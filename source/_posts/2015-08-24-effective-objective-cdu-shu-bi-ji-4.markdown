---
layout: post
title: "Effective Objective-C读书笔记4"
date: 2015-08-24 14:58:20 +0800
comments: true
categories: iOS
published: true
---

第四部分开始讨论OC的两大重要特性，Protocols和Categories。Protocols类似Java中的interfaces，弥补了OC没有多继承的缺点，常被用于实现OC委托模式，但还有很多其他方面的用途。Categories则是提供了不继承而向类添加拓展的方法，这要归功于OC动态语言特性，但同时了解它使用时的常见问题也非常重要。

<!--more-->

##Item23 Use Delegate and Data Source Protocols for Interobject Communication

1. Delegate是用于对象之间进行数据交互的设计模式，使用它的好处是可以让不同的业务逻辑解耦，实现代码模块化。而在OC中实现这一模式，主要依靠Protocols。
2. 怎么使用Delegate不介绍了，注意点是：1）命名最好和你需要委托的类相关，例如UITableView,UITableViewDelegate；2）类的delegate property是weak属性，原因是接受委托的对象一般会持有需要委托的对象，如果delegate设置为strong，需要委托的对象也会持有接受委托的对象，这样就会出现retain cycle。
3. Delegate一般都定义为option，除非一些方法是一定要被委托者实现的，同时对于option的方法，委托者需要在调用之前使用*respondsToSelector:*来内省，确保被委托者实现了该方法。
4. Delegate中定义的方法一定要清楚，而且一定要包括被委托者自身作为其中一个参数，这样如果存在同类型多个实例对象时，委托者可以在同一个方法中区分这些实例变量。
5. Protocols还可以用于DataSource模式，与Delegate模式区别是，对于一个Class来说，Delegate的信息是流出Class的，而DataSource的信息是流入Class的，设计Protocols也可以参照这一原则。
6. 对于option的方法要进行*respondsToSelector:*来检测，但是对于一些需要频繁调用的方法，采用这一方式非常影响性能，作者利用了C中的由多个1bit字段组成的结构体来标识被委托对象是否响应所有方法，这基于被委托对象一般不会动态改变对方法的响应： 
 
```objectivec
@interface EOCNetworkFetcher(){
	struc {
		unsigned int didReceiveData :1;
		unsigned int didFailWithError :1;
		unsigned int didUpdateProgressTo :1;
	} _delegateFlags;
}
@end

@implementation EOCNetworkFetcher

-(void)setDelegate:(id<EOCNetworkFetcherDelegate>delegate) {
	_delegate = delegate;
	_delegateFlags.didReceiveData = [delegate respondsToSelector:@selector(networkFetcher: didReceiveData:)];
	_delegateFlags.didFailWithError = [delegate respondsToSelector:@selector(networkFetcher: didFailWithError:)];
	_delegateFlags.didUpdateProgressTo = [delegate respondsToSelector:@selector(networkFetcher: didUpdateProgressTo:)];
}

/*
if(_delegateFlags.didUpdateProgressTo) {
	[_delegate networkFetcher:self didUpdateProgressTo:currentProgress];
}
*/

@end
```