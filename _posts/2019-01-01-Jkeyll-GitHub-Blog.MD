---
title:  "Jekyll 搭建静态博客"
categories: Jekyll GitHub Valine
tags: Jekyll GitHub Valine
author: LIUREN
---

# Jekyll 搭建静态博客

> Jekyll 是一个简单的博客形态的静态站点生产机器。它有一个模版目录，其中包含原始文本格式的文档，通过 Markdown （或者 Textile） 以及 Liquid 转化成一个完整的可发布的静态网站，你可以发布在任何你喜爱的服务器上。Jekyll 也可以运行在 GitHub Page 上，也就是说，你可以使用 GitHub 的服务来搭建你的项目页面、博客或者网站，而且是完全免费的。



## 搭建过程

在jekyll的官网上 <http://jekyllrb.com/> 其实已经说得比较明白了，我在这里还是简单的说一下吧。我用的是Windows系统。 

主要环节有：安装`Ruby`，安装`jekyll`，安装`jekyll-paginate`

### 安装Ruby

ruby官网下载安装：<https://www.ruby-lang.org/en/downloads/>

![001](http://www.codepeople.cn/imges/jekyll/001.png)

选择这个包好Devkit的，注意上边信息是 WITH DEVKIT

![002](http://www.codepeople.cn/imges/jekyll/002.png)

![003](http://www.codepeople.cn/imges/jekyll/003.png)

![001](http://www.codepeople.cn/imges/jekyll/004.png)

![001](http://www.codepeople.cn/imges/jekyll/005.png)

等待安装成功，安装成功后Ruby会自动配置环境变量：

打开命令行输入 `ruby -v` 获取对应的版本号信息，如果正常说明ruby安装成功

打开命令行输入`gem -v` 获取对应版本号信息，如果正常说明Devkit安装成功

### 安装Jekyll

执行下面的语句安装：

`gem install jekyll`

等待安装成功

![001](http://www.codepeople.cn/imges/jekyll/005.png)

### 安装jekyll-paginate

执行下面的语句安装：

`gem install jekyll-paginate`

等待安装成功

![001](http://www.codepeople.cn/imges/jekyll/006.png)

### 创建博客

进入D盘然后创建一个工作控件jekyll_blog

然后进入目录打开cmd命令行

然后输入创建的博客

```ruby
jekyll new blog  //blog为你的博客文件名
```

### 开启jekyll内置服务器

实现转入blog的目录，输入：

```ruby
cd blog//一定要进入创建的对应blog目录，否则服务无法开启
```

然后输入：

```ruby
jekyll serve  //开启服务器，可以按ctrl+c停止
```

Jekyll服务器默认端口是4000，所以打开浏览器输入：http://localhost:4000 就可以看到生成的博客页面。如下：

![001](http://www.codepeople.cn/imges/jekyll/007.png)

### 使用jekyll写博文

你可能喜欢markdown或html来写博文，都可以，但是博文文件的**命名规则**要服从下面的规则：

```
 year-month-title.markup //markup为你的文件格式的后缀名
```

在你的文章头部添加yaml头信息

```
---
layout: post
title:  "Jekyll+Github搭建个人博客"
date:   2017-01-14 15:03:25
categories: original
---
```

写上自己的博文内容，将这个文件保存在blog里面的_posts目录里面即可。在重启jekyll内置服务器，刷新页面：http://localhost:4000

### 用github 展示你的博客

接下来的操作都是用GIT命令完成的，不再是cmd了。首先，大家应该都拥有了github账号，没有的注册一个就好了。

- 创建个人仓库
  就是建立一个新的仓库，但是这个仓库的名字必须为你的github的名字+github+io，即yourname.github.io

- 将目录切换到你想要放github博客的文件目录下，在这个目录git bash 将刚才建的仓库克隆下来：

  ```shell
    git clone git@github.com:yourname/yourname.github.io.git
  ```

  这时，你会发现你的文件夹下会多出一个yourname的文件，我们把之前的blog下的所有文件复制到里面。

- 然后把里面的所有文件push到刚刚建的远程仓库，步骤我就不写了。
  这时，在浏览器里面输入网址：http://yourname.github.io 就可以看你的个人博客网站了，这就是你的博客网站的地址了。
  **前面所说的yourname指的是你的github账号名字。**

- 嗯，接下来你就可以查看你的博客网站了。