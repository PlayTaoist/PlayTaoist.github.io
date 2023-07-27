---
title:  "SpringBoot 自定义启动画面"
categories: SpringBoot banner
tags: SpringBoot banner
author: LIUREN
---

# SpringBoot 自定义启动画面

> 
>



## 第一步　我们启动项目的之后，会在控制台上看到类似下面的画面：

![](https://www.codepeople.cn/imges/00x153.png)

## 第二步  自定 Banner

上面👆的截图信息就是 `Banner` 信息，我们可以在项目的 `resources` 下新建 `banner.txt` 来实现，比如：

```
// banner.txt
Jimmy
```

然后我们重新启动项目，则有：

![](https://www.codepeople.cn/imges/00x154.png)

## 第三步  获取属性

当然，我们可以获取项目的属性，通过 `${属性名}` 来获取，比如：

```bash
${spring-boot.version}
```

获取的属性可以有：

- **AnsiColor.BRIGHT_RED**: 设置控制台中输出内容的颜色，具体参考 `org.springframework.boot.ansi.AnsiColor`
- **application.version**: 用来获取 `MANIFEST.MF` 文件的版本信息
- **application.formatted-version**: 格式化后的 `application.version` 版本信息
- **spring-boot.version**: `Spring Boot` 的版本号
- **spring-boot.formatted-version**: 格式化后的 `sprin-boot.version` 版本信息

## 第四步  设置颜色

刚才我们提到了 `AnsiColor` 可以设置控制台输出内容的颜色。那么，我们应该如何设置呢？

这很简单，只需要在这段文本前面加入 `${AnsiColor.BRIGHT_RED}` 即可。添加的**枚举类型**颜色根据个人喜好更改～

```java
package org.springframework.boot.ansi;

public enum AnsiColor implements AnsiElement {
    DEFAULT("39"),
    BLACK("30"),
    RED("31"),
    GREEN("32"),
    YELLOW("33"),
    BLUE("34"),
    MAGENTA("35"),
    CYAN("36"),
    WHITE("37"),
    BRIGHT_BLACK("90"),
    BRIGHT_RED("91"),
    BRIGHT_GREEN("92"),
    BRIGHT_YELLOW("93"),
    BRIGHT_BLUE("94"),
    BRIGHT_MAGENTA("95"),
    BRIGHT_CYAN("96"),
    BRIGHT_WHITE("97");

    private final String code;

    private AnsiColor(String code) {
        this.code = code;
    }

    public String toString() {
        return this.code;
    }
}
```

## 第四步  实操

`OK`，我们了解得差不多了，来实操一下。通过 [Text to ASCII Art Generator](https://link.juejin.cn/?target=http%3A%2F%2Fpatorjk.com%2Fsoftware%2Ftaag%2F%23p%3Ddisplay%26f%3DBig%26t%3DJimmy) 获取生成的艺术字。

然后在 `banner.txt` 上添加文本如下：

```
启动成功！
${AnsiColor.BRIGHT_RED}
       _ _
      | (_)
      | |_ _ __ ___  _ __ ___  _   _
  _   | | | '_ ` _ | '_ ` _ | | | |
 | |__| | | | | | | | | | | | | |_| |
  ____/|_|_| |_| |_|_| |_| |_|__, |
                                __/ |
                               |___/

${AnsiColor.BRIGHT_WHITE}
欢迎使用～
spring boot 版本为 ${spring-boot.version}
```

我们重新运行下项目，效果如下：

![](https://www.codepeople.cn/imges/00x155.png)

## 关闭 Banner

我们不想使用 `Banner`，那么，我们在入口文件中进行关闭即可。

原先入口文件：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class LaunchApplication {

    public static void main(String[] args) {
        SpringApplication.run(LaunchApplication.class, args);
    }

}
```

上面代码其实可以写成这样：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class LaunchApplication {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(LaunchApplication.class);
        app.run(args);
    }

}
```

我们添加一行代码禁用 `banner.txt`，更改后：

```java
import org.springframework.boot.Banner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class LaunchApplication {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(LaunchApplication.class);
        app.setBannerMode(Banner.Mode.OFF); // 设置 Banner 模式为关闭
        app.run(args);
    }

}
```

启动后，控制台就没有相关的 `banner` 信息了：

![](https://www.codepeople.cn/imges/00x156.png)



> 作者：Jimmy
> 链接：https://juejin.cn/post/7259965990960201787
> 来源：稀土掘金

