---
title:  "Jekyll 中使用GitHub服务"
categories: Jekyll GitHub Valine
tags: Jekyll GitHub Valine
author: LIUREN
---

# Jekyll 中使用GitHub服务

> 很多人都知道，Linus在1991年创建了开源的Linux，从此，Linux系统不断发展，已经成为最大的服务器系统软件了。
>
> Linus虽然创建了Linux，但Linux的壮大是靠全世界热心的志愿者参与的，这么多人在世界各地为Linux编写代码，那Linux的代码是如何管理的呢？
>
> 事实是，在2002年以前，世界各地的志愿者把源代码文件通过diff的方式发给Linus，然后由Linus本人通过手工方式合并代码！
>
> 你也许会想，为什么Linus不把Linux代码放到版本控制系统里呢？不是有CVS、SVN这些免费的版本控制系统吗？因为Linus坚定地反对CVS和SVN，这些集中式的版本控制系统不但速度慢，而且必须联网才能使用。有一些商用的版本控制系统，虽然比CVS、SVN好用，但那是付费的，和Linux的开源精神不符。
>
> 不过，到了2002年，Linux系统已经发展了十年了，代码库之大让Linus很难继续通过手工方式管理了，社区的弟兄们也对这种方式表达了强烈不满，于是Linus选择了一个商业的版本控制系统BitKeeper，BitKeeper的东家BitMover公司出于人道主义精神，授权Linux社区免费使用这个版本控制系统。
>
> 安定团结的大好局面在2005年就被打破了，原因是Linux社区牛人聚集，不免沾染了一些梁山好汉的江湖习气。开发Samba的Andrew试图破解BitKeeper的协议（这么干的其实也不只他一个），被BitMover公司发现了（监控工作做得不错！），于是BitMover公司怒了，要收回Linux社区的免费使用权。
>
> Linus可以向BitMover公司道个歉，保证以后严格管教弟兄们，嗯，这是不可能的。实际情况是这样的：
>
> Linus花了两周时间自己用C写了一个分布式版本控制系统，这就是Git！一个月之内，Linux系统的源码已经由Git管理了！牛是怎么定义的呢？大家可以体会一下。
>
> Git迅速成为最流行的分布式版本控制系统，尤其是2008年，GitHub网站上线了，它为开源项目免费提供Git存储，无数开源项目开始迁移至GitHub，包括jQuery，PHP，Ruby等等。
>
> 历史就是这么偶然，如果不是当年BitMover公司威胁Linux社区，可能现在我们就没有免费而超级好用的Git了。



## 在Windows 10 上安装Git

在Windows上使用Git，可以从Git官网直接[下载安装程序](https://git-scm.com/downloads)，（网速慢的同学请移步[国内镜像](https://pan.baidu.com/s/1kU5OCOB#list/path=%2Fpub%2Fgit)），然后按默认选项安装即可。

安装完成后，在开始菜单里找到“Git”->“Git Bash”，蹦出一个类似命令行窗口的东西，就说明Git安装成功！

![](http://www.codepeople.cn/imges/jekyll/008.jpg)

安装完成后，还需要最后一步设置，在命令行输入：

```shell
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

注意`git config`命令的`--global`参数，用了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置，当然也可以对某个仓库指定不同的用户名和Email地址。



> 我们在VSCode里编辑的代码会推送到GutHub，但是是如何实现的呢，下面来讲一下如何配置能把VSCode与GitHub连接。

#### 首先你必须确保已经配置好Git了，即把下面的命令敲入GitBash：

```shell
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

#### 首先我们需要取得一个SSH key：

> 打开GitHub，依次点击①Settings——②SSH and GPG keys——③generating SSH keys——④Generating a new SSH key and adding it to the ssh-agent——⑤复制选中的命令到GitBash，点击3次回车，得到图⑥泡泡的样子说明成功~接着我们再在GitBash里输入命令：cat ~/.ssh/id_rsa.pub，回车，得到一大段英文，如图⑦，复制下来这段英文！以上步骤可参考以下图示：

第①步：

![009](http://www.codepeople.cn/imges/jekyll/009.png)

第②步：

![010](http://www.codepeople.cn/imges/jekyll/010.png)

第③步：

![011](http://www.codepeople.cn/imges/jekyll/011.png)

第④步：

![012](http://www.codepeople.cn/imges/jekyll/012.png)

第⑤步：

![013](http://www.codepeople.cn/imges/jekyll/013.png)

第⑥步：

![014](http://www.codepeople.cn/imges/jekyll/014.png)

第⑦步：

![014](http://www.codepeople.cn/imges/jekyll/015.png)



> 复制下的英文要粘贴到哪里呢(⊙▽⊙)请看图⑧所示，看到类似图⑨里的文字，就说明已经成功地生成了SSH key啦~~

第⑧步：

![014](http://www.codepeople.cn/imges/jekyll/016.png)

第⑨步：

![014](http://www.codepeople.cn/imges/jekyll/017.png)

> 接下来，我们需要回到GitHub的首页来创建一个repository，点击⑩ New repository，起一个名字my blog（可随便取），其他什么都不要点，直接点Creat repository，如图⑪，之后会得到图⑫（确保图⑫中是SSH链接），以上步骤看以下图示~

第⑩步：

![014](http://www.codepeople.cn/imges/jekyll/018.png)

第⑪步：

![014](http://www.codepeople.cn/imges/jekyll/019.png)

第⑫步：

![014](http://www.codepeople.cn/imges/jekyll/020.png)

#### 注意事项：

#### 把代码提交到github上

##### 1.首先初始化项目

```shell
git init
```

##### 2.然后参看状态

```shell
git status
```

##### 3.把项目添加进入git管理

```shell
git add .
git status
```

##### 4.把项目提交到本地仓库

```shell
git commit -m "init my project"
git status
```

##### 5.采用免密码提交方式SSH

如果已经是origin方式(HTTPS)则选择SSH提交方式

如果是HTTPS报如下错误fatal: remote origin already exists.

移出origin方式：`git remote rm origin`

```shell
git remote add origin https://github.com/xxx/xxxx.git
git push -u origin master
```



## 以上都是如何安装和使用Git工具，下面介绍Jkeyll和Git的应用，Jekyll + Github Pages搭建静态博客

好吧，终于说到正题了，其实这里搭建的过程很简单，只要你找好一个博客模板然后fork到自己的github上。但是我之前花费了好长的时间才搞得定，因为我是想自己新建一个Jekyll项目，而不是fork别人的，但是由于Jekyll最新版本新建的项目把主题功能分开了，缺少了`_includes`和`_layouts`目录，不兼容现在的Github Pages的模式，所以搞了我好久，后来就直接去github上找博客模板了。

我用的是[PlayTaoist](https://link.jianshu.com?t=https%3A%2F%2Fhuangxuan.me%2F)大神的博客模板，[github仓库](https://github.com/PlayTaoist/PlayTaoist.github.io)在这里，进入仓库，点击Fork

![021](http://www.codepeople.cn/imges/jekyll/021.png)

然后回到你自己的仓库，进入刚刚fork的博客项目，点击setting，将仓库名字改为： **<username>.github.io**，`<username>`就是你的github用户名，然后点击 **Rename**

![022](http://www.codepeople.cn/imges/jekyll/022.png)

如果你有自己的域名则可以直接指定域名

![023](http://www.codepeople.cn/imges/jekyll/023.png)

然后等几分钟，在浏览器打开`https://www.codepeople.cn`，你就可以看到博客页面，但是此时页面的内容都不是你自己的，这是你首先需要修改一下`_config.yml`内容。具体的教程还是直接看黄玄大神的教程吧，有个[中文版教程](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2FHuxpro%2Fhuxpro.github.io%2Fblob%2Fmaster%2FREADME.zh.md)，我也就懒得班门弄斧了。

## 写博客

`_posts`目录就是专门存放博客文件的，你可以使用`markdown`、`Textile`（这个没听过）或者`html`格式的文件来写博客，我个人是用`markdown`格式写的。但是不管是哪种格式的文件都需要包含 [YAML](https://link.jianshu.com/?t=http%3A%2F%2Fyaml.org%2F) 头信息， Jekyll 才会把它当做一个特殊的文件来处理。

```yaml
title:  "Jekyll 中使用GitHub服务"
categories: Jekyll GitHub Valine
tags: Jekyll GitHub Valine
author: LIUREN
```

markdown编写的文本的命名规则如下：

![024](http://www.codepeople.cn/imges/jekyll/024.png)

编写好博客，然后就可以使用vscode直接提交到GitHub上了，然后刷新http://www.codepeople.cn就能看到新编写的博客了。