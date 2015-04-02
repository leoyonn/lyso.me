title: 我的博客站点搭建过程
date: 2014-05-07 18:44:12
updated: 2014-05-10 18:44:12
permalink: setup-this-site
tags:
 - Hexo
 - github pages
 - Jekyll
categories:

---
![builder](http://lyso.qiniudn.com/mm.1.jpg)
## 用什么搭建的

* 空间用的Github Pages
 - 用天航的话说，cnblogs/csdn/sina-blog等太低端太屌丝
 - 流量不算太大时买空间又要花钱不太划算
 - 我的代码大都在github上，Github Pages发布方便
* 域名在godaddy上购买
 + 国外的，不需要备案
* DNS用DNSPod
 + 方便好用
* 静态网页内容在本地用hexo生成（模板+markdown），并发布到github上
 + 用天航的话说，wordpress太低端
 + markdown简洁方便，在Jekyll、Octopress、Pelican、FarBox、Ghost、Marboo、Hexo、Medium中综合选取和Hexo，选取原因包括：
   - 名字好听
   - 台湾人搞的，中文支持肯定不疼
   - 对Node.js的印象比Ruby好
   具体情况可以随便搜一下这些关键词，网上一大堆。

* 评论用多说（会换掉成我们自己开发的产品），分享用jiathis
 + 不用disqus/addthis是因为在天朝用的人不多，不需要装B的地方就别装了
 + 我们已经做出来一套消息通信平台，可以做任何disqus可以做的和做不了的事情，敬请期待：）

## 已有环境

已经安装好`git`，本地有`shell`命令终端，有个文本编辑器如`editplus`或`sublime text 2`或`记事本`，有`github`账号。

## 安装node.js

直接到[node.js官方网站](http://nodejs.org/) 下载安装即可。

## 安装hexo
`npm install -g hexo`

## 初始化你的博客站点

```
mkdir your-site
cd your-site
hexo init # 初始化站点
hexo g # 同 hexo generate，生成静态文件
npm install # 仅需运行一次
hexo server # 起服务
```

此时在http://localhost:4000 就能看到服务了。

## 上传到github
首先，需要在github上有个账号，并且把本机的ssh加入到github的authed-keys中。这些默认有节操的程序员都有的，自然不必再讲解了。如果你正在朝有节操的路上奔走，赶紧搜个教程搞定吧：）

其次，在github上新建一个特殊的repository，比如我的github账号是leoyonn，则我新建是 leoyonn.github.io 这个repository。

然后，在你的本地hexo站点目录下的_config.yml文件中添加配置：

```
deploy:
  type: github
  repository: git@github.com:leoyonn/leoyonn.github.io.git
  branch: master
```

然后使用命令 `hexo g -d` 就能通过git的commit和push命令直接发布到githubpages上了。等个十来分钟试一下吧： http://leoyonn.github.io

## 域名配置
有个个性的域名更拉风和装B，比如 http://lyso.me ^^。
域名在godaddy上买，支持支付宝支付。有一堆教程教如何购买，基本上搜一下一路next就行了。还有教如何使用优惠券的，这里不多说了。*建议大家申请.com或.me域名。据说.info因垃圾网站太多，被搜索引擎惩罚，而且续费较贵。*
但由于墙的原因，godaddy的域名解析在国内基本上无法使用，看大家都用dnspod，我试着也不错。

首先，在godaddy帐户里设置本域名对应的dns解析地址为dnspod，如下表：

| # | Nameserver | Status |
|:--------|:---|:---|
|1 | F1G1NS1.DNSPOD.NET | Active |
| 2 | F1G1NS2.DNSPOD.NET | Active |

其次，在这里找到github的ip地址：https://help.github.com/articles/my-custom-domain-isn-t-working ，并在dnspod上的域名配置项里添加两个A记录，如下表：

|	主机记录 |	记录类型|	线路类型|记录值|	MX优先级|TTL|	操作|
|:--------|:---|:---|:---|:---|:---|:---|:---|
| @ |A | 默认| 192.30.252.153| -| 600| 删除  暂停|
| @| A| 默认| 192.30.252.154| -| 600| 删除 暂停 |
| @| NS| 默认| f1g1ns1.dnspod.net.| -| 600| 删除  暂停| 
| @| NS| 默认| f1g1ns2.dnspod.net.| -| 600| 删除  暂停| 
| email| CNAME| 默认| email.secure...r.net.| -| 600| 删除  暂停| 

然后，在本地hexo根目录中`source`目录下添加`CNAME`文件，文件内容为一行，就是你的域名：`lyso.me`。
好了，用`hexo g -d`发布上去等一会试试吧。

## 写一篇新文章

写新文章的原理就是在`source/_posts`目录下添加新的md文档。你既可以手动新建一个文本文件扔进去，也可以通过以下命令新建：

```
hexo new [layout] "postName" 
```

其中layout是可选参数，默认值为post。layout有哪些可以在scaffolds目录下查看，也可以添加自己的layout或编辑现有的layout。至于怎么写文章，就是看你怎么用简洁的markdown语法玩转自己的思路了。文章里可以添加标签、title、摘要、创建时间、永久链接等（可以伪造时间呢，哈哈）：

```
title: 我的博客站点搭建过程
date: 2014-05-07 18:44:12
updated: 2014-05-10 18:44:12
permalink: setup-this-site
tags:
 - Hexo
 - github pages
 - Jekyll
categories:

---
![builder](http://lyso.qiniudn.com/mm.1.jpg)
## 用什么搭建的

```

## 使用模板
主题模板的使用很简单，到[这里](http://github.com/tommy351/hexo/wiki/Themes) 找到你喜欢的主题，然后用git下载到themes目录：
```
git clone https://github.com/heroicyang/hexo-theme-modernist.git themes/modernist
```

并修改`_config.yml`指定为此模板：

```
theme: modernist
```

很多模板里都内置了评论、分享等插件，慢慢挖掘吧：）

## 图床
考虑到博客的速度，同时也为了便于博客的迁移，图床是很有需求的，大家可以看看我的[羽毛球亚军那篇博客](http://lyso.me/2014/06/27/badminton-second-mi/) 体验一下速度：）。看到大家都墙裂推荐七牛，还访问速度极快，支持日志、防盗链和水印，我也试了试，果然不错。本想直接用百度云网盘的外链等方法的，解决不了连接过期问题，还没这个方便。七牛还支持网站直接备份，使用起来非常方便傻瓜，不介绍啦。

## 评论和分享
在很多模板的`_config.yml`中都可以找到配置，不再多说。

## 自定义页面
我做了两个自定义页面，一个`about`，一个`404.html`。用以下命令可以创建自定义页面：
```
hexo new page "about"
```

404页面我做了腾讯的找孩子公益，比如你点[这里](http://lyso.me/404) 进我博客的一个不存在的页面，页面下方有教你怎么做（非常简单，只需要把404.html放在source下，并把那句js代码放进去^^）

