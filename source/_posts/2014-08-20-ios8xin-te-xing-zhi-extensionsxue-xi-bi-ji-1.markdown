---
layout: post
title: "iOS8新特性之Extensions学习笔记1-SharingExtension"
date: 2014-08-20 09:56:53 +0800
comments: true
categories: iOS
published: false
---

<img src="/images/Extensions_Sample.png">

Extensions是iOS8一个重要的新特性，可以理解为iOS系统中的插件功能吧，但是在iOS大的封闭环境下，它只能完成一些受限的App间交互，在一定程度上弥补iOS沙盒机制对应用间通信的限制，但是总的来说对开发者还是好事，毕竟这能使我们的产品给用户带来更多样的服务和体验。

<!--more-->

iOS目前公布支持的Extension类型有：

* Today screen widgets：在通知中心的今日栏目里展示小工具（widget），可以自定义一些快捷操作；
* Share extensions：自定义内容分享组件供其他应用使用；
* Actions：在另一个应用里执行应用插件操作，如直接打开一个应用；
* Photo editing：使用第三方应用插件在苹果自带的照片应用里编辑照片或视频；
* Storage providers：应用间共享部分存储控件，如协作修改同一份文档；
* Custom keyboards：自定义键盘输入法，替代系统输入法。

这篇博客将主要介绍Share extensions的构建方法，其他的Extension将会在之后陆续介绍。

<!--more-->

##创建ShareExtension

Extension是需要包含在一个已有app当中的编译二进制文件，它不能脱离宿主app而独立存在，Xcode会提供相应extension类型的模板，如下图，我们为app新建了一个ShareExtension：

<img src="/images/SetupShareExtension.png">

新的ShareExtension主要是一个storyboard和一个`SLComposeServiceViewController`的子类，这些会提供默认的分享UI，包括字数统计、图片、文字输入，取消和发送按钮，当然你可以自己重新去定义UI。这里选择使用默认UI，来看下`SLComposeServiceViewController`提供的一些新方法：

* `presentationAnimationDidFinish()`是一个用于完成繁琐任务的钩子方法，我们这里用来取出待分享的图片；
* `contentText`用户输入的文字；
* `didSelectPost()`用户点击**Post**按钮，我们在这儿加入发送任务；
* `didSelectCancel()`用户点击**Cancel**按钮；
* `isContentValid()`用户输入内容发生变化时调用，返回布尔值告诉系统是否超出规定字数，我们在这儿做了剩余可字数统计的方法；
* `charactersRemaining`剩余可输入字数，会显示在编辑框下；

在我们进一步开始完成Extension前，先了解下如何调试Extension，方便下面的开发。

##调试ShareExtension

理论上说，你要测试你的Extension，你要通过另外一个app来调现在这个app的Extension，不过Xcode提供的调试app只能是自己开发的app，不能调系统自带的app（如果可以的话，可直接调用`photos`来测试），这样的话，我们只能采用下面的方法来调试：

1. 直接在该Extension的宿主app下加入以下代码，这样宿主app会提供一张图片和一个按钮来弹出一个通用的分享表单，里面会有我们Extension的入口；

	```objc
	@IBAction func handleShareSampleTapped(sender: AnyObject) {
	  shareContent(sharingText: "Highland Cow",
	                       sharingImage: sharingContentImageView.image)
	}
	
	// Utility methods
	func shareContent(#sharingText: String?, sharingImage: UIImage?) {
	  var itemsToShare = [AnyObject]()
	
	
	  if let text = sharingText {
	    itemsToShare.append(text)
	  }
	  if let image = sharingImage {
	    itemsToShare.append(image)
	  }
	
	
	  let activityViewController = UIActivityViewController(activityItems: itemsToShare, applicationActivities: nil)
	  presentViewController(activityViewController, animated: true, completion: nil)
	}
	```

2. 点击我们自己的Extension，会运行我们开发的Extension进程；
3. 你也可以在你的其他app中进行类似的测试。或者也可以保持你的app运行的基础上，自己打开`photos`应用，也可进行同样的测试。


