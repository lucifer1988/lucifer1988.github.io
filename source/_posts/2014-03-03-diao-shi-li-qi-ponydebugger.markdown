---
layout: post
title: "调试利器-ponyDebugger"
date: 2014-03-03 15:12:01 +0800
comments: true
categories: iOS
description: "调试利器-ponyDebugger"
keywords: ponyDebugger,调试
published: true 
---

<img src="/images/ponyDebugger_icon.png">  

ponyDebugger是git上一个利用Chrome开发者工具来进行iOS客户端远程调试的工具包，与其他远类型程调试工具相比，它有着非常IMBA且又非常实用的功能：网络活动监控、查看CoreData对象、视图层级查看等，下面让我们看看如何驾驭这只神奇的小马驹吧！

<!--more-->

##功能

ponyDebugger提供了四个特色的功能，包括：监控网络、CoreData对象查看、视图分层查看和远程日志打印。

###监控网络

所有通过NSURLConnection进行的网络访问，都会被监测到，也就是说那些基于NSURLConnection的第三方网络组件，如AFNetworking，都可以被监测到，而且看以方便的查看到：访问类型、接口名、错误类型，返回类型、网络延时等信息，非常强大！

<img src="/images/ponyDebugger_Network.png">

###CoreData对象查看

这个功能就不必多说了，你可能见过很多方便的sqlite工具，比如FireFox的SQLite Manager插件，但是这样直接查看CoreData对象的不多见吧？

<img src="/images/ponyDebugger_CoreData.png">

###视图分层查看

这个功能对于前端开发者来说是最重要不过了，ponyDebugger将你应用的视图关系以xml的形式分层展示出来，选择相应xml，会在客户端进行对应视图的高亮响应，xml的属性信息可以在客户端进行配置，而且竟然支持直接修改xml属性值，而客户端UI会实时做出改变！

<img src="/images/ponyDebugger_ViewHierarchy.png">

###远程日志打印

这个可能你觉得没什么亮点，上次介绍的NSLogger可是专门做这件事的，不过ponyDebugger的这一功能也不会让你失望，它的语句类型不多，主要分为```PDLog()```和```PDLogObjects()```，```PDLog()```负责打印字符串信息，```PDLogObjects()```负责打印对象和数组。不过正如下图所示，```PDLogObjects()```打印出的对象也是分层展示的，比如查看一个数组中自定义modal对象的一个属性的值这样的事情就非常轻松，而且不用你再回Xcode中加断点，这是不很爽啊！

<img src="/images/ponyDebugger_Console.png">

##配置
##使用
##小结


