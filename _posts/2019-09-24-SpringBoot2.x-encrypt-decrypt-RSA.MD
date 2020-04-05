---
title:  "SpringBoot2.x中使用RSA加密解密"
categories: SpringBoot2.0 RSA
tags: SpringBoot2.0 RSA
author: LIUREN
---

# SpringBoot2.x中使用RSA加密解密

> 起一段时间部署的网站被安全公司告知有密码泄露的漏洞，这怎么行，要改啊。不然密码泄露了安全问题担不起责任啊。本想使用MD5加密呢，但是这个比较普遍。今天正好在网上浏览帖子的时候发现了Bobby写的加密和解码的帖子，吼吼吼！然后研究了一下，感觉挺不错的，可以参考使用。下面就是参考的实现的demo方法。



###  一、介绍

**rsa-encrypt-body-spring-boot**
实现了对Spring Boot接口返回值、参数值通过注解的方式自动加解密。

### 二、使用方法

***Apache Maven***

```xml
<dependency>
  <groupId>cn.shuibo</groupId>
  <artifactId>rsa-encrypt-body-spring-boot</artifactId>
  <version>1.0.1.RELEASE</version>
</dependency>
```

### 三、以Maven为例

- **以Maven为例，在pom.xml中引入依赖**

  ```xml
  <dependency>
      <groupId>cn.shuibo</groupId>
      <artifactId>rsa-encrypt-body-spring-boot</artifactId>
      <version>1.0.1.RELEASE</version>
  </dependency>
  ```

- **启动类Application中添加@EnableSecurity注解**

  ```java
  @SpringBootApplication
  @EnableSecurity
  public class DemoApplication {
      public static void main(String[] args) {
          SpringApplication.run(DemoApplication.class, args);
      }
  }
  ```

- **在application.yml或者application.properties中添加RSA公钥及私钥**

  ```yaml
  rsa:
    encrypt:
      open: true # 是否开启加密 true  or  false
      showLog: true # 是否打印加解密log true  or  false
      publicKey: MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDgTQG5aE2kvnS2wX9Qbx9YlNrMdeMhr2+7PS9kG+OmxuvPyOHxwB8HOl/k/jD5ds2m8et1pkfEXbn56D0sK/JANJYXoJBE79FHl6O4CjY5O/99cTTiupiIp7LFV1e7ITodPeXbCQUW/KtYiDV8YuPh19oQJU4d7Q5RKlYuE7KXawIDAQAB # RSA公钥
      privateKey: MIICdQIBADANBgkqhkiG9w0BAQEFAASCAl8wggJbAgEAAoGBAOBNAbloTaS+dLbBf1BvH1iU2sx14yGvb7s9L2Qb46bG68/I4fHAHwc6X+T+MPl2zabx63WmR8RdufnoPSwr8kA0lhegkETv0UeXo7gKNjk7/31xNOK6mIinssVXV7shOh095dsJBRb8q1iINXxi4+HX2hAlTh3tDlEqVi4TspdrAgMBAAECgYBuen+W6yETPBwQyE4FODagUvFSM96kAN1vxDtLe9EtnF0aA0AG6zJdRQtmpy1IMpQo3o+zayt+NQwdY55920d+HA938UIBbcrUJjqik3/EeE04M8hDfMq+/+HmadmgDC7dFuvNW+Ozo6ke6Qh8ZyEdXKm4/LqzmtEB2yXFCFchQQJBAPvPpuy9tFWWF/jEqAXDYdHmNj+PADfHAlCdG9JzRwFqvFxJzVuf6dFcIme2BIwrut5QlCNzYKFci4wav6wNN68CQQDkCDNosDhJ8I85Pgh7c9tY9IoezRl6RHTveNcK83mZD9QNICzUyTuTgvZIyko5lgv3Aarnv9YPEWjkZ7bfts8FAkAs4LU/PBAHsNhU3beUT13/qqfAbBv91Nn0xAtRVnoxJnSrffu7eKSTS3hWQ3pMAP8s3++G4ek22OeF8L54S2LBAkAksbUkGYdVTqKdgX/37bLUY7V+M17wL+fpSTbBimB84vEqk4zjSOaPXDmpiPPcxGp0ew43cMVPc2u/pXjmExLJAkAryDLD72MQhP18fcfXAzWLPIRdwKKuUV/YQVO1UDNWKGZvBlx7dd58jB8TgXSF22OTgQXCpuKL8h8V+ueVQIcb  #RSA私钥
  ```

  公钥和私钥可以在线生成，生成地址是：<http://web.chacuo.net/netrsakeypair>

  ![](https://www.codepeople.cn/imges/rsa.png)

- **对返回值进行加密**

  ```java
  @Encrypt
  @GetMapping("/encryption")
  public TestBean encryption(){
      TestBean testBean = new TestBean();
      testBean.setName("shuibo.cn");
      testBean.setAge(18);
      return testBean;
  }
  ```

- **对传过来的加密参数解密**

  ```java
  @Decrypt
  @PostMapping("/decryption")
  public String Decryption(@RequestBody TestBean testBean){
      return testBean.toString();
  }
  ```

  对返回值进行加密比较容易理解，本地测试用例如下：

  java代码：

  ```java
      @Encrypt
      @GetMapping("/getUser")
      @ResponseBody
      public User getUser() {
      	User user = new User();
      	user.setId(1);
      	user.setName("shuibo.cn");
      	user.setPassword("123456");
      	user.setPerms("update");
      	return user;
      }
  ```

  请求的URL：`http://127.0.0.1:8002/sys/getUser`

  返回的数据：

  ```shell
  "JoGVJPE5zrk+NeuN9dGWW2ga1KLUQM9ArvFet7c426SBV//vbX6h+VGkzYYyfw/y/MmdmedslWqqOxlt/j96OQyUbD/Vxv7XDwAXZ7ypOfXUcoXWPtKpK54jJTGMqtzDYdgxzYu1VZ4bU2WLeyjiBTFifPtwlLBWtwGfDDx8Tro="
  ```

  ![](https://www.codepeople.cn/imges/rsa001.png)

  对请求参数进行加密用例：

  HTML页面

  ```html
  <!DOCTYPE html>
  <html>
  <head>
  <meta charset="UTF-8">
  <title>登录页面</title>
  <script src="https://code.jquery.com/jquery-1.11.1.min.js"></script>
  <script src="https://cdn.bootcss.com/jsencrypt/3.0.0-beta.1/jsencrypt.js"></script>
  </head>
  <body>
  
  <h3>小白兔系统</h3>
  
  <h3 style="color: red;">${msg!""}</h3>
  <div>
  <button onclick="btn();"> 提交 </button>
  </div>
  </body>
  </html>
  <script>
  var PUBLIC_KEY = 'MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDgTQG5aE2kvnS2wX9Qbx9YlNrMdeMhr2+7PS9kG+OmxuvPyOHxwB8HOl/k/jD5ds2m8et1pkfEXbn56D0sK/JANJYXoJBE79FHl6O4CjY5O/99cTTiupiIp7LFV1e7ITodPeXbCQUW/KtYiDV8YuPh19oQJU4d7Q5RKlYuE7KXawIDAQAB';
  //使用公钥加密
  var encrypt = new JSEncrypt();
    //encrypt.setPrivateKey('-----BEGIN RSA PRIVATE KEY-----'+PRIVATE_KEY+'-----END RSA PRIVATE KEY-----');
  encrypt.setPublicKey(PUBLIC_KEY);
  function btn(){
  	var str = JSON.stringify({name:"张三" , password : "123456"});
  	var encrypted = encrypt.encrypt(str);
  	console.log('加密后数据:%o', encrypted);
  	$.ajax({
  	    url : "login",
  	    type : "post",
  	    contentType: 'application/json',
  	    data: encrypted,
  	});
  }
  </script>
  ```

  java方法：

  ```java
      @Decrypt
      @PostMapping("/login")
      @ResponseBody
      public User getLoginInfo(@RequestBody User user) {
      	System.out.println("======>");
      	System.out.println(user.toString());
      	return user;
      }
  ```

  请求的URL：`http://127.0.0.1:8002/sys/toLogin`

  返回的数据：`{"id":null,"name":"张三","password":"123456","perms":null}`

  ![](https://www.codepeople.cn/imges/rsa002.png)


### 测试的用例和demo源代码如下：

<https://github.com/PlayTaoist/springboot-shiro-beetl>

=====================================================================

博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)