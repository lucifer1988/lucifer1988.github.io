---
layout: post
title: "Swifter读书笔记1"
date: 2015-11-27 14:24:30 +0800
comments: true
categories: iOS
published: true
---
继续Swift的进阶学习，这一阶段使用的是[王巍（onevcat）](http://weibo.com/onevcat)的Swifter一书，书中记录了100个Swift开发的Tips，本章包含了Tip1~Tip20。

<!--more-->

## 1.柯里化（Currying）

Currying我们在之前的Swift by Tutorials中也学习过，是一种可以使函数分布调用的方式，也可以充当产生新方法的工厂方法，例如：

```objectivec
func greaterThan(comparor: Int)(Input: Int) -> Bool {
    return Input > comparor
}

let greaterThan10 = greaterThan(10)

greaterThan10(Input: 12)
greaterThan10(Input: 1)
```

作者举了一个实践的例子，我们之前也讲过Swift中的Selector只能通过String类型来赋值，从实用角度讲，是很好的与OC的Target/Action做了桥接，但是任然无法保证调用的安全性，用例就是借助Currying对selector调用做了改造，[这篇博客](http://oleb.net/blog/2014/07/swift-instance-methods-curried-functions/?utm_campaign=iOS_Dev_Weekly_Issue_157&utm_medium=email&utm_source=iOS%2BDev%2BWeekly)做了详细说明。

首先通过以下的例子，我们可以得知，一个类的方法也是可以被获取的，这因为Swift中方法是第一类对象，例如最后depositor的调用，它的类型就是**BankAccount -> (Double) -> ()**，其实也是currying调用，利用这一特性，博主自己对selector的调用进行了改造。

```objectivec
class BankAccount {
    var balance = 0.0
    
    func deposit(amount: Double) {
        balance += amount
    }
}

let account = BankAccount()
account.deposit(100)
print(account.balance)

let depositor = BankAccount.deposit
depositor(account)(100)
print(account.balance)
```

这样改变了selector的调用方式，现在开发者需要传入target和一个(T) -> () -> ()类型的action，而不采用之前的字符串输入，这当然保证了该方法一定会被响应，因为如果，该类型没有这个方法，在编译期间就会报错，而不会是在runtime中。

```objectivec
protocol TargetAction {
    func performAction()
}

struct TargetActionWrapper<T: AnyObject>: TargetAction {
    weak var target: T?
    let action: (T) -> () -> ()
    
    func performAction() -> () {
        if let t = target {
            action(t)()
        }
    }
}

enum ControlEvent {
    case TouchUpInside
    case ValueChanged
    //...
}

class Control {
    var actions = [ControlEvent: TargetAction]()
    
    func setTarget<T: AnyObject>(target: T, action: (T) -> () -> (), controlEvent: ControlEvent) {
        actions[controlEvent] = TargetActionWrapper(target: target, action: action)
    }
    
    func removeTargetForCotrolEvent(controlEvent: ControlEvent) {
        actions[controlEvent] = nil
    }
    
    func performActionForControlEvent(controlEvent: ControlEvent) {
        actions[controlEvent]?.performAction()
    }
}
```

```objectivec
class MyViewController {
    let button = Control()

    func viewDidLoad() {
        button.setTarget(self, action: MyViewController.onButtonTap, controlEvent: .TouchUpInside)
    }

    func onButtonTap() {
        println("Button was tapped")
    }
}
```

## 2.Struct Mutable的方法

这个问题，之前的[Swift by Tutorials](http://lucifer1988.github.io/blog/2015/10/15/swift-by-tutorials-generics/)也提到过，结构体的实例方法是不能修改自己声明的变量的，因为Struct是数值型的，默认是不可变的，所以如果某个方法需要修改结构体的变量，就需要添加**mutating**关键字，使得编译器可以对结构体进行copy-on-write操作。

```objectivec
struct User {
    var age: Int
    var weight: Int
    var height: Int
    mutating func gainWeight(newWeight: Int) {
        weight += newWeight
    }
}
```

## 3.将protocol的方法声明为mutating

这个问题是承接上一个问题的，Swift中的protocol是可以被class、struct、enum实现的，所以如果protocol中一些方法正好会修改到自身变量，但没有加**mutating**关键字的话，struct和enum实现该方法时就无法编过，所以一定要在设计protocol时考虑这个问题。另外，我补充一点，如果这个protocol只是给class实现的，那么可以不加mutating，同时也可以使用**protocol ClassOnlyProtocol: class**这样的写法，告诉使用者，该协议只能实现于class。

```objectivec
import UIKit

protocol Vehicle {
    var numberOfWheels: Int {get}
    var color: UIColor {get set}
    
    mutating func changeColor()
}

struct MyCar: Vehicle {
    let numberOfWheels = 4
    var color = UIColor.blueColor()
    
    mutating func changeColor() {
        color = UIColor.redColor()
    }
}
```

## 4.Sequence

在Swift中，只要实现了SequenceType，就可以使用for...in快速遍历，而需要先实现一个GeneratorType来控制遍历的顺序。

```objectivec
class ReverseGenerator: GeneratorType {
    typealias Element = Int
    
    var counter: Element
    init<T>(array: [T]) {
        self.counter = array.count - 1
    }
    
    init(start: Int) {
        self.counter = start
    }
    
    func next() -> Element? {
        return self.counter < 0 ? nil : counter--
    }
}

struct ReverseSequence<T>: SequenceType {
    var array: [T]
    
    init (array: [T]) {
        self.array = array
    }
    
    typealias Generator = ReverseGenerator
    
    func generate() -> Generator {
        return ReverseGenerator(array: array)
    }
}

let arr = [0, 1, 2, 3, 4]

for i in ReverseSequence(array: arr) {
    print("Index \(i) is \(arr[i])")
}
```

for...in如何利用这两个协议，作者给出了一个解释。

```objectivec
var g = array.generate()
while let obj = g.next() {
	print(obj)
}
```

而且你也可以免费使用map，filter，reduce这些函数，因为SequenceType的几个extension实现了他们。关于这几个函数的使用，我们在Swift by Tutorials做了很多介绍。

## 5.多元组(Tuple)

Tuple我们之前也介绍过，主要用于简单场景的复合数据类型，作者举了一个CGRect的方法在OC和Swift中的实践。

CGRect有个函数**CGRectDivide()**，用来切割矩形区域，由于会返回两个新的CGRect，而OC又没有返回多个结果的机制，只能先声明两个CGRect，然后将指针传入函数，等待函数对其赋值。

```objectivec
//CGRectDivide(CGRect rect, CGRect *slice, CGRect *remainder, CGFloat amount, CGRectEdge edge)
CGRect rect = CGRectMake(0, 0, 100, 100);
CGRect small;
CGRect large;
CGRectDivide(rect, &small, &large, 20, CGRectMinXEdge);
```

而在Swift中由于有了元组，可以优雅的进行实现，同时，由于Swift中结构体可以添加方法，所以将原来的函数直接作为了CGRect的一个方法，使得实现更加简洁。

```objectivec
//func rectsByDividing(atDistance: CGFloat, fromEdge: CGRectEdge) -> (slice: CGRect, remainder: CGRect)
let rect = CGRectMake(0, 0, 100, 100)
let (small, large) = rect.rectsByDividing(20, fromEdge: .MinXEdge)
```

## 6.@autoclosure和??

@autoclosure是简化代码的一个关键字，之前说过方法是Swift中的第一类对象，所以会在Swift中出现大量的匿名方法，即闭包closure作为参数，如下例。

```objectivec
func logIfTrue(@autoclosure predicate:() -> Bool) {
    if predicate() {
        print("True")
    }
}
logIfTrue({return 2>1})
```

当然按照之前介绍的简化策略，上面的调用可以简化为：

```objectivec
logIfTrue{2>1}
```

然而我们可以在方法定义时加上@autoclosure，这样调用时，Swift会自动将输入的参数转化为closure，但需要注意的是使用@autoclosure关键字时，闭包的参数只能是()，如果包含参数时，不能使用该关键字。

```objectivec
func logIfTrue(@autoclosure predicate:() -> Bool) {
    if predicate() {
        print("True")
    }
}
logIfTrue(2>1)
```

??可以用来对Optional类型进行空值判断，并赋默认值，用法如下：

```objectivec
var level: Int?
var startLevel = 1
var currentLevel = level ?? startLevel
```

??的具体实现如下，两个版本的区别是默认值是否是optional类型，但是需要注意的是输入的第二个参数被封装成了一个闭包的类型，作者猜测了??方法的具体实现。

```objectivec
@warn_unused_result
public func ??<T>(optional: T?, @autoclosure defaultValue: () throws -> T?) rethrows -> T?
@warn_unused_result
public func ??<T>(optional: T?, @autoclosure defaultValue: () throws -> T) rethrows -> T
```

```objectivec
func ??<T>(optional: T?, @autoclosure defaultValue: () -> T) -> T {
    switch optional {
        case .Some(let value):
            return value
        case .None:
            return defaultValue()
    }
}
```

这里没有直接用T类型作为默认值的参数，而是封转了一个方法，这是一种优化手段。这样，系统在开始判断optional是否有值之前就不需要开辟默认值的内存，只有在optional为nil时，才去准备默认值。但是在开发者调用??方法时又不需要输入一个闭包参数，而仍是一个数值，这就是巧妙的利用了@autoclosure关键字。

作者留出了关于Swift的||和&&的实现，其实他们的情况也和??一样，需要分布判断，下面是我给出的答案。

```objectivec
func ||<T : BooleanType, U : BooleanType>(lhs: T, @autoclosure rhs: () -> U) -> Bool {
    switch lhs.boolValue {
    case true:
        return true
    case false:
        return rhs().boolValue
    }
}
```

```objectivec
func &&<T : BooleanType, U : BooleanType>(lhs: T, @autoclosure rhs: () -> U) -> Bool {
    switch lhs.boolValue {
    case false:
        return false
    case true:
        return rhs().boolValue
    }
}
```






