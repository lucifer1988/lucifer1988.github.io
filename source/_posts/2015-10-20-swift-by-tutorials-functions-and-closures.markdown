---
layout: post
title: "Swift by Tutorials--Functions and Closures"
date: 2015-10-20 17:28:15 +0800
comments: true
categories: iOS
published: true
---

函数是现代编程的一个重要部分，它将执行一个特定任务的逻辑打包到一个单元，可以复用，也可以提供给其他开发者作为黑盒接入使用。Swift支持全局的函数和类以及结构体的方法，还支持闭包，可以当做对象来传递。在这一章，会深入介绍Swift的函数，包括语法、类型及参数，还有Swift的命名习惯如何受OC的影响。最后，还有巧妙和灵活的闭包，这也是Swift作为一个函数式语言的重要原因。

<!--more-->

## Functions

### Your first functon

1.定义一个全局函数，Swift的函数全部是全局的，而方法是被定义在类或结构体中的，这与其他绝大多数语言室相同的。  

```objectivec
func square(number: Double) -> Double {
	return number * number
}
```

### Functions are first-class

1.函数在Swift中是一级对象，也就意味着它可以赋值，可以作为函数参数，也可以作为函数结果返回，这也是第七章要讲的函数式编程的基础。  
2.从前面几章得知，Swift是非常强调类型的安全性，那么函数的类型是什么呢？  

```objectivec
let operation = square
```

```objectivec
let operation:(Double) -> Double = square
```

3.面对一些函数类型中嵌套过于复杂，可以考虑使用typealias来梳理代码。  

```objectivec
func doMath(operation:(Double) -> Double) -> Double {...}
```

```objectivec
typealias OperationType = (Double) -> Double
func doMath(operation:OperationType) -> Double {...}
```

### Function syntax

1.除了一些常见的参数和返回结果这些必须条件，Swift函数无返回值时，**Void**并不是一个关键字，而是一个类型。  

```objectivec
func logDouble(number: Double) -> Void {
	print(String(format: "%.2f", number))
}
```

```objectivec
typealias Void = ()
```

2.所以你可以将返回空的函数返回值写为**()**，其实也可以不写返回符号和类型，但是函数类型没变，还是**(Double) -> ()**。  

```objectivec
func logDouble(number: Double) -> () {
	print(String(format: "%.2f", number))
}
```

```objectivec
func logDouble(number: Double) {
	print(String(format: "%.2f", number))
}
```

### Overloading and generics

1.Swift是支持重载的，如下两个方法是可以共存的。  

```objectivec
func checkAreEqual(value: Int, expected: Int, message: String) {
    if expected != value {
        print(message)
    }
}
func checkAreEqual(value: String, expected: String, message: String) {
    if expected != value {
        print(message)
    }
}
```

2.这里复习下重载（overload）和重写（override）的区别，overload是指在同一类中同样方法名，但参数与返回结果可以不同；而重写是指子类覆盖父类的方法，方法名、参数、返回都相同，实现不同，重写Overriding是父类与子类之间多态性的一种表现，重载Overloading是一个类中多态性的一种表现。  
3.但是其实更好的解决方案是泛型，上一章也介绍过。  

```objectivec
func checkAreEqual<T: Equatable> (value: T, expected: T, message: String) {
    if value != expected {
        print(message)
    }
}
```

4.经测试会出现这一现象，这一现象出现的原因是checkAreEqual会对前两个参数类型进行检查，一是其类型必须遵循Equatable协议，二是两个参数类型必须相同，第一个出现error就是因为类型不同造成的，而后者成功是因为45并没有指明类型，所以Swift通过"cat"推测出T应该是String类型，所以45也被推测为String类型，所以通过。

```objectivec
//error
checkAreEqual(Int(45), expected: "cat", message: "is not cat")
//success
checkAreEqual(45, expected: "cat", message: "is not cat")
```

### In-out variables

1.开发中有些情况是需要我们改变参数值的，比如下面的square方法  

```objectivec
func square(number: Double) {
    number = number*number
}
```

2.但是这么写会直接报错，因为Swift中传入方法的参数类型默认都是let常量，是不可改变的，所以我们先将number改为var类型。  

```objectivec
func square(var number: Double) {
    number = number*number
}
```

3.这次编译通过了，但是结果测试不对，仍输出原值，其实这里Swift与其他主流语言（OC、Java、C#、JS）一样，虽然表面上允许参数值修改，但是其实修改的是参数的本地copy，并不影响原值。  
4.但是Swift还是支持直接修改原参数的，通过添加inout关键字，就可以将参数的指针地址直接传入，从而修改参数，如下：  

```objectivec
func square(inout number: Double) {
    number = number*number
}
var a = 5.0
square(&a)
print(a)
```

5.但是作者还是推荐谨慎使用，尤其在调用者对该技术不了解时，可能会造成困惑。  

### Classes and structures as function parameters

1.这一节讨论下类和结构体作为函数参数时的区别，首先看下Class作为参数的情况。  

```objectivec
class Person {
    var age = 34, name = "Colin"
    func growOlder() {
        self.age++
    }
}
func celebrateBirthday(person: Person) {
    print("Happy Birthday \(person.name)")
    person.growOlder()
}
let person = Person()
celebrateBirthday(person)
print(person.age)
```

2.结果person成功完成了age的增加，因为函数参数person和声明的实例person都是常量，并引用了相同的实例，当函数将class作为参数类型时，swift会将该类的实例的指针传入函数。  
3.再看下结构体的实现，显然结构体和上一节的基本数据类型是一样的，仍需要声明参数为inout，而且在修改自身变量时需要在方法前加mutating关键字。    

```objectivec
struct Person {
    var age = 34, name = "Colin"
    mutating func growOlder() {
        self.age++
    }
}
func celebrateBirthday(inout person: Person) {
    print("Happy Birthday \(person.name)")
    person.growOlder()
}
var person = Person()
celebrateBirthday(&person)
print(person.age)
```

4.另外由于array和dictionary都是struct，所以需要在函数中，他们作为参数并允许被修改时，也需要inout关键字。  

### Variadics

1.这一节来讨论有变长参数的函数类型，下面就是一个用例：  

```objectivec
func longestWord(words: String...) -> String? {
    var currentLongest: String?
    for word in words {
        if currentLongest != nil {
            if word.characters.count > currentLongest!.characters.count {
                currentLongest = word
            }
        } else {
            currentLongest = word
        }
    }
    return currentLongest
}
let long = longestWord("chick", "fish", "cat", "elephant")
print(long)
```

2.words相当于一个常量数组，在函数中可以直接使用for-in来遍历其成员，可变长参数可能并不常用，但是在特定场景下还是一种比较优雅的实现方式。  
3.这里补充一点，计算String长度的函数已经从**countElements(word)【Swift1.0】** -> **count(word)【Swift1.2】** -> **word.characters.count【Swift2.0】**，经历了这一变化。  
4.如果结合第七章要讲的函数式编程，其实该方法可以用reduce函数来实现，如下：  

```objectivec
func longestWord(words: String...) -> String? {
    return words.reduce(String?()) {
    (longest, word) in
    longest == nil || word.characters.count > longest!.characters.count
        ? word : longest
    }
}
```

### External parameter names

1.有时一些参数的作用表示的不是很明确，Swift不像OC，习惯在每个参数前写一段方法名，比如上述的比较字符串输入验证，光从调用来看无法得知那个是需要的名称。  

```objectivec
checkAreEqual("cat", "dog", "Incorrect input")
```
2.为了解决这一问题，Swift允许开发者添加供外部使用的参数名，可以和内部的参数名共存，内部外部各自使用不同的参数名：  

```objectivec
func checkAreEqual(value val: String, expected exp: String, message msg: String) {
    if exp != val {
        print(msg)
    }
}
checkAreEqual(value: "cat", expected: "dog", message: "Incorrect input")
```

3.但是需要注意的是，即使是使用了另外一套参数名，调用的顺序还是要和原来一致，像下面的调用是不会成功的。  

```objectivec
checkAreEqual(expected: "dog", value: "cat", message: "Incorrect input")
```

4.如果你的内外部参数名一致，而且也确实需要外部参数名，可以在参数名前加#来使内外部参数名相同。  

```objectivec
func checkAreEqual(#value: String, #expected: String, #message: String) {	if expected != value { 
		println(message)
	} 
}checkAreEqual(value: "dog", expected: "cat", message:"Incorrect input")
```

2.当然这是一些场景下的问题，很多方法其实从函数名即可推断出参数的意义，而有些方法则是必须加参数名的，这一决定由具体情况来决定，**大部分需要添加的属于有多个相同类型的参数**。  

```objectivec
//purpose is clear
dateFromString("2014-03-14")
//not clear, which is row? which is colume?
convertCellAt(42, 13)
//to be clear
convertCellAt(column: 42, row: 13)
```

## Methods

与函数相对的是方法，方法是与类、结构体、枚举关联的函数，本章会对各类方法进行讨论。  

### Instance methods

1.下面是一个名叫cell的类，其中包含单个参数和多个参数的方法。    

```objectivec
class Cell: CustomStringConvertible {
    private(set) var row = 0
    private(set) var column = 0
    func move(x: Int = 0, y: Int = 0) {
        row += y
        column += x
    }
    func moveByX(x: Int) {
        column += x
    }
    func moveByY(y: Int) {
        row += y
    }
    var description: String {
        get {
            return "Cell [row=\(row), col=\(column)]"
        }
    }
}
```

2.调用单个参数方法与函数没有区别，而调用多参数时需要在第二个参数后加上外部变量名，这与函数是不同的。其原因是函数默认是不添加外部参数名的，而方法除了第一个参数，后面的参数都是默认添加外部参数名，且与内部参数名一致，这一点也是从OC的命名风格继承而来。  

```objectivec
let cell = Cell()
cell.moveByX(4)
cell.move(4, y: 7)
```

3.你可以通过在参数前加** _ **来禁止这一默认添加外部参数名的行为，如下：  

```objectivec
func move(x: Int, _ y: Int) { 
	row += y	column += x 
}
cell.move(4, 7)
```

4.还有一个酷的特性是可以为参数赋默认值，这样调用时可以只输入单个参数，如下：  

```objectivec
func move(x: Int = 0, y: Int = 0) {
    row += y
    column += x
}
cell.move(4, y: 7)
cell.move(2)
cell.move(y: 3)
```

### Methods are first-class,too

1.与function类似，Method也是第一类对象，同样可以作为参数、返回结果、或则赋值给其他变量。  

```objectivec
var cell = Cell()var instanceFunc = cell.moveByY 
instanceFunc(34)
```

2.除此之外，还有一个函数式编程的特性，直接通过类获取方法，在调用时需要绑定一个类的实例，这被称为柯里化函数，会在第七章的函数式编程介绍。    

```objectivec
var cell = Cell()var moveFunc = Cell.moveByY 
moveFunc(cell)(34)
```

## Closures

闭包如同函数和方法一样，是一组代码，可以被调用或传递，但是闭包是匿名的，而且它会将它范围内的值进行持有，这是它的特性。  

### Closure expressions as anonymous function

1.

