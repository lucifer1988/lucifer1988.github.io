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
		unsigned int didReceiveData:1;
		unsigned int didFailWithError:1;
		unsigned int didUpdateProgressTo:1;
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

//调用委托时
/*
if(_delegateFlags.didUpdateProgressTo) {
	[_delegate networkFetcher:self didUpdateProgressTo:currentProgress];
}
*/

@end
```  

<!--more-->

##Item24 Use Categories to Break Class Implementations into Manageable Segments

1. 分类这一特性主要为了解决一个类在开发中无限膨胀的问题，将一个类的方法按照功能进行分类处理是常规做法。
2. 第二种用途是为了对代码进行分割增加可读性，例如NSURLRequest想增加专门的HTTP请求，单纯继承不是一个很好的选择，原因是NSURLRequest封装了一组针对CFURLRequest的C方法，无法通过继承获得，而直接添加这些HTTP的方法则会导致一些代码理解错误，例如开发者使用FTP协议，去发现可以调用关于HTTP的方法，所以将HTTP部分的方法做成NSHTTPURLRequest的分类是最好的选择。
3. 第三个用途是方便调试，原因是分类中的方法在日志里会显示为类似：*-[EOCPerson(Friendship) addFriend:]*，可以方便定位该方法。
4. 另外，在做一个库时，把一些私有方法用名为Private的分类封装，这样这些方法不用暴露在外，而内部又可以随意调用，而且外部万一用到了，也可以在日志中看到private的标志，起到了文档的作用。

<!--more-->

##Item25 Always Prefix Category Names on Third-Party Classes

1. 为一个类添加分类后，运行时runtime会遍历category每个方法，顺便加入类的方法列表，如果这时category重写了类的某个方法，这将覆盖原有的方法，如果多个category都出现这个情况，那么最后被载入的那个分类的方法会被采用，这两种情况都将导致Bug，且难以定位。
2. 解决这一问题的方法只能是添加namespace，规则参考Item15，最好就是公司+项目这样的方式，而且最好将分类的名字也加namespace，这样可以避免warning。
3. 要记住添加到一个类的category中的方法，只要被添加，在所有类的示例都可以调用（这里还是需要导入这个category才可以），尤其在为Cocoa中的类添加分类时时刻注意添加命名空间，去刻意重写类中的方法是一个非常坏的习惯，它带来的问题可能比好处大得多。

<!--more-->

##Item26 Avoid Properties in Categories

1. category默认是不支持添加property的（匿名分类除外），虽然这一做法可以在技术上实现，但是依然不推荐这么做。
2. category不支持property，主要是无法自动合成setter和getter方法，解决这一问题有两个方法：1）使用Item12的做法，用@dynamic声明，重写message-forwarding的方法，在runtime添加setter和getter方法；2）使用Item10，使用associated objects，自己在getter和setter进行关联。
3. 上述两个方法均可行，但作者认为这两个方法都不完美，缺点有二：1）内存管理，你很容易忘记这个property的特殊性，而只去修改property的关键字，而忘记去修改setter方法；2）如果你想让自己的property对象在内部支持mutable，可以在内部声明一个，mutable拷贝，但是这又会出现一个进入源代码的混乱路径，所以在category中定义property的代价是很高的。
4. 作者建议的方法是把所有的变量都放入原类中，而category只提供额外的方法。
5. 但有时category中可以添加只读变量，而且也不涉及读写原类的变量，但是虽然不报错，还是推荐使用一个方法来完成，因为真的没必要这么做。

<!--more-->

##Item27 Use the Class-Continuation Category to Hide Implementation Detail

1.OC是没有真正的私有方法的，但我们还是不希望把不需要暴露的方法和变量暴露在外，所以匿名分类就是一种隐藏这些细节的手段。  
2.你可以将实例变量声明在匿名变量或implementation中，可以完全不用暴露你要导入的头文件等一切信息，例如：  

```objectivec
@interface EOCPerson(){
	NSString *_anInstanceVariable;
}
//Method declarations here
@end

@implemenation EOCPerson {
	int _anotherInstanceVariable;
}
//Method implemenations here
@end
```

3.一般OC代码中使用C++一般两种情况：一些游戏相关的后端代码需要用C++，使用的第三方库使用了C++，而你作为使用者除非特殊情况，一定要使用匿名分类来使用C++，这样其他类使用你的类时，不用再因为C++的原因，将.m文件命名为.mm，而使编译器将其编为Objective-C++。Cocoa的web browser framework和CoreAnimation使用了这一模式。  
4.还有一种应用就是在外部声明readonly的property，然后在匿名分类中再将其声明为readwrite，这样可以实现外部只能通过方法设置值，而内部可以正常使用该变量，可能会出现外部在访问，内部同时在赋值同步的问题，将在Item41讨论。  
5.接下来就是可以在匿名分类声明私有方法，虽然这不是必须的，而作者比较推荐先列好方法，理清思路，再开始实现，如果是比较大的项目，还是需要这么干的。  
6.最后就是可以在匿名分类添加委托。

<!--more-->

##Item28 Use a Protocol to Provide Anonymous Objects

1. 利用Protocol可以实现创建一些匿名对象，例如：id<EOCDelegate> delegate。
2. 例子1：来自多个第三方类库的数据库管理对象对应不同数据库类型，现在需要提供一个统一的Manager，来返回这些不同的对象，依靠基类继承是不可能的，只能通过定义一组数据库通用的操作作为Protocol，然后分别继承这些类，而新类则遵从这一protocol，这样Manager只需返回id<Protocol>类型的对象即可，而使用者也只需要知道它们实现了这些方法也足够了。
3. 例子2：已确定只有一个类型，但其是一个内部使用的数据类型，不需要将其所有细节暴露，只需要暴露其中一部分方法即可，那么将这些方法声明为Protocol，然后返回类型定义为id<Protocol>即可，其实就是实现了对对象的大部分封装。


