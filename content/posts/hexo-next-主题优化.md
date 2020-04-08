---
title: "hexo next 主题优化 | 这里可能有你遇到所有问题的答案"
date: 2020-03-19T14:19:42+08:00
categories: ["博客建站tips"]
tags: ["hexo"]
draft: false
---

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/way-4005288_1280.jpg)

hello 大家好，我是鱼叔。我最近花了两天左右的时间基于hexo和next搭建了个人的blog（传输门），中途遇到了一堆的bug以及相关的优化问题，想写这篇文章记录下来给想搭建blog的你一个参考。

<!--more-->

## 搭建blog的初衷

现在网络平台里有很多可以发表个人看法的平台，比如我派，比如知乎、简书等，那花费功夫搭建自己的个人blog到底有什么意义呢？在我看来，个人博客的意义在于自由性，因为这是你自己的平台，你可以自由地去发表个人想法和文章，而不用担心内容是否迎合具体某个平台的读者。其次，我将个人博客作为自己的一个分享型知识记录，我希望自己的经历或者知识可以给别人带来一点启发和帮助，也许某一个瞬间，当一个人在搜寻相关的知识，参考了你博客的内容并且留言感时，你会发现自己其实在给这个社会做贡献，你在无声中被别人所认可。



## 相关的优化

### 前言

- 关于hexo

我所搭建的blog是基于hexo + GitHub的，因为hexo凭借它轻便快捷的特点，已经成为现在个人博客搭建的主流。我派已经有较为具体的搭建教程文章了，就不再详述搭建步骤。

<https://sspai.com/post/59337>

- 关于NexT

NexT 是hexo主题中非常流行的一个，因为它功能齐全，而且插件丰富，成为大多数人的首选主题。当然流行就会出现一个问题，很容易撞车，大家的博客外表都非常相似。不过鱼叔认为博客只是载体，内容才是王道，所以在精力有限的情况下，没必要太纠结于博客外表的华丽，找一个顺眼的主题驻扎即可。本文接下去的内容也主要适用于NexT主题。

![next 主题样式](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200318145642.png)



### 

### 

### 菜单栏修改问题

Next在主题的_config.yml文件中提供菜单栏的修改，只要搜索menu就可以找到，通过去除注释即可以在网页上进行渲染。然而鱼叔在修改next中的menu时会发现存在这样的报错 “cannot get %20” -- 无法找到相应的菜单。出现的原因是官方给的代码中多加了一个空格，导致网页无法渲染，个人除了删除注释外，要将“||”前的空格删除不然会导致菜单没法跳转。

```yml
menu:
  home: /|| home
  about: /about/|| user
  tags: /tags/|| tags
  categories: /categories/|| th
  archives: /archives/|| archive
  #schedule: /schedule/ || calendar #官方给的代码 || 前多加了空格
```

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200318150930.png)



### Hexo博客出现“Cannot GET/tags or categories”错误

next提供了标签和分类功能，但是在实现这个功能时常常会遇到"cannot get" 的问题，这是因为我们没有对它们进行初始化。具体解决方法如下：

1. 在git bash 中输入以下代码创建相应的page：

   ```python
   hexo new page "tags"
   hexo new page "categories"
   ```

2. 在第一步完成后会在source文件夹中出现tags和categories的文件夹，在各自的文件夹里打开index.md文件进行修改(多加上一个type属性)：

   ```markdown
   title: categories
   date: 2020-03-15 14:19:53
   type: "categories"
   
   title: tags
   date: 2020-03-15 14:20:32
   type: "tags"
   ```



### 底部下一页跳转问题

鱼叔在建立博客的时候发现底部下一页跳转按钮没有正常显示，如下图

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/fsfsfewf.png)

修改方式：

打开主题 layout/_partials/pagination.swg 进行修改，将“i class”模块删除修改成如下模式(其中prev 和 next可以自定字样)：

```python
{% if page.prev or page.next %}
  <nav class="pagination">
    {{
      paginator({
        prev_text: 'prev',
        next_text: 'next',
        mid_size: 1
      })
    }}
  </nav>
{% endif %}
```

修改后结果为下图，实现正常的跳转按钮：

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/ljoifowenfewnf.png)



### 如何让首页文章部分显示

next主题默认的是将你的文章全篇显示在自己的首页上，这就会导致一个问题，首页各个文章太长了不利于翻阅，那如何部分显示自己的文章呢？很简单，只要在个人的文章Markdown 中在想要显示的文章部分下面加上代码`<!--more>`，即可只在首页显示所需的文章部分。



### 修改头像问题

将自己的头像复制到blog文件夹中的\themes\next\source\images中覆盖原来的avatar.png 文件即可。



### 修改站点icon

当套用完next主题后，个人博客的站点图标会自动为next的logo，作为博主当然不能忍这个logo啦，那怎么修改呢？

我们打开next的主题配置文件会发现有这样的设定：

```yml
favicon:
  small: /images/favicon-16x16-next.png
  medium: /images/favicon-32x32-next.png
  apple_touch_icon: /images/apple-touch-icon-next.png
  safari_pinned_tab: /images/logo.svg
```

这就可以很清晰地发现去哪里修改了，只要在对应的路径上把原有的图片文件给替换成我们需要的文件就行了。这里给大家提供一个网站用于生成所需的icon同尺寸图像：<https://www.favicon-generator.org/>



### 增加阅读时长和字数统计功能

next自带阅读时长和字数统计插件，我们所需要做的就是调用这两个工具。首先在主题的配置文件中修改：

```yml
post_wordcount:
  item_text: true
  wordcount: true         # 单篇 字数统计
  min2read: true          # 单篇 阅读时长
  totalcount: false       # 网站 字数统计
  separated_meta: true
```

完成配置后，我们需要安装word-count 插件，在git bash中输入：

```
npm i --save hexo-wordcount
```

完成插件安装后，为了更好的显示，我们可以打开xxx_blog/themes/next/layout/_macro/post.swig ，在对应地方添加字words 和 min。可以通过搜索‘wordcount’ 和 ‘min2read’ 来定位。

```
 <span title="{{ __('post.wordcount') }}">
                  {{ wordcount(post.content) }} words
                </span>
         
           <span title="{{ __('post.min2read') }}">
                  {{ min2read(post.content) }} min
                </span>
```



### 配置搜索功能

next自带一个搜索功能，可以实现对站内内容的搜索。

首先需要通过如下命令安装对应的搜索插件：

然后在全局的配置文件（hexoblog目录下的_config.yml）中，增加配置如下内容：

```yml
# Search Config
search:
  path: search.xml
  field: post
  format: html
  limit: 100
```

然后在git hash 中加载相应的插件：

```
npm install hexo-generator-search --save
npm install hexo-generator-searchdb --save
```

打开主题内的配置文件，找到 local_search 属性，配置开启本地搜索功能。

```yml
local_search:
  enable: true
  # if auto, trigger search by changing input
  # if manual, trigger search by pressing enter key or search button
  trigger: auto
  # show top n results per article, show all results by setting to -1
  top_n_per_article: 1
```



### 将自己的博客收录到谷歌搜索中

为了让自己的博客能被更多人搜索到，自然需要将自己的博客收录到谷歌引擎中，具体的操作如下：

1. 添加资源

打开[Google search Console](https://search.google.com/search-console?hl=zh), 在左上角添加资源：

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200318161417.png)

2. 网址验证

选择网址前缀输入自己的博客网址。接下去就是对网站的验证，即证明网站是自己的。google 提供了多种方式进行验证，比较方便的是HTML 文件方式，它会产生一个HTML 文件，你只要将它放到博客source里然后再进行重新部署就行,（tips: 为了避免验证文件被hexo渲染，可以在验证html里添加‘layout：false’ 代码）。鱼叔采用的是HTML标记的方式，具体的方法就是将google提供的代码copy到 xxx_blog/themes/next/layout/_partials/head.swig中，放在头部的<meta>下面即可。

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200318161752.png)

3. 增加sitemap

   通过在线网址 [sitemap](https://www.xml-sitemaps.com/)生成自己博客的站点地图，然后将站点地图下载下来放到自己博客源public文件夹中（xxx_blog/public）。然后打开 Google console，点开左侧的添加站点地图，输入sitemap.xml，点击提交即可，一般会花2-3天时间进行收录。

   ![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200319115742.png)



### 增加评论功能

目前比较流行的两种评论系统是Valine 和 Giement，这里主要讲Valine的配置。

1. 注册leancloud， [leancloud注册网址](https://leancloud.cn/dashboard/login.html#/signin)

2. 注册登陆后，访问控制台，创建应用，选择开发版，创建好之后就生成了对应的id和key

   ![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200319121641.png)

4. 在主题配置文件中进行修改

   ```
    Valine.
   # You can get your appid and appkey from https://leancloud.cn
   # more info please open https://valine.js.org
   valine:
     enable: true 
     appid:   # 填入 App ID
     appkey:  # 填入 App Key
     notify: false 
     verify: false #
     placeholder: 请在此输入您的留言 # 默认留言框内的文字
     avatar: "" # 修改留言者头像，具体代码可以参考valine官网
     guest_info: nick,mail # 默认留言框的头部需要访问者输入的信息
     pageSize: 10 #默认单页的留言条数
   ```

5. 最后效果

   ![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200319122124.png)



### 博客中图床问题

这里推荐图床神器PicGo, 可以非常快捷地将图片上传到自己所需的云端中，而我采用的是PicGo结合腾讯云COS来搭建在线个人图床。

相关的教程链接：<https://sspai.com/post/59284>



## 总结

基本上讲完了我搭建博客时遇到的所有问题，我之前看过一个博主写的一句话，“搭建博客就像玩了一个游戏，你的博客就是你的角色，你总是会想方设法地给他升级“。博客优化其实是一个无止境的路，需要耐心和时间，但往往我们会过度的为了美化它而忘记了自己搭建的初衷。好的内容才是一个博客真正的灵魂。



## 参考

[Next 官方指南](https://theme-next.iissnan.com/theme-settings.html)





