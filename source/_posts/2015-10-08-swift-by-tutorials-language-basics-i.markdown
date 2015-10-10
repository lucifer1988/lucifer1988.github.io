---
layout: post
title: "Swift by Tutorials--Language Basics I"
date: 2015-10-08 14:26:37 +0800
comments: true
categories: iOS
published: true
---

Swift更新到2.0了，是时候来一波Swift的集中学习了，这次用的教材是raywenderlich出版的Swift by Tutorials，我手里的版本是2014年12月份的，可能有些在Swift2.0中发生了变化，我会尽量标注出来。开始第一章，介绍一些Swift的基本语法。

<!--more-->

##Variables,constants and strings

1.Playground是Xcode新加入的一种文件，实际上就是一个可以自动编译的swift文件，可以用来测试一些简单的代码，也能显示一些资源文件，下面是申明一个string类型，可以直接在playground运行。  

```objectivec
var greeting: String = "Hello"
//也可以不指名类型
//var greeting = "Hello"
//如对greeting赋值int型，会报错
//greeting = 9
print(greeting)
```

2.string可以不声明类型，即可运行，是源于Swift的type interface的特性，即通过赋值自动确定变量的类型，但是之后再对其赋值整形，则会报错。因为Swift是静态输入语言，编译期间会进行类型检查。**type interface是推荐使用的，可以使代码简洁，增加可读性**。  
3.Swift的string类型是可变的，而且改变方式不像NSMutableString那么复杂，直接使用+=方式即可。  

```objectivec
var greeting = "Hello"
greeting += "World"
print(greeting)
```

4.如果想声明不可变String，只需添加let关键字即可。在Swift中，控制内建类型的可变性是通过添加let或var关键字来实现的，这不同于OC，需要两种类型来实现。**你应该尽量使用不可变类型，这不仅使你的程序更加健壮，也会使编译器做更多优化，实际上let使用的场景是远远多于var的**。  

```objectivec
let greeting = "Hello"
```

5.let关键字不仅限于Swift内建类型，自定义的结构体和类也可以使用，但有些细微不同，这将在第三章中详细讲到。  
6.改变string也可以利用append()方法，这是Swift String自带的一些API，但是数量不多。幸运的是，Swift String与OC的NSString是可转换的，NSString的全部方法，String也可全部使用，但是还是推荐尽量使用String自带的API，比较简洁。  

```objectivec
//Swift String API
greeting.append(Character("!"))
//bridge to NSString
//logout Hello World
import Foundation

var greeting = "hello world".capitalizedString
print(greeting)
//NSString style append
import Foundation

var greeting = "hello".stringByAppendingString(" world")
print(greeting) 
```

7.Swift String是一个值类型，当它被赋值给变量、常量、或者作为参数传入方法时，它的值是被copy的，如下例，改变alternateGreeting不会影响greeting的值。  

```objectivec
var alternateGreeting = greeting
alternateGreeting += " and beyond!"
print(alternateGreeting)
print(greeting)
```

<!--more-->

##Semicolons(分号)

1. Swift中分号只有在同一行中添加多条代码时才强制使用，其他情况可以省略不写，**这又是Swift代码简洁的一大表现**。

<!--more-->

##Numeric types and conversion

1.这一节主要讲Swift的数值类型，下面创建了两个变量，Int类型的radius和Double类型的pi，Swift还有很多其他类型，如Int8、UInt16、Float等。除非你有特殊需求，那么Int和Double类型是你的首选，这两个类型有广泛的使用，而且编译器会自动选择Int的最佳长度，32或64，基于机器的字长，来生成最高效的代码。  

```objectivec
var radius = 4
let pi = 3.14159
```

2.在Swift中，你可以使用**_**来作为千分号，如下。  

```objectivec
let milion = 1_000_000
```

3.以下代码会报错，是因为*无法直接将Int和Double相乘，而Swift中也不会隐式转化，需要开发者显式转化，需要注意这里并不是类型转化，而是新生成了一个Double类型，在第三章中会详细讲解初始化方法。  

```objectivec
//error
var area = radius * radius * pi
//soluation
var area = Double(radius) * Double(radius) * pi
```

4.显式转化变量是Swift的安全策略之一，另外一个是越界检查，下列代码在其他语言中可能会生成一个负数，而在Swift中会直接将其视为一个runtime error。同时，为了避免integer的越界计算error，Apple提供了&+、&-、&*、&/、&%这些越界运算符，使用后会像常规计算一样，生成负数。  

```objectivec
var overflow = Int.max + 1
//overflow operators
var overflow = Int.max &+ 1
```

<!--more-->

##Booleans

1.Swift的布尔类型为Bool，值为true或false，需要说明的一点是，作为Swift的安全特性之一，控制流中需要进行布尔判断的只能使用Bool类型，而不同于在OC中，可以将任意非零值作为判断条件，例如你不能在Swift中使用一个整数当做判断条件。  

```objectivec
let alwaysTrue = true
```

<!--more-->

##Tuples(元组)

1.元组用来将多个值组成一个类型，但是不像类和结构体，你不需要显式的定义这个类型，如下，你可以通过索引来访问每个值，也可以通过索引来改变每个值（前提是你得元组的是可变的），而越界访问，编译器会报error。  

```objectivec
var address = (742, "Evergreen Terrace")
print(address.0)
print(address.1)
address.0 = 744
```

2.你也可以声明元组的类型，如下。如果想将元组的Int值类型改为Double有三种方式，同如下：  

```objectivec
var address: (Int, String) = (742, "Evergreen Terrace")
//change Int to Double
//1)using a type annotation
var address1: (Double, String) = (742, "Evergreen Terrace")
//2)by explicit creation of a Double
var address2 = (Double(742), "Evergreen Terrace")
//3)by using a double literal value
var address3 = (742.0, "Evergreen Terrace")
```

3.你也可以把元组解析成单个元素，而这也是一个copy操作，改变新值不会影响原始的元组值，如下：  

```objectivec
var address = (742, "Evergreen Terrace")
let (house, street) = address
print(house)
print(street)
```

4.此外，你可以为元组的每个值加一个key，提高可读性，同时上述的访问方法依然有效。  

```objectivec
var address = (number: 742, street: "Evergreen Terrace")
print(address.number)
print(address.street)
```

5.元组只是一个类型，也可以进行嵌套，元组作为其他元组的一个元素，类和结构体虽然包含了元组的所有功能，但是在一些轻量的场景下，元组可以更加快速简单地去构建一个复合类型。  

<!--more-->

##String interpolation

1.打印出一个类的信息，是常见的需求，例如OC中NSObject的description方法，如果我们想打印出上一节中元组的信息，我们可能这么做：  

```objectivec
var address = (742, "Evergreen Terrace")
let (house, street) = address
print("I live at " + String(house) + ", " + street)
```

2.这利用了Swift的+拼接字符串的技术，不过在Swift中有更加方便的字符串拼接技术来处理这一场景，string interpolation，如下：  

```objectivec
var address = (742, "Evergreen Terrace")
let (house, street) = address
print("I live at \(house), \(street)")
```

3.当然这不是打印日志专用的，在你需要从一系列变量中构建String时，都可使用。  

```objectivec
import Foundation

var address = (742, "Evergreen Terrace")
let (house, street) = address
let str = "I live at \(house+10), \(street.uppercaseString)" 
```

4.最后，如果想只打印出\时，请用\\\转义。  

<!--more-->

##Control flow

###For loops

1.首先要介绍的Swift中的for循环的新特性是closed range operator，即闭区间运算符，例子如下：   

```objectivec
let greeting = "Swift by Tutorials!"
for i in 1...5 {
	print("\(i) - \(greeting)")
}
```

2.注意，其中的i并不是一个var类型的变量，而是每次循环创建一个let的常量，是不可被赋值的。  
3.1...5是一个Range类型，与for循环并无关系，下例说明了这一问题：  

```objectivec
var range = 1...5
for i in range {
	print("\(i) - \(greeting)")
}
//what's 1..5?
var range = Range(start: 1, end: 6)
```

4.x...y只是Range类型的一个简化的语法糖，你可以用x..<y来创建半开半闭的区间，最后一个值为y-1。  

```objectivec
//means 1,2,3,4,5
var range1 = 1...5
//means 1,2,3,4
var range2 = 1..<5
```

5.那么还有个问题，for循环是怎么处理这个Range的？从而实现循环？其实是这样的，for循环可以处理很多可遍历的Swift类型，例如array、dictionary，还比如string也可以，所以range只是这些可遍历类型的其中之一而已。  

```objectivec
for i in "Swift" {
	print(i)
}
```

6.**再挖的深一点，为什么String和Range一样可以被遍历？去看一下它们的定义就可以得知，它们都遵循了Sequence protocol，通过实现协议中的方法，就可以得到generator，继而循环请求其中的值，在第四章我们将自己创建类型来实现这一协议和利用generator。**  

###While loops

1.Swift同时支持while循环和do-while循环(**Swift2.0将do-while改为了repeat-while**)。  

```objectivec
//while loop
let greeting = "Swift by Tutorials!"
var i = 0
while i < 5 {
	print("\(i) - \(greeting)")
	i++
}
//repeat-while loop
let greeting = "Swift by Tutorials!"
var i = 0
repeat {
	print("\(i) - \(greeting)")
	i++
} while i < 5
```

###If statements

1.Swift支持常规的if-else用法，需要注意的就是前面提到过的，if的条件必须是Bool类型，而且**还有一点就是即使只有一条执行的语句，也要用{}，这是Swift和其他语言if的一个区别，不然会报错，这也是为了避免{}误用或少写导致的bug**，此外，还有一个和if有关的关于optional value的技巧会在下一章讲到。  

```objectivec
import Foundation

let greeting = "Swift by Tutorials!"

for i in 1...5 {
	if i == 5 {
		print(greeting.uppercaseString)
	} else {
		print(greeting)
	}
}
```

###Switch statements

1.Swift支持常规的switch语句，与OC不同的是，Swift的switch条件可以使任意类型，而OC只是原始类型，编译器来确保每个case的条件与switch条件类型一致。  
2.第二点，Swift中的switch不需要在每个case后添加break，这是因为在Swift中，每个case执行完后，会自动跳出switch，所以不需要手动添加break，同时这也是安全策略之一。  

```objectivec
var direction = "up"switch direction { 
	case "down":		println("Going Down!") 	case "up":		println("Going Up!") 	default:		println("Going Nowhere") }
```
3.第三，Swift的switch相当智能，如果你提供的switch条件的可能值没有在case中被全部覆盖，会提示你添加default，如上例的String类型，不然会报error；而如果你的case覆盖了switch条件的所有值，如enum类型，那么不添加default也不会报错。  
4.如何在switch在一个case中匹配多个值，参照下例：  

```objectivec
var direction = "up"switch direction { 
	case "down", "up":		println("Going Somewhere!") 	default:		println("Going Nowhere") }
```

5.可以利用上一节介绍的Range来实现case的区间匹配：  

```objectivec
var score = 570

switch score {
	case 1..<10:
		print("novice")
	case 10..<100:
		print("proficient")
	case 100..<1000:
		print("rock-star")
	default:
		print("awesome")
}
```

6.另外switch与元组的结合，可以创造出更加复杂的场景处理：  

```objectivec
let somePoint = (1, 1)
switch somePoint {
	case (0, 0):
		print("(0, 0) is at the origin")
	case (_, 0):
		print("(\(somePoint.0), 0) is on the x-axis")
	case (0, _):
		print("(0,\(somePoint.1)) is on the y-axis")
	case (-2...2, -2...2):
		print("(\(somePoint.0), \(somePoint.1)) is inside the box")
	default:
		print("(\(somePoint.0), \(somePoint.1)) is outside the box")
}
```

7.利用value binding技术，可以在判断期间将tuple的元素赋值给let变量（当然也可以声明为var类型，且它的作用范围只存在该case中）：  

```objectivec
let anotherPoint = (2, 0)
switch anotherPoint {
	case (let x, 0):
		print("on the x-axis with an x value of \(x)")
	case (0, let y):
		print("on the y-axis with an y value of \(y)")
	case let (x, y):
		print("somewhere else at (\(x), \(y))")
}
```

8.switch可以添加where语句，为每个case添加额外的判断条件：  

```objectivec
let yetAnotherPoint = (1, -1)
switch yetAnotherPoint {
	case let (x, y) where x == y:
		print("(\(x), \(y)) is on the line x == y")
	case let (x, y) where x == -y:
		print("(\(x), \(y)) is on the line x == -y")
	case let(x, y):
		print("(\(x), \(y)) is just some arbitrary point")
}
```










