---
title: "hexo 博客拓展 | CODING平台托管以及个性化域名"
date: 2020-03-30T17:46:38+08:00
categories: ["博客建站tips"]
tags: ["hexo"]
draft: false
---

![img](https://cdn.sspai.com/2020/03/30/3a7e7aec64856772c43f0efe5ea2d1b3.jpg?imageMogr2/quality/95/thumbnail/!1420x708r/gravity/Center/crop/1420x708/interlace/1)

「hello 大家好，我是鱼叔。在上一篇讲完关于next的优化tips之后，这一篇我们来聊一下CODING平台的代码托管以及如何个性化你自己的域名」

<!--more-->

## 初衷

一般大家的hexo博客都是基于GIthub 进行托管，但是这个托管存在一个问题就是国内网络访问会比较慢，同时GitHub对百度搜索的爬虫存在限制，也就是说你的博客如果基于Github可能就没法被百度搜索给收录，这时候咱们就需要另一个代码托管平台来应对国内的网络访问。笔者所采用的平台是CODING,有点类似Github的一个国内代码托管平台。

关于域名，纯粹是个人的喜好问题了。为了让自己的博客更加个性化，一个独特的域名自然是很有必要的，再加上国内购买域名的方式很方便（笔者便通过阿里云购买了域名），所以有兴趣的博主也可以考虑。



## CODING 平台代码托管

### CODING 的注册

CODING 作为一个国内代码托管平台，同时支持git操作。首先在[coding.net](https://coding.net/company/about) 进行注册。

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200326192154.png)

### 将ssh 公钥配置到CODING上

一般之前用GitHub部署博客时已经生成了ssh密钥，如果没有生成则从第一步开始，如果已经生成了则从第二步开始。

1. 生成ssh密钥

   打开gitbash，输入 `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"` ，然后一直回车（建议用同样的邮箱注册CODING 和 GitHub）。

2. 复制公钥

   打开C盘下user 文件夹中的.ssh文件夹，再打开其中的id_rsa.pub，将其中的公钥进行复制。

   ![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200326193749.png)

   打开CODING个人设置，点击左侧的SSH公钥，选择添加新的公钥，将之前复制的所有内容进行粘贴即可。

   ![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200326194043.png)



3. 公钥配置测试

   为了测试公钥添加是否完成，我们需要在本地的GitBash里进行测试。打开GitBash，输入

   ```
   ssh -T git@e.coding.net
   ```

   然后会询问是否继续，一直输入yes即可，最后会显示success的标志。

   ![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200326204311.png)

   

### 创建CODING 仓库

在自己的项目管理平台上新建一个代码托管项目。

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/1.gif)



在创建完项目后，会在右侧看到相关的HTTP协议的链接，将它进行复制用于后面的代码上传。

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200326201646.png)



### 部署博客

1. 在我们的blog文件夹中打开_config.yml文件，在deploy下面添加CODING平台的链接(链接为上一步复制的链接)，注意后方逗号后的master是分支的命名格式。

   ```
   deploy:
     type: git
     repo:
       github: https://github.com/xxx/xxx.github.io.git,master
       coding: https://e.coding.net/xxx/xxx.git,master
   
   ```

2. 在source文件中新建一个Staticfile（用于生成静态网页），注意大小写以及没有后缀的特点。可以通过gitbash输入以下代码来生成，也可通过新建一个txt文件然后修改名字来实现。

   ```
   cd source
   touch Staticfile
   ```

3. 在Git Bash中输入`hexo clean && hexo g && hexo d` 来进行部署。



### 生成静态网页

刚生成的项目仓库中是不存在生成静态网页的选项的，因此我们要将这个按钮打开。打开方式也很简单，点击项目栏下方的“项目设置”，选择“通知设置”，将其中的“构建与部属”打开，然后就可以发现项目栏中会出现“构建静态网页"选项，通过这个选项便可以生成我们所需的博客了，并且CODING平台会提供相应的网站链接（注意，静态网页生成需要实名注册）。

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/2.gif)





## 域名个性化

域名就是一个网站的名字，通过网域名称系统（DNS）来实现对具体IP地址的访问。为了保证域名的唯一性，所有的域名都需要通过域名注册商实现注册，笔者便是通过阿里云实现注册，接下去的教程也是基于阿里云。



### 购买域名

1. 在[阿里云万网](https://wanwang.aliyun.com/)上查询想要的域名。

2. 将域名加入清单并结算。

   ![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200330121817.png)

3. 创建信息模板。（注意信息模板中要求实名认证，并且推荐使用身份证，会比较快捷，笔者曾用护照去实名花费了大量的审核时间）

4. 付钱。



### 解析

当购买完域名后，需要在域名的控制平台里加入我们想要这个域名映射到的实际网站。点击“解析”操作进入域名控制台。	

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200330122544.png)



在域名控制台中加入解析记录，主要填的内容为`记录类型`、`主机记录`、`解析线路`、`记录值`。具体填写如下：

```html
记录类型		主机记录	解析线路	记录值
CNAME			www		   境外		github 博客的网址
CNAME			www		   默认  		前面CODING平台产生的静态网页的网址
A               @          默认       185.199.108.153
A               @          默认       185.199.109.153
A               @          默认       185.199.110.153
A               @          默认       185.199.111.153
```

其中将GitHub部署的博客和CODING部署的博客分别解析线路改成境外和境内，是为了让境外用户和境内用户能更加快捷的访问你的博客而不受地域限制。

注意，针对CODING 平台还需要加一个A记录类型的记录，记录值为CODING Page 的IP。CODING Page 的IP 可通过在gitbash中输入`ping 你的名字.coding.me` 来获得。



### 购买免费的SSL证书

阿里云为个人的域名提供免费的SSL证书，具体方法如下：

1. 打开个人控制台选择配置SSL证书

   ![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200330150227.png)

2. 然后在购买证书中选择免费版（个人）DV 即可。

   ![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200330150341.png)

### GitHub本地配置

1. 新建CNAME文件

   进入自己博客的source文件夹中，新建一个`CNAME`文件，注意没有后缀。然后打开该文件，在文件中加入自己的域名，注意不需要加http或者www，比如笔者的博客为unclefish.ink，直接输入unclefish.ink 保存即可。

   ![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200330124648.png)

2. 部署

   在GitBash 中输入 `hexo g`、`hexo d`进行部署，然后打开自己GitHub博客仓库，打开设置查看GitHub Pages 中的`Custom domain` 是否是自己的博客新域名。



### CODING 部署新域名

在CODING平台的静态网页部署中选择设置，然后在自定义域名中加了自己的域名后保存即可（注意要求要在阿里云的DNS中先加入CNAME 导向CODING的静态网页）。

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/3.gif)



修改完域名后，网站的浏览数会重新开始计数，所以算是一种弊端吧。



## 将博客重新收录到谷歌和百度中

### 谷歌收录

由于换了域名，所以需要重新加入收录库，在控制台中增加收录网页，然后在验证中选择dns验证，这时候谷歌会提供一个TXT，将这个TXT复制到阿里云的DNS解析记录中，通过验证后即可被收录到谷歌中（注意验证后要提交[站点网页](https://sspai.com/post/59568)）。新增记录格式如下：

```
记录类型		主机记录	解析线路	记录值
TXT				@			默认		google-xxx(复制来的值)
```

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200330161605.png)



### 百度收录

#### DNS 验证

因为GitHub不允许百度的爬虫，所以这里我们通过将百度的搜索导向我们在CODING平台上建立的静态网页。

1. 在百度的[站点网站](https://ziyuan.baidu.com/site/index#/)中添加自己的博客。

2. 选择CNAME验证方式，这时候百度会要求将`xxxxx.你的博客`使用CNAME解析到ziyuan.baidu.com, 这时候我们要将前面的xxxx给复制下来。

3. 打开阿里云DNS解析工具，新增如下记录，注意xxxx为之前复制的值。

   ```
   记录类型		主机记录	解析线路	记录值
   CNAME			xxxx		默认		ziyuan.baidu.com
   ```

4. 在百度站点平台点击验证即可。



#### 提交站点文件

在gitbash中输入` npm install hexo-generator-baidu-sitemap --save` 来加载插件，然后在博客的_config.yml 文件中添加以下代码。

```yml
baidusitemap:
  path: baidusitemap.xml
```

最后通过`hexo g; hexo d `实现部署，会发现在public文件夹中多了一个baidusitemap.xml 文件，这就是百度所要抓取的站点文件。随后在百度链接提交处选择sitemap输入自己的文件路径即可。

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200330165828.png)