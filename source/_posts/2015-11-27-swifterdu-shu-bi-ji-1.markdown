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

## 7.Optional Chaining

Optional Chaining在之前也介绍过，可以摆脱Swift中不必要的有值判断，但使用时也需注意其返回值和执行方法的不确定性。

```objectivec
class Toy {
    let name: String
    init(name: String) {
        self.name = name
    }
}

class Pet {
    var toy: Toy?
}

class Child {
    var pet: Pet?
}

let xiaoming = Child()
let toyName = xiaoming.pet?.toy?.name
```
如上，虽然name定义为toy的String类型，非Optional，但由于Optional Chaining的使用，是可能提前返回nil的，所以最终toyName的类型，还是String？。

实际开发中，一般还是用Optional Binding+Optional Chaining的方式来取值。

```objectivec
if let toyname = xiaoming.pet?.toy?.name {
    print(toyname)
}
```

如果，我们给Toy再加一个扩展：

```objectivec
extension Toy {
    func play() {
        //...
    }
}
```

如果我们想将play()封装成一个以Child为参数的closure，那么就要注意closure的返回类型了。以下列出了错误和正确的写法。

```objectivec
//wrong
//let playClosure = {(child: Child) -> () in child.pet?.toy?.play()}
//right
let playClosure = {(child: Child) -> ()? in child.pet?.toy?.play()}
```

由于toy之前的值的不确定性，所以play()不一定可以正常执行，所以应该是()?类型，或者Void?类型，最后列出我们实际使用的场景，依然使用Optional Binding。

```objectivec
if let result: () = playClosure(xiaoming) {
    //...
}
```

## 8.操作符

Swift与OC的一大区别是支持操作符的重载，比如我们需要定义以下结构体的相加运算，我们可以重载加号操作符。

```objectivec
struct Vector2D {
    var x = 0.0
    var y = 0.0
}

func +(left: Vector2D, right: Vector2D) -> Vector2D {
    return Vector2D(x: left.x+right.x, y: left.y+right.y)
}

let v1 = Vector2D(x: 2.0, y: 3.0)
let v2 = Vector2D(x: 1.0, y: 4.0)
let v3 = v1+v2
print(v3)
```

类似的，我们也可以重载减号（-），负号（-）这样的运算符。

但是如果我们要定义一个全新的运算符，比如点积运算，单纯重载是无效的，因为该方法并不存在于Swift之中，需要我们先声明，如下：

```objectivec
infix operator +* {
    associativity none
    precedence 160
}
```

infix表示这是一个中位操作符，即前后都是输入，其他修饰子还包括prefix（一个输入，并操作符作用于输入之前，类似++i，!i）和postfix（一个输入，并操作符作用于输入之后，类似i++，i--）。

associativity定义的事结合律，就是计算顺序，比如常见的加法和减法就是left，我们这里定义的点积的结果是Double，不会再和其他点积结合，所以就是none。

precedence是运算的优先级，越高的优先级优先进行运算，Swift中乘除法是150，加减法是140，这里的点积是160，是优先于乘除法的。

之后就可以定义点积运算了。

```objectivec
func +* (left: Vector2D, right: Vector2D) -> Double {
    return left.x*right.x+left.y*right.y
}

let result = v1 +* v2
```

需要注意的是，重载的操作符都是函数，所以是全局的，也就是说多次重载会导致冲突，尤其是多个module同时重载时。所以应该尽量将其作为其他某个方法的简便写法，避免实现大量逻辑或提供独一无二的功能，因为这样即使出现冲突，使用者还可以通过方法名调用的方式来使用，总之，重载操作符的特性不要滥用，除非确实存在大量并全局的需求。

## 9.func的参数修饰

这一问题在之前也说过，一般我们声明Swift函数时都不会添加参数修饰符，如：

```objectivec
func incrementor(variable: Int) -> Int {
    return variable + 1
}
```

而很多人可能不会这么写，因为有++这样的运算符，一般都会这么写：

```objectivec
func incrementor(variable: Int) -> Int {
    return ++variable
}
```

然而直接报错，原因是Swift默认参数是let型的，所以++对自己会重新赋值，所以会报错，修改也很简单。修改参数为var就好。

```objectivec
func incrementor(var variable: Int) -> Int {
    return ++variable
}

var putIn = 3
let resultOut = incrementor(putIn)
print(resultOut)
print(putIn)
```

但在我们使用时会发现，返回值没错，但是作为输入的putIn并没有改变，这一原因是即使我们将参数声明为var，但Swift在使用之前还是会将其copy一份，然后对其copy修改，所以实际输入的变量并不会改变。

如果我们想对输入参数在方法内修改，就要使用inout关键字，这样我们就将参数的指针输入了方法，所以在调用时也需要传入参数的指针，其实这与OC中的block使用外部变量是一样的，默认就是值拷贝，无法修改原值，如需修改，在原值加_block关键字。

```objectivec
func incrementor(inout variable: Int) {
    ++variable
}

var putIn = 3
incrementor(&putIn)
print(putIn)
```

作者补充的一点是，参数修饰符是有传递限制性的，入股统一参数要跨层调用，需要保证同一参数的修饰统一，如下例：

```objectivec
func makeIncrementor(addNumber: Int) -> ((inout variable: Int) -> ()) {
    func incrementor(inout variable: Int) -> () {
        variable + addNumber
    }
    return incrementor
}
```

## 10.字面量转换

字面量就是指特定的数字，字符串或布尔值这样可以直接表明类型，并直接赋值的值。如下：

```objectivec
let aNumber = 3
let aString = "Hello"
let aBool = true
let anArray = [1, 2, 3]
let aDictionary = ["key1": "value1", "key2": "value2"]
```

Swift提供了一组协议，用来将字面量转换为特定类型，对于实现了字面量转换接口的类型，在字面量赋值时，可以直接通过赋值的方式，将其转化为对应类型。这些协议有：

* ArrayLiteralConvertible
* BooleanLiteralConvertible
* DictionaryLiteralConvertible
* FloatLiteralConvertible
* NilLiteralConvertible
* IntegerLiteralConvertible
* StringLiteralConvertible

下面举例BooleanLiteralConvertible的实现，BooleanLiteralConvertible的定义是：

```objectivec
protocol BooleanLiteralConvertible {
	typealias BooleanLiteralConvertible
	
	init(booleanLiteral value: BooleanLiteralType)
}
```

BooleanLiteralConvertible在Swift中已有定义：

```objectivec
typealias BooleanLiteralType = Bool
```

那么我们自己实现字面量转换时，就需要实现init方法就好了，这样我们就可以直接使用true和false对MyBool类型赋值：

```objectivec
enum MyBool: Int {
    case myTrue, myFalse
}

extension MyBool: BooleanLiteralConvertible {
    init(booleanLiteral value: Self.BooleanLiteralType) {
        self = value ? myTrue : myFalse
    }
}

let myTrue: MyBool = true
let myFalse: MyBool = false

myTrue.rawValue
myFalse.rawValue
```

BooleanLiteralConvertible是这些协议中最简单的了，如果是StringLiteralConvertible协议，还要实现另外两个方法，这两个方法我们一般开发中不会遇到，是对应的[字符簇和字符](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Strings/Articles/stringsClusters.html)的字面量转换。

* ExtendedGraphemeClusterLiteralConvertible
* UnicodeScalarLiteralConvertible

继续举例比如，我们声明一个Person类，想通过String赋值，来生成Person对象，那么可以这样实现。

```objectivec
class Person: StringLiteralConvertible {
    let name: String
    init(name value: String) {
        self.name = value
    }
    
    required init(stringLiteral value: String) {
        self.name = value
    }
    
    required init(extendedGraphemeClusterLiteral value: String) {
        self.name = value
    }
    
    required init(unicodeScalarLiteral value: String) {
        self.name = value
    }
}
```

下面三个init方法都加上了**required**关键字，这是由初始化方法的完备性需求所决定的，该类的子类都需要保证能实现类似的字面量转换，以确保类型安全。

上面多次调用了self.name = value，其实是不必要的，我们可以在之后的每个init中调用最初的init方法，当然这需要在其他init方法前加**convenience**关键字。

```objectivec
class Person: StringLiteralConvertible {
    let name: String
    init(name value: String) {
        self.name = value
    }
    
    required convenience init(stringLiteral value: String) {
        self.init(name: value)
    }
    
    required convenience init(extendedGraphemeClusterLiteral value: String) {
        self.init(name: value)
    }
    
    required convenience init(unicodeScalarLiteral value: String) {
        self.init(name: value)
    }
}

let p: Person = "xiaoming"
print(p.name)
```

Person例子中，没有像MyBool一样，使用extension来扩展，因为在extension中，我们无法定义required的初始化方法，也就是说我们无法为非final的class添加字面量转换。

综上，字面量转换是一个强大的功能，使用得当对代码简洁度有很大提高，但是对于使用者来讲，如果没有详细的文档支持，会造成很多困惑，所以它的最佳实践还有待开发者们继续挖掘。

## 11.下标

Swift中的Array和Dictionary是支持下标读写的，这一点与新版OC是一致的。

```objectivec
var arr = [1, 2, 3]
arr[2]
arr[2] = 4

var dict = ["cat":"meow", "goat":"mie"]
dict["cat"]
dict["cat"] = "miao"
```

而Swift比较先进的是，Swift是允许自定义下标的，比如作者给出的，使用subscript关键字为Array添加支持以[Int]为index来读取的extension。

```objectivec
extension Array {
    subscript(input: [Int]) -> ArraySlice<Element> {
        get {
            var result = ArraySlice<Element>()
            for i in input {
                assert(i < self.count, "index out of range")
                result.append(self[i])
            }
            return result
        }
        set {
            for (index, i) in input.enumerate() {
                assert(i < self.count, "index out of range")
                self[i] = newValue[index]
            }
        }
    }
}

var arr = [1, 2, 3, 4, 5]
arr[[0, 2, 3]]
arr[[0, 2, 3]] = [-1, -3, -4]
arr
```

## 12.方法嵌套

方法在Swift中成为了第一对象，可以作为参数，可以作为结果返回，而且可以在方法中定义子方法，这对内容过长的方法进行重构提供了新的思路。如果子方法只会被调用一次，那么它更适合被声明到父方法内部。

```objectivec
func appendQuery(var url: String, key: String, value: AnyObject) -> String {
    func appendQueryDictionary(var url: String, key: String, value: [String: AnyObject]) {
        //...
        return result
    }
    
    func appendQueryArray(var url: String, key: String, value: [AnyObject]) {
        //...
        return result
    }
    
    func appendQuerySingle(var url: String, key: String, value: AnyObject) {
        //...
        return result
    }
    
    if let dictionary = value as? [String: AnyObject] {
        return appendQueryDictionary(url, key: key, value: dictionary)
    } else if let array = value as? [AnyObject] {
        return appendQueryArray(url, key: key, value: array)
    } else {
        return appendQuerySingle(url, key: key, value: value)
    }
}
```

还有在方法模板中（就是在Swift By Tutorials中提到的[Partial application](http://lucifer1988.github.io/blog/2015/11/23/swift-by-tutorials-functional-programming/)），开发者一方面希望提供一个模板来让使用者定制方法，而同时不希望暴露太多的的实现细节。

```objectivec
func makeIncrementor(addNumber: Int) -> ((inout Int) -> Void {
    func incrementor(inout variable: Int) -> Void {
        variable += addNumber;
    }
    return incrementor;
}
```

## 13.命名空间

OC是没有命名空间的，开发时只能通过自己添加前缀来区分，但是还是有可能出现类文件相同的情况，而另一种情况是，一些第三方库本身引用了其他第三方库，而开发者同时使用这两个库时，就会出现冲突。









