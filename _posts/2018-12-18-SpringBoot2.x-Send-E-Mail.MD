---
title:  "SpringBoot2.0配置发送E-mail"
categories: SpringBoot2.0
tags: SpringBoot2.0
author: LIUREN
---

## SpringBoot2.0配置发送E-mail

> 本文记录了如何使用SpringBoot2.0集成Email并发送简单邮件。



## SpringBoot2.0配置发送E-mail

### 引入依赖

在 pom.xml 文件中引入邮件配置：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

### 配置文件

```
# JavaMailSender 邮件发送的配置
spring.mail.host=smtp.163.com
spring.mail.username=用户163邮箱
spring.mail.password=邮箱密码
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
```

**注意：**若使用QQ邮箱发送邮件，则需要修改为`spring.mail.host=smtp.qq.com`，同时`spring.mail.password`改为QQ邮箱的授权码。 
QQ邮箱->设置->账户->POP3/SMTP服务:开启服务后会获得QQ的授权码 

![获取QQ授权 ](https://www.codepeople.cn/imges/email/001.jpg)

但是真正运行程序时，还是会爆 535 认证失败。 

![启动报错](https://www.codepeople.cn/imges/email/002.jpg)

解决方案：因为JDK1.8中jre\lib\security中两个 jar 包替换的缘故。将下载后的local_policy.jar和US_export_policy.jar替换到JDK1.8的jre\lib\security文件夹即可。 

地址：http://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html

### 发送简单文本邮件

贴 java 代码：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@ActiveProfiles("163")
public class EmailTest {

    @Autowired
    private JavaMailSender mailSender; //自动注入的Bean

    @Value("${spring.mail.username}")
    private String Sender; //读取配置文件中的参数

    @Test
    public void sendSimpleMail() throws Exception {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(Sender);
        message.setTo(Sender); //自己给自己发送邮件
        message.setSubject("主题：简单邮件");
        message.setText("测试邮件内容");
        mailSender.send(message);
    }
}
```

### 发送Html邮件

java 代码：

```java
 @Test
    public void sendHtmlMail() {
        MimeMessage message = null;
        try {
            message = mailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(Sender);
            helper.setTo(Sender);
            helper.setSubject("标题：发送Html内容");

            StringBuffer sb = new StringBuffer();
            sb.append("<h1>大标题-h1</h1>")
                    .append("<p style='color:#F00'>红色字</p>")
                    .append("<p style='text-align:right'>右对齐</p>");
            helper.setText(sb.toString(), true);
        } catch (Exception e) {
            e.printStackTrace();
        }
        mailSender.send(message);
    }
```

### 发送带附件的邮件

java 代码：

```java
@Test
    public void sendAttachmentsMail() {
        MimeMessage message = null;
        try {
            message = mailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(Sender);
            helper.setTo(Sender);
            helper.setSubject("主题：带附件的邮件");
            helper.setText("带附件的邮件内容");
            //注意项目路径问题，自动补用项目路径
            FileSystemResource file = new FileSystemResource(new File("src/main/resources/static/image/picture.jpg"));
            //加入邮件
            helper.addAttachment("图片.jpg", file);
        } catch (Exception e){
            e.printStackTrace();
        }
        mailSender.send(message);
    }
```

### 发送带静态资源的邮件

java 代码：

```java
 @Test
    public void sendInlineMail() {
        MimeMessage message = null;
        try {
            message = mailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(Sender);
            helper.setTo(Sender);
            helper.setSubject("主题：带静态资源的邮件");
            //第二个参数指定发送的是HTML格式,同时cid:是固定的写法
            helper.setText("<html><body>带静态资源的邮件内容 图片:<img src='cid:picture' /></body></html>", true);

            FileSystemResource file = new FileSystemResource(new File("src/main/resources/static/image/picture.jpg"));
            helper.addInline("picture",file);
        } catch (Exception e){
            e.printStackTrace();
        }
        mailSender.send(message);
    }
```

### 发送模板邮件

在Spring Boot中也能使用模板引擎来实现模板化的邮件发送。关于模板邮件，SpringBoot 原本是支持 velocity，在 1.4 版本后又抛弃了 velocity，暂时只支持 freemaker。 
引入 freemaker 依赖:

```xml
 <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-freemarker</artifactId>
 </dependency>
```

java 代码：

```java
@Autowired
    private FreeMarkerConfigurer freeMarkerConfigurer;  //自动注入

    @Test
    public void sendTemplateMail(){
        MimeMessage message = null;
        try {
            message = mailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(Sender);
            helper.setTo(Sender);
            helper.setSubject("主题：模板邮件");

            Map<String, Object> model = new HashedMap();
            model.put("username", "zggdczfr");

            //修改 application.properties 文件中的读取路径
//            FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
//            configurer.setTemplateLoaderPath("classpath:templates");
            //读取 html 模板
            Template template = freeMarkerConfigurer.getConfiguration().getTemplate("mail.html");
            String html = FreeMarkerTemplateUtils.processTemplateIntoString(template, model);
            helper.setText(html, true);
        } catch (Exception e) {
            e.printStackTrace();
        }
        mailSender.send(message);
    }
```

### 效果截图

![启动报错](https://www.codepeople.cn/imges/email/003.jpg)

### 项目参考地址

项目参考地址：<https://github.com/FunriLy/springboot-study/tree/master/%E6%A1%88%E4%BE%8B6>



参考文章：

文章一：<https://blog.csdn.net/u011244202/article/details/54809696>

文章二：<https://my.oschina.net/zyw205/blog/1918222>

文章三：<http://www.ityouknow.com/springboot/2017/05/06/springboot-mail.html>

考虑如何把定时任务和邮件发送关联起来，考虑如何把[RabbitMQ](http://www.ityouknow.com/springboot/2016/11/30/spring-boot-rabbitMQ.html)和Mybatis以及Spring Boot2.0 结合起来

