---
title:  "SpringBoot+Shiro手机号码登录 多realm认证登陆 密码不正确时抛出错误异常，无法捕获问题"
categories: Shiro
tags: SpringBoot
author: LIUREN
---

## Shiro 多realm认证登陆 密码不正确时抛出错误异常，无法捕获问题

>   大部分场景下，我们都会在项目中实现自定义 `Realm` 搭配 `UsernamePasswordToken` 来完成用户的登录认证流程，但是如果登录方式包括“第三方登录”、“手机号登录”等，仅凭 `UsernamePasswordToken` 就难以实现了，因为以上的两种登录方式都是免密登录，而 `UsernamePasswordToken` 却必须要有 `username` 和 `password`，因此需要自定义多个 Realm 和 Token 才能实现上述功能。当多个Realm同时存在时候异常捕获就会出现问题，原来的异常会被最新的Realm覆盖，导致原来的异常信息被覆盖，捕获不到一直抛`Authentication token of type [class org.apache.shiro.authc.UsernamePasswordToken] could not be authenticated by any configured realms. Please ensure that at least one realm can authenticate this tokens.`
>
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。





### 1. `原来的UserRealm`

```java
package com.kelan.framework.shiro.realm;

import java.util.HashSet;
import java.util.Set;

import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.ExcessiveAttemptsException;
import org.apache.shiro.authc.IncorrectCredentialsException;
import org.apache.shiro.authc.LockedAccountException;
import org.apache.shiro.authc.SimpleAuthenticationInfo;
import org.apache.shiro.authc.UnknownAccountException;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;

import com.kelan.common.exception.user.CaptchaException;
import com.kelan.common.exception.user.RoleBlockedException;
import com.kelan.common.exception.user.UserBlockedException;
import com.kelan.common.exception.user.UserNotExistsException;
import com.kelan.common.exception.user.UserPasswordNotMatchException;
import com.kelan.common.exception.user.UserPasswordRetryLimitExceedException;
import com.kelan.framework.shiro.service.SysLoginService;
import com.kelan.framework.util.ShiroUtils;
import com.kelan.system.domain.SysUser;
import com.kelan.system.service.ISysMenuService;
import com.kelan.system.service.ISysRoleService;

/**
 * 自定义Realm 处理登录 权限
 * 
 * @author kelan
 */
public class UserRealm extends AuthorizingRealm
{
    private static final Logger log = LoggerFactory.getLogger(UserRealm.class);

    @Autowired
    private ISysMenuService menuService;

    @Autowired
    private ISysRoleService roleService;

    @Autowired
    private SysLoginService loginService;

    /**
     * 授权
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection arg0)
    {
        SysUser user = ShiroUtils.getSysUser();
        // 角色列表
        Set<String> roles = new HashSet<String>();
        // 功能列表
        Set<String> menus = new HashSet<String>();
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        // 管理员拥有所有权限
        if (user.isAdmin())
        {
            info.addRole("admin");
            info.addStringPermission("*:*:*");
        }
        else
        {
            roles = roleService.selectRoleKeys(user.getUserId());
            menus = menuService.selectPermsByUserId(user.getUserId());
            // 角色加入AuthorizationInfo认证对象
            info.setRoles(roles);
            // 权限加入AuthorizationInfo认证对象
            info.setStringPermissions(menus);
        }
        return info;
    }

    /**
     * 登录认证
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException
    {
        UsernamePasswordToken upToken = (UsernamePasswordToken) token;
        String username = upToken.getUsername();
        String password = "";
        if (upToken.getPassword() != null)
        {
            password = new String(upToken.getPassword());
        }

        SysUser user = null;
        try
        {
            user = loginService.login(username, password);
        }
        catch (CaptchaException e)
        {
            throw new AuthenticationException(e.getMessage(), e);
        }
        catch (UserNotExistsException e)
        {
            throw new UnknownAccountException(e.getMessage(), e);
        }
        catch (UserPasswordNotMatchException e)
        {
            throw new IncorrectCredentialsException(e.getMessage(), e);
        }
        catch (UserPasswordRetryLimitExceedException e)
        {
            throw new ExcessiveAttemptsException(e.getMessage(), e);
        }
        catch (UserBlockedException e)
        {
            throw new LockedAccountException(e.getMessage(), e);
        }
        catch (RoleBlockedException e)
        {
            throw new LockedAccountException(e.getMessage(), e);
        }
        catch (Exception e)
        {
            log.info("对用户[" + username + "]进行登录验证..验证未通过{}", e.getMessage());
            throw new AuthenticationException(e.getMessage(), e);
        }
        SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(user, password, getName());
        return info;
    }

    /**
     * 清理缓存权限
     */
    public void clearCachedAuthorizationInfo()
    {
        this.clearCachedAuthorizationInfo(SecurityUtils.getSubject().getPrincipals());
    }
    
    @Override
	public boolean supports(AuthenticationToken token) {
		return token instanceof UsernamePasswordToken;
	}
}
```

必须重写 `supports()` 方法，在后面起到了至关重要的作用。

### 2. `手机短信登录的UserMobileRealm`

```java
package com.kelan.framework.shiro.realm;

import java.util.HashSet;
import java.util.Set;

import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.SimpleAuthenticationInfo;
import org.apache.shiro.authc.credential.AllowAllCredentialsMatcher;
import org.apache.shiro.authc.credential.CredentialsMatcher;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.beans.factory.annotation.Autowired;

import com.kelan.common.constant.Constants;
import com.kelan.common.exception.user.CaptchaException;
import com.kelan.common.exception.user.UserNotExistsException;
import com.kelan.common.utils.MessageUtils;
import com.kelan.framework.config.MobileToken;
import com.kelan.framework.manager.AsyncManager;
import com.kelan.framework.manager.factory.AsyncFactory;
import com.kelan.framework.shiro.service.SysLoginService;
import com.kelan.framework.util.ShiroUtils;
import com.kelan.system.domain.SysUser;
import com.kelan.system.service.ISysMenuService;
import com.kelan.system.service.ISysRoleService;

public class UserMobileRealm extends AuthorizingRealm {

    @Autowired
    private ISysMenuService menuService;

    @Autowired
    private ISysRoleService roleService;

    @Autowired
    private SysLoginService loginService;

    @Override
    public void setCredentialsMatcher(CredentialsMatcher credentialsMatcher) {
        //这儿的CredentialsMatcher的new的对象必须是AllowAllCredentialsMatcher
        CredentialsMatcher matcher = new AllowAllCredentialsMatcher();
        super.setCredentialsMatcher(matcher);
    }
    /**
     * 授权
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection arg0)
    {
        SysUser user = ShiroUtils.getSysUser();
        // 角色列表
        Set<String> roles = new HashSet<String>();
        // 功能列表
        Set<String> menus = new HashSet<String>();
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        // 管理员拥有所有权限
        if (user.isAdmin())
        {
            info.addRole("admin");
            info.addStringPermission("*:*:*");
        }
        else
        {
            roles = roleService.selectRoleKeys(user.getUserId());
            menus = menuService.selectPermsByUserId(user.getUserId());
            // 角色加入AuthorizationInfo认证对象
            info.setRoles(roles);
            // 权限加入AuthorizationInfo认证对象
            info.setStringPermissions(menus);
        }
        return info;
    }

    /**
     * 登录认证
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException
    {
        MobileToken mobileToken = null;
        if (token instanceof MobileToken) {
			mobileToken = (MobileToken) token;
		} else {
			return null;
		}
        //获取我发送验证码存入session中的验证码和手机号
        String sessionmobileVerificationCode = (String)SecurityUtils.getSubject().getSession().getAttribute("mobileVerificationCode");
        String sessionmobile = (String)SecurityUtils.getSubject().getSession().getAttribute("mobile");
        String mobile = mobileToken.getPhoneNum();//PC端获取的用户手机号码
        String mobileVerificationCode = mobileToken.getMobileVerificationCode();//PC端填写的手机收到的短信验证码
        //1.先判断session中存储的手机号和用户填写的手机号是否一致，如果不一致就直接报错
        if (!sessionmobile.trim().equalsIgnoreCase(mobile)) {
        	 AsyncManager.me().execute(AsyncFactory.recordLogininfor(mobile, Constants.LOGIN_FAIL, MessageUtils.message("not.null")));
             throw new UserNotExistsException();
		}
        //2.判断验证码是否正确
        if (!sessionmobileVerificationCode.trim().equalsIgnoreCase(mobileVerificationCode)) {
        	AsyncManager.me().execute(AsyncFactory.recordLogininfor(mobile, Constants.LOGIN_FAIL, MessageUtils.message("user.jcaptcha.error")));
        	throw new CaptchaException();
		}
        SysUser user = loginService.mobileLogin(mobile); 
        return new SimpleAuthenticationInfo(user,mobile,getName());
    }

    /**
     * 清理缓存权限
     */
    public void clearCachedAuthorizationInfo()
    {
        this.clearCachedAuthorizationInfo(SecurityUtils.getSubject().getPrincipals());
    }
    
    
	@Override
	public boolean supports(AuthenticationToken token) {
		return token instanceof MobileToken;
	}
    
}
```

必须重写 `supports()` 方法，在后面起到了至关重要的作用。

### 3. 自定义MobileToken

网上很多文章都继承 `UsernamePasswordToken` 来创建自己的 Token，但我不建议这样写，如果继承 `UsernamePasswordToken` ，在后面的操作中会变得相对麻烦。

我们直接查看 `UsernamePasswordToken` 的源码，可以看到它实现了 `HostAuthenticationToken` 和 `RememberMeAuthenticationToken` ，而这两个类又分别实现了 `AuthenticationToken` ，因此在这里我们直接实现 `AuthenticationToken` 即可，同时重写 `getPrincipal()` 和 `getCredentials()` 两个方法。

  ```java
package com.kelan.framework.config;

import java.io.Serializable;

import org.apache.shiro.authc.AuthenticationToken;
import org.springframework.stereotype.Component;

@Component
public class MobileToken implements AuthenticationToken, Serializable {

	private static final long serialVersionUID = 1L;
	
	/**
	 * 手机号码
	 */
	private String phoneNum;
	
	/** 前端输入的验证码 */
	private String mobileVerificationCode;
	
	/** 无参构造方法 */
	public MobileToken(){}
	
	public MobileToken(String phoneNum, String mobileVerificationCode) {
		this.phoneNum = phoneNum;
		this.mobileVerificationCode = mobileVerificationCode;
	}


	@Override
	public Object getPrincipal() {
		return phoneNum;
	}
	
	@Override
    public Object getCredentials() {
         return phoneNum;
    }

	public String getPhoneNum() {
		return phoneNum;
	}

	public void setPhoneNum(String phoneNum) {
		this.phoneNum = phoneNum;
	}

	public String getMobileVerificationCode() {
		return mobileVerificationCode;
	}

	public void setMobileVerificationCode(String mobileVerificationCode) {
		this.mobileVerificationCode = mobileVerificationCode;
	}

	@Override
	public String toString() {
		return "MobileToken [phoneNum=" + phoneNum + ", mobileVerificationCode=" + mobileVerificationCode + "]";
	}
	
	
}
  ```

### 4. 创建自定义CustomModularRealmAuthenticator

  `Token` 和 `Realm` 创建完成之后，需要再创建一个 `ModularRealmAuthenticator` 来进行绑定操作，让程序知道碰到这个 `Token` 时进入对应的 `Realm`。

这步其实很简单，只需要重写 `doMultiRealmAuthentication(Collection<Realm> realms, AuthenticationToken token)` 方法，该方法有两个参数，其中 `realms` 是 `ShiroConfig` 中配置的所有 realm 集合，`token` 就是登录时传入的用户信息 token，在我们这边只会是 `UsernamePasswordToken` 或 `MobileToken`。

在 `doMultiRealmAuthentication()` 方法中遍历所有的 realm，通过每个 realm 的 `supports()` 方法来进行匹配。

```java
package com.kelan.common.exception.user;

import java.util.Collection;

import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.pam.ModularRealmAuthenticator;
import org.apache.shiro.authc.pam.UnsupportedTokenException;
import org.apache.shiro.realm.Realm;

public class CustomModularRealmAuthenticator  extends ModularRealmAuthenticator{
	/**
     * 自定义Realm的分配策略
     * 通过realm.supports()方法匹配对应的Realm，因此才要在Realm中重写supports()方法
     * @param realms
     * @param token
     * @return
     */
    @Override
    protected AuthenticationInfo doMultiRealmAuthentication(Collection<Realm> realms, AuthenticationToken token) {
        // 判断getRealms()是否返回为空
        assertRealmsConfigured();

        // 通过supports()方法，匹配对应的Realm
        Realm uniqueRealm = null;
        for (Realm realm : realms) {
            if (realm.supports(token)) {
                uniqueRealm = realm;
                break;
            }
        }
        if (uniqueRealm == null) {
            throw new UnsupportedTokenException();
        }
        return uniqueRealm.getAuthenticationInfo(token);
    }

}
```

### 5.修改ShiroConfig.java的配置信息，把多个Realm注入进去，同时把自定义异常也同时注入进去

```java
/**
     * 针对多Realm，使用自定义身份验证器
     * @return
     */
    @Bean
    public ModularRealmAuthenticator modularRealmAuthenticator(){
        CustomModularRealmAuthenticator authenticator = new CustomModularRealmAuthenticator();
        authenticator.setAuthenticationStrategy(new AtLeastOneSuccessfulStrategy());
        return authenticator;
    }

    /**
     * 自定义短信登录Realm
     */
    @Bean
    public UserMobileRealm mobileRealm(CacheManager cacheManager) {
    	UserMobileRealm mobileRealm = new UserMobileRealm();
    	mobileRealm.setCacheManager(cacheManager);
        return mobileRealm;
    }
    
    /**
     * 自定义Realm
     */
    @Bean
    public UserRealm userRealm(CacheManager cacheManager)
    {
        UserRealm userRealm = new UserRealm();
        userRealm.setCacheManager(cacheManager);
        return userRealm;
    }
```

两个Reaml分别是UserRealm 账号密码登录；UserMobileRealm手机短信登录；ModularRealmAuthenticator身份验证器也需要注入交给Shiro管理。

然后把`ModularRealmAuthenticator` 交给`SecurityManager`管理

```java
securityManager.setAuthenticator(modularRealmAuthenticator());
```

多个Reaml也交给`SecurityManager`管理

```java
List<Realm> realms = new ArrayList<Realm>();
realms.add(userRealm);
realms.add(mobileRealm);
securityManager.setRealms(realms);
```

完整的ShiroConfig.java

```java
package com.kelan.framework.config;

import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;
import javax.servlet.Filter;
import org.apache.commons.io.IOUtils;
import org.apache.shiro.authc.pam.AtLeastOneSuccessfulStrategy;
import org.apache.shiro.authc.pam.ModularRealmAuthenticator;
import org.apache.shiro.cache.CacheManager;
import org.apache.shiro.cache.ehcache.EhCacheManager;
import org.apache.shiro.codec.Base64;
import org.apache.shiro.config.ConfigurationException;
import org.apache.shiro.io.ResourceUtils;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.realm.Realm;
import org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.CookieRememberMeManager;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.apache.shiro.web.servlet.SimpleCookie;
import org.crazycake.shiro.RedisCacheManager;
import org.crazycake.shiro.RedisManager;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.kelan.common.exception.user.CustomModularRealmAuthenticator;
import com.kelan.common.utils.StringUtils;
import com.kelan.common.utils.spring.SpringUtils;
import com.kelan.framework.shiro.realm.UserMobileRealm;
import com.kelan.framework.shiro.realm.UserRealm;
import com.kelan.framework.shiro.session.OnlineSessionDAO;
import com.kelan.framework.shiro.session.OnlineSessionFactory;
import com.kelan.framework.shiro.web.filter.LogoutFilter;
import com.kelan.framework.shiro.web.filter.captcha.CaptchaValidateFilter;
import com.kelan.framework.shiro.web.filter.kickout.KickoutSessionFilter;
import com.kelan.framework.shiro.web.filter.online.OnlineSessionFilter;
import com.kelan.framework.shiro.web.filter.sync.SyncOnlineSessionFilter;
import com.kelan.framework.shiro.web.session.OnlineWebSessionManager;
import com.kelan.framework.shiro.web.session.SpringSessionValidationScheduler;
import at.pollux.thymeleaf.shiro.dialect.ShiroDialect;
import redis.clients.jedis.JedisPoolConfig;

/**
 * 权限配置加载
 * 
 * @author kelan
 */
@Configuration
public class ShiroConfig
{
    public static final String PREMISSION_STRING = "perms[\"{0}\"]";

    // Session超时时间，单位为毫秒（默认30分钟）
    @Value("${shiro.session.expireTime}")
    private int expireTime;

    // 相隔多久检查一次session的有效性，单位毫秒，默认就是10分钟
    @Value("${shiro.session.validationInterval}")
    private int validationInterval;

    // 同一个用户最大会话数
    @Value("${shiro.session.maxSession}")
    private int maxSession;

    // 踢出之前登录的/之后登录的用户，默认踢出之前登录的用户
    @Value("${shiro.session.kickoutAfter}")
    private boolean kickoutAfter;

    // 验证码开关
    @Value("${shiro.user.captchaEnabled}")
    private boolean captchaEnabled;

    // 验证码类型
    @Value("${shiro.user.captchaType}")
    private String captchaType;

    // 设置Cookie的域名
    @Value("${shiro.cookie.domain}")
    private String domain;

    // 设置cookie的有效访问路径
    @Value("${shiro.cookie.path}")
    private String path;

    // 设置HttpOnly属性
    @Value("${shiro.cookie.httpOnly}")
    private boolean httpOnly;

    // 设置Cookie的过期时间，秒为单位
    @Value("${shiro.cookie.maxAge}")
    private int maxAge;

    // 登录地址
    @Value("${shiro.user.loginUrl}")
    private String loginUrl;

    // 权限认证失败地址
    @Value("${shiro.user.unauthorizedUrl}")
    private String unauthorizedUrl;

 // 权限认证失败地址
    @Value("${spring.redis.host}")
    private String redisHost;
    
    /**
     * 缓存管理器 使用Ehcache实现
     */
    @Bean
    @ConditionalOnProperty(prefix = "spring.redis", name = "session-enable", havingValue = "false")
    public EhCacheManager getEhCacheManager()
    {
        net.sf.ehcache.CacheManager cacheManager = net.sf.ehcache.CacheManager.getCacheManager("kelan");
        EhCacheManager em = new EhCacheManager();
        if (StringUtils.isNull(cacheManager))
        {
            em.setCacheManager(new net.sf.ehcache.CacheManager(getCacheManagerConfigFileInputStream()));
            return em;
        }
        else
        {
            em.setCacheManager(cacheManager);
            return em;
        }
    }

    /** 缓存管理器 使用redis实现 By Nick*/
    @Bean
    @ConditionalOnProperty(prefix = "spring.redis", name = "session-enable", havingValue = "true")
    public RedisCacheManager getRedisCacheManager()
    {
        RedisCacheManager redisCacheManager = new RedisCacheManager();
        RedisManager redisManager = new RedisManager();
        redisManager.setHost(this.redisHost);
        redisManager.setTimeout(500);
        redisCacheManager.setPrincipalIdFieldName("userId");
         
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        //设置最大实例总数
        jedisPoolConfig.setMaxTotal(150);
        //控制一个pool最多有多少个状态为idle(空闲的)的jedis实例。
        jedisPoolConfig.setMaxIdle(30);
        jedisPoolConfig.setMinIdle(10);
        //表示当borrow(引入)一个jedis实例时，最大的等待时间，如果超过等待时间，则直接抛出JedisConnectionException；
        jedisPoolConfig.setMaxWaitMillis(4 * 1000);
        // 在borrow一个jedis实例时，是否提前进行alidate操作；如果为true，则得到的jedis实例均是可用的；
        jedisPoolConfig.setTestOnBorrow(true);
        // 在还会给pool时，是否提前进行validate操作
        jedisPoolConfig.setTestOnReturn(true);
        jedisPoolConfig.setTestWhileIdle(true);
        jedisPoolConfig.setMinEvictableIdleTimeMillis(500);
        jedisPoolConfig.setSoftMinEvictableIdleTimeMillis(1000);
        jedisPoolConfig.setTimeBetweenEvictionRunsMillis(1000);
        jedisPoolConfig.setNumTestsPerEvictionRun(100);
         
        redisManager.setJedisPoolConfig(jedisPoolConfig);
        redisCacheManager.setRedisManager(redisManager);
        return redisCacheManager;
    }
    /**
     * 返回配置文件流 避免ehcache配置文件一直被占用，无法完全销毁项目重新部署
     */
    protected InputStream getCacheManagerConfigFileInputStream()
    {
        String configFile = "classpath:ehcache/ehcache-shiro.xml";
        InputStream inputStream = null;
        try
        {
            inputStream = ResourceUtils.getInputStreamForPath(configFile);
            byte[] b = IOUtils.toByteArray(inputStream);
            InputStream in = new ByteArrayInputStream(b);
            return in;
        }
        catch (IOException e)
        {
            throw new ConfigurationException(
                    "Unable to obtain input stream for cacheManagerConfigFile [" + configFile + "]", e);
        }
        finally
        {
            IOUtils.closeQuietly(inputStream);
        }
    }

    /**
     * 针对多Realm，使用自定义身份验证器
     * @return
     */
    @Bean
    public ModularRealmAuthenticator modularRealmAuthenticator(){
        CustomModularRealmAuthenticator authenticator = new CustomModularRealmAuthenticator();
        authenticator.setAuthenticationStrategy(new AtLeastOneSuccessfulStrategy());
        return authenticator;
    }

    /**
     * 自定义短信登录Realm
     */
    @Bean
    public UserMobileRealm mobileRealm(CacheManager cacheManager) {
    	UserMobileRealm mobileRealm = new UserMobileRealm();
    	mobileRealm.setCacheManager(cacheManager);
        return mobileRealm;
    }
    
    /**
     * 自定义Realm
     */
    @Bean
    public UserRealm userRealm(CacheManager cacheManager)
    {
        UserRealm userRealm = new UserRealm();
        userRealm.setCacheManager(cacheManager);
        return userRealm;
    }

    /**
     * 自定义sessionDAO会话
     */
    @Bean
    public OnlineSessionDAO sessionDAO()
    {
        OnlineSessionDAO sessionDAO = new OnlineSessionDAO();
        return sessionDAO;
    }

    /**
     * 自定义sessionFactory会话
     */
    @Bean
    public OnlineSessionFactory sessionFactory()
    {
        OnlineSessionFactory sessionFactory = new OnlineSessionFactory();
        return sessionFactory;
    }

    /**
     * 会话管理器
     */
    @Bean
    public OnlineWebSessionManager sessionManager(CacheManager cacheManager)
    {
        OnlineWebSessionManager manager = new OnlineWebSessionManager();
        // 加入缓存管理器
        //manager.setCacheManager(getEhCacheManager());
        manager.setCacheManager(cacheManager);
        // 删除过期的session
        manager.setDeleteInvalidSessions(true);
        // 设置全局session超时时间
        manager.setGlobalSessionTimeout(expireTime * 60 * 1000);
        // 去掉 JSESSIONID
        manager.setSessionIdUrlRewritingEnabled(false);
        // 定义要使用的无效的Session定时调度器
        manager.setSessionValidationScheduler(SpringUtils.getBean(SpringSessionValidationScheduler.class));
        // 是否定时检查session
        manager.setSessionValidationSchedulerEnabled(true);
        // 自定义SessionDao
        manager.setSessionDAO(sessionDAO());
        // 自定义sessionFactory
        manager.setSessionFactory(sessionFactory());
        return manager;
    }

    /**
     * 安全管理器
     */
    @Bean
    public SecurityManager securityManager(UserRealm userRealm, UserMobileRealm mobileRealm,CacheManager cacheManager)
    {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        // 设置realm.
        //securityManager.setRealm(userRealm);
        securityManager.setAuthenticator(modularRealmAuthenticator());
        List<Realm> realms = new ArrayList<Realm>();
        realms.add(userRealm);
        realms.add(mobileRealm);
        securityManager.setRealms(realms);
        // 记住我
        securityManager.setRememberMeManager(rememberMeManager());
        // 注入缓存管理器;
        securityManager.setCacheManager(cacheManager);
        // session管理器
        securityManager.setSessionManager(sessionManager(cacheManager));
        return securityManager;
    }

    /**
     * 退出过滤器
     */
    public LogoutFilter logoutFilter(CacheManager cacheManager)
    {
        LogoutFilter logoutFilter = new LogoutFilter();
        //logoutFilter.setCacheManager(getEhCacheManager());
        logoutFilter.setCacheManager(cacheManager);
        logoutFilter.setLoginUrl(loginUrl);
        return logoutFilter;
    }

    /**
     * Shiro过滤器配置
     */
    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager,  CacheManager cacheManager)
    {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        // Shiro的核心安全接口,这个属性是必须的
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        // 身份认证失败，则跳转到登录页面的配置
        shiroFilterFactoryBean.setLoginUrl(loginUrl);
        // 权限认证失败，则跳转到指定页面
        shiroFilterFactoryBean.setUnauthorizedUrl(unauthorizedUrl);
        // Shiro连接约束配置，即过滤链的定义
        LinkedHashMap<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
        // 对静态资源设置匿名访问
        filterChainDefinitionMap.put("/favicon.ico**", "anon");
        filterChainDefinitionMap.put("/kelan.png**", "anon");
        filterChainDefinitionMap.put("/css/**", "anon");
        filterChainDefinitionMap.put("/docs/**", "anon");
        filterChainDefinitionMap.put("/fonts/**", "anon");
        filterChainDefinitionMap.put("/img/**", "anon");
        filterChainDefinitionMap.put("/ajax/**", "anon");
        filterChainDefinitionMap.put("/js/**", "anon");
        //filterChainDefinitionMap.put("/ruoyi/**", "anon");
        filterChainDefinitionMap.put("/kelan/**", "anon");
        //短信发送请求不需要拦截
        filterChainDefinitionMap.put("/sms/**", "anon");
        filterChainDefinitionMap.put("/mobile/login**", "anon");
        filterChainDefinitionMap.put("/captcha/captchaImage**", "anon");
        // 退出 logout地址，shiro去清除session
        filterChainDefinitionMap.put("/logout", "logout");
        // 不需要拦截的访问
        filterChainDefinitionMap.put("/login", "anon,captchaValidate");
        // 注册相关
        filterChainDefinitionMap.put("/register", "anon,captchaValidate");
        // 系统权限列表
        // filterChainDefinitionMap.putAll(SpringUtils.getBean(IMenuService.class).selectPermsAll());

        Map<String, Filter> filters = new LinkedHashMap<String, Filter>();
        filters.put("onlineSession", onlineSessionFilter());
        filters.put("syncOnlineSession", syncOnlineSessionFilter());
        filters.put("captchaValidate", captchaValidateFilter());
        filters.put("kickout", kickoutSessionFilter(cacheManager));
        // 注销成功，则跳转到指定页面
        filters.put("logout", logoutFilter(cacheManager));
        shiroFilterFactoryBean.setFilters(filters);

        // 所有请求需要认证
        filterChainDefinitionMap.put("/**", "user,kickout,onlineSession,syncOnlineSession");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);

        return shiroFilterFactoryBean;
    }

    /**
     * 自定义在线用户处理过滤器
     */
    @Bean
    public OnlineSessionFilter onlineSessionFilter()
    {
        OnlineSessionFilter onlineSessionFilter = new OnlineSessionFilter();
        onlineSessionFilter.setLoginUrl(loginUrl);
        return onlineSessionFilter;
    }

    /**
     * 自定义在线用户同步过滤器
     */
    @Bean
    public SyncOnlineSessionFilter syncOnlineSessionFilter()
    {
        SyncOnlineSessionFilter syncOnlineSessionFilter = new SyncOnlineSessionFilter();
        return syncOnlineSessionFilter;
    }

    /**
     * 自定义验证码过滤器
     */
    @Bean
    public CaptchaValidateFilter captchaValidateFilter()
    {
        CaptchaValidateFilter captchaValidateFilter = new CaptchaValidateFilter();
        captchaValidateFilter.setCaptchaEnabled(captchaEnabled);
        captchaValidateFilter.setCaptchaType(captchaType);
        return captchaValidateFilter;
    }

    /**
     * cookie 属性设置
     */
    public SimpleCookie rememberMeCookie()
    {
        SimpleCookie cookie = new SimpleCookie("rememberMe");
        cookie.setDomain(domain);
        cookie.setPath(path);
        cookie.setHttpOnly(httpOnly);
        cookie.setMaxAge(maxAge * 24 * 60 * 60);
        return cookie;
    }

    /**
     * 记住我
     */
    public CookieRememberMeManager rememberMeManager()
    {
        CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
        cookieRememberMeManager.setCookie(rememberMeCookie());
        cookieRememberMeManager.setCipherKey(Base64.decode("fCq+/xW488hMTCD+cmJ3aQ=="));
        return cookieRememberMeManager;
    }

    /**
     * 同一个用户多设备登录限制
     */
    public KickoutSessionFilter kickoutSessionFilter(CacheManager cacheManager)
    {
        KickoutSessionFilter kickoutSessionFilter = new KickoutSessionFilter();
        kickoutSessionFilter.setCacheManager(cacheManager);
        kickoutSessionFilter.setSessionManager(sessionManager(cacheManager));
        // 同一个用户最大的会话数，默认-1无限制；比如2的意思是同一个用户允许最多同时两个人登录
        kickoutSessionFilter.setMaxSession(maxSession);
        // 是否踢出后来登录的，默认是false；即后者登录的用户踢出前者登录的用户；踢出顺序
        kickoutSessionFilter.setKickoutAfter(kickoutAfter);
        // 被踢出后重定向到的地址；
        kickoutSessionFilter.setKickoutUrl("/login?kickout=1");
        return kickoutSessionFilter;
    }

    /**
     * thymeleaf模板引擎和shiro框架的整合
     */
    @Bean
    public ShiroDialect shiroDialect()
    {
        return new ShiroDialect();
    }

    /**
     * 开启Shiro注解通知器
     */
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(
            @Qualifier("securityManager") SecurityManager securityManager)
    {
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
        return authorizationAttributeSourceAdvisor;
    }
}
```

### 6. 编写手机短信登录功能

```
 @PostMapping("/mobile/login")
    @ResponseBody
    public AjaxResult ajaxMobileLogin(String mobile, String smsValidateCode)
    {
    	MobileToken token = new MobileToken(mobile, smsValidateCode);
        Subject subject = SecurityUtils.getSubject();
        try
        {
            subject.login(token);
            return success();
        }
        catch (AuthenticationException e)
        {
            String msg = "用户或密码错误";
            if (StringUtils.isNotEmpty(e.getMessage()))
            {
                msg = e.getMessage();
            }
            return error(msg);
        }
    }
```





博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)