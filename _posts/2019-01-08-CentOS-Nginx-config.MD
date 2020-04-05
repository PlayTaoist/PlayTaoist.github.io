---
title:  "Nginx性能优化功能-Gzip压缩(大幅度提高页面加载速度)"
categories: Nginx
tags: Nginx
author: LIUREN
---



# Nginx性能优化功能- Gzip压缩(大幅度提高页面加载速度)

> [Nginx](https://www.centos.bz/category/web-server/nginx/)开启Gzip压缩功能， 可以使网站的css、js 、xml、html 文件在传输时进行压缩，提高访问速度, 进而优化[Nginx](https://www.centos.bz/tag/nginx-2/)性能! Web网站上的图片，视频等其它多媒体文件以及大文件，因为压缩效果不好，所以对于图片没有必要支压缩，如果想要优化，可以图片的生命周期设置长一点，让客户端来缓存。 开启Gzip功能后，Nginx服务器会根据配置的策略对发送的内容, 如css、js、xml、html等静态资源进行压缩, 使得这些内容大小减少，在用户接收到返回内容之前对其进行处理，以压缩后的数据展现给客户。这样不仅可以节约大量的出口带宽，提高传输效率，还能提升用户快的感知体验, 一举两得; 尽管会消耗一定的cpu资源，但是为了给用户更好的体验还是值得的。
>
> 经过Gzip压缩后页面大小可以变为原来的30%甚至更小，这样，用户浏览页面的时候速度会快得多。Gzip 的压缩页面需要浏览器和服务器双方都支持，实际上就是服务器端压缩，传到浏览器后浏览器解压并解析。浏览器那里不需要我们担心，因为目前的巨大多数浏览器 都支持解析Gzip过的页面。



Gzip压缩作用：将响应报⽂发送⾄客户端之前可以启⽤压缩功能，这能够有效地节约带宽，并提⾼响应⾄客户端的速度。Gzip压缩可以配置http,[server](https://www.centos.bz/tag/server/)和[location](https://www.centos.bz/tag/location/)模块下。Nginx开启Gzip压缩功能的配置如下:

```shell
#修改nginx配置文件 /usr/local/nginx/conf/nginx.conf
[root@localhost ~]# vim /usr/local/nginx/conf/nginx.conf        #将以下配置放到nginx.conf的http{ ... }节点中

#修改配置为
gzip on;                    #开启gzip压缩功能
gzip_min_length 10k;         #设置允许压缩的页面最小字节数; 这里表示如果文件小于10个字节，就不用压缩，因为没有意义，本来就很小.
gzip_buffers 4 16k;         #设置压缩缓冲区大小，此处设置为4个16K内存作为压缩结果流缓存
gzip_http_version 1.1;      #压缩版本
gzip_comp_level 2;   #设置压缩比率，最小为1，处理速度快，传输速度慢；9为最大压缩比，处理速度慢，传输速度快; 这里表示压缩级别，可以是0到9中的任一个，级别越高，压缩就越小，节省了带宽资源，但同时也消耗CPU资源，所以一般折中为6
gzip types text/css text/xml application/javascript;      #制定压缩的类型,线上配置时尽可能配置多的压缩类型!
gzip_disable "MSIE [1-6]\.";       #配置禁用gzip条件，支持正则。此处表示ie6及以下不启用gzip（因为ie低版本不支持）
gzip vary on;    #选择支持vary header；改选项可以让前端的缓存服务器缓存经过gzip压缩的页面; 这个可以不写，表示在传送数据时，给客户端说明我使用了gzip压缩
```



## 线上使用的Gzip压缩配置

```shell
[root@external-lb02 ~]# cat /data/nginx/conf/nginx.conf
........
http {
.......
    gzip  on;
    gzip_min_length  1k;
    gzip_buffers     4 16k;
    gzip_http_version 1.1;
    gzip_comp_level 9;
    gzip_types       text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php application/javascript application/json;
    gzip_disable "MSIE [1-6]\.";
    gzip_vary on;

}
```

如果不开启Gzip压缩功能(即注释掉Gzip的相关配置), 查看某个图片大小

```shell
[root@external-lb02 ~]#  ll  -h /data/web//www/test.bmp
-rw-r--r-- 1 root root 453K 3月  14 18:43 /data/web//www/test.bmp
```

如下可知, 文件没有被压缩,文件传输大小还是400多K

![001](http://www.codepeople.cn/imges/centos/nginx/001.png)

如果开启Nginx的Gzip压缩功能(即打开Gzip的相关配置), 然后再次访问test.bmp图片, 发现压缩后的该图片文件传输大小只有200多K !

![001](http://www.codepeople.cn/imges/centos/nginx/002.png)



通过上面测试对比, 发现Nginx开启Gzip压缩功能后, 定义的gzip type的文件在传输时的大小明显变小, 这样这会大大提高nginx访问性能.



直接用curl测试命令:

```shell
[root@fvtlb02 ~]# curl -I -H "Accept-Encoding: gzip, deflate" "http://fvtvfc-web.kevin.com/service-worker.js"
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Mon, 26 Nov 2018 02:19:16 GMT
Content-Type: application/javascript; charset=utf-8
Connection: keep-alive
Vary: Accept-Encoding
Last-Modified: Sun, 25 Nov 2018 22:28:15 GMT
Vary: Accept-Encoding
ETag: W/"5bfb21ff-40be"
Content-Encoding: gzip

如上,response header头信息中出现"Conten_Encoding: gzip" , 就说明Nginx已开启了压缩 (在浏览器访问, 通过F12看请求的响应头部 也是一样)
```



## **Nginx的Gzip压缩功能虽然好用，但是下面两类文件资源不太建议启用此压缩功能。**

**1) 图片类型资源 (还有视频文件)**

原因：图片如jpg、png文件本身就会有压缩，所以就算开启gzip后，压缩前和压缩后大小没有多大区别，所以开启了反而会白白的浪费资源。（可以试试将一张jpg图片压缩为zip，观察大小并没有多大的变化。虽然zip和gzip算法不一样，但是可以看出压缩图片的价值并不大）

**2) 大文件资源**

原因：会消耗大量的cpu资源，且不一定有明显的效果。





原文地址：<https://www.cnblogs.com/kevingrace/p/10018914.html>