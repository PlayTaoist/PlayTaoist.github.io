---
title:  "windows安装jekyll步骤及问题"
categories: windows
tags: windows jekyll
author: LIUREN
---



# windows安装jekyll步骤及问题

> 既然都看到这篇文章了，想必也对jekyll有所了解，我也是新手，将自己遇到的一些问题分享出来，希望能对大家有帮助。
>
> jekyll是一个简单的免费的Blog生成工具，类似WordPress。但是和WordPress又有很大的不同，原因是jekyll只是一个生成静态网页的工具，不需要数据库支持。但是可以配合第三方服务,例如Disqus。最关键的是jekyll可以免费部署在Github上，而且可以绑定自己的域名。



## Jektyll本地安装步骤

步骤： 
安装 Ruby 
安装 DevKit 
安装 Jekyll

# 1、安装 Ruby

下载地址：<https://rubyinstaller.org/downloads/>

![我的图片](https://www.codepeople.cn/imges/12.png)

注意版本要选 2.0 到 3.0 之间，本文使用的是：rubyinstaller-2.3.3-x64

如果是第一次安装，推荐默认路径，不要乱改路径，避免一些不必要的问题，比如我的路径是：

`C:\Ruby23-x64`

安装的时候注意勾选把ruby添加到路径PATH，如果不勾选也可以手动添加 
例如，按照我的路径，就应该添加：

`C:\Ruby22-x64\bin;`
检查ruby是否正常安装，会出现版本号

`ruby -v`


2、安装DevKit
回到刚刚的下载 ruby 的页面，往下滑。。。 
下载 DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe

![](https://www.codepeople.cn/imges/windows-jekyll/001.png)

解压文件，路径不要太长，推荐：

`C:\DevKit`
进入目录，初始化

`ruby dk.rb init`
打开 config.yml 添加

`- C:/Ruby22-x64`


依次执行以下命令

`ruby dk.rb review  # 审查（非必须）`

`ruby dk.rb install  # 安装`

`gem -v  # 查看gem是否正常安装`

![](https://www.codepeople.cn/imges/windows-jekyll/002.png)

均无报错，ok

3、安装jekyll
`gem install jekyll`
1
测试一下

`jekyll --version`

![](https://www.codepeople.cn/imges/windows-jekyll/003.png)

新建项目

`jekyll new myblog`

如果没有任何报错，会在当前目录下回生产一个 myblog 文件夹

![](https://www.codepeople.cn/imges/windows-jekyll/004.png)

4、运行服务器
进入 myblog 文件夹，运行服务器

`cd myblog`

`myblog>jekyll serve`

![](https://www.codepeople.cn/imges/windows-jekyll/005.png)

访问测试：http://127.0.0.1:4000/

![](https://www.codepeople.cn/imges/windows-jekyll/006.png)

一切都很完美，不过这才刚刚开始。。。

5、遇到的问题
问题 1：MSYS2 could not be found
运行：gem install jekyll 时报错

![](https://www.codepeople.cn/imges/windows-jekyll/007.png)

安装msys2之后也不行，经过反复测试，发现是ruby版本的问题，注意到这句话： 
For use with Ruby 2.0 to 2.3 (x64 - 64bits only)

![](https://www.codepeople.cn/imges/windows-jekyll/008.png)

删除已安装的ruby，重新下载对应的版本 
ruby: rubyinstaller-2.2.6-x64 
建议将dev-kit也删除重装

问题 2：ERROR: Failed to build gem native extension
运行：gem install jekyll 时报错

![](https://www.codepeople.cn/imges/windows-jekyll/009.png)

由于之前已经装过 dev ，重装ruby之后没有重新初始化，此处最好重新安装DevKit（查看第二步）

问题 3：cannot load such file – bundler (LoadError)
运行：jekyll serve 时报错 

![](https://www.codepeople.cn/imges/windows-jekyll/010.png)

问题 4：in any of the gem sources listed in your Gemfile. (Bundler::GemNotFound)
运行：jekyll serve 时报错

![](https://www.codepeople.cn/imges/windows-jekyll/011.png)

问题4 基本里边有一个共同点：bundle 
通过以下命令安装需要的组件

`myblog>bundle install`

问题5：缺少分页插件

`进入myblog文件夹，然后执行 gem install jekyll-paginate `