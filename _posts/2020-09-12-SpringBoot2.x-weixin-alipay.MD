---
title:  "Springboot整合支付宝和微信支付"
categories: 支付宝 微信支付
tags: 支付宝 微信支付
author: LIUREN
---

## Springboot整合支付宝和微信支付

> 现如今，支付宝和微信支付改变了人们的支付方式，使支付更加便捷，那我们如何把自己的产品也整合支付宝支付与微信支付呢？
>
> 本文使用springboot以及支付宝、微信官方的依赖，实现了微信支付与支付宝支付，其中屏蔽掉了持久层的代码。
>
> 看本文之前最好先去官网了解具体的支付的集中场景，以便于更好的理解支付流程。
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



## 一、支付宝支付

### 1、支付信息配置

引入支付宝支付依赖

 

```xml
 <!-- alipay-sdk-java -->
<dependency>
    <groupId>com.alipay.sdk</groupId>
    <artifactId>alipay-sdk-java</artifactId>
    <version>3.4.27.ALL</version>
</dependency>
```

在配置文件中添加如下支付宝支付配置

```properties
#支付网关
pay.alipay.gatewayUrl=https://openapi.alipay.com/gateway.do
#商户APPid
pay.alipay.appid=
#私钥
pay.alipay.appPrivateKey=
#公钥
pay.alipay.alipayPublicKey=
#支付同步通知
pay.alipay.returnUrl=http://luciar.natapp1.cc/alipay/returnUrl
#支付异步通知
pay.alipay.notifyUrl=http://luciar.natapp1.cc/alipay/notify
#扫码付款成功后跳转的url
pay.alipay.returnUrlForScan=http://luciar.natapp1.cc/success
# 商品定价(以元为单位)
pay.alipay.totalAmount=0.01

#########支付公共配置############
#商品名
pay.subjectName=啄影佳品
#前缀
pay.prefixUrl=http://luciar.natapp1.cc
```

编写支付宝支付配置类 Alipay Properties

```java
/**
 * 支付宝支付的参数配置
 */
@PropertySource("classpath:application.properties")
//配置信息自动注入参数
@ConfigurationProperties(prefix = "pay.alipay")
public class AlipayProperties {
    private static final Logger logger = LoggerFactory.getLogger(AlipayProperties.class);
    /** 支付宝gatewayUrl */
    private String gatewayUrl;
    /** 商户应用id */
    private String appid;
    /** RSA私钥，用于对商户请求报文加签 */
    private String appPrivateKey;
    /** 支付宝RSA公钥，用于验签支付宝应答 */
    private String alipayPublicKey;
    /** 签名类型 */
    private String signType = "RSA2";
    /** 格式 */
    private String formate = "json";
    /** 编码 */
    private String charset = "UTF-8";
    /** 同步地址 */
    private String returnUrl;
    /** 异步地址 */
    private String notifyUrl;
    /** 最大查询次数 */
    private static int maxQueryRetry = 5;
    /** 查询间隔（毫秒） */
    private static long queryDuration = 5000;
    /** 最大撤销次数 */
    private static int maxCancelRetry = 3;
    /** 撤销间隔（毫秒） */
    private static long cancelDuration = 3000;

    private AlipayProperties() {}
    /*省略set和get方法*/
    //setters and getters
}
```

将配置注入到Alipay Client ，并将Alipay Client注入容器

```java
@Configuration
@EnableConfigurationProperties(AlipayProperties.class)
public class AlipayConfiguration {

    @Autowired
    private AlipayProperties properties;

    @Bean
    public AlipayClient alipayClient(){
        return new DefaultAlipayClient(
                properties.getGatewayUrl(),
                properties.getAppid(),
                properties.getAppPrivateKey(),
                properties.getFormate(),
                properties.getCharset(),
                properties.getAlipayPublicKey(),
                properties.getSignType());
    }
}
```

 

### 2、扫码支付

直接在controller中写方法，返回from。当页面请求这个方法的时候，会自动跳转到支付宝扫码页面

扫码付款成功后会跳转到同步通知的URL，一般为跳转到支付成功页面，同时向后台发送异步通知。

```java
@Controller
@RequestMapping("/alipay")
public class AlipayPagePayController {

    @Autowired
    private AlipayProperties alipayProperties;

    @Autowired
    private AlipayClient alipayClient;

    /**
     * 商品定价
     */
    @Value("${pay.alipay.totalAmount}")
    private String totalAmount;

    /**
     * 商品名
     */
    @Value("${pay.subjectName}")
    public String subjectName;

    /**
     * 扫码付款成功后跳转的url
     */
    @Value("${pay.alipay.returnUrlForScan}")
    private String returnUrlForScan;

    @ApiOperation(value="网页扫码支付", notes="跳转到支付宝扫码支付页面，用户可通过扫码进行支付，成功则跳转到成功页面")
    @GetMapping("/gotoPayPage")
    public void gotoPayPage(HttpServletResponse response) throws AlipayApiException, IOException {
        // 订单模型
        String productCode = "FAST_INSTANT_TRADE_PAY";
        AlipayTradePagePayModel model = new AlipayTradePagePayModel();
        model.setOutTradeNo(UUID.randomUUID().toString());
        model.setSubject(subjectName);
        model.setTotalAmount(totalAmount);
        model.setBody(subjectName+"，共0.01元");
        model.setProductCode(productCode);

        AlipayTradePagePayRequest pagePayRequest =new AlipayTradePagePayRequest();
        pagePayRequest.setReturnUrl(returnUrlForScan);
        pagePayRequest.setNotifyUrl(alipayProperties.getNotifyUrl());
        pagePayRequest.setBizModel(model);

        // 调用SDK生成表单, 并直接将完整的表单html输出到页面
        String form = alipayClient.pageExecute(pagePayRequest).getBody();
        response.setContentType("text/html;charset=" + alipayProperties.getCharset());
        response.getWriter().write(form);
        response.getWriter().flush();
        response.getWriter().close();
    }
}
```

**同步通知**

支付成功后由支付宝发起请求，跳转到支付成功页面

```java
/**
     * 支付宝页面跳转同步通知页面
     */
    @ApiIgnore
    @GetMapping("/returnUrl")
    public String returnUrl(HttpServletRequest request, HttpServletResponse response) throws UnsupportedEncodingException, AlipayApiException {
        response.setContentType("text/html;charset=" + aliPayProperties.getCharset());

        //获取支付宝GET过来反馈信息
        Map<String,String> params = new HashMap<>();
        Map requestParams = request.getParameterMap();
        for (Object iter : requestParams.keySet()) {
            String name = (String) iter;
            String[] values = (String[]) requestParams.get(name);
            String valueStr = "";
            for (int i = 0; i < values.length; i++) {
                valueStr = (i == values.length - 1) ? valueStr + values[i]
                        : valueStr + values[i] + ",";
            }
            //乱码解决，这段代码在出现乱码时使用。如果mysign和sign不相等也可以使用这段代码转化
            valueStr = new String(valueStr.getBytes("ISO-8859-1"), "utf-8");
            params.put(name, valueStr);
        }
        logger.info("params is "+ params.toString());
        //验签
        boolean verifyResult = AlipaySignature.rsaCheckV1(params, aliPayProperties.getAlipayPublicKey(),
                aliPayProperties.getCharset(), "RSA2");
        if(verifyResult){
            //验证成功
            //请在这里加上商户的业务逻辑程序代码，如保存支付宝交易号
            //商户订单号
            String outTradeNo = new String(request.getParameter("out_trade_no").getBytes("ISO-8859-1"),"UTF-8");
            //支付宝交易号
            String tradeNo = new String(request.getParameter("trade_no").getBytes("ISO-8859-1"),"UTF-8");

            return "success";
        }else{
            return  "error";
        }
    }
```

**异步通知**

支付成功后由支付宝发起异步请求，一般为入库逻辑

此处把支付成功的异步通知和退款成功的异步通知都给写到了一起

详细解释见代码里面的注释

```java
/**
     * 支付异步通知
     * 接收到异步通知并验签通过后，一定要检查通知内容，包括通知中的app_id、out_trade_no、total_amount是否与请求中的一致，
     * 并根据trade_status进行后续业务处理
     * https://docs.open.alipay.com/194/103296
     */
@ApiIgnore
@PostMapping("/notify")
@ResponseBody
public String notify(HttpServletRequest request) throws AlipayApiException, UnsupportedEncodingException {
    // 一定要验签，防止黑客篡改参数
    Map<String, String[]> parameterMap = request.getParameterMap();
    StringBuilder notifyBuild = new StringBuilder("/****************************** alipay notify **********" +
                                                  "********************/\n");
    parameterMap.forEach((key, value) -> notifyBuild.append(key + "=" + value[0] + "\n") );
    //订单信息
    logger.info("notifyBuild"+notifyBuild.toString());

    //校验签名
    boolean flag = this.rsaCheckV1(request);
    if (flag) {
        /**
             * 需要严格按照如下描述校验通知数据的正确性
             * 商户需要验证该通知数据中的out_trade_no是否为商户系统中创建的订单号，
             * 并判断total_amount是否确实为该订单的实际金额（即商户订单创建时的金额），
             * 同时需要校验通知中的seller_id（或者seller_email) 是否为out_trade_no这笔单据的对应的操作方（有的时候，一个商户可能有多个seller_id/seller_email），
             * 上述有任何一个验证不通过，则表明本次通知是异常通知，务必忽略。
             * 在上述验证通过后商户必须根据支付宝不同类型的业务通知，正确的进行不同的业务处理，并且过滤重复的通知结果数据。
             * 在支付宝的业务通知中，只有交易通知状态为TRADE_SUCCESS或TRADE_FINISHED时，支付宝才会认定为买家付款成功。
             */
        //交易状态
        String tradeStatus = new String(request.getParameter("trade_status").getBytes("ISO-8859-1"),"UTF-8");
        //支付宝交易号
        String trade_no = new String(request.getParameter("trade_no").getBytes("ISO-8859-1"),"UTF-8");
        //商户id
        String app_id = new String(request.getParameter("app_id").getBytes("ISO-8859-1"),"UTF-8");
        //退款费用 用于判断是否是退款
        String refund_fee = null;
        try{
            refund_fee = new String(request.getParameter("refund_fee").getBytes("ISO-8859-1"),"UTF-8");
        }catch (Exception e){
            refund_fee = "";
        }
        /*判断是否是支付异步通知*/
        if(refund_fee == null || refund_fee == ""){
            //付款金额
            String total_amount = new String(request.getParameter("total_amount").getBytes("ISO-8859-1"),"UTF-8");
            //判断订单号是否已处理
            if (alipayService.containsByOutTradeNo(trade_no)){
                logger.warn("订单已处理，支付宝重复调用");
                return "success";
            }
            //判断订单金额是否一致 防止被篡改
            if (!total_amount.equals(totalAmount)){
                logger.warn("订单金额不一致，请防止黑客恶意篡改信息");
                return "fail";
            }
            //判断商户id是否一致，防止被篡改
            if (!app_id.equals(aliPayProperties.getAppid())){
                logger.warn("操作的商户id不一致，请防止黑客恶意篡改信息");
                return "fail";
            }

            // TRADE_FINISHED(表示交易已经成功结束，并不能再对该交易做后续操作);
            // TRADE_SUCCESS(表示交易已经成功结束，可以对该交易做后续操作，如：分润、退款等);
            if("TRADE_FINISHED".equals(tradeStatus)){
                //判断该笔订单是否在商户网站中已经做过处理
                //如果没有做过处理，根据订单号（out_trade_no）在商户网站的订单系统中查到该笔订单的详细，
                // 并判断total_amount是否确实为该订单的实际金额（即商户订单创建时的金额），并执行商户的业务程序
                //请务必判断请求时的total_fee、seller_id与通知时获取的total_fee、seller_id为一致的
                //如果有做过处理，不执行商户的业务程序
                //注意：
                //如果签约的是可退款协议，退款日期超过可退款期限后（如三个月可退款），支付宝系统发送该交易状态通知
                //如果没有签约可退款协议，那么付款完成后，支付宝系统发送该交易状态通知。
            } else if ("TRADE_SUCCESS".equals(tradeStatus)){
                //判断该笔订单是否在商户网站中已经做过处理
                //如果没有做过处理，根据订单号（out_trade_no）在商户网站的订单系统中查到该笔订单的详细，
                // 并判断total_amount是否确实为该订单的实际金额（即商户订单创建时的金额），并执行商户的业务程序
                //请务必判断请求时的total_fee、seller_id与通知时获取的total_fee、seller_id为一致的
                //如果有做过处理，不执行商户的业务程序
                logger.info("签约可退款协议");
                alipayService.saveToDb(parameterMap);
                //注意：
                //如果签约的是可退款协议，那么付款完成后，支付宝系统发送该交易状态通知。
            }
        }
        /*判断是否是退款异步通知*/
        else{
            if (alipayRefundService.containsByOutTradeNo(trade_no)){
                logger.warn("退款已处理，用户可能重复提交");
                return "have refunded";
            }
            //判断订单金额是否一致 防止被篡改
            if (!refund_fee.equals(totalAmount)){
                logger.warn("订单金额不一致，请防止黑客恶意篡改信息");
                return "fail";
            }
            //判断商户id是否一致，防止被篡改
            if (!app_id.equals(aliPayProperties.getAppid())){
                logger.warn("操作的商户id不一致，请防止黑客恶意篡改信息");
                return "fail";
            }
            if("TRADE_FINISHED".equals(tradeStatus)){
                //判断该笔订单是否在商户网站中已经做过处理
                //如果没有做过处理，根据订单号（out_trade_no）在商户网站的订单系统中查到该笔订单的详细，
                // 并判断total_amount是否确实为该订单的实际金额（即商户订单创建时的金额），并执行商户的业务程序
                //请务必判断请求时的total_fee、seller_id与通知时获取的total_fee、seller_id为一致的
                //如果有做过处理，不执行商户的业务程序
                //注意：
                //如果签约的是可退款协议，退款日期超过可退款期限后（如三个月可退款），支付宝系统发送该交易状态通知
                //如果没有签约可退款协议，那么付款完成后，支付宝系统发送该交易状态通知。
            } else if ("TRADE_SUCCESS".equals(tradeStatus)){
                //判断该笔订单是否在商户网站中已经做过处理
                //如果没有做过处理，根据订单号（out_trade_no）在商户网站的订单系统中查到该笔订单的详细，
                // 并判断total_amount是否确实为该订单的实际金额（即商户订单创建时的金额），并执行商户的业务程序
                //请务必判断请求时的total_fee、seller_id与通知时获取的total_fee、seller_id为一致的
                //如果有做过处理，不执行商户的业务程序
                alipayRefundService.saveToDb(parameterMap);
                //注意：
                //如果签约的是可退款协议，那么付款完成后，支付宝系统发送该交易状态通知。
            }
        }

        return "success";
    }
    return "fail";
}
```

 

### 3、H5支付

前端传入商户订单号，如果不传入，由系统通过`System.nanoTime()`生成。

通过SDK调用支付宝客户端下单, 并直接将返回的完整表单html输出到页面

```java
@GetMapping("/alipage")
@ResponseBody
public String gotoPayPage(@RequestParam String outTradeNo) throws AlipayApiException {
    // 订单模型
    String productCode="QUICK_WAP_WAY";
    AlipayTradeWapPayModel model = new AlipayTradeWapPayModel();
    //防止传空参数
    if (outTradeNo == null || outTradeNo=="" ||outTradeNo.length()<4){
        outTradeNo = String.valueOf(System.nanoTime());
    }
    //        String outTradeNo = UUID.randomUUID().toString();
    model.setOutTradeNo(outTradeNo);
    model.setSubject(subjectName);
    model.setTotalAmount(totalAmount);
    model.setBody(subjectName+"，共0.01元");
    model.setTimeoutExpress("2m");
    model.setProductCode(productCode);

    AlipayTradeWapPayRequest wapPayRequest =new AlipayTradeWapPayRequest();
    wapPayRequest.setReturnUrl(aliPayProperties.getReturnUrl());
    wapPayRequest.setNotifyUrl(aliPayProperties.getNotifyUrl());
    wapPayRequest.setBizModel(model);

    // 调用SDK生成表单, 并直接将完整的表单html输出到页面
    String form = alipayClient.pageExecute(wapPayRequest).getBody();
    return form;
}
```

同步通知与异步通知与扫码支付一样，可重用。

4、退款

```java
@ApiOperation(value="退款", notes="通过商户订单号退款")
@ApiParam(name = "out_trade_no", value = "商户订单号", required = true)
@GetMapping("/refund")
@ResponseBody
public String refund(@RequestParam String out_trade_no){
    //若订单已处理，直接返回
    if (alipayRefundService.containsByOutTradeNo(out_trade_no)){
        logger.warn("退款已处理，用户可能重复提交");
        return "have refunded";
    }
    AlipayTradeRefundRequest request = new AlipayTradeRefundRequest();
    AlipayTradeRefundModel refundModel = new AlipayTradeRefundModel();
    //        refundModel.setTradeNo(out_trade_no);
    refundModel.setOutTradeNo(out_trade_no);
    refundModel.setRefundAmount(totalAmount);
    refundModel.setRefundReason("用户归还胶片，退还费用");
    request.setBizModel(refundModel);
    AlipayTradeRefundResponse response = null;
    try {
        response = alipayClient.execute(request);
        logger.info(response.toString());
    } catch (AlipayApiException e) {
        logger.error("申请退款出错");
        e.printStackTrace();
        return "error";
    }
    if(Objects.requireNonNull(response).isSuccess()){
        logger.info("申请退款成功");
        Map<String, String> responseMap = new HashMap<>(2);
        responseMap.put("return_code", "200");
        responseMap.put("return_msg", "refund success");
        return JSON.toJSONString(responseMap);
    } else {
        logger.info("申请退款失败");
        Map<String, String> responseMap = new HashMap<>(2);
        responseMap.put("return_code", "200");
        responseMap.put("return_msg", "refund error");
        return JSON.toJSONString(responseMap);
    }
}
```

 

------

 

## 二、微信支付

微信支付相较于支付宝支付，复杂一些

**注意:**

```java
1、证书文件不能放在web服务器虚拟目录，应放在有访问权限控制的目录中，防止被他人下载

2、建议将证书文件名改为复杂且不容易猜测的文件名

3、商户服务器要做好病毒和木马防护工作，不被非法侵入者窃取证书文件

4、请妥善保管商户支付密钥、公众帐号SECRET，避免密钥泄露

5、参数为Map<String, String>对象，返回类型也是Map<String, String>

6、方法内部会将参数会转换成含有appid、mch_id、nonce_str、sign_type和sign的XML

7、可选HMAC-SHA256算法和MD5算法签名

8、通过HTTPS请求得到返回数据后会对其做必要的处理（例如验证签名，签名错误则抛出异常）
```

### 1、支付信息配置

微信支付配置

```properties
#############微信支付配置###########
#公众账号ID
pay.wxpay.appID:
#商户号
pay.wxpay.mchID:
#API密钥
pay.wxpay.key:
#支付密钥，用于JSAPI支付中生成open_id
pay.wxpay.secret:
#证书路径
pay.wxpay.certPath:
#异步通知地址
pay.wxpay.notifyUrl:http://luciar.natapp1.cc/wxpay/notify
#退款url
pay.wxpay.refundNotifyUrl:http://luciar.natapp1.cc/wxpay/refund/notify
#是否使用沙箱
pay.wxpay.useSandbox:false
#沙箱环境密钥
pay.wxpay.sandboxKey:sandboxKey
# 商品定价(以分为单位)
pay.wxpay.totalAmount=1
```

公共配置

此外，还需要一些支付公用配置，主要处理订单的数据

```properties
#商品名
pay.subjectName=啄影佳品
#前缀
pay.prefixUrl=http://luciar.natapp1.cc

#解决读取配置文件中文乱码问题
spring.http.encoding.charset=UTF-8
spring.http.encoding.enabled=true
spring.messages.encoding=UTF-8
spring.http.encoding.force=true
server.tomcat.uri-encoding=UTF-8
```

#### 支付配置类

编写微信支付配置类，将配置注入到属性中

```java
@ConfigurationProperties(prefix = "pay.wxpay")
public class MyWXPayConfig implements WXPayConfig {
    private  static Logger logger = LoggerFactory.getLogger(MyWXPayConfig.class);

    /** 公众账号ID */
    private String appID;

    /** 商户号 */
    private String mchID;

    /** API密钥 */
    private String key;

    /** API 沙箱环境密钥 */
    private String sandboxKey;

    /** API证书绝对路径 */
    private String certPath;

    /** 退款异步通知地址 */
    private String notifyUrl;

    private Boolean useSandbox;

    /** HTTP(S) 连接超时时间，单位毫秒 */
    private int httpConnectTimeoutMs = 8000;

    /** HTTP(S) 读数据超时时间，单位毫秒 */
    private int httpReadTimeoutMs = 10000;


    /**
     * 获取商户证书内容
     *
     * @return 商户证书内容
     */
    @Override
    public InputStream getCertStream()  {
        File certFile = new File(certPath);
        InputStream inputStream = null;
        try {
            inputStream = new FileInputStream(certFile);
        } catch (FileNotFoundException e) {
            logger.error("cert file not found, path={}, exception is:{}", certPath, e);
        }
        return inputStream;
    }
}
```

#### 支付客户端

编写微信支付客户端（`WXPayClient`）,主要用于支付和退款时调用

```java
public class WXPayClient extends WXPay {
    private static final Logger logger = LoggerFactory.getLogger(WXPayClient.class);
    /** 密钥算法 */
    private static final String ALGORITHM = "AES";
    /** 加解密算法/工作模式/填充方式 */
    private static final String ALGORITHM_MODE_PADDING = "AES/ECB/PKCS5Padding";
    /** 用户支付中，需要输入密码 */
    private static final String ERR_CODE_USERPAYING = "USERPAYING";
    private static final String ERR_CODE_AUTHCODEEXPIRE = "AUTHCODEEXPIRE";
    /** 交易状态: 未支付 */
    private static final String TRADE_STATE_NOTPAY = "NOTPAY";

    /** 用户输入密码，尝试30秒内去查询支付结果 */
    private static Integer remainingTimeMs = 10000;

    private WXPayConfig config;

    public WXPayClient(WXPayConfig config, WXPayConstants.SignType signType, boolean useSandbox) {
        super(config, signType, useSandbox);
        this.config = config;
    }

    /**
     * 刷卡支付
     * 对WXPay#microPay(Map)增加了当支付结果为USERPAYING时去轮询查询支付结果的逻辑处理
     * 注意：该方法没有处理return_code=FAIL的情况，暂时不考虑网络问题，这种情况直接返回错误
     */
    public Map<String, String> microPayWithPOS(Map<String, String> reqData) throws Exception {
        // 开始时间(毫秒)
        long startTimestampMs = System.currentTimeMillis();

        Map<String, String> responseMapForPay = super.microPay(reqData);
        logger.info(responseMapForPay.toString());

        // // 先判断 协议字段返回(return_code)，再判断 业务返回，最后判断 交易状态(trade_state)
        // 通信标识，非交易标识
        String returnCode = responseMapForPay.get("return_code");
        if (WXPayConstants.SUCCESS.equals(returnCode)) {
            String errCode = responseMapForPay.get("err_code");
            // 余额不足，信用卡失效
            if (ERR_CODE_USERPAYING.equals(errCode) || "SYSTEMERROR".equals(errCode) || "BANKERROR".equals(errCode)) {
                Map<String, String> orderQueryMap = null;
                Map<String, String> requestData = new HashMap<>();
                requestData.put("out_trade_no", reqData.get("out_trade_no"));

                // 用户支付中，需要输入密码或系统错误则去重新查询订单API err_code, result_code, err_code_des
                // 每次循环时的当前系统时间 - 开始时记录的时间 > 设定的30秒时间就退出
                while (System.currentTimeMillis() - startTimestampMs < remainingTimeMs) {
                    // 商户收银台得到USERPAYING状态后，经过商户后台系统调用【查询订单API】查询实际支付结果。
                    orderQueryMap = super.orderQuery(requestData);
                    String returnCodeForQuery = orderQueryMap.get("return_code");
                    if (WXPayConstants.SUCCESS.equals(returnCodeForQuery)) {
                        // 通讯成功
                        String tradeState = orderQueryMap.get("trade_state");
                        if (WXPayConstants.SUCCESS.equals(tradeState)) {
                            // 如果成功了直接将查询结果返回
                            return orderQueryMap;
                        }
                        // 如果支付结果仍为USERPAYING，则每隔5秒循环调用【查询订单API】判断实际支付结果
                        Thread.sleep(1000);
                    }
                }

                // 如果用户取消支付或累计30秒用户都未支付，商户收银台退出查询流程后继续调用【撤销订单API】撤销支付交易。
                String tradeState = orderQueryMap.get("trade_state");
                if (TRADE_STATE_NOTPAY.equals(tradeState) || ERR_CODE_USERPAYING.equals(tradeState) || ERR_CODE_AUTHCODEEXPIRE.equals(tradeState)) {
                    Map<String, String> reverseMap = this.reverse(requestData);
                    String returnCodeForReverse = reverseMap.get("return_code");
                    String resultCode = reverseMap.get("result_code");
                    if (WXPayConstants.SUCCESS.equals(returnCodeForReverse) && WXPayConstants.SUCCESS.equals(resultCode)) {
                        // 如果撤销成功，需要告诉客户端已经撤销订单了
                        responseMapForPay.put("err_code_des", "用户取消支付或尚未支付，后台已经撤销该订单，请重新支付！");
                    }
                }
            }
        }

        return responseMapForPay;
    }

    /**
     * 从request的inputStream中获取参数
     */
    public Map<String, String> getNotifyParameter(HttpServletRequest request) throws Exception {
        InputStream inputStream = request.getInputStream();
        ByteArrayOutputStream outSteam = new ByteArrayOutputStream();
        byte[] buffer = new byte[1024];
        int length = 0;
        while ((length = inputStream.read(buffer)) != -1) {
            outSteam.write(buffer, 0, length);
        }
        outSteam.close();
        inputStream.close();

        // 获取微信调用我们notify_url的返回信息
        String resultXml = new String(outSteam.toByteArray(), "utf-8");
        logger.info("resultXml为："+resultXml);
        Map<String, String> notifyMap = WXPayUtil.xmlToMap(resultXml);

        return notifyMap;
    }

    /**
     * 解密退款通知
     * <a href="https://pay.weixin.qq.com/wiki/doc/api/micropay.php?chapter=9_16&index=11>退款结果通知文档</a>
     */
    public Map<String, String> decodeRefundNotify(HttpServletRequest request) throws Exception {
        // 从request的流中获取参数
        Map<String, String> notifyMap = this.getNotifyParameter(request);
        logger.info(notifyMap.toString());

        String reqInfo = notifyMap.get("req_info");
        //（1）对加密串A做base64解码，得到加密串B
        byte[] bytes = new BASE64Decoder().decodeBuffer(reqInfo);

        //（2）对商户key做md5，得到32位小写key* ( key设置路径：微信商户平台(pay.weixin.qq.com)-->账户设置-->API安全-->密钥设置 )
        Cipher cipher = Cipher.getInstance(ALGORITHM_MODE_PADDING);
        SecretKeySpec key = new SecretKeySpec(WXPayUtil.MD5(config.getKey()).toLowerCase().getBytes(), ALGORITHM);
        cipher.init(Cipher.DECRYPT_MODE, key);

        //（3）用key*对加密串B做AES-256-ECB解密（PKCS7Padding）
        // java.security.InvalidKeyException: Illegal key size or default parameters
        // https://www.cnblogs.com/yaks/p/5608358.html
        String responseXml = new String(cipher.doFinal(bytes),"UTF-8");
        Map<String, String> responseMap = WXPayUtil.xmlToMap(responseXml);
        return responseMap;
    }

    /**
     * 获取沙箱环境验签秘钥API
     * <a href="https://pay.weixin.qq.com/wiki/doc/api/micropay.php?chapter=23_1">获取验签秘钥API文档</a>
     */
    public Map<String, String> getSignKey() throws Exception {
        Map<String, String> reqData = new HashMap<>();
        reqData.put("mch_id", config.getMchID());
        reqData.put("nonce_str", WXPayUtil.generateNonceStr());
        String sign = WXPayUtil.generateSignature(reqData, config.getKey(), WXPayConstants.SignType.MD5);
        reqData.put("sign", sign);
        String responseXml = this.requestWithoutCert("https://api.mch.weixin.qq.com/sandboxnew/pay/getsignkey", reqData,
                config.getHttpConnectTimeoutMs(), config.getHttpReadTimeoutMs());

        Map<String, String> responseMap = WXPayUtil.xmlToMap(responseXml);

        return responseMap;
    }

    /**
     * 根据订单号查询订单状态
     * @param orderId 订单号
     * @return 订单状态
     */
    public Map<String, String> queryWeiXinPay(String orderId){
        Map<String, String> resp  = null;

        WXPay wxpay = new WXPay(config,WXPayConstants.SignType.MD5,false);
        Map<String, String> data = new HashMap<String, String>();
        data.put("out_trade_no", orderId);//订单号

        try {
            resp = wxpay.orderQuery(data);
        } catch (Exception e) {
            logger.info("查询订单失败，订单号为："+orderId);
            e.printStackTrace();
        }
        return resp;
    }
}
```

#### 注入容器

将微信支付配置封装到`WXPay`和`WXPayClient`中，并将它们注入到容器。

```java
@Configuration
@EnableConfigurationProperties(MyWXPayConfig.class)
public class WXPayConfiguration {

    @Autowired
    private MyWXPayConfig wxPayConfig;

    @Bean
    public WXPay wxPay() {
        return new WXPay(wxPayConfig, WXPayConstants.SignType.MD5, wxPayConfig.getUseSandbox() );
    }

    @Bean
    public WXPayClient wxPayClient() {
        return new WXPayClient(wxPayConfig, WXPayConstants.SignType.MD5, wxPayConfig.getUseSandbox());
    }
}
```

 2、扫码支付

与支付宝支付不同，微信扫码支付不会自己跳转到扫码支付页面，甚至不会自己生成支付的二维码，因此所有的操作要自己来做。

当前端调用此方法时，方法体内部会对订单进行下单，返回二维码图片的二进制流给前端，之后需要前端自行处理图片二进制流

统一下单

```java
@ApiOperation(value="网页扫码支付", notes="返回二维码的二进制流和订单号组成的串，在前端进行处理")
    @GetMapping("/precreate")
    @ResponseBody
    public String preCreate(HttpServletRequest request) throws Exception {
        Map<String, String> reqData = new HashMap<>();
        reqData.put("out_trade_no", String.valueOf(System.nanoTime()));
        reqData.put("trade_type", "NATIVE");
        reqData.put("product_id", "10001");
        reqData.put("body", subjectName);
        // 订单总金额，单位为分
        reqData.put("total_fee", totalAmount);
        // APP和网页支付提交用户端ip，Native支付填调用微信支付API的机器IP。
        reqData.put("spbill_create_ip", "");
        // 异步接收微信支付结果通知的回调地址，通知url必须为外网可访问的url，不能携带参数。
        reqData.put("notify_url", wxPayConfig.getNotifyUrl());
        // 自定义参数, 可以为终端设备号(门店号或收银设备ID)，PC网页或公众号内支付可以传"WEB"
        reqData.put("device_info", "WEB");
        // 附加数据，在查询API和支付通知中原样返回，可作为自定义参数使用。
        reqData.put("attach", "");

        Map<String, String> responseMap = wxPay.unifiedOrder(reqData);
        logger.info(responseMap.toString());
        String returnCode = responseMap.get("return_code");
        String resultCode = responseMap.get("result_code");
        if (WXPayConstants.SUCCESS.equals(returnCode) && WXPayConstants.SUCCESS.equals(resultCode)) {
            String prepayId = responseMap.get("prepay_id");
            String codeUrl = responseMap.get("code_url");
            //获取二维码
            BufferedImage image = PayUtil.getQRCodeImge(codeUrl);

            ByteArrayOutputStream out = new ByteArrayOutputStream();
            ImageIO.write(image, "gif", out);

            Map<String,Object> resultMap = new HashMap<>();
            //图片转为二进制流
            byte[] imageData = out.toByteArray();
            //封装返回集合
            resultMap.put("image",imageData);
            resultMap.put("orderId",reqData.get("out_trade_no"));
            return JSON.toJSONString(resultMap);
        }
        return null;
    }
```

#### 查询订单状态

微信支付的前端不知道用户是否支付成功，所以前端需要定时查询支付状态是否已经支付成功

如下是后端查询支付状态的代码

```java
@ApiOperation(value="订单查询", notes="通过传入的订单号查询订单")
@ApiParam(name = "orderId", value = "订单号", required = true)
@GetMapping("/status")
@ResponseBody
public String getPayStatus(@RequestParam(value = "orderId") String orderId) throws Exception {
    Map<String, String> stringStringMap = queryWeiXinPay(orderId);
    //  Map<String,Object> resultMap = new HashMap<>();
    if (stringStringMap != null){
        if ("SUCCESS".equals(stringStringMap.get("trade_state"))){
            Map<String, String> responseMap = new HashMap<>(2);
            responseMap.put("return_code", "SUCCESS");
            responseMap.put("return_msg", "OK");
            return JSON.toJSONString(responseMap);
        }

    }
    Map<String, String> responseMap = new HashMap<>(2);
    responseMap.put("return_code", "ERROR");
    responseMap.put("return_msg", "ERROR");
    return JSON.toJSONString(responseMap);
}
```

 

#### 生成二维码图片

在如上的代码中，将生成订单后返回的跳转链接制作成二维码图片的代码如下

代码生成的二维码做了美化，在二维码中间加入了如片，并对二维码做了圆角处理，使其看上去与微信生成的二维码一样

```java
public class PayUtil {

    private static final Logger logger = LoggerFactory.getLogger(PayUtil.class);
    //用于设置图案的颜色
    private static final int BLACK = 0xFF000000;
    //用于背景色
    private static final int WHITE = 0xFFFFFFFF;

    /**
     * 根据url生成二位图片对象
     */
    public static BufferedImage getQRCodeImge(String codeUrl) throws WriterException{
        Hashtable<EncodeHintType, Object> hints = new Hashtable<EncodeHintType, Object>();
        // 二维码图片宽度和高度
        int width = 300;
        int height = 300;
        // 指定纠错等级,纠错级别（L 7%、M 15%、Q 25%、H 30%）
        hints.put(EncodeHintType.ERROR_CORRECTION, ErrorCorrectionLevel.M);
        hints.put(EncodeHintType.CHARACTER_SET, "utf-8");
        //设置二维码边的空度，非负数
        hints.put(EncodeHintType.MARGIN, 1);

        BitMatrix bitMatrix = new MultiFormatWriter().encode(
                codeUrl, BarcodeFormat.QR_CODE, width, height,
                //生成条形码时的一些配置,此项可选
                hints);

        BufferedImage image = toBufferedImage(bitMatrix);
        try {
            image = LogoMatrix(image);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return image;
    }

    /**
     * 设置 logo
     * @param matrixImage 源二维码图片
     * @return 返回带有logo的二维码图片
     */
    public static BufferedImage LogoMatrix(BufferedImage matrixImage) throws IOException {
        //读取二维码图片，并构建绘图对象
        Graphics2D g2 = matrixImage.createGraphics();

        int matrixWidth = matrixImage.getWidth();
        int matrixHeigh = matrixImage.getHeight();

        //判断文件是否存在
        if (ResourceUtils.isUrl("classpath:static/pic/logo.png")) {
            try {
                File file = ResourceUtils.getFile("classpath:static/pic/logo.png");
                //读取Logo图片
                BufferedImage logo = ImageIO.read(file);

                //开始绘制图片
                //绘制
                g2.drawImage(logo,matrixWidth/5*2,matrixHeigh/5*2, matrixWidth/5, matrixHeigh/5, null);
                BasicStroke stroke = new BasicStroke(5,BasicStroke.CAP_ROUND,BasicStroke.JOIN_ROUND);
                // 设置笔画对象
                g2.setStroke(stroke);
                //指定弧度的圆角矩形
                RoundRectangle2D.Float round = new RoundRectangle2D.Float(matrixWidth/5*2, matrixHeigh/5*2, matrixWidth/5, matrixHeigh/5,20,20);
                g2.setColor(Color.white);
                // 绘制圆弧矩形
                g2.draw(round);

                //设置logo 有一道灰色边框
                BasicStroke stroke2 = new BasicStroke(1,BasicStroke.CAP_ROUND,BasicStroke.JOIN_ROUND);
                // 设置笔画对象
                g2.setStroke(stroke2);
                RoundRectangle2D.Float round2 = new RoundRectangle2D.Float(matrixWidth/5*2+2, matrixHeigh/5*2+2, matrixWidth/5-4, matrixHeigh/5-4,20,20);
                g2.setColor(new Color(128,128,128));
                // 绘制圆弧矩形
                g2.draw(round2);

                g2.dispose();
                matrixImage.flush() ;
            } catch (FileNotFoundException ignored) {
                logger.info("读取文件失败");
            }
        }
        return matrixImage ;
    }

    /**
     * bit图像转为BufferedImage
     * @param matrix bit图像
     */
    public static BufferedImage toBufferedImage(BitMatrix matrix) {
        int width = matrix.getWidth();
        int height = matrix.getHeight();
        BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
        for (int x = 0; x < width; x++) {
            for (int y = 0; y < height; y++) {
                image.setRGB(x, y,  (matrix.get(x, y) ? BLACK : WHITE));
            }
        }
        return image;
    }
}
```

#### 前端处理逻辑

1、前端用html承接二维码，用隐藏的input存储订单号

```html
<img id="ImagePic" alt="二维码特效" width="100" height="100"/>
<!--影藏标签-->
<input type="hidden" id="orderId" value="" />
```

2、通过ajax调用扫码支付请求，获取返回的二维码二进制流，并将其添加到页面的image标签下

```javascript
function getImg() {
    $.ajax({
        type: 'get',
        data: {}, //参数
        dataType: 'json',
        url: "/wxpay/precreate",
        success: function (data) {
            console.info(data)
            //将图片的Base64编码设置给src
            $("#ImagePic").attr("src", "data:image/png;base64," + data.image);
            $("#orderId").val(data.orderId);
        },
        error: function (data) {
            alert('响应失败！');
        }
    });
}
```

3、前端请求查询支付状态，当支付状态为成功时跳转到支付成功页面

```javascript
function checkOrder() {
    var orderId= $("#orderId").val();
    if (orderId != null) {
        console.info("订单号为：" + orderId);
        // var result = ajaxWithServer("wxpay/status",orderId);
        $.ajax({
            type: 'get',
            data: {
                "orderId":orderId
            },
            dataType: 'json',
            url: "wxpay/status",
            success: function (data) {
                // console.info(data);
                if(data.return_code === "SUCCESS" ){
                    window.location.href="/success";
                }
            },
            error: function () {
            }
        });
    }
}
```

 

4、通过定时器，各两秒向后台发送订单是否完成的请求

```javascript
$('body').everyTime('2s',function(){
    checkOrder();
});
```

这个定时器需要引入导入`jquery.timers.js`包，也可以直接访问

https://us.softpedia-secure-download.com/dl/a81e3905125b6a7c4fe5e30c06d8501d/5c6cd04e/700044280/webscripts/javascripts/jquery.timers.js下载

### 3、H5支付

微信的h5支付不会像支付宝支付那样直接自动跳转处理，而是要下单获取url并传到前端，再由前端进行跳转调出微信支付客户端

#### h5支付后端代码

在第三方浏览器中调用微信客户端进行支付

```java
@ApiOperation(value="调用手机微信支付", notes="通过连接调用本地手机微信支付")
@GetMapping("/order")
public Object h5pay(HttpServletRequest request, HttpServletResponse response,
                    @RequestParam(value = "ip",required=false)String ip) throws Exception {
    //解决跨域调用
    response.setHeader("Access-Control-Allow-Origin", "*");
    Map<String, String> reqData = new HashMap<>(10);
    reqData.put("out_trade_no", String.valueOf(System.nanoTime()));
    reqData.put("trade_type", "MWEB");
    reqData.put("product_id", totalAmount);
    reqData.put("body", subjectName);
    // 订单总金额，单位为分
    reqData.put("total_fee", totalAmount);
    // APP和网页支付提交用户端ip，Native支付填调用微信支付API的机器IP。
    if (ip == null || "".equals(ip)){
        ip = getRemoteHost(request);
    }
    reqData.put("spbill_create_ip", ip);
    // 异步接收微信支付结果通知的回调地址，通知url必须为外网可访问的url，不能携带参数。
    reqData.put("notify_url", wxPayConfig.getNotifyUrl());
    // 自定义参数, 可以为终端设备号(门店号或收银设备ID)，PC网页或公众号内支付可以传"WEB"
    reqData.put("device_info", "");
    // 附加数据，在查询API和支付通知中原样返回，可作为自定义参数使用。
    reqData.put("attach", "");
    reqData.put("scene_info", "{\"h5_info\": {\"type\":\"Wap\",\"wap_url\": \""+prefixUrl+"/wxpay/order\",\"wap_name\": \""+subjectName+"\"}}");
    Map<String, String> responseMap = wxPay.unifiedOrder(reqData);
    logger.info(responseMap.toString());
    String returnCode = responseMap.get("return_code");
    String resultCode = responseMap.get("result_code");
    if (WXPayConstants.SUCCESS.equals(returnCode) && WXPayConstants.SUCCESS.equals(resultCode)) {
        // 预支付交易会话标识
        String prepayId = responseMap.get("prepay_id");
        // 支付跳转链接(前端需要在该地址上拼接redirect_url,该参数不是必须的)
        // 正常流程用户支付完成后会返回至发起支付的页面，如需返回至指定页面，则可以在MWEB_URL后拼接上redirect_url参数，来指定回调页面
        // 需对redirect_url进行urlencode处理

        // 由于没有实际账号，还没找到为啥不是普通链接的原因
        String mwebUrl = responseMap.get("mweb_url");
        logger.info(returnCode+"\t"+prepayId+"\t"+mwebUrl);
        return JSON.toJSONString(responseMap);
    }

    return responseMap;
}
```

#### 前端逻辑

向后台发起请求，获取url，且直接跳转到该url即可调出微信支付客户端

```javascript
// 创建订单 发送请求
$(document).on('click', '.wxpay', function() {
    //获取用户端ip
    var localIp = get_client_ip();
    $.ajax({
        type: 'get',
        url: "http://luciar.natapp1.cc/wxpay/order",
        dataType: 'json',
        jsonp: "callback",
        data: {
            'ip': localIp
        },
        success: function(data) {
            var mweb_url = data.mweb_url;
            console.log(mweb_url);
            window.location.href = mweb_url;
        },
        error: function() {

        }
    })
});
```

微信H5支付存在一个坑，要求发起请求的用户端ip和实际调用微信客户端支付的ip要一致

**经测试，部分浏览器使用前端传ip方式可以调出支付客户端，而另外一些却要用后端获取方式（代码如下），目前还不知道通过哪种方式解决，唯一庆幸的是这两种方法至少有一种是可以用的**

**前端获取ip**

引入依赖

```javascript
<script src="http://pv.sohu.com/cityjson?ie=utf-8"></script>
```

直接获取当前访问的ip

```javascript
function get_client_ip() {
   // document.write(returnCitySN["cip"] + ',' + returnCitySN["cname"])
   return returnCitySN["cip"];
}
```

 

**后端获取ip**

```java
private String getRemoteHost(HttpServletRequest request){
     String ipAddress = request.getHeader("x-forwarded-for");
     if(ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
         ipAddress = request.getHeader("Proxy-Client-IP");
     }
     if (ipAddress == null || ipAddress.length() == 0 || "unknow".equalsIgnoreCase(ipAddress)) {
         ipAddress = request.getHeader("WL-Proxy-Client-IP");
     }
     if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
         ipAddress = request.getRemoteAddr();

         if(ipAddress.equals("127.0.0.1") || ipAddress.equals("0:0:0:0:0:0:0:1")){
             //根据网卡获取本机配置的IP地址
             InetAddress inetAddress = null;
             try {
                 inetAddress = InetAddress.getLocalHost();
             } catch (UnknownHostException e) {
                 e.printStackTrace();
             }
             ipAddress = inetAddress.getHostAddress();
         }
     }
     //对于通过多个代理的情况，第一个IP为客户端真实的IP地址，多个IP按照','分割
     if(null != ipAddress && ipAddress.length() > 15){
         //"***.***.***.***".length() = 15
         if(ipAddress.indexOf(",") > 0){
             ipAddress = ipAddress.substring(0, ipAddress.indexOf(","));
         }
     }
     return ipAddress;
 }
```

 

### 4、JSAPI支付

#### 获取code

4.1、获取code

请求地址(回调地址一定要encode)

```http
https://open.weixin.qq.com/connect/oauth2/authorize?appid=你的appid&redirect_uri=你的回调地址&response_type=code&scope=snsapi_base&state=STATE#wechat_redirect
```

注意：

4.1.1`redirect_uri`参数：授权后重定向的回调链接地址, 请使用`urlEncode`对链接进行处理。

4.1.2 scope：用`snsapi_base`。

#### 换取openid

4.2、根据code 换取openid

```java
@PostMapping("getOpenId")
private String getOpenId(@RequestParam("code") String code) {
    String openId = "";
    String url = "https://api.weixin.qq.com/sns/oauth2/access_token?appid=" + wxPayConfig.getAppID() +
        "&secret=" + secret +
        "&code=" + code +
        "&grant_type=authorization_code";
    JSONObject jsonObject = HttpRequest.httpsRequestToJsonObject(url, "POST", null);
    logger.info("jsonObject:" + jsonObject);
    Object errorCode = jsonObject.get("errcode");
    if (errorCode != null) {
        logger.info("code不合法");
    } else {
        openId = jsonObject.getString("openid");
        logger.info("openId:" + openId);
    }
    return openId;
}
```

#### 统一下单

4.3、调用统一下单接口 成功后获取返回集合

```java
//通过code获取openId
String openId = getOpenId(code);
Map<String, String> reqData = new HashMap<>(10);
reqData.put("out_trade_no", String.valueOf(System.nanoTime()));
reqData.put("trade_type", "JSAPI");
reqData.put("openid", openId);
reqData.put("product_id", totalAmount);
reqData.put("body", subjectName);
// 订单总金额，单位为分
reqData.put("total_fee", totalAmount);
// APP和网页支付提交用户端ip，Native支付填调用微信支付API的机器IP。
reqData.put("spbill_create_ip", ip);
// 异步接收微信支付结果通知的回调地址，通知url必须为外网可访问的url，不能携带参数。
reqData.put("notify_url", wxPayConfig.getNotifyUrl());
// 自定义参数, 可以为终端设备号(门店号或收银设备ID)，PC网页或公众号内支付可以传"WEB"
reqData.put("device_info", "WEB");
// 附加数据，在查询API和支付通知中原样返回，可作为自定义参数使用。
reqData.put("attach", "");
reqData.put("scene_info", "{\"h5_info\": {\"type\":\"Wap\",\"wap_url\": \""+prefixUrl+"/wxpay/order\",\"wap_name\": \""+subjectName+"\"}}");
Map<String, String> responseMap = wxPay.unifiedOrder(reqData);
```

#### 生成paySign

4.4、根据预支付id 等参数生成paySign

**获取参数**

```java
Date beijingDate = Calendar.getInstance(Locale.CHINA).getTime();
String prepayId = responseMap.get("prepay_id");
String appId = responseMap.get("appid");
String nonceStr = responseMap.get("nonce_str");
String signType = "MD5";
String timeStamp = String.valueOf(beijingDate.getTime() / 1000);
```

**获取paySign**

其中`wxPayUtil`是微信自带的工具类里面的

```java
//获取paySign
Map<String, String> payMap = new HashMap<String, String>();
payMap.put("appId", appId);
payMap.put("timeStamp", timeStamp);
payMap.put("nonceStr",nonceStr);
payMap.put("signType",signType );
payMap.put("package", "prepay_id=" + prepayId);
String paySign = WXPayUtil.generateSignature(payMap, wxPayConfig.getKey());
```

返回数据

```java
payMap.put("paySign", paySign);
return JSON.toJSONString(payMap);
```

#### 前端拉起支付

4.5、前端拉起支付页面

```javascript
function onBridgeReady(){
    var a=(new Date()).toLocaleDateString();//获取当前日期
    WeixinJSBridge.invoke(
        'getBrandWCPayRequest', {
            "appId":$("#appId").val(),     //公众号名称，由商户传入
            "timeStamp":$("#timeStamp").val(),////时间戳，自1970年以来的秒数
            "nonceStr":$("#nonceStr").val(), //随机串
            "package":$("#package").val(),
            "signType":"MD5",         //微信签名方式：
            "paySign":$("#paySign").val() //微信签名
        },
        function(res){
            if(res.err_msg == "get_brand_wcpay_request:ok" ){
                // 使用以上方式判断前端返回,微信团队郑重提示：
                //res.err_msg将在用户支付成功后返回ok，但并不保证它绝对可靠。
                alert("支付成功");
                $("#data").val("支付成功");
                window.history.go(-1);
            }
        });
}
if (typeof WeixinJSBridge == "undefined"){
    if( document.addEventListener ){
        document.addEventListener('WeixinJSBridgeReady', onBridgeReady, false);
    }else if (document.attachEvent){
        document.attachEvent('WeixinJSBridgeReady', onBridgeReady);
        document.attachEvent('onWeixinJSBridgeReady', onBridgeReady);
    }
}else{
    onBridgeReady();
}
```

 

## 三、提示

### 1、跨域调用

因前后端分离，会存在跨域问题，可以用如下代码处理，只需将下面的代码加入到controller层

```html
response.setHeader("Access-Control-Allow-Origin", "*");
```



### 最后给大家安利一波：[WxJava](https://gitee.com/binary/weixin-java-tools/) 使用该微信工具包能够减少不少代码。所以大家能用尽量用封装好了。完成工作也快。项目进度在你手。

博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)