---
layout: post
title: "调试利器-PonyDebugger"
date: 2014-03-03 15:12:01 +0800
comments: true
categories: iOS
description: "调试利器-PonyDebugger"
keywords: PonyDebugger,调试
published: true 
---

<img src="/images/ponyDebugger_icon.png">  

PonyDebugger是git上一个利用Chrome开发者工具来进行iOS客户端远程调试的工具包，与其他远类型程调试工具相比，它有着非常IMBA且又非常实用的功能：网络活动监控、查看CoreData对象、视图层级查看等，下面让我们看看如何驾驭这只神奇的小马驹吧！

<!--more-->

##功能

PonyDebugger提供了四个特色的功能，包括：监控网络、CoreData对象查看、视图分层查看和远程日志打印。

###监控网络

所有通过NSURLConnection进行的网络访问，都会被监测到，也就是说那些基于NSURLConnection的第三方网络组件，如AFNetworking，都可以被监测到，而且看以方便的查看到：访问类型、接口名、错误类型，返回类型、网络延时等信息，非常强大！

<img src="/images/ponyDebugger_Network.png">

###CoreData对象查看

这个功能就不必多说了，你可能见过很多方便的sqlite工具，比如FireFox的SQLite Manager插件，但是这样直接查看CoreData对象的不多见吧？

<img src="/images/ponyDebugger_CoreData.png">

###视图分层查看

这个功能对于前端开发者来说是最重要不过了，PonyDebugger将你应用的视图关系以xml的形式分层展示出来，选择相应xml，会在客户端进行对应视图的高亮响应，xml的属性信息可以在客户端进行配置，而且竟然支持直接修改xml属性值，而客户端UI会实时做出改变！

<img src="/images/ponyDebugger_ViewHierarchy.png">

###远程日志打印

这个可能你觉得没什么亮点，上次介绍的NSLogger可是专门做这件事的，不过PonyDebugger的这一功能也不会让你失望，它的语句类型不多，主要分为```PDLog()```和```PDLogObjects()```，```PDLog()```负责打印字符串信息，```PDLogObjects()```负责打印对象和数组。不过正如下图所示，```PDLogObjects()```打印出的对象也是分层展示的，比如查看一个数组中自定义modal对象的一个属性的值这样的事情就非常轻松，而且不用你再回Xcode中加断点，这是不很爽啊！

<img src="/images/ponyDebugger_Console.png">

##配置

配置工作包括配置服务器端和客户端。

###配置服务器端

* 安装Xcode's Command Line Tools，在之前的版本中可直接在Xcode中安装，如果你的环境更新到10.9和Xcode5之后，不妨参考[这里](http://ourcoders.com/thread/show/1208/)进行安装，这是之前10.9更新CocoaPods时发现的一个问题。
* 在终端执行以下命令，进行安装

```
curl -sk https://cloud.github.com/downloads/square/PonyDebugger/bootstrap-ponyd.py | \
  python - --ponyd-symlink=/usr/local/bin/ponyd ~/Library/PonyDebugger
```
注：如果在安装过程中报出如下错误：

```
···
  Running setup.py (path:/Users/user/Library/PonyDebugger/build/tornado/setup.py) egg_info for package tornado

    warning: no previously-included files matching '_auto2to3*' found anywhere in distribution
Downloading/unpacking pybonjour (from ponydebugger)
  Could not find any downloads that satisfy the requirement pybonjour (from ponydebugger)
  Some externally hosted files were ignored (use --allow-external pybonjour to allow).
Cleaning up...
```
可以参考[这里](https://github.com/square/PonyDebugger/issues/100)的解决方法，在我安装的过程中也遇到了这个问题，按照提示是```pip```安装时没有配置```--allow-external pybonjour --allow-unverified pybonjour```。

* 成功安装后，在终端输入```ponyd serve --listen-interface=127.0.0.1```，打开监听。
* 最后打开浏览器，输入地址```http://localhost:9000```，如果访问到如下结果，说明安装成功。

<img src="/images/ponyDebugger_install.png">

###配置客户端

由于现在PonyDebugger支持CocoaPods安装了，可以直接在你的podfile中加入：

```
pod 'PonyDebugger'
```
然后，安装一下就可以了。

```
pod install
```
不了解CocoaPods的童鞋可以参考下唐巧大哥的[这篇介绍](http://blog.devtang.com/blog/2012/12/02/use-cocoapod-to-manage-ios-lib-dependency/)，希望手动加入PonyDebugger的童鞋可以参考[官方文档](https://github.com/square/PonyDebugger)中Manual Installation一项。

##使用

相比起搭建环境，PonyDebugger的使用非常简单。你需要在你应用的```AppDelegate.m```的```- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {}```方法中加入以下代码：

```
    PDDebugger *debugger = [PDDebugger defaultInstance];
    //设置网络监控
    [debugger enableNetworkTrafficDebugging];
    [debugger forwardAllNetworkTraffic];
    //通过TCP连接至服务端
    [debugger connectToURL:[NSURL URLWithString:@"ws://localhost:9000/device"]];
    // 也可通过bonjour自动连接
    //[debugger autoConnect];
    // 或连接至指定bonjour服务
    //[debugger autoConnectToBonjourServiceNamed:@"MY PONY"];
    //设置CoreData监控
    [debugger enableCoreDataDebugging];
    [debugger addManagedObjectContext:self.managedObjectContext withName:@"TIME Test MOC"];
    //设置视图分层监控
    [debugger enableViewHierarchyDebugging];
    [debugger setDisplayedViewAttributeKeyPaths:@[@"frame", @"hidden", @"alpha", @"opaque", @"accessibilityLabel", @"text"]];
    //设置远程日志打印
    [debugger enableRemoteLogging];
```
##小结

本文可以作为```PonyDebugger```一个入门级的文档。相比上一次介绍的```NSLogger```来说，```PonyDebugger```的展示方式和查看方式更加直观和方便，作为通用的调试工具非常不错，而```NSLogger```拥有强大的日志记录功能，但想要发挥```NSLogger```的作用，对程序员的经验和能力就有了一定的要求，所以两种工具各有千秋，大家各取所需就好。