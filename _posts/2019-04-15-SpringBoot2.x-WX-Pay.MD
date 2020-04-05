---
title:  "SpringBoot2.0小程序支付功能实现weixin-java-pay"
categories: SpringBoot2.0 Shiro
tags: SpringBoot2.0 Shiro
author: LIUREN
---

# SpringBoot2.0小程序支付功能实现weixin-java-pay

>  WxJava - 微信开发 Java SDK（开发工具包); 支持包括微信支付、开放平台、公众号、企业微信/企业号、小程序等微信功能的后端开发。



## 第一步： SDK使用方式

Maven方式引入：

```xml
<dependency>
  <groupId>com.github.binarywang</groupId>
  <artifactId>（不同模块参考下文）</artifactId>
  <version>3.3.0</version>
</dependency>
```

各模块的**artifactId:**

- 微信小程序：`weixin-java-miniapp`
- 微信支付：`weixin-java-pay`
- 微信开放平台：`weixin-java-open`
- 公众号（包括订阅号和服务号）：`weixin-java-mp`
- 企业号/企业微信：`weixin-java-cp`



## 第二步：配置微信相关信息

application.yml

```yaml
#微信小程序配置
wx:
  miniapp:
    configs:
        - appid: #微信公众号或者小程序等的appid 必填
          secret: #微信公众号或者小程序等的secret 必填
          token: #微信小程序消息服务器配置的token
          aesKey: #微信小程序消息服务器配置的EncodingAESKey
          msgDataFormat: JSON
  pay:
    appId:  #微信公众号或者小程序等的appid 必填
    mchId:  #微信支付商户号    必填
    mchKey: #微信支付商户密钥
    subAppId: #服务商模式下的子商户公众账号ID
    subMchId: #服务商模式下的子商户号
    keyPath: classpath:cert/apiclient_cert.p12 # p12证书的位置，可以指定绝对路径，也可以指定类路径（以classpath:开头）
  notifyUrl: https://www.xxx.com/wx/notify #微信支付回调地址 自己定义但是必须放到外网微信可以访问的到
```



## 第三步：编写统一订单接口业务

调用统一下单支付接口:**wxService.createOrder(orderRequest)**

```java
/**
 * @Title: WxUserVedioController.java
 * @Package io.renren.modules.wx.controller
 * @Description: 
 * Copyright: Copyright (c) 2019 www.codepeople.cn Inc. All rights reserved. 
 * Website: www.codepeople.cn
 * 
 * @Author 刘仁
 * @DateTime 2019年4月6日 上午11:54:08
 * @version V1.0
 */

package io.renren.modules.wx.controller;

import java.math.BigDecimal;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

import com.baomidou.mybatisplus.core.conditions.update.UpdateWrapper;
import com.github.binarywang.wxpay.bean.notify.WxPayNotifyResponse;
import com.github.binarywang.wxpay.bean.notify.WxPayOrderNotifyResult;
import com.github.binarywang.wxpay.bean.request.WxPayUnifiedOrderRequest;
import com.github.binarywang.wxpay.bean.result.BaseWxPayResult;
import com.github.binarywang.wxpay.constant.WxPayConstants;
import com.github.binarywang.wxpay.exception.WxPayException;
import com.github.binarywang.wxpay.service.WxPayService;

import io.renren.common.utils.DateUtils;
import io.renren.common.utils.IPUtils;
import io.renren.common.utils.PageUtils;
import io.renren.common.utils.R;
import io.renren.modules.soft.entity.AuthorizationEntity;
import io.renren.modules.soft.entity.HospitalEntity;
import io.renren.modules.soft.entity.OrderEntity;
import io.renren.modules.soft.entity.PayFlowEntity;
import io.renren.modules.soft.entity.PurchaseRecordEntity;
import io.renren.modules.soft.entity.RefundRecordEntity;
import io.renren.modules.soft.service.AuthorizationService;
import io.renren.modules.soft.service.BedBaseService;
import io.renren.modules.soft.service.DeviceService;
import io.renren.modules.soft.service.HospitalService;
import io.renren.modules.soft.service.OrderService;
import io.renren.modules.soft.service.PayFlowService;
import io.renren.modules.soft.service.PurchaseRecordService;
import io.renren.modules.soft.service.RefundRecordService;
import io.renren.modules.soft.service.StatementService;
import io.renren.modules.soft.service.ThirdUserService;
import io.renren.modules.soft.service.UserAuthorizationService;
import io.renren.modules.soft.service.UserService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import lombok.extern.slf4j.Slf4j;

/**
 * @ClassName: WxUserVedioController
 * @Description: 
 * @Author 刘仁
 * @DateTime 2019年4月6日 上午11:54:08 
 */
@Api(tags = "普通用户操作接口")
@RestController
@RequestMapping("/wx")
@Slf4j
public class WxxxxUserVedioController {
	@Value("${wx.notifyUrl}")
	private String notifyUrl;
	@Autowired
	private WxPayService wxService;
	@Autowired
	private HospitalService hospitalService;
	@Autowired
	private PayFlowService payFlowService;
	@Autowired
	private OrderService orderService;

	/**
     * @Title: buyVedio
     * @Description: 
     * @Author 刘仁
     * @DateTime 2019年4月6日 下午12:12:11
     * @param
     * @param
     * @param
     * @return
     */
    @PostMapping("/user/createOrder")
    public R createOrder(HttpServletRequest request,@RequestBody Map<String,String> map)  throws WxPayException{
    	String bed_id = map.get("bedId");
    	log.info("参数的bed_id：{}", bed_id);
    	if (StringUtils.isAnyBlank(bed_id)) {
    		return R.error("参数不能为null");
		}
    	Long bedId = Long.valueOf(bed_id).longValue();
    	//根据bedId 查找所属医院的价格
	    HospitalEntity hospitalEntity = hospitalService.hospitalByBedId(bed_id.trim());
	    BigDecimal money = hospitalEntity.getCharge();

	    HttpSession session = request.getSession();
    	String mobile = (String) session.getAttribute("userphone");
	    String userId = (String) session.getAttribute("userId");
    	// TODO 插入订单记录
    	// TODO 记录用户的购买记录
    	// TODO 调用统一生成订单接口
    	WxPayUnifiedOrderRequest orderRequest = new WxPayUnifiedOrderRequest();
    	orderRequest.setSignType(WxPayConstants.SignType.MD5);
    	orderRequest.setBody("中国科技");
        orderRequest.setOutTradeNo(order_No); //自己生成order_No
        orderRequest.setTradeType(WxPayConstants.TradeType.JSAPI);
//        orderRequest.setTotalFee(BaseWxPayRequest.yuanToFen(yuanMoney));//直接分
        orderRequest.setTotalFee(money.intValue());//直接分
        orderRequest.setOpenid(userEntity.getOpenid()); // 获取微信支付用户的openId
        orderRequest.setSpbillCreateIp(IPUtils.getIpAddr(request));
        Date now = new Date();
        Date afterDate = DateUtils.addDateMinutes(now, 10);//10分钟后
        orderRequest.setTimeStart(DateUtils.format(now, "yyyyMMddHHmmss"));
        orderRequest.setTimeExpire(DateUtils.format(afterDate, "yyyyMMddHHmmss"));
        orderRequest.setNotifyUrl(notifyUrl);
	    Object order = wxService.createOrder(orderRequest);
	    
	    return R.ok().put("order", order);
    }
    
    @ApiOperation("微信支付回调地址")
    @ResponseBody
    @PostMapping("/notify")//返回订单号
    public String payNotify(HttpServletRequest request, HttpServletResponse response) {
      try {
    	/*HttpSession session = request.getSession();
    	String mobile = (String) session.getAttribute("userphone");
    	if (StringUtils.isBlank(mobile)) {
    		return R.error(401, "session获取不到授权手机号！");
		}
      	//获取用户手机号,根据用户手机号获取用户ID
      	AuthorizationEntity user = authorizationService.getOneByMobile(mobile);*/
        String xmlResult = IOUtils.toString(request.getInputStream(), request.getCharacterEncoding());
        WxPayOrderNotifyResult notifyResult = wxService.parseOrderNotifyResult(xmlResult);
        // 结果正确 outTradeNo
        String orderId = notifyResult.getOutTradeNo();
        String tradeNo = notifyResult.getTransactionId();
        String totalFee = BaseWxPayResult.fenToYuan(notifyResult.getTotalFee());
        if("SUCCESS".equals(notifyResult.getResultCode())) {
        	PayFlowEntity entity = new PayFlowEntity();
            entity.setPayFee(BigDecimal.valueOf(notifyResult.getCashFee()));
            entity.setPayFlowNo(orderId);
            entity.setPayUserInfo(notifyResult.getOpenid());
            entity.setThreeInNo(tradeNo);
            payFlowService.save(entity);
            OrderEntity order_entity = new OrderEntity();
            SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMddHHmmss");
            order_entity.setCreateTime(new Date());
            //更新订单信息
            UpdateWrapper<OrderEntity> updateWrapper = new UpdateWrapper<>();
            updateWrapper.set("order_status", "1");
            updateWrapper.set("pay_id", notifyResult.getOpenid());
            updateWrapper.set("pay_status", "1");
            updateWrapper.set("pay_end_time", sdf.parse(notifyResult.getTimeEnd()));
	        updateWrapper.eq("order_no",notifyResult.getOutTradeNo());
            orderService.update(updateWrapper);
        }
        //自己处理订单的业务逻辑，需要判断订单是否已经支付过，否则可能会重复调用
         return WxPayNotifyResponse.success("成功");
      } catch (Exception e) {
        log.error("微信回调结果异常,异常原因{}", e.getMessage());
       // WxPayNotifyResponse.fail(e.getMessage());

        return WxPayNotifyResponse.success("code:"+9999+"微信回调结果异常,异常原因:"+e.getMessage());
      }
    }
    
}
```

## 第四步：小程序端可以操作了

小程序端主要先调用小程序接口获取用户信息主要是code,然后传给后台，后台通过code获取openid获取方式：

```java
WxMaService  wxService = WxMaConfiguration.getMaService(appid);
			WxMaJscode2SessionResult wx_session = wxService.getUserService().getSessionInfo(code);
			openId = wx_session.getOpenid();
			log.info("通过微信获取用户的openId==>{}", openId);
```

然后根据回调地址返回的数据，小程序就可以对数据进行解析封装调用支付接口进行付款了。

这里主要讲的后端的使用方法



==================================================================

[博客地址](https://www.codepeople.cn)：<https://www.codepeople.cn>

==================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)