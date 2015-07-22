---
layout: post
title: "Effective Objective-C读书笔记1"
date: 2015-07-20 14:47:04 +0800
comments: true
categories: iOS
published: true
---

关于书不多做介绍了，很有名的一本书，是Mattt Thompson大神写的，他是AFNetworking的主要作者，同时维护了[NSHipster](http://nshipster.com)，这本书之前看了一次，但是没那么细致，打算再看一次，同时做做笔记。

<!--more-->

###Item1 Familiarize Yourself with Objective-C's Roots

1. OC采用消息传递而非函数调用的基本结构，二者最大区别是消息传递中运行时才决定执行的代码，而函数调用中编译器会决定执行的代码。所以运行期承担了OC运作的大部分工作，所以每当运行期更新时你的应用都会从中受益，而不需等到重新编译（最后这段，不是太明白）。
2. 学好C会让你更好理解OC，诸如内存模型和引用计数这些概念。所有OC对象的内存都是[开辟在堆上的，不在栈上](http://mobile.51cto.com/iphone-394484.htm)，栈是编译器控制的，堆是程序员控制的，而这些对象的指针是存放在栈上的，所以当指针不存在，而程序员又没有释放堆上的对象，就导致了内存泄露。
3. OC是通过引用计数来模拟内存的开辟与释放。
4. 有些变量是直接开辟在栈上的，如CGRect，他是一个结构体，不同于对象，他们的使用不会影响性能。

<!--more-->

###Item2 Minimize Importing Headers in Headers

1. 尽量避免在类的头文件直接#import其他class的头文件，能使用@class尽量使用，有俩个好处：1、避免引用头文件的连锁效应，增加编译时间；2、避免了互相#import头文件而出现的循环导入的特殊情况。
2. 一些需要导入头文件到.h文件的请款：1、class所继承的父类；2、使用protocol类型。
3. 遵从protocol可以放在匿名分类中。

<!--more-->

###Item3 Prefer Literal Syntax over the Equivalent Methods

1. 尽量多去使用文字型语法，这样可减少代码量，增加可读性。
2. 关于NSArray的文字型创建语法，如果其中一个对象为nil，则会立即抛出异常，而使用传统的*arrayWithObjects:*则会在加入nil对象时停下，并不会报错，这使得我们更难发现这一问题。
3. 唯一一个不足是文字型语法只接受Foundation框架的对象，而不接受自定义对象。

<!--more-->

###Item4 Prefer Typed Constants to Preprocessor #define

1. 尽量多使用静态常量，而不是预编译常量。原因只要是预编译常量是代码整体进行替换，容易被重赋值，常量的范围不好控制。类似*static const NSTimeInterval kAnimationDuration = 0.3*
2. 而如果要使用全局常量（比如注册和接受通知的名称），采用以下方式

```objectivec
//in the header file
extern NSString *const EOCStringConstant;

//in the implementation file
NSString *const EOCStringConstant = @"VALUE";

//基本类型常量
//EOCAnimatedView.h
extern const NSTimeInterval EOCAnimatedViewAnimationDuration;

//EOCAnimatedView.m
const NSTimeInterval EOCAnimatedViewAnimationDuration = 0.3;

```
 
<!--more-->
    
###Item5 Use Enumerations for States, Options, and Status Codes

1.使用枚举类型主要是用于定义状态和选项，可读性好是它最大的优点，c++11后OC开始支持自定义枚举类型所用的数据类型。  
2.用枚举做选项时，可用位移的方式实现多个选项合并使用，这种方式广泛用于UIKit。

```objectivec
typedef NS_OPTIONS(NSUInteger, EOCPermittedDirection) {
  EOCPermittedDirectionUp = 1<<0,
  EOCPermittedDirectionDown = 1<<1,
  EOCPermittedDirectionLeft = 1<<2,
  EOCPermittedDirectionRight = 1<<3,
}

EOCPermittedDirection permittedDirection = EOCPermittedDirectionUp | EOCPermittedDirectionDown;
if(permittedDirection & EOCPermittedDirectionUp) {
  //EOCPermittedDirectionUp is set
}
```
    
3.OC定义了专门定义枚举的宏，NS_ENUM和NS_OPTIONS，他们对兼容新旧编译器做了自动判断，推荐使用，如想使用可合并的枚举，必须使用NS_OPTIONS来定义。  
4. 最后一点，对枚举型值执行switch语句时，不要添加default处理。
   
<!--more--> 

