---
layout: post
title: "Swift by Tutorials--Functions and Closures"
date: 2015-10-20 17:28:15 +0800
comments: true
categories: iOS
published: false
---

函数是现代编程的一个重要部分，它将执行一个特定任务的逻辑打包到一个单元，可以复用，也可以提供给其他开发者作为黑盒接入使用。Swift支持全局的函数和类以及结构体的方法，还支持闭包，可以当做对象来传递。在这一章，会深入介绍Swift的函数，包括语法、类型及参数，还有Swift的命名习惯如何受OC的影响。最后，还有巧妙和灵活的闭包，这也是Swift作为一个函数式语言的重要原因。

<!--more-->

## Functions

### Your first functon

1.定义一个全局函数，Swift的函数全部是全局的，而方法是被定义在类或结构体中的，这与其他绝大多数语言室相同的。  

```objectivec
func square(number: Double) -> Double {
	return number*number
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