---
title:  "Spring Security 自定义短信登录"
categories: Spring Security
tags: Spring Security
author: LIUREN
---

# Spring Security 自定义短信登录

> Spring Securit自带的是用户账号密码登录, 如果需要用短信登录,要重新写一套验证逻辑. 手机号登录与用户名密码登录逻辑相同，所以我们在使用手机号登录系统的时候可以完全拷贝用户名密码登录的逻辑
>
> 作者：任未然
>
> 链接：https://www.jianshu.com/p/5c57ba5a40d8
>
> 来源：简书
>
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



# 用户登陆的整个处理逻辑

用户登陆的整个处理逻辑：spring security在验证的时候是通过过滤器来完成，通过 用户名密码过滤器拦截到用户的登录请求(根据请求方式以及请求uri地址)，然后将给AuthenticationManager(认证管理器)来处理，因为AuthenticationManager(具体是由ProviderManager)管理着众多的Provider, 对于用户请求的过滤器就通过遍历的方式，来确定由那个Provider(DaoAuthenticationProvider)来处理对应的过滤器，在这个Provider中获取用户名到数据库中查询用户信息（UserDetailsService），比对用户的密码。



#### 导入依赖配置

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.17</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.social</groupId>
            <artifactId>spring-social-web</artifactId>
            <version>1.1.6.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-text</artifactId>
            <version>1.8</version>
        </dependency>
        <!-- 阿里云发送短信 -->
        <dependency>
            <groupId>com.aliyun</groupId>
            <artifactId>aliyun-java-sdk-core</artifactId>
            <version>4.0.3</version>
        </dependency>
```

#### 1. 编写Token

编写手机号认证Token, 模仿UsernamePasswordAuthenticationToken这个类来实现。

```java
/**
 * 短信验证码Token, 用于封装用户使用手机登录的相关信息。
 */
public class SmsAuthenticationToken extends AbstractAuthenticationToken {
    private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;
    /**
     * 在验证之前封装电话信息，
     * 在验证之后存储 用户信息
     */
    private final Object principal;
    public SmsAuthenticationToken(Object principal) {
        super(null);
        this.principal = principal;
        setAuthenticated(false);
    }
    public SmsAuthenticationToken(Object principal,
                                  Collection<? extends GrantedAuthority> authorities) {
        super(authorities);
        this.principal = principal;
        super.setAuthenticated(true); // must use super, as we override
    }
    @Override
    public Object getCredentials() {
        return null;
    }
    @Override
    public Object getPrincipal() {
        return this.principal;
    }
    @Override
    public void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException {
        if (isAuthenticated) {
            throw new IllegalArgumentException(
                    "Cannot set this token to trusted - use constructor which takes a GrantedAuthority list instead");
        }
        super.setAuthenticated(false);
    }
    @Override
    public void eraseCredentials() {
        super.eraseCredentials();
    }
}
```

#### 2. 编写Filter

手机号的过滤器可以模仿 UsernamePasswordAuthenticationFilter 来实现。

```java
/**
 * 处理用户登录的过滤器。
 * 过滤器的作用就是获取到手机号，然后封装成 SmsAuthenticationToken
 */
public class SmsAuthenticationFilter extends AbstractAuthenticationProcessingFilter {
    public static final String SPRING_SECURITY_FORM_MOBILE_KEY = "mobile";
    private String mobileParameter = SPRING_SECURITY_FORM_MOBILE_KEY;
    private boolean postOnly = true;
    public SmsAuthenticationFilter() {
        super(new AntPathRequestMatcher("/authentication/sms", "POST"));
    }
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request,
                                                HttpServletResponse response) throws AuthenticationException {
        if (postOnly && !request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException(
                    "Authentication method not supported: " + request.getMethod());
        }
        String mobile = obtainMobile(request);
        if (mobile == null) {
            mobile = "";
        }
        // 反射
        mobile = mobile.trim();
        SmsAuthenticationToken authRequest = new SmsAuthenticationToken(mobile);
        // Allow subclasses to set the "details" property
        setDetails(request, authRequest);
        return this.getAuthenticationManager().authenticate(authRequest);
    }
    protected String obtainMobile(HttpServletRequest request) {
        return request.getParameter(mobileParameter);
    }
    protected void setDetails(HttpServletRequest request,
                              SmsAuthenticationToken authRequest) {
        authRequest.setDetails(authenticationDetailsSource.buildDetails(request));
    }
    public void setUsernameParameter(String mobileParameter) {
        Assert.hasText(mobileParameter, "Username parameter must not be empty or null");
        this.mobileParameter = mobileParameter;
    }
    public void setPostOnly(boolean postOnly) {
        this.postOnly = postOnly;
    }

    public final String getMobileParameter() {
        return mobileParameter;
    }
}
```

#### 3. 编写Provider

Provider的作用是用来处理对应的Token，校验用户名密码使用的Provider为DaoAuthenticationProvider,  在实现我们自己的Provider的时候，我们去实现AuthenticationProvider。

```java
/**
 * 具体校验处理逻辑
 */
public class SmsAuthenticatoinProvider implements AuthenticationProvider {
    private UserSecurityService userSecurityService;
    public UserSecurityService getUserSecurityService() {
        return userSecurityService;
    }
    public void setUserSecurityService(UserSecurityService userSecurityService) {
        this.userSecurityService = userSecurityService;
    }
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        //过滤器传过来的SmsAuthenticationToken
        SmsAuthenticationToken smsAuthenticationToken = (SmsAuthenticationToken)authentication;
        String mobile = (String)smsAuthenticationToken.getPrincipal();  //获取到电话
        UserDetails userDetails = userSecurityService.loadUserByUsername(mobile);
        SmsAuthenticationToken token = new SmsAuthenticationToken(userDetails, userDetails.getAuthorities());
        // 设置用户的其他的详细信息(登录一些)
        token.setDetails(smsAuthenticationToken.getDetails());
        return token;
    }
    /**
     * 该方法就是来判断，该Provider要处理哪一个Filter丢过来的Token,
     * 返回true, 上面的方法 authenticate(Authentication authentication)
     */
    @Override
    public boolean supports(Class<?> authentication) {
        // 判断类型是否一致
        return authentication.isAssignableFrom(SmsAuthenticationToken.class);
    }
}
```

#### 发送短信实现

##### 短信发送模板

```java
public abstract class AbstractSendSmsCode {
    private String accessKeyId;  //运营商提供的key
    private String accessKeySecret; // 运营商提供的secret
    private String templateId; //模板ID
    private String code; //随机验证码
    private String mobile; //手机号
    public AbstractSendSmsCode(String accessKeyId, String accessKeySecret, String templateId, String code, String mobile) {
        this.accessKeyId = accessKeyId;
        this.accessKeySecret = accessKeySecret;
        this.templateId = templateId;
        this.code = code;
        this.mobile = mobile;
    }
    public String getAccessKeyId() {
        return accessKeyId;
    }
    public void setAccessKeyId(String accessKeyId) {
        this.accessKeyId = accessKeyId;
    }
    public String getAccessKeySecret() {
        return accessKeySecret;
    }
    public void setAccessKeySecret(String accessKeySecret) {
        this.accessKeySecret = accessKeySecret;
    }
    public String getTemplateId() {
        return templateId;
    }
    public void setTemplateId(String templateId) {
        this.templateId = templateId;
    }
    public String getCode() {
        return code;
    }
    public void setCode(String code) {
        this.code = code;
    }
    public String getMobile() {
        return mobile;
    }
    public void setMobile(String mobile) {
        this.mobile = mobile;
    }
    //不同运营商发送短信逻辑不同
    public abstract Object sendSms();
}
```

##### 具体继承类

```java
public class AliSmsSendCodeService extends AbstractSendSmsCode {
    //签名, 阿里独有的方式
    private String sign;
    public AliSmsSendCodeService(String accessKeyId, String accessKeySecret,
                                 String templateId, String code, String mobile, String sign) {
        super(accessKeyId, accessKeySecret, templateId, code, mobile);
        this.sign = sign;
    }
    public String getSign() {
        return sign;
    }
    public void setSign(String sign) {
        this.sign = sign;
    }
    @Override
    public Object sendSms() {
        DefaultProfile profile = DefaultProfile.getProfile("cn-hangzhou", this.getAccessKeyId(), this.getAccessKeySecret());
        IAcsClient client = new DefaultAcsClient(profile);
        CommonRequest request = new CommonRequest();
        request.setMethod(MethodType.POST);
        request.setDomain("dysmsapi.aliyuncs.com");
        request.setVersion("2017-05-25");
        request.setAction("SendSms");
        request.putQueryParameter("RegionId", "cn-hangzhou");
        request.putQueryParameter("PhoneNumbers", this.getMobile());  //手机号
        request.putQueryParameter("SignName", this.getSign()); //签名保证数据的安装传输
        request.putQueryParameter("TemplateCode", this.getTemplateId());  //模板id
        // 因为在模板中定义了  $code
        request.putQueryParameter("TemplateParam", "{\"code\":\"" + this.getCode() + "\"}"); //
        try {
            CommonResponse response = client.getCommonResponse(request);
            return response;
        } catch (ServerException e) {
            e.printStackTrace();
        } catch (ClientException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

#### 短信验证码Filter

```java
/**
 * 验证短信验证码的过滤器。
 */
@Component
public class SmsCodeValidateFilter extends OncePerRequestFilter {
    private SessionStrategy sessionStrategy = new HttpSessionSessionStrategy();
    @Autowired
    private MyAuthenticationFailureHandler myAuthenticationFailureHandler;
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        if(request.getMethod().equals("POST") && "/authentication/sms".equals(request.getRequestURI())) {


            try{
                validateSmsCode(request);
            }catch (InternalAuthenticationServiceException exception) {
                myAuthenticationFailureHandler.onAuthenticationFailure(request, response, exception);
                return;
            }

            filterChain.doFilter(request, response);
        }else {
            filterChain.doFilter(request, response);
        }
    }
    private void validateSmsCode(HttpServletRequest request) {
        String smsCode = request.getParameter("smsCode"); //获取短信验证码
        // 取到验证码
        ValidateCode validateCode = (ValidateCode)sessionStrategy.getAttribute(new ServletWebRequest(request),
                SmsController.SMS_CODE_KEY);
        // 验证码不能为空或者
        if(StringUtils.isEmpty(smsCode) || "".equals(smsCode.trim())) {
            throw new InternalAuthenticationServiceException("短信验证码不能为空.");
        }
        if(validateCode.isExpire()) {  //是否过期
            throw new InternalAuthenticationServiceException("短信验证码过期.");
        }
        if(null == validateCode) {
            throw new InternalAuthenticationServiceException("短信验证码不存在.");
        }
        if(!smsCode.equals(validateCode.getCode())) {
            throw new InternalAuthenticationServiceException("短信验证码不正确");
        }
    }
}
```

#### 自定义认证

```java
// 该类的作用是处理用户登录名和密码
@Component
public class UserSecurityService implements UserDetailsService {
    private static Logger logger = LoggerFactory.getLogger(UserSecurityService.class);
    @Resource
    private SysUserService sysUserService;
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        logger.info("用户名或者电话：" + username);
        SysUser sysUser = null;
        try{
            sysUser = sysUserService.getSysUserByUsenameOrMobile(username);
        }catch (EmptyResultDataAccessException exception) {  //没有对应的用户名异常
            throw new UsernameNotFoundException("用户或密码错误");
        }
        if(null == sysUser) {
            throw new UsernameNotFoundException("用户名或密码错误");
        }else {
            /**
             * User第一参数是：用户名
             *     第二个参数是：pssword, 是从数据库查出来的
             *     第三个参数是: 权限
             */
            User user =  null;
            try{
                user = new User(username,
                        sysUser.getPassword(),
                        Arrays.asList(new SimpleGrantedAuthority("admin")));
            }catch (InternalAuthenticationServiceException exception) {
                throw exception;  // 在此处，将异常接着往外抛，抛给AuthenticationFailureHandler处理
            }
            return user;
        }
    }
}
```

#### 安全配置

```java
@Configuration
public class SmsAuthenticationConfig extends SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity> {
    @Autowired
    private MySuccessAuthenticationHandler mySuccessAuthenticationHandler;
    @Autowired
    private MyAuthenticationFailureHandler myAuthenticationFailureHandler;
    @Autowired
    private UserSecurityService userSecurityService;
    @Override
    public void configure(HttpSecurity http) throws Exception {
        // 实例化过滤器
        SmsAuthenticationFilter filter = new SmsAuthenticationFilter();
        // 设置AuthenticatoinManager, 因为filter和Provider中间的桥梁就是 AuthenticationManager
        filter.setAuthenticationManager(http.getSharedObject(AuthenticationManager.class));
        filter.setAuthenticationSuccessHandler(mySuccessAuthenticationHandler);   //设置成功后的处理逻辑
        filter.setAuthenticationFailureHandler(myAuthenticationFailureHandler);
        SmsAuthenticatoinProvider smsAuthenticatoinProvider = new SmsAuthenticatoinProvider();
        /**
         *  因为在 SmsAuthenticatoinProvider要根据电话查询用户信息
         *  UserDetails userDetails = userSecurityService.loadUserByUsername(mobile);
         */
        smsAuthenticatoinProvider.setUserSecurityService(userSecurityService);

        // 统一设置Filter和Provider
        http.authenticationProvider(smsAuthenticatoinProvider)
                .addFilterAfter(filter, UsernamePasswordAuthenticationFilter.class);
    }
}
```

```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    /**
     * 该bean的作用是，在UserDetailsService接口的loadUserByUsername返回的UserDetail中包含了
     * password, 该bean就将用户从页面提交过来的密码进行处理，处理之后与UserDetail中密码进行比较。
     * @return
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    @Autowired
    private MySuccessAuthenticationHandler mySuccessAuthenticationHandler;
    @Autowired
    private MyAuthenticationFailureHandler myAuthenticationFailureHandler;
    @Autowired
    private DataSource dataSource;
    @Autowired
    private UserSecurityService userSecurityService;
    @Autowired
    private ImageCodeValidateFilter imageCodeValidateFilter;
    @Autowired
    private SmsCodeValidateFilter smsCodeValidateFilter;
    @Autowired
    private SmsAuthenticationConfig smsAuthenticationConfig;
    @Bean
    public PersistentTokenRepository persistentTokenRepository() {
        JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
        jdbcTokenRepository.setDataSource(dataSource);
        jdbcTokenRepository.setCreateTableOnStartup(true);  //true, 自动创建表，
        return jdbcTokenRepository;
    }
    @Override
    protected void configure(HttpSecurity http) throws Exception {

        // 意思是将图片验证码过滤器，加载用户名密码验证过滤器之前
        http.addFilterBefore(imageCodeValidateFilter, UsernamePasswordAuthenticationFilter.class) //
                .addFilterBefore(smsCodeValidateFilter, UsernamePasswordAuthenticationFilter.class)
                .formLogin()  //使用form进行登录
            .loginPage("/login.html")   //指定登录页面
                .loginProcessingUrl("/authentication/form")  //表示form往哪里进行提交
                .successHandler(mySuccessAuthenticationHandler)   //成功后的处理
                .failureHandler(myAuthenticationFailureHandler)   //失败处理
                .and()
                .rememberMe()
                .tokenValiditySeconds(36000000)
                .tokenRepository(persistentTokenRepository())
                .userDetailsService(userSecurityService)    // 因为用户传入过来的token, 需要再次进行校验
                .alwaysRemember(true)
                .and()  //表示进行其他的配置
                .authorizeRequests()   //表示所有的都需要认证
                .antMatchers("/login.html", "/js/**", "/css/**", "/sms/code",
                        "/validate/code").permitAll() //意思是让登录页面直接过
                .anyRequest() // 对于所有的请求
                .authenticated()
                .and()
                .csrf().disable()
                .apply(smsAuthenticationConfig);  //引用手机号登录整个逻辑
    }
    public static void main(String[] args) {
        BCryptPasswordEncoder bCryptPasswordEncoder = new BCryptPasswordEncoder();
        System.out.println(bCryptPasswordEncoder.encode("1"));
    }
}
```

#### Service

```java
@短信发送Service
public class SmsService {
    @Value("${aliyun.sms.accessKeyId}")
    private String accessKeyId;
    @Value("${aliyun.sms.accessKeySecret}")
    private String accessKeySecret;
    @Value("${aliyun.sms.templateId}")
    private String templateId;
    @Value("${aliyun.sms.sign}")
    private String sign;
    // 发送短信, 要调用第三方的短信运营商
    public void send(String code, String mobile) {
        // http://wwww.cloud.alibaba/sms
        /**
        System.out.println("模板Id为：" + templateId);
        System.out.println("往手机号为：" + mobile + " 发送的验证码为：" + code);
         */
        AbstractSendSmsCode abstractSendSmsCode =
                new AliSmsSendCodeService(accessKeyId, accessKeySecret, templateId, code, mobile, sign);
        CommonResponse commonResponse = (CommonResponse)abstractSendSmsCode.sendSms();
    }
}
```

#### 短信发送controller

```java
@Controller
@RequestMapping("/sms/code")
public class SmsController {
    public final static String SMS_CODE_KEY = "SMS_CODE_KEY";
    @Resource
    private SmsService smsService;
    // Session的工具类
    private SessionStrategy sessionStrategy = new HttpSessionSessionStrategy();
    // 发送短信
    @RequestMapping
    public void sendSms(HttpServletRequest request, String mobile) {
        // 生成随机数子的工具类
        RandomStringGenerator randomStringGenerator
                = new RandomStringGenerator.Builder().withinRange(new char[]{'0','9'}).build();
        //randomStringGenerator.generate(4) 生成0-9的四位随机字符串
        ValidateCode validateCode = new ValidateCode(randomStringGenerator.generate(4) ,120);
        sessionStrategy.setAttribute(new ServletWebRequest(request), SMS_CODE_KEY, validateCode);
        System.out.println("短信验证码：" + validateCode.getCode());
        smsService.send(validateCode.getCode(), mobile); //调用短信运营商的发送短信的功能
    }
}
```

#### 页面的实现



![](https://www.codepeople.cn/imges/passwordlogin.png)

![](https://www.codepeople.cn/imges/smslogin.png)

博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)