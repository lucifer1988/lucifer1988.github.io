---
layout: post
title: "Swift by Tutorials--Language Basics II"
date: 2015-10-10 14:41:12 +0800
comments: true
categories: iOS
published: true
---

继上一篇后，本章将继续介绍Swift的基础知识，但是相比第一章会有所提升，包括了Optional类型对象的用法、Swift中的Collection类型用法以及与OC的Collection的不同之处。

<!--more-->

##Optionals

1.空指针是一个困扰着各类语言的常见问题，在Java中，调用了空指针会直接抛出异常，在OC中向nil指针发送消息会返回nil，也就是说空指针是安全的，但有很多时候你并不希望指针为空，一般会加判断对象是否为nil的断言判断，但在Swift中，针对这个问题，有了新的解决方案。  
2.Swift在对没有初始化赋值的变量使用时，会直接报错，而且像String类型也不能初始化直接赋nil值，这也保证了空指针不会出现。但是如果我们真的需要一个空值的变量怎么办呢？可以使用optional机制，它是用来指明一个变量是可能有值的，相当于给变量一个nil的默认值，这也是空指针的问题所在，它是一个合法的指针，但没有指向一个合法的对象。  

###Declaring optionals

1.使用optional很简单，如下，不赋值的话str默认为nil，在这里你可以把String?理解为一个不同于String的类型，所以能给String?直接赋值String类型实际上是Swift在内部进行了封装，Swift将String的值封装到了一个String?类型的实例中，然后再赋值给了str。  

```objectivec
//no assignment
var str: String?
//an assignment
var str: String? = "Swift by Tutorials!"
```

2.如果你现在对str使用uppercaseString方法，会报错，这也验证了上面所说String?已是另一个类型的说法，那么如何让str使用String的方法呢？如下即可，通过if语句对str进行解封，并将其赋值给一个let型的String，这就是optional和if在Swift中的经典配合，这么做的好处就是让开发者可以在必选确认指针不为空的时候强制去进行空指针的检查。  

```objectivec
if let unwrappedStr = str {
	print("Unwrapped! \(unwrappedStr.uppercaseString)")
} else {
	print("Was nil")
}
```

###Forced unwrapping

1.在你了解optional机制下，在一些optional中你确定有值的时候，你可以使用强制解封，如下：  

```objectivec
var str: String? = "Swift by Tutorials!"
print("Force unwrapped! \(str!.uppercaseString)")
```

2.但是需要注意的是，如果optional类型中的是nil值，那么会出现runtime error，所以使用强制解封，**一定要在你100%确定你的optional对象不是空值**。  

###Implicit unwrapping

1.你也可以不用!或者let来进行optional解封，使用以下方法，可以直接对变量使用方法，这看起来和没使用optional差不多，但是它在实质上和上述两种解封方法是一致的，只是语法不通而已，如果不去初始化赋值，那么你会得到和强制解封一个nil的optional的值一样的error。  

```objectivec
var str: String! = "Swift by Tutorials!"
str = str.lowercaseString
print(str)
```

2.你也可以通过if来检查隐式解封的optional值，但你会发现这和OC中的做法一样，只不过在OC中你拿nil作为一个false的判断条件，而在Swift中你将nil作为一个无值的状态来判断。  

```objectivec
if str != nil {
	str = str.lowercaseString
	print(str)
}
```

3.**最后注意，你要将隐式解封和强制解封一样重视，因为除了声明的地方，它和普通变量是一样的，这很容易忽视。**  

###Optional chaining

1.最后要介绍的是Optional chaining，这是上述三个解封方式之外的另一种optional来执行方法的方式，它的设计参照了OC中常用的delegate模式，即在optional类型变量执行方法时会先判断它是否为nil，不是nil的话直接执行，而如果是nil的话，则直接返回nil，其实和OC中对nil发送消息的处理是一样的。  

```objectivec
var maybeString: String? = "Swift by Tutorials!"
let uppercase = maybeString?.uppercaseString
```

2.由于在对象声明和方法执行时两次使用optional，所以形成了Optional chaining。  

<!--more-->

##Collection

1.任何语言都会有集合类型，OC中有NSArray、NSDictionary、NSSet，其中包含可变和不可变类型，而在Swift中只保留了Array和Dictionary两种类型。  

###Arrays

1.Swift的Array有着其他语言中共同的特性，如下：  

```objectivec
//initialize array
var array = [1, 2, 3, 4, 5]
print(array[2])
//add an element
array.append(6)
print(array)
```

2.Swift中你可以通过添加一个序列来扩展一个Array，比如上一节提到的Range。  

```objectivec
//add 7,8,9,10
//Swift2.0中将extend()改为了appendContentsOf()
//array.extend(7...10)
array.appendContentsOf(7...10)
```

3.在上述数组中试图添加一个String，会直接报错，这在OC中可能是很正常的需求，可以在一个数组中添加不同类型的对象，但在Swift中只能在一个数组中添加同一类型的对象，在上面的Array初始化中是使用了type interface，如果制定类型声明的话应该是*var array: Array<Int> = [1, 2, 3, 4, 5]*（会在第四章详细说明），不过更常见的写法是*var arrray: [Int] = [1, 2, 3, 4, 5]*，这是Apple的语法糖，用来简化语法。  

```objectivec
//Array Initializer
var array: Array<Int> = [1, 2, 3, 4, 5]
var arrray: [Int] = [1, 2, 3, 4, 5]
```

4.当然也可以让Array像NSArray那样工作，可以将类型声明为Array<Any>，但是仍然不推荐这么做，因为这样Swift的很多Array方法会因为类型不一而不能使用，而且也会失去Swift的提供的安全性保护。   

```objectivec
//add multiple type instance
var array: Array<Any> = []
array.append(6)
array.append("Swift By Tutorials!")
```

###Dictionaries

1.Swift的Dictionary与OC的NSDicionary大致相同，只是语法上略有变化，但需要注意的是，Dictionary也存在只能添加固定类型的键值对的情况，与上述的Array相同。  

```objectivec
var dictionary = [1: "Dog", 2: "Cat"]
//Another Initializer
//var dictionary: Dictionary<Int:String> = [1: "Dog", 2: "Cat"]
//var dictionary: [Int:String] = [1: "Dog", 2: "Cat"]
print(dictionary[1])
dictionary[3] = "Mouse"
print(dictionary)
dictionary[3] = nil
print(dictionary)
```

2.从Dictionary中通过key直接获取值时，该值是optional类型的，因为有可能是不存在该key对应的值的，所以推荐读取Dictionary时还是使用上一章中介绍的安全拆解的方法，这又是Swift强制开发者随时考虑安全问题的一个表现。  

```objectivec
if let value = dictionary[1] {
	print("Value is \(value)")
}
```

###Reference and copies

1.这一节讨论Dictionary和Array在Swift中与OC所不同的内存管理策略，如下，从结果发现，Swift中将一个Dictionary直接赋值给另外的变量或常量，都是执行copy操作的，即改变新变量，并不会影响原来的Dictionary。  

```objectivec
var dictionaryA = [1: 1, 2: 4, 3: 9, 4: 16]
var dictionaryB = dictionaryA
print(dictionaryA)
print(dictionaryB)
dictionaryB[4] = nil
print(dictionaryA)
print(dictionaryB)
```

2.那么关于Array呢？答案是一样的，Array也是会执行copy操作，这与OC中的NSDictionary和NSArray的指针赋值是完全不同的，所以单独强调一下。  

```objectivec
var arrayA = [1, 2, 3, 4, 5]
var arrayB = arrayA
print(arrayA)
print(arrayB)
arrayB.removeAtIndex(0)
print(arrayA)
print(arrayB)
```

###Constant collection

1.上面都是定义的var类型的Dictionary和Array，那么如果定义为let的话，Dictionary和Array是不能进行任何修改操作的（其实就是OC中的不可变类型）。  

```objectivec
let constantArray = [1, 2, 3, 4, 5]
//error
constantArray.append(6) 
constantArray.removeAtIndex(0)
```


