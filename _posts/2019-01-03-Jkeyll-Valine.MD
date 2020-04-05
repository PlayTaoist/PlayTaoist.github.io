---
title:  "Jekyll 中添加Valine评论功能"
categories: Jekyll Valine
tags: Jekyll Valine
author: LIUREN
---

# Jekyll 中添加Valine评论功能

> 关于github搭建jekyl博客，在这里不做过多描述，详情参考：
>
> ①[github搭建jekyll博客](https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=0&rsv_idx=1&tn=baidu&wd=github%E6%90%AD%E5%BB%BAjekyll%E5%8D%9A%E5%AE%A2&rsv_pq=cecac38b0002878a&rsv_t=362evSQHeS%2BnOn3iYGtNoUoaipbPHcEtIUUj%2BJ%2BgGxQVIN0EqkFglRaw0Eo&rqlang=cn&rsv_enter=1&rsv_sug3=2&rsv_sug1=1&rsv_sug7=001)
>
> ②官网：[https://www.jekyll.com.cn](https://www.jekyll.com.cn/)



关于评论插件：

1. Disqus: 需要翻墙
2. 网易云评论：需要自己的域名
3. Duoshuo: 挂了
4. gitment：逼格太高

其他评论插件没有用过，最后发现了Valine，一款快速、简洁且高效的无后端评论系统。
详情请访问官网：https://valine.js.org
这个评论系统是基于LeanCloud的，对Jekyll的博客是支持的，官网网址：https://leancloud.cn

#### 获取 APP ID 和 APP KEY

点击这里登录或注册Leancloud

点这里创建应用，应用名看个人喜好。

![](http://www.codepeople.cn/imges/jekyll/025.png)

#### 选择刚刚创建的应用>设置>选择应用 Key，然后你就能看到你的APP ID和APP KEY了



![](http://www.codepeople.cn/imges/jekyll/026.png)

#### 为了您的数据安全，请填写应用>设置>安全设置中的Web 安全域名，如下图：

![](http://www.codepeople.cn/imges/jekyll/027.png)

#### 创建数据存储

![](http://www.codepeople.cn/imges/jekyll/028.png)

#### 然后修改博客模板中的`_includes/comments.html`

这里面开头的`{% if site.valine_comment.enable %}` 和结尾的`{% endif %}`都展示不出来，记得自己添加。

完整的看代码后面跟的图

```vue
{% if site.valine_comment.enable %}
<div id="comments"></div>
<script src="//cdn1.lncld.net/static/js/3.0.4/av-min.js"></script>
<script src='//unpkg.com/valine/dist/Valine.min.js'></script>
<script>
    new Valine({
        el: '#comments',
        app_id: '{{ site.valine_comment.leancloud_appid }}',
        app_key: '{{ site.valine_comment.leancloud_appkey }}',
        placeholder:'{{ site.valine_comment.placeholder }}',
        visitor: true
    });
</script>
{% endif %}
```

![](http://www.codepeople.cn/imges/jekyll/031.png)

#### 修改`_config.yml`

```yaml
valine_comment:
    enable: true
    leancloud_appid: TyycwmVygccDruMiF6w0XwNb-gzGzoHsz
    leancloud_appkey: 63tegakam3VgktVHeLjcr5RG
    placeholder: "Just go go"
```

然后看到生成的评论功能界面如下：

![](http://www.codepeople.cn/imges/jekyll/030.png)

#### 注意：

![](http://www.codepeople.cn/imges/jekyll/029.png)