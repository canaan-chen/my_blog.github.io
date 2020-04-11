---
title: "浅谈hugo建站"
date: 2020-04-10T10:59:21+08:00
categories: ["博客建站tips"]
tags: ["hugo"]
draft: false
---

![img](https://cdn.sspai.com/2020/04/09/0d0641fc7dda81e960abc227c48eb8a1.png?imageMogr2/quality/95/thumbnail/!1420x708r/gravity/Center/crop/1420x708/interlace/1)

「大家好，我是鱼叔，前不久我刚用hexo搭建完我的博客，但是最近在隔离期无所事事，秉着“生命在于折腾”的精神，我又将博客从Hexo迁移到了Hugo上，这篇文章主要讲为什么我要选择Hugo，以及Hugo搭建博客的一些tips。」

HUGO 博客链接：http://unclefish.ink/my_blog.github.io/

<!--more-->

### 为什么选择Hugo

现在主流的博客大多都是Hexo 加 Github Page，甚至说再绝对一点，常常点开一篇文章就是Hexo Next 主题的模板。这个其实很容易理解，因为hexo比较容易上手，并且有很稳定、功能齐全的主题（说的就是你，Next），那么在这样一个强有力的竞争对手下，Hugo 又是如何生存下来，并且吸引到用户的呢？

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200409141402.png)

首先，第一点得提的就是Hugo 打出的标语 -- “The world’s fastest framework for building websites”。Hugo 依靠Go语言进行开发，号称世界上最快的构建网站工具，到底有多快？根据相关博主提供的数据，他200篇左右的博文用Hexo 需要10分钟去生成静态网页，而Hugo 只需要10秒。就我个人的体验来说，Hugo确实大大加快了网页生成的速度，平常增加一篇博文然后再生成渲染需要4秒左右，而Hugo基本上是1秒内完成。正是依赖于Hugo 快速生成的特点，调试方便成了Hugo的第二大特点。基本上我在源文件处修改的内容可以实时地显示在网页上，而不用再次敲代码生成再预览，这对于博主来说简直就是一个福音。

![img](https://cdn.sspai.com/2020/04/09/39e11e09980ac738c1deb091d1bb2436.jpg?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

### Hugo 目前存在的问题

Hugo 在传播度上不及Hexo，相应的搭建教程及bug修复上也没有Hexo来的齐全，因此会要求用户有一定的代码能力和debug能力。此外，从Hexo迁移到Hugo会存在一定的时间成本，因为两者的markdown文件中对于Front Matter 的格式定义不同，因此需要修改每篇博文的该部分（当然用脚本去修改是最好的）。最后，是主题问题，Hugo上面还没有像Next一样完善成熟的主题，不过这也让大家不会一窝蜂地选取同一个样式（Hugo的主题样式还是很多的，官网提供了将近300个主题，选择犹豫症慎入）。



如果你看完上面我的使用感受，对Hugo还感兴趣的话就可以接着往下看技术部分。

------



### 技术部分

#### 1. 安装 Git 和 Go 

使用Hugo前需要安装[Git](https://git-scm.com/) 和 [Go](https://golang.org/dl/) 语言开发环境，点击对应网址下载安装包即可。



#### 2. 安装Hugo

网上存在很多用工具安装的方式，我这里讲述一个我认为最简单的方式，不用涉及太多的代码（本方法针对pc）。

（1）在Hugo的[官网](https://github.com/gohugoio/hugo/releases)中选择想要的版本下载zip，将其中的hugo.exe文件解压到想要的地方，比如 `C:\Hugo\bin`。

（2）将Hugo添加到Windows的环境变量 `PATH`中。具体添加方法可参考[Windows 10 如何添加环境变量](https://jingyan.baidu.com/article/8ebacdf02d3c2949f65cd5d0.html)。

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/3.gif)

（3）添加完`PATH`后，打开Git Bash 输入 `hugo version ` 出现`hugo static site generator`相关信息表示安装完成。

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200409131219.png)



#### 3. 生成博客

（1）打开Git Bash 输入`hugo new site "你的文件名字"`，便可以生成一个用于存放博客的文件夹。

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200409131452.png)

（2）安装主题。

不同于hexo，hugo没有自带主题，所以建立完文件夹后要导入主题文件。导入主题方式和hexo相似，可以用`git clone` 的方式，也可以到相应主题的github中下载zip文件然后解压到自己博客的`themes`文件夹中。

推荐几个主题：[Pure](https://themes.gohugo.io/hugo-theme-pure/)、[Even](https://themes.gohugo.io/hugo-theme-even/)、[Coder](https://themes.gohugo.io/hugo-coder/)

官网主题库：[Hugo Themes](https://themes.gohugo.io/)

（3）配置文件

Hugo配置文件放置在源文件下，并且支持三种格式：toml，yaml，yml。这个配置文件可以直接从主题文件中的exampleSite 里copy到博客文件夹下，然后进行修改。

- 注意点1：有些主题没有提供相应的配置文件，得进行自己修改，不建议选用这类主题。
- 注意点2：配置文件中要确保里面的主题名字和你`themes`文件夹中相应的主题文件夹名字一样，比如我的主题是pure，那么配置文件里的`theme = pure`，并且themes 文件夹中也有一个pure的文件夹。这是为了保证工具能依据名字找到相应的主题文件。

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/4.gif)



（4）生成博文

在 Git Bash 中输入 `hugo new posts/xxxx.md `，这时候就会在文件夹 `content/posts`形成你要的markdown文件，打开进行编辑即可。

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/5.gif)



（5）渲染查看效果

在博客文件夹中打开Git Bash，输入 `hugo server `，然后打开 http://localhost:1313/ 来查看效果。注意，markdown文件中的 `front matter` 部分有一个`draft` 参数，如果`draft`设置为`true` 则可正常渲染，如果设置为`false`则不予以渲染。相应的如果想查看全部效果则输入`hugo server -D ` 表示将草稿文件也进行渲染。



#### 4. 代码托管

这里主要以GitHub 作为代码托管，假定你已经建立了一个xxx.github.io的一个仓库。官方提供了三四种[上传方式](https://gohugo.io/hosting-and-deployment/hosting-on-github/)，本文采用生成docs的方式进行部署，个人认为这种方式比较简单明了。

（1）修改配置文件。在配置文件config.toml中添加`publishDir = docs`，其他格式的配置文件类似。

（2）打开Git Bash，输入`hugo`，就会发现博客文件中出现了docs文件夹，这是因为hugo将网页的信息都存储在docs里，而不是public中。

（3）在博客文件夹中，打开Git Bash，依次输入以下代码（注意 git remote add 后跟随自己github的对应地址）：

```git
git remote add origin  https://github.com/xxxxx/xxxx.github.io.git
git add .
git commit -m "first commit"
git push -u origin master
```

（4）在GitHub对应的仓库设置中，将Github Pages source改成branch/docs 。

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200409140359.png)



（5）这时候点击网址会发现内容已经成功渲染了，但是跳转连接出现问题，这是因为我们没有在配置文件中的`baseURL`中更新我们未来发布的网址链接。因此我们将GitHub Pages 对应的网址进行复制然后添加到配置文件的第一个 `baseURL`中，重新进行第二步和第三步即可。

------



### 总结

到这里就说完了我所有的操作过程，Hugo 的快速生成特性很适合那些拥有大量博文的博主，从长远的角度来看，伴随着博文数量的增加，Hugo的这个优势也会越来越明显。





