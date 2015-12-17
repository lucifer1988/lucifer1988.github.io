---
layout: post
title: "为Octopress做迁移"
date: 2015-12-17 15:30:54 +0800
comments: true
categories: Octopress
published: true
---
最近刚换了工作，电脑也换了一个全新的，所有东西都做了一次大迁移，第一件事还是把Blog的环境迁过来，因为之前介绍的Octopress搭建是基于之前没有Octopress，第一次搭建的情况，而现在是git上已有了我们的博客环境，而要在新机器上将环境再重建起来，还是有很多不同的，所以这一篇博客会详细说明一下。

<!--more-->

## Octopress运作原理

Octopress的代码仓库有两个分支，**source**和**master**。**source**分支负责生成博客网站，而**master**分支就是我们的博客网站。

在我们[第一次搭建博客环境时](http://lucifer1988.github.io/blog/2014/02/13/hello-world-octopressfa-bu-wen-zhang-bu-zou/)，**master**分支是存储在**_deploy**子文件夹中的。由于该文件夹以下划线开头，所以在你执行**git push origin source**时，**_deploy**文件夹是被ignored，它只会在你执行**rake deploy**时更新。

所以我们重建octopress时，也要按照这一顺序，clone source分支，将master分支重建于**_deploy**文件夹，安装依赖gems。

## 从github迁移新的Octopress环境

从github clone Octopress的source分支，因为git默认的clone命令是针对master分支的，所以很多人卡在了这一步，stackOverflow上有一个回答专门解决这一问题，[如何clone一个指定的git分支](http://stackoverflow.com/questions/1911109/clone-a-specific-git-branch)。

此外，在clone时还有可能遇到**SSH connect to github fail**的问题，github的官博有[解决方法](https://help.github.com/articles/updating-credentials-from-the-osx-keychain/)。

```ruby
git clone -b source git@github.com:username/username.github.io.git octopress 
cd octopress
git clone git@github.com:username/username.github.com.git _deploy
```

安装依赖的gems，这一步与我们第一次搭建时没有区别。

```ruby
gem install bundler
bundle install
rake install # 注意如果你之前自己修改过主题，可不做这一步，否则会重新安装默认主题
```

之后的新建，提交等步骤，和之前是一致的，就不赘述了。

## 如何在两台电脑之间更新博客

可能有人会有这样的需求，其实知道原理的话，也很容易，在一台电脑提交新博客之后，那么在另一台电脑开始工作之前，先从git上更新分支，再开始新博客的书写。

```ruby
cd octopress
git pull origin source
cd ./_deploy
git pull origin master
```

本文主要参考了以下两篇博文：

1. [zerosharp的博文](http://blog.zerosharp.com/clone-your-octopress-to-blog-from-two-places/)
2. [Pazzilivo的笔记](https://github.com/Pazzilivo/Notes/blob/master/Git/octopress.md)




