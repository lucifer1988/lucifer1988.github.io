---
layout: post
title: "基于Octopress搭建自己的博客网站"
date: 2014-02-12 17:19:13
comments: true
categories: BlogBasics
description: "基于Octopress搭建自己的博客网站"
keywords: Octopress,git
published: true
---
本文将介绍当前主流的一个基于git的博客管理工具：[Octopress](http://octopress.org/)，它是利用[Jekyll](https://github.com/jekyll/jekyll)博客引擎开发的一个博客系统，生成的页面可以利用github page来展现，大致原理是利用git来管理你的博客文章，然后发布到github上成为独立博客站点。本博客就是基于Octopress搭建的。

<!--more-->

##安装
安装Octopress的详细步骤在其官方文档有详细说明，地址是<http://octopress.org/docs/setup/>，这里只列一下大致步骤：

###安装Ruby
Mac环境下一般是自带Ruby的，不过你需要注意下Ruby的版本：

```
ruby --version
```

如果低于1.9.3，那么就需要更新：

```
rvm install 1.9.3
rvm use 1.9.3
rvm rubygems latest
```
###安装Octopress
确保Ruby环境OK后，继续安装Octopress，先从git上clone其代码：

```
git clone git://github.com/imathis/octopress.git octopress
cd octopress    # If you use RVM, You'll be asked if you trust the .rvmrc file (say yes).
ruby --version  # Should report Ruby 1.9.3
```
继续安装依赖项：

```
gem install bundler
rbenv rehash    # If you use rbenv, rehash to be able to run the bundle command
bundle install
```
最后一步：

```
rake install
```
###一些配置工作
安装好后要进行一些配置工作：

 * 修改_config.yml中有关博客名，作者名之类的信息，填写git上对应的仓库url（见下一小节）。
 * 删去_config.yml中twitter相关的信息，否则由于GFW的原因，将会造成页面load很慢。
 * 修改定制文件/source/_includes/custom/head.html 把google的自定义字体去掉，原因同上。

注：在修改_config.yml时注意，所有的键值格式都是类似```title: 刘毅的技术博客```，即```:```后是有个空格的，如果在修改时不慎删去，可能在```rake generate```出现类似这样的问题：<http://stackoverflow.com/questions/10086806/i-can-not-do-any-modify-after-octopress-installed/13898285#13898285>，在这里提醒下各位。

更进一步的配置参见：<http://octopress.org/docs/configuring/>

<!--more-->

##部署
我们利用[Github Pages](http://pages.github.com)来托管博客。

首先，你需要在github上创建一个新的仓库，命名方式是username.github.io，这就是上一步需要填写的url，也是将来博客地址的域名。

创建好仓库之后，需要在octopress文件夹下运行rake命令来部署git仓库，期间需要输入你的仓库地址：

```
rake setup_github_pages
```
然后生成博客内容，并部署到仓库，这也是每次编辑好新博客后需要做的事：

```
rake generate
rake deploy
```
现在已经就可以在```http://username.github.io```访问到新的博客内容了，不过octopress的source部分更新需要手动提交：

```
git add .
git commit -m 'your commit words'
git push origin source 
```

<!--more-->

##写博客
上面都完成之后，写博客要做的事就很easy了，首先依然需要rake命令new一个新博客文件：

```
rake new_post["你的文章名"]
```
命名支持中英文，会在source/_posts/下生成一个markdown文件，抬头是：

```
layout: post
title: "基于Octopress搭建自己的博客网站"
date: 2014-02-12 17:19:13
comments: true
published: false
categories: BlogBasics
```
这里的published: false字段是我自己添加的，它置为false的作用是即使你部署这篇博客到git上，也不会被访问到，这可以满足你在彻底完成并润色好文章之后再发布的需要。另外，这里文章的编辑使用markdown语法，我会在之后写一篇博客介绍下的。

现在假设你精心撰写的文章已经OK了，那么就将它发布吧：

```
rake generate
git add .
git commit -am "Some comment here." 
git push origin source
rake deploy
```
现在你应该已经能访问到你的新博客了，如果需要在部署之前先本地预览下博客，可以在终端输入：

```
rake preview
```
然后在本地浏览器访问：```http://localhost:4000/```预览你的博客。

<!--more-->

##高级配置
因为Octopress自带的Disqus Comments评论系统比较慢，所以你可以选择国内的评论和分享系统，本博客分享系统用的是[加网](http://www.jiathis.com)，评论系统用的是[友言](http://www.uyan.cc)，具体步骤是：

 * 增加文件：source/_includes/post/weibo.html 
 * 访问 <http://www.jiathis.com/>，获取分享的代码 
 * 访问 <http://uyan.cc/>，获得评论的代码 
 * 将上面2步获得的代码都加入weibo.html中即可

<!--more-->

##参考
我的博客搭建和这篇文章的完成主要参考了Octopress官方文档和[唐巧](http://blog.devtang.com/blog/2012/02/10/setup-blog-based-on-github/)、[破船](http://beyondvincent.com/blog/2013/08/03/108-creating-a-github-blog-using-octopress/)的相关教程，他们都是我很尊敬的iOS开发者，可以多多关注他们，一定会受益匪浅！