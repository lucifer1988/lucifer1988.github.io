---
layout: post
title: "Swift by Tutorials--Swift Quick Reference"
date: 2015-11-26 17:09:12 +0800
comments: true
categories: iOS
published: true
---
这是Swift by Tutorials的最后一章，主要是提供一个Swift实践代码的快速查找手册，方便开发时快速查阅。

<!--more-->

## Language basics

```objectivec
// variables can be updated
var variableNumber: Int = 1
variableNumber = variableNumber + 1

// constants cannot be updated
let constNumber: Int = 1
// constNumber = constNumber + 1 <- error!

// inferred type
let inferredInt = 1

// Swift types
let int: Int = 20
let double: Double = 3.5
let float: Float = 4.5
let bool: Bool = false
let str: String = "Hello Swift!"

// explicit type conversion
let pi = 3.1415
let multiplier = 2
let twoPi = pi * Double(multiplier)
```

Tips:

* 尽量将数值定义为常量let，不仅会使代码更健壮，也可以获得更多的编译器优化
* Swift是强类型的语言，所以很多地方需要手动类型转换

## Basic control structures

```objectivec
for index in 1..<3 {
  // loops with index taking values 1,2
}

for index in 1...3 {
  // loops with index taking values 1,2,3
}

//http://stackoverflow.com/questions/32197250/using-stride-in-swift-2-0
for index in 10.stride(to: 20, by: 2) {
  // loops form 10 to 20 (inclusive) in steps of 2
}

var index = 0
while index < 5 {
  // loops 5 times
  index++
}

index = 0
repeat {
  // loops 5 times
  index++
} while index < 5

// if/else
let temperature = 45
if temperature > 60 {
  print("It's hot!")
} else if temperature > 40 {
  print("It's warm")
} else {
  print("It's chilly")
}
```

## Tuples

```objectivec
let tuple = (1, 3, 5)   // inferred type (Int, Int, Int)
let tuple2 = (1, 5.0)   // inferred type (Int, Double)
let tuple3: (Double, Double) = (5, 6)

// indexing a tuple
print(tuple.0)
print(tuple.1)
print(tuple.2)

// a tuple with named elements
let product = (id: 24, name: "Swift Book")
print(product.name)

// decomposing a tuple
let (x, y, z) = tuple
print("\(x), \(y), \(z)") // "1, 3, 5"
```

Tips：

* Tuples在快速创建简易的复合类型时非常实用，而大多数情况struct是更好的选择，它有更强的类型安全和更好的功能性

## Arrays

```objectivec
import Foundation

// empty array creation
let empty1 = [String]()
let empty2 = Array<String>()
let empty3: [String] = []

// a constant array with inferred type
let animals = ["cat", "dog", "chicken"]
// animals.append("cow")  <- error!
// animals[2] = "fish"    <- error!

// a variable array with explicit type
var mutableAnimals: [String] = ["cat", "dog", "chicken"]
mutableAnimals.append("cow")
mutableAnimals[2] = "fish"

// iteration
for animal in animals {
  print(animal)
}

// array API
animals[0]                      // "cat"
animals[1..<3]                  // "dog", "chicken"
animals.count                   // "3"
animals.contains("cat")        // true
mutableAnimals.removeAtIndex(0) // "cat"

// functional API
animals.map { $0.uppercaseString }   // "CAT", "DOG", ...
animals.filter { $0.hasPrefix("c") } // "cat", "chicken"
animals.sort { $0 < $1 }           // "cat", "chicken", ...

// bridged NSArray API
let nsArray = animals as NSArray
nsArray.objectAtIndex(2)        // "chicken"
```

Tips：

* Swift中的Array是数值类型

## Dictionaries

```objectivec
// empty dictionary creation
let empty1 = [Int:String]()
let empty2 = Dictionary<Int, String>()
let empty3: [Int:String] = [:]

// a constant dictionary with inferred type
let animals = [24 : "cat", 36 : "dog"]
// animals[88] = "fish"   <- error!

// a variable dictionary with explicit type
var mutableAnimals: [Int:String] = [24 : "cat", 36 : "dog"]
mutableAnimals[55] = "fish"
mutableAnimals[24] = "chicken"

// dictionary API
animals[24]                 // "cat"
animals[1]                  // nil
animals.count               // "2"
mutableAnimals.removeValueForKey(24) // "chicken"

// dictionary values are returned as optionals
animals[24]!.hasPrefix("c") // true

// iteration
for (key, value) in animals {
  print(key)
  print(value)
}
```

Tips：

* Swift中的Dictionary是数值类型

## Optionals

```objectivec
// an optional variable
var maybeString: String?   // defaults to nil
maybeString = nil          // can be assigned a nil value
maybeString = "fish"       // can be assigned a value

// unwrapping an optional
if let unwrappedString = maybeString {
  // unwrappedString is a String rather than an optional String
  print(unwrappedString.hasPrefix("f")) // "true"
} else {
  print("maybeString is nil")
}

// forced unwrapping - will fail at runtime if the optional is nil
if maybeString!.hasPrefix("f") {
  print("maybeString starts with 'f'")
}

// optional chaining, returning an optional with the 
// result of hasPrefix, which is then unwrapped
if let hasPrefix = maybeString?.hasPrefix("f") {
  if hasPrefix {
    print("maybeString starts with 'f'")
  }
}

// null coalesce
var anOptional: Int?
var coalesced = anOptional ?? 3 // anOptional is nil, coalesced to 3
```

## Implicitly unwrapped optionals

```objectivec
// an implicitely unwrapped optional variable
var maybeString: String!
maybeString = nil
maybeString = "fish"

// methods invoked directly, failing at runtime if the optional is nil
if maybeString.hasPrefix("f") {
  print("maybeString starts with 'f'")
}else {
  print("maybeString does not start with an 'f'")
}
```

Tips：

* 隐式拆解的optional一般的使用场景是，类和结构体中初始化时不会给一个property赋值，而在稍后在使用前会进行赋值
* 虽然使用上不需要再去拆解，但是它仍是optional类型，所以依然需要注意它为nil时的调用

## Switch

```objectivec
let bit = Bit.Zero

// a simple switch statement on an enum
switch bit {
case .Zero:
  print("zero")
case .One:
  print("one")
}

// interval matching
let time = 45
switch time {
case 0..<60:
  print("A few seconds ago")
case 60..<(60 * 4):
  print("A few minutes ago")
default:                 // default required in order
  print("Ages ago!")   // to be exhaustive
}

// tuples and value bindings
let boardLocation = (2, 5)
switch boardLocation {
case (3, 4), (3, 3), (4, 3), (4, 4):
  print("central location")
case (let x, let y):
  print("(\(x), \(y) is not in the center")
}
```

Tips：

* Swift中的switch需要覆盖所有的case

## Enums

```objectivec
// enum declaration
enum Direction {
  case North, South, East, West
}

// assignment
var direction = Direction.North
direction = .South   // enum type inferred

// switching on enums
switch direction {
case .North:
  print("Going North")
default:
  print("Going someplace else!")
}

// advanced enumerations - using generics
enum Result<T> {
  case Failure
  // enumeration member with associated value
  case Success(T)
}

// creating an instance - where the type T is an Int
var result = Result.Success(22)

// switching and extracting the associated value
switch result {
case .Failure:
  print("Operation failed")
case .Success(let value):
  print("Operation returned value \(value)")
}
```

## Functions

```objectivec
// a simple function
func voidFunc(message: String) {
  print(message);
}
voidFunc("Hello Swift!")

// a function that returns a value
func multiply(arg1: Double, arg2: Double) -> Double {
  return arg1 * arg2
}
let result = multiply(20.0, arg2: 35.2)

// external and default parameters names
func multiplyTwo(first first: Double, andSecond second: Double) -> Double {
  return first * second
}
let result2 = multiplyTwo(first: 20.0, andSecond: 35.2)

// inout parameters
func square(inout number: Double) {
  number *= number
}
var number = 4.0
square(&number) // number = 16

// function types
let myFunc: (Double, Double) -> Double = multiplyTwo

// a generic function
func areEqual<T: Equatable>(op1: T, op2: T) -> Bool {
  return op1 == op2
}
```

## Closures

```objectivec
let animals = ["fish", "cat" , "chicken", "dog"]

animals.sort({
  (one: String, two: String) -> Bool in
  return one > two
})

animals.sort({
  (one, two) -> Bool in // inferred argument types
  return one > two
})

animals.sort({
  (one, two) in        // inferred return type
  return one > two
})

animals.sort({
  // no brackets around parameters
  one, two in return one > two
})

animals.sort({
  // no return keyword
  one, two in one > two
})

// shorthand arguments
animals.sort({ $0 > $1 })

// trailing closure
animals.sort() { $0 > $1 }
animals.sort { $0 > $1 }
animals.sort(>)
```

## Classes and protocols

```objectivec
public class BaseClass {
  private let id: Int            // private constant property
  
  init(id: Int) {
    self.id = id
  }
    
    class func numberOfLegs() ->Int {
    return 4;
    }
}

protocol NamedType {
  var name: String { get }       // a property with a getter
}

public class Animal: BaseClass, NamedType {
  
  private(set) var name: String  // variable with public getter
                                 // and private setter
  var size: Double = 45.0        // implicit internal property
  public let fullName: String    // public constant property
  
  init (id: Int, name: String, fullName: String) {
    
    // all properties initialized before base init invoked
    self.name = name;
    self.fullName = fullName;
    
    // super initialiser invoked
    super.init(id: id)
    
    // methods on self can now be called
  }
}

// creating an instance
var animal = Animal(id: 24, name: "cat", fullName: "Felis catus")
```