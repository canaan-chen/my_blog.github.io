---
title: "hexo Next 主题修改tips"
date: 2020-04-04T15:01:48+08:00
categories: ["博客建站tips"]
tags: ["hexo"]
draft: false
---

![img](https://cdn.sspai.com/2020/03/19/dcc16be00b1827f820c01bfe47c4d29d.jpg?imageMogr2/quality/95/thumbnail/!1420x708r/gravity/Center/crop/1420x708/interlace/1)

本文讲述了四个next优化方案：文章置顶、添加博客背景、调整文章内容透明度、增加近期文章list。

<!--more-->



## 1. 如何让文章置顶

### 博文置顶

在gitbash里输入以下代码加入置顶插件。

```py
$ npm uninstall hexo-generator-index --save
$ npm install hexo-generator-index-pin-top --save
```

然后在文章的`Front-matter`中加上`top:true`即可实现置顶。



### 置顶标志体现

实现置顶后如果没有置顶标志就会表现得很奇怪，这时候我们需要添加相关的置顶标志来实现。

打开/blog/theme/next/layout/_macro 中的`post.swig` 文件，定位到 `<div class="post-block">`下，添加如下代码：

```
{% if post.top %}
            <i class="fa fa-thumb-tack"></i>
            <font color=7D26CD>置顶</font>
            <span class="post-meta-divider">|</span>
 {% endif %}
```



## 2. 增加博客背景图片

1) 在`themes/next/source/css/_custom/custom.styl`中添加CSS样式

文件位置：hexo/themes/next/source/css/_custom/custom.styl// 背景图片

```css
body::before {
    background-image: url(https://背景图.jpg);
    background-repeat: no-repeat;
    background-size: cover;
    background-position: 50% 50%;
    content: " ";
    position: fixed;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    z-index: -2;
}
```

2）jquery-backstretch插件

文件位置：hexo/themes/next/layout/_layout.swig

```
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery-backstretch/2.0.4/jquery.backstretch.min.js"></script>
+  <script>
+  $("body").backstretch("https://背景图.jpg");
+  </script>
</body>
```

加入到文件最后面前面即可。



## 3. 博客内容透明度调整

NexT主题的博客文章均是不透明的，下面的方法可以使博客内容透明化：

在`blog/themes/next/source/css/_custom/custom.styl`中添加以下内容：

```
// page-opacity

.content-wrap {
  opacity: 0.85;
}

.sidebar {
  opacity: 0.85;
}

.header-inner {
  background: rgba(255,255,255,0.85);
}

.popup {
  opacity: 0.9;
}
```

其中`content-wrap`是文章内容的透明度设置，`sidebar`是侧边框的透明度设置，`header-inner`是菜单栏的透明度设置，`popup`是搜索框（`local-search`）的透明度设置。

可以修改上面的数字来自定义透明度。

注意其中`header-inner`不能使用`opacity`进行配置。
因为`header-inner`包含`header.swig`中的所有内容。
若使用`opacity`进行配置，子结点会出很多问题



## 4. 增加近期文章list

```
{% if theme.recent_posts %}
    <div class="links-of-blogroll motion-element {{ "links-of-blogroll-" + theme.recent_posts_layout  }}">
      <div class="links-of-blogroll-title">
        <!-- modify icon to fire by szw -->
        <i class="fa fa-history fa-{{ theme.recent_posts_icon | lower }}" aria-hidden="true"></i>
        {{ theme.recent_posts_title }}
      </div>
      <ul class="links-of-blogroll-list">
        {% set posts = site.posts.sort('-date') %}
        {% for post in posts.slice('0', '5') %}
          <li>
            <a href="{{ url_for(post.path) }}" title="{{ post.title }}" target="_blank">{{ post.title }}</a>
          </li>
        {% endfor %}
      </ul>
    </div>
{% endif %}
```

将此代码贴在`next/layout/_macro/sidebar.swig`中的`if theme.links`对应的`endif`后面，就ok了，是不是很简单。。。。
为了配置方便，在主题的`_config.yml`中添加了几个变量，如下：

```
recent_posts_title: 近期文章
recent_posts_layout: block
recent_posts: true
```



