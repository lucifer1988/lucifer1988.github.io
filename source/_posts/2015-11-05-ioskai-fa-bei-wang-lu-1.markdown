---
layout: post
title: "iOS开发备忘录1"
date: 2015-11-05 13:41:03 +0800
comments: true
categories: iOS
published: true
---

总结一些iOS开发必备的知识点，结构可能会比较杂，可以当做备忘录使用，不断更新中。

<!--more-->

## 1.library和framework的比较

library也就是我们常用的.a文件，而framework就是.framework文件，当然还有.dylib这样的文件。


### 静态库和动态库的区别

* 静态库：链接时完整地拷贝至可执行文件夹中，被多次使用时会有多份冗余拷贝。
* 动态库：链接时不复制，程序运行是由系统动态加载到内存，供程序调用，系统只加载一次，多个程序共用，节省内存。

### iOS库的形式

* 静态库：.a和.framework
* 动态库：.dylib和.framework
* 系统提供的.framework是动态的，而自己开发的.framework是静态的

### .a和.framework区别

* .a就是一个纯二进制的文件，而.framework还会包含头文件和资源文件
* .a一般是要配合.h头文件使用的，.framework是可以直接使用的
* 实际上.framework = .a + .h + sourceFile

### 需要注意的问题

* 注意理解：无论是.a静态库还.framework静态库，我们需要的都是二进制文件+.h+其它资源文件的形式，不同的是，.a本身就是二进制文件，需要我们自己配上.h和其它文件才能使用，而.framework本身已经包含了.h和其它文件，可以直接使用。
* 图片资源的处理：两种静态库，一般都是把图片文件单独的放在一个.bundle文件中，一般.bundle的名字和.a或.framework的名字相同。.bundle文件很好弄，新建一个文件夹，把它改名为.bundle就可以了，右键，显示包内容可以向其中添加图片资源。
* category是我们实际开发项目中经常用到的，把category打成静态库是没有问题的，但是在用这个静态库的工程中，调用category中的方法时会有找不到该方法的运行时错误（selector not recognized），解决办法是：在使用静态库的工程中配置other linker flags的值为-ObjC。
* 如果一个静态库很复杂，需要暴露的.h比较多的话，就可以在静态库的内部创建一个.h文件（一般这个.h文件的名字和静态库的名字相同），然后把所有需要暴露出来的.h文件都集中放在这个.h文件中，而那些原本需要暴露的.h都不需要再暴露了，只需要把.h暴露出来就可以了。
* 创建.a文件的一篇[博客](http://www.raywenderlich.com/41377/creating-a-static-library-in-ios-tutorial)，创建.framework的一篇[博客](http://www.raywenderlich.com/65964/create-a-framework-for-ios)。

<!--more-->

## 2.事件响应链The Responder Chain

### 事件的传递顺序

当用户触发的一个事件发生，UIKit会创建一个包含要处理的事件信息的事件对象。然后她会将事件对象放入active app’s（应用程序对象，每个程序对应唯一一个）事件队列。对于触摸事件，事件对象就是UIEvent对象封装的一系列触摸集合。对于动作事件，这个事件对象依赖于使用的framework和你关心哪种动作事件。

### 事件类型

事件通过特殊的路径传递直到被传递到一个可以处理该事件的对象。首先，单例的UIApplication对象从顶层的队列中获取事件，然后分发。典型的，它将事件发送到App的key window对象，window则为了处理该事件而发送它到初始化对象（initial object），这个初始化对像依靠事件类型。

* 触摸事件（Touch events）。对于触摸事件，window对象首先会尝试将事件传递给事件发生的view。这个view就是所谓的hit-test view。寻找hit-test view的方法叫hit-testing，具体描述见[Apple文档](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/event_delivery_responder_chain/event_delivery_responder_chain.html#//apple_ref/doc/uid/TP40009541-CH4-SW4)。
* 动作事件和远程控制事件（Motion and remote control events）。在这些事件中，window对象发送事件到第一个响应器。第一个响应器的具体描述见[Apple文档](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/event_delivery_responder_chain/event_delivery_responder_chain.html#//apple_ref/doc/uid/TP40009541-CH4-SW1)。

事件传递路径的最终目的时找出能处理和响应该事件的对象。因此，UIKit给适合处理该事件的对象发送事件。对于触摸事件，这个对象就是hit-test view，对于其他事件，这个对象就是第一个响应器（first responder）。

### 触摸事件的响应链

iOS使用hit-testing寻找触摸的view。 Hit-Testing通过检查触摸点是否在关联的view边界内，如果在，则递归地（recursively）检查该view的所有子view。在层级上处于lowest（就是用户直接接触view）且边界范围包含触摸点的view成为hit-test view。确定hit-test view后，它传递触摸事件给该view。

举例说明，假设用户触摸了图中的view E。iOS通过如下顺序查找hit-test view。

![image](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/hit_testing_2x.png)

1. 触摸点在view A中，所以要检查子view B和C。
2. 触摸点不在view B中，但在C中，所以检查C的子view D和E。
3. 触摸点不在D中，但在E中。

[hitTest:withEvent:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/instm/UIView/hitTest:withEvent:)方法通过传递进来CGPoint和UIEvent返回hit test view。该方法调用[pointInside:withEvent:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/instm/UIView/pointInside:withEvent:)方法，如果传入hitTest:withEvent:的point在view的边界范围内，则pointInside:withEvent:返回YES。然后，这个方法会在view的所有子view中递归的调用hitTest:withEvent:。

如果传入hitTest:withEvent:的point不在view的边界范围内，则pointInside:withEvent:返回NO。这个point会被忽略，hitTest:withEvent:返回nil。如果一个子view返回NO，则它所在的view的层级上的分支的子view都会被忽略。

Hit-test view是处理触摸事件的第一选择，如果hit-test view不能处理事件，该事件将从事件响应链中寻找响应器，直到系统找到一个处理事件的对象。

### 响应器链

一些类型的事件的传递依赖响应器链。响应器链（responder chain）是一系列相关的响应器对象。它开始于第一个响应器终止于应用对象（application object）。如果第一个responder不处理事件，则会根据responder chain将event传递给下一个responder。

Responder object，即可以响应和处理事件的对象。UIResponder类是所有responder对象的基类，它定义了动态的接口，不仅处理事件也包括处理响应行为。包括UIApplication，UIViewController，和UIView类都是responder，这意味着所有view和大部分关键的controller对象都是responder。但是Core Animation layers不是responders。

First responder被设计来第一个接收事件。典型的，first responder是一个view object。之所以成为第一个responder由于两个原因：

1. 覆盖canBecomeFirstResponder方法，返回YES。
2. 接收becomeFirstResponder消息。如果必须，一个object能发送给自身这个消息。

### 响应器链的传输路径

如果初始化对象（initial object）—— 即hit-test view或者first responder —— 不处理事件，UIKit会将事件传递给responder chain的下一个responder。每个responder决定它是传递事件还是通过nextResponder方法传递给它的下一个responder。这个操作继续直到一个responder处理event或者没有responder了。

Responder chain 序列在iOS确定一个事件并将它传递给initial object（通常是view）时开始。所以initial view有处理事件的第一个机会。下图描述了两个不同的事件传递路径（因为不同的app 设置）。一个App的事件传递路径由app特殊的构成决定，但事件传递路径会遵守相同的规则。

<img src="/images/responser_chain.jpeg">

### 手动指定当前view不响应事件

```objectivec
-(BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event {
    for (UIView *view in self.subviews) {
        if (!view.hidden && view.userInteractionEnabled && [view pointInside:[self convertPoint:point toView:view] withEvent:event])
            return YES;
    }
    return NO;
}
```

### 总结

事件的传递和响应分两个链：

* 传递链：由系统向离用户最近的view传递。UIKit –> active app’s event queue –> window –> root view –>……–>lowest view
* 响应链：由离用户最近的view向系统传递。initial view –> super view –> …..–> view controller –> window –> Application









