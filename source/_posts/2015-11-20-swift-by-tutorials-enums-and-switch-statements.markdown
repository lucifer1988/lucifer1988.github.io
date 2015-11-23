---
layout: post
title: "Swift by Tutorials--Enums and Switch Statements"
date: 2015-11-20 10:11:34 +0800
comments: true
categories: iOS
published: true
---

enum枚举类型是很多编程语言的基本特性，一般是用来存储一组表示不同type的值，比如UILabel使用的NSTextAlignment，会有.Center，.Left多个type。而在Swift中，enum除了原始的用法，它的用法更像class或struct，enum可以拥有自己的方法，甚至是构造方法，然后通过配合Switch，可以实现更灵活的代码流控制，所以才会专门拿出来讲解，足见enum+switch在Swift的重要性。

<!--more-->

## Basic enumerations

先看一个Swift中定义的简单enum，如下：

```objectivec
enum Shape {
    case Rectangle
    case Square
    case Triangle
    case Circle
}
```

而使用时，可以按照下面两种方式赋值，如果事先定义了类型，那么可以不用在后面写类型，这还是Swift type inference的表现。

```objectivec
var aShape = Shape.Triangle
var bShape: Shape = .Triangle
```

而在之后的再次赋值中，都不用再指定类型。

```objectivec
aShape = .Square
```

### Raw values

在OC中，我们一般使用NS_ENUM()来定义枚举，并可以指定值的类型，也可以单独指定某个type的值。而在Swift中，也可以做到，而且更简单。

```objectivec
enum Shape: Int {
    case Rectangle
    case Square
    case Triangle
    case Circle
}
```

使用上述代码就指定了enum的原始值类型为Int，同时该enum获得了一个property，**rawValue**，用于读取当前枚举值的raw value，和一个新的初始化方法，**init(rawValue:)**，用raw value来初始化enum。

```objectivec
var triangle = Shape.Triangle
triangle.rawValue
var square = Shape(rawValue: 1)
```

有一点需要注意，就是通过**init(rawValue:)**生成的enum值，是Optional类型的，因为可能会出现输入的raw value没有对应的enum，如下，会返回nil。

```objectivec
var notAShape = Shape(rawValue: 100)
```

enum可以直接给某个type赋值，然后其他未赋值的type会自动加1。

```objectivec
enum Shape: Int {
    case Rectangle = 10
    case Square
    case Triangle
    case Circle
}
```

此外，enum可以使用其他类型的raw value，比如Double，Float，String，但是需要为所有type指定对应的raw value，因为除了Int，其他类型的enum是不会对没赋值的enum自动加1的。

```objectivec
enum Shape: String {
    case Rectangle = "Rectangle"
    case Square = "Square"
    case Triangle = "Triangle"
    case Circle = "Circle"
}
```

## Switch statements

switch与enum的配合在很多语言都是常见的用法，Swift中一样可以，而且有很多实用的改进。

```objectivec
enum Shape {
    case Rectangle
    case Square
    case Triangle
    case Circle
}

var aShape = Shape.Rectangle

switch(aShape) {
case .Rectangle:
    print("a rectangle")
case .Square:
    print("a square")
default:
    print("other shape")
}
```

之前在控制流的章节我们介绍过Swift中的switch，它是需要所有case全部覆盖的，如果你实现了全部case的覆盖，则不用添加default，反之，一定要添加defaut处理。而且，你不需要在每个case后添加break，系统会自动为你添加。

```objectivec
switch(aShape) {
case .Rectangle, .Square:
    print("a quadrilateral")
case .Circle:
    print("a circle")
default:
    break
}
```

一个case处理多个值也很简单，只需要逗号隔开多个值即可，并不需要其他语言中，用不加break的多行处理。

## Associated values

Associated values是Swift与其他语言的enum类型最大的不同之处，你可以向每个type添加一个类似元组类型的附加值，来作为二级判断条件，下面是有附带值的enum声明，可以为参数添加参数名。

```objectivec
enum Shape {
    case Rectangle(width: Float, height: Float)
    case Square(side: Float)
    case Triangle(base: Float, height: Float)
    case Circle(radius: Float)
}
```

使用时，可以像类或结构体初始化一样赋值，但记住不能像两者一样用**.**来访问。

```objectivec
var rectangle = Shape.Rectangle(width: 5, height: 10)
```

你可以使用这些associated values的场景只能是在switch语句中，同时你还可以使用where关键字，为case添加二级条件。

```objectivec
switch (rectangle) {
case .Rectangle(let width, let height) where width <= 10:
    print("Narrow rectangle:\(width)*\(height)")
case .Rectangle(let width, let height):
    print("Wide rectangle:\(width)*\(height)")
default:
    print("other shape")
}
```

需要注意的是，判断条件越来越复杂，便会出现符合多个case的情况，但像前面所说，系统会自动在case后添加break，所以也只会执行最先符合的case，这需要注意。

## Eunms as types

Swift中的enum还有另外一个重要特性，就是可以添加方法，配合associated value，可以封转一些方便的方法，比如为上面的Shape添加一个计算面积的方法。

```objectivec
func area() -> Float {
    switch(self) {
    case .Rectangle(let width, let height):
        return width * height
    case .Square(let side):
        return side * side
    case .Triangle(let base, let height):
        return 0.5 * base * height
    case .Circle(let radius):
        return Float(M_PI) * powf(radius, 2)
    }
}
```

然后申明一个Shape的变量，就可以直接调用该方法计算面积了。

```objectivec
var circle = Shape.Circle(radius: 5)
circle.area()
```

除此之外，可以添加新的构建方法，这是Shape实例的构造方法，所以切记一定要给self赋值，否则self没有值是无法返回对象的。

```objectivec
init(_ rect: CGRect) {
    let width = Float(CGRectGetWidth(rect))
    let height = Float(CGRectGetHeight(rect))
    if width == height {
        self = Square(side: width)
    } else {
        self = Rectangle(width: width, height: height)
    }
}
```

```objectivec
var shape = Shape(CGRectMake(0, 0, 5, 10))
```

所以如下这个构建方法是不合法的，因为会存在无法给self赋值的情况。

```objectivec
init(_ string: String) {
    switch(string) {
    case "rectangle":
        self = Rectangle(width: 5, height: 10)
    case "square":
        self = Square(side: 5)
    case "triangle":
        self = Triangle(base: 5, height: 10)
    case "circle":
        self = Circle(radius: 5)
    default:
        break
    }
}
```

但是想通过这种思路来初始化Shape，也可以实现，就是构建一个static的工厂方法，这样就不存在一定要有值的限制了，可以返回Optional类型。

```objectivec
static func fromString(string: String) -> Shape? { 
switch(string) {
    case "rectangle":
        return Rectangle(width: 5, height: 10)
    case "square":
        return Square(side: 5)
    case "triangle":
        return Triangle(base: 5, height: 10)
    case "circle":
        return Circle(radius: 5)
    default:
        return nil
    }
}
```

对应的，使用时也要用if来拆包。

```objectivec
if let anotherShape = Shape.fromString("rectangle") {
    anotherShape.area()
}
```

### Optionals are enums

Swift的Optional类型是enum类型，下面是部分实现：

```objectivec
enum Optional<T> : NilLiteralConvertible { 
  case None  case Some(T)  init()  init(_ some: T)  static func convertFromNilLiteral() -> T? 
}
```

这里使用了泛型，并将该类型作为Some的associated value，可以使Optional持有任何类型的值。

同时enum可以实现协议，NilLiteralConvertible协议使你可以用nil来替代optional，编译器自动会将赋值nil时去调用**convertFromNilLiteral**，最终转化为.None。正是因为实现了这个协议，才可以对optional类型赋nil值，不然会报错。

## JSON parsing using enums

本节用解析JSON这一常见用例，来实践enum在Swift中的使用。

### Parsing JSON the hard way

这是常规的解析方法，看起来确实比较繁琐（但相比OC是还是简单多了）。

```objectivec
let json = "{\"success\":true,\"data\":{\"numbers\":[1,2,3,4,5],\"animal\":\"dog\"}}"

if let jsonData = (json as NSString).dataUsingEncoding(NSUTF8StringEncoding) {
  let parsed: AnyObject? = NSJSONSerialization.JSONObjectWithData(jsonData, options: NSJSONReadingOptions(0), error: nil)
  
  // Actual JSON parsing section
  if let parsed = parsed as? [String:AnyObject] {
    if let success = parsed["success"] as? NSNumber {
      if success.boolValue == true {
        if let data = parsed["data"] as? NSDictionary {
          if let numbers = data["numbers"] as? NSArray {
            print(numbers)
          }
          if let animal = data["animal"] as? NSString {
            print(animal)
          }
        }
      }
    }
  } 
}
```

### Introducing JSON.swift

这个文件之前在第四章解析JSON时用过，下面来分析下它的结构，首先是一个enum定义，因为JSON文件就是一些基本元素和dictionary或array的组合，所以使用enum和associated value来定义JSON的基本对象，这是这一解决方案的核心思想。

```objectivec
enum JSONValue {	case JSONObject([String:JSONValue]) 	case JSONArray([JSONValue])	case JSONString(String)	case JSONNumber(NSNumber)	case JSONBool(Bool)	case JSONNull}
```

接下来为了获取.JSONObject和.JSONArray，我们利用Swift中方便添加角标访问的方式，使用subscript技术。

```objectivec
subscript(i: Int) -> JSONValue? {
  get {
    switch self {
    case .JSONArray(let value):
      return value[i]
    default:
      return nil
    }
  }
}

subscript(key: String) -> JSONValue? {
  get {
    switch self {
    case .JSONObject(let value):
      return value[key]
    default:
      return nil
    }
  }
}
```

那么访问一般元素时呢？我们采用了computed properties来访问。

```objectivec
var object: [String:JSONValue]? {
  switch self {
  case .JSONObject(let value):
    return value
  default:
    return nil
  }
}

var array: [JSONValue]? {
  switch self {
  case .JSONArray(let value):
    return value
  default:
    return nil
  }
}

var string: String? {
  switch self {
  case .JSONString(let value):
    return value
  default:
    return nil
  }
}

var integer: Int? {
  switch self {
  case .JSONNumber(let value):
    return value.integerValue
  default:
    return nil
  }
}

var double: Double? {
  switch self {
  case .JSONNumber(let value):
    return value.doubleValue
  default:
    return nil
  }
}

var bool: Bool? {
  switch self {
  case .JSONBool(let value):
    return value
  case .JSONNumber(let value):
    return value.boolValue
  default:
    return nil
  }
}
```

最后将对象转化为JSONValue也需要一个方法，而且是递归调用的，

```objectivec
static func fromObject(object: AnyObject) -> JSONValue? {
  switch object {
  case let value as NSString:
    return JSONValue.JSONString(value)
  case let value as NSNumber:
    return JSONValue.JSONNumber(value)
  case let value as NSNull:
    return JSONValue.JSONNull
  case let value as NSDictionary:
    var jsonObject: [String:JSONValue] = [:]
    for (k: AnyObject, v: AnyObject) in value {
      if let k = k as? NSString {
        if let v = JSONValue.fromObject(v) {
          jsonObject[k] = v
        } else {
          return nil
        }
      }
    }
    return JSONValue.JSONObject(jsonObject)
  case let value as NSArray:
    var jsonArray: [JSONValue] = []
    for v in value {
      if let v = JSONValue.fromObject(v) {
        jsonArray.append(v)
      } else {
        return nil
      }
    }
    return JSONValue.JSONArray(jsonArray)
  default:
    return nil
  }
}
```

### Putting it into practice

现在让我们使用JSON.swift来完成JSON的解析，进过比较，现在只需要两层嵌套就完成了原来五层的嵌套解析，而且由于使用了Optional类型，也增加了安全性。所以在Swift中一定要注重利用enum这些新特性，它非常适用于可以预定义为一组不同的子类型的类型，就像JSON。

```objectivec
let json = "{\"success\":true,\"data\":{\"numbers\":[1,2,3,4,5],\"animal\":\"dog\"}}"
    
if let jsonData = (json as NSString).dataUsingEncoding(NSUTF8StringEncoding) {
  if let parsed: AnyObject = NSJSONSerialization.JSONObjectWithData(jsonData, options: NSJSONReadingOptions(0), error: nil) {
    if let jsonParsed = JSONValue.fromObject(parsed) {
      
      // Actual JSON parsing section
      if jsonParsed["success"]?.bool == true {
        if let numbers = jsonParsed["data"]?["numbers"]?.array {
          print(numbers)
        }
        if let animal = jsonParsed["data"]?["animal"]?.string {
          print(animal)
        }
      }
    }
  }
}
```










