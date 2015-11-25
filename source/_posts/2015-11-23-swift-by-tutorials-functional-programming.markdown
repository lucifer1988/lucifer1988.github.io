---
layout: post
title: "Swift by Tutorials--Functional Programming"
date: 2015-11-23 14:09:04 +0800
comments: true
categories: iOS
published: true
---
前几篇分别介绍了泛型、类、枚举、范围运算符这些Swift的语言新特性，当然有一部分是对已有技术的改进，但这已经表明Swift是比OC更具表现力和更简洁的语言。而Swift不仅仅是提供了更好的语法，在Swift中，函数式编程会成为编程中可行的并且非常重要的编程工具。函数式编程，简而言之，就是一种通过数学式的函数概括计算的编程范式，不可变且具表现力，同时使用最少的变量和状态值。函数式编程对于测试是非常友好的。随着多核设备的普及，并行和并发处理显得非常重要，而函数式编程就是可以很好处理并行和并发处理的，这也是它日趋重要的原因之一。

<!--more-->

## Simple array filtering

这个简单的例子要求我们在1到10之间找到所有的偶数的数字，看起来是一个简单的工作，但是是介绍函数式编程的一大步。

### Filtering the old way

```objectivec
var evens = [Int]()
for i in 1...10 {
    if i % 2 == 0 {
    evens.append(i)
    }
}
print(evens)
```

这是一个很简单的小程序，而且运行也没问题，但是最核心的逻辑，检测一个数是否偶数，被隐藏在了for循环中。另外，添加数字到数组的逻辑在判断条件中，如果你想要打印每个偶数，那么除了复制粘贴，没法儿复用这部分代码。

### Functional filtering

让我们来看看函数式的解决方案，其中filter是核心，通过传递isEven给filter，直接返回了我们需要的新数组。

```objectivec
func isEven(number: Int) -> Bool {
    return number%2 == 0
}
evens = Array(1...10).filter(isEven)
print(evens)
```

通过上一章我们知道函数只是有名字的闭包，按照之前的介绍，利用type inference我们可以对该方案进一步简化。

```objectivec
evens = Array(1...10).filter { number in number % 2 == 0 } 
println(evens)
```

或者最简化的使用参数简化符号。

```objectivec
evens = Array(1...10).filter{ $0%2 == 0 }
print(evens)
```

对于简化符号的使用，作者表示，对于比较简单的逻辑（如上例）推荐使用简化符号，而对于复杂的逻辑，则不推荐使用，因为即使减少了代码量，但牺牲了可读性，还是不太划算。

上面的函数式编程较上一节的方案，更为简洁，而同时它也反映了，函数式编程三个共同的特点。

1. Higher-order functons：你需要将这些函数当做参数传入其他函数，该例中isEven即是Higher-order functons。
2. First-class function：这也是我们多次强调的一点，函数作为第一类对象，可以作为参数或者返回结果，这也是函数式编程的基础，Swift全面支持这一点。
3. Closures：闭包，可以使代码更简洁，相当于匿名函数。

你可能觉得OC的block也具有类似的特点，但Swift较之更胜一筹，主要是因为它拥有很多内建的函数式语法，比如该例中的filter就是。

### The magic behind filter

通过自定义一个myFilter方法，让我们来看一下filter的背后实现。

```objectivec
func myFilter<T>(source: [T], predicate:(T) -> Bool) -> [T] {
    var result = [T]()
    for i in source {
        if predicate(i) {
            result.append(i)
        }
    }
    return result
}
```

从上面发现，实际上的实现和我们在第一节的逻辑是相同的，只不过通过泛型和函数作为参数完成了filter这一过程的抽象，这也是函数式编程的本质。

原文还提出一个将myFilter()作为Array的一个方法，而不是函数，其实只需要添加一个Array的扩展，将泛型改为Array自身的元素就行，下面给出自己的解答。

```objectivec
extension Array {
    func myFilter<T>(predicate:(T) -> Bool) -> [T] {
        var result = [T]()
        for i in self {
            if predicate(i as! T) {
                result.append(i as! T)
            }
        }
        return result
    }
}
evens = Array(1...10).myFilter{$0%2 == 0}
```

## Reducing

这一节将介绍Swift中内建的，比较复杂的reduce函数，进一步了解函数式编程。

### Manual reduction

reduce的效果是输入一个数组，但最终得到一个结果，例子是要求我们找出1到10的所有偶数，并计算出他们的和，先看下常规控制流的实现。

```objectivec
var evens = [Int]()
for i in 1...10 {
    if i % 2 == 0 {
        evens.append(i)
    }
}
var evenSum = 0
for i in evens {
    evenSum += i
}
print(evenSum)
```

### Functional reduce

下面是reduce的函数式实现，这里采用的是全简写符号。

```objectivec
let evenSum = Array(1...10).filter{$0%2 == 0}.reduce(0){$0+$1}
print(evenSum)
```

reduce的函数原型如下，有两个参数，initial和combine，initial为U类型，最终的返回值也是U类型，而combine的参数是U和T类型，返回值也是U类型，每次执行完后，返回值都会成为combine新的参数，所以实现了上述的累加效果。

```objectivec
func reduce<U>(initial: U, combine: (U, T) -> U) -> U
```

这里还有一个寻找数组中最大值的例子。

```objectivec
let maxNumber = Array(1...10).reduce(0){total, number in max(total, number)}
print(maxNumber)
```

我们发现，输入值T和最终返回值U可以是不同的类型，所以可以有更多的应用，比如下面的字符串输出。

```objectivec
let numbers = Array(1...10).reduce("numbers:"){$0+"\($1)"}
print(numbers)
```

这里又有一个附加问题，要求输入一个["3", "1", "4", "1"]数组，而输出Int值3141，我的思路是先拼接字符串，然后转为Int。

```objectivec
let digits = Int(["3", "1", "4", "1"].reduce(""){$0+$1})
print(digits)
```

### The magic behind reduce

下面我们自己为Array添加一个自定义的reduce方法，实现方法与我们在之前的实现是一致的，还是做了一步抽象。

```objectivec
extension Array {
    func myReduce<T, U>(seed:U, combiner:(U, T) -> U) -> U {
        var result = seed;
        for i in self {
            result = combiner(result, i as! T)
        }
        return result
    }
}
```

## Building an index

接下来，我们将实践一次函数式编程，题目是将下面的String数组，按照首字母进行分组。

```objectivec
let words = ["Cat", "Chicken", "fish", "Dog", "Mouse", "Guinea Pig", "monkey"]
```

首先现有一个大致的思路，建立一个元组，包含首字母和其对应的String数组，最后通过一个函数返回一个该元组的数组，然后就完成了任务。

```objectivec
typealias Entry = (Character, [String])

func buildIndex(words: [String]) -> [Entry] {
    return [Entry]()
}
```

### Building an index imperatively

下面是常规控制流实现，使用了两个for循环，一个用于记录所有的首字母key，第二个用于将原数组添加到对应的元组中。

```objectivec
func buildIndex(words: [String]) -> [Entry] {
  var result = [Entry]()
  var letters = [Character]()
  for word in words {
    let firstLetter = Character(word.substringToIndex(
      advance(word.startIndex, 1)).uppercaseString)
    if !contains(letters, firstLetter) {
      letters.append(firstLetter)
    }
  }
  for letter in letters {
    var wordsForLetter = [String]()
    for word in words {
      let firstLetter = Character(word.substringToIndex(
        advance(word.startIndex, 1)).uppercaseString)
      if firstLetter == letter {
        wordsForLetter.append(word)
      }
    }
    result.append((letter, wordsForLetter))
  }
  return result
}
```

### Building an index the functional way

下面是该问题的函数式解决方案，首先我们利用Array的map函数，得到words对应的首字母数组。

```objectivec
func buildIndex(words: [String]) -> [Entry] {
    let letters = words.map { word -> Character in
    Character(word.substringToIndex(word.startIndex.advancedBy(1)).uppercaseString)
    }
    return [Entry]()
}
```

map与前面的filter，reduce一样都是Array的内建函数，它的作用是输出一个与原数组对应的相关数组。而且数组元素可以与原数组不同，这里通过map得到了words对应的首字母数组，但是不足的是重复字母并没有过滤。我们可以像之前一样自定义一个过滤相同字母的函数。

```objectivec
func distinct<T: Equatable> (source: [T]) -> [T] {
    var unique = [T]()
    for item in source {
        if !unique.contains(item) {
            unique.append(item)
        }
    }
    return unique
}
```

利用这一函数，过滤重复字母。

```objectivec
func buildIndex(words: [String]) -> [Entry] {
    let letters = words.map { word -> Character in 
    Character(word.substringToIndex(word.startIndex.advancedBy(1)).uppercaseString)
    }
    let distinctLetters = distinct(letters)
    print(distinctLetters)
    return [Entry]()
}
```

最后，我们利用map和filter的嵌套使用，完成最终的结果数组。

```objectivec
func buildIndex(words: [String]) -> [Entry] {
    let letters = words.map { word -> Character in 
    Character(word.substringToIndex(word.startIndex.advancedBy(1)).uppercaseString)
    }
    let distinctLetters = distinct(letters)
    
    return distinctLetters.map { letter -> Entry in
        return (letter, words.filter { word -> Bool in 
        Character(word.substringToIndex(word.startIndex.advancedBy(1)).uppercaseString) == letter
            })
    }
}
```

在完成基础上我们可以进一步重构，我们在buildIndex()函数中声明了一个新的函数firstLetter()，该函数的作用范围只在外围函数中，这得益于Swift中function作为第一类对象，所以可以被视作本地变量，也有作用范围。

```objectivec
func buildIndex(words: [String]) -> [Entry] {
    func firstLetter(str: String) -> Character {
        return Character(str.substringToIndex(str.startIndex.advancedBy(1)).uppercaseString)
    }
    
    let letters = words.map { word -> Character in
        return firstLetter(word)
    }
    let distinctLetters = distinct(letters)
    print(distinctLetters)
    
    return distinctLetters.map { letter -> Entry in
        return (letter, words.filter { word -> Bool in
            firstLetter(word) == letter
            })
    }
}
```

然而，这还不是最简形式，追求最简形式，可以将所有函数连续调用，这是没问题的，以下是最终结果。

```objectivec
func buildIndex(words: [String]) -> [Entry] {
    func firstLetter(str: String) -> Character {
        return Character(str.substringToIndex(str.startIndex.advancedBy(1)).uppercaseString)
    }
    
    return distinct(words.map(firstLetter))
        .map {
          letter -> Entry in
          return (letter, words.filter {
            word -> Bool in
            firstLetter(word) == letter
          })
        }
}
```

通过比较，你会发现，在常规做法中，你会声明很多临时变量，而在函数式编程中，你会使用对应的常量替代，这就意味着是不可变的，而不可变的类型更容易测试和协助并发，函数式编程与不可变类型是一体的，而且会导致你的代码更简洁，错误更少。

这节的挑战任务是需要输出的结果按字母排序，我们可以添加之前介绍过的最简的sort()方法，在distinct()之后进行调用。

```objectivec
func buildIndex(words: [String]) -> [Entry] {
    func firstLetter(str: String) -> Character {
        return Character(str.substringToIndex(str.startIndex.advancedBy(1)).uppercaseString)
    }
    
    return distinct(words.map(firstLetter)).sort(<)
        .map {
          letter -> Entry in
          return (letter, words.filter {
            word -> Bool in
            firstLetter(word) == letter
          })
        }
}
```

## Partial application and currying

前面几节围绕Array的三个方法，map，reduce，filter来介绍函数式编程。这一节将进一步介绍函数式编程的Partial application和currying。这里有一篇[博客](http://segmentfault.com/a/1190000000765247)也进行了简单的介绍。

### Partial application

为了说明Partial application，先提出一个问题，切割字符串为数组，我们的常用解决方案就是就是直接调用NSString的componentsSeparatedByString()方法。

```objectivec
import Foundation
let data = "5,7;3,4;55,6"
print(data.componentsSeparatedByString(";"))
print(data.componentsSeparatedByString(","))
```

这么调用也没问题，但是如果有场景需要我们多次用分号，或者逗号切割字符串，这样处理会出现许多重复代码，那么partical application就是这类问题的解决方案。

```objectivec
func creatSplitter(separator: String) -> (String -> [String]) {
    func split(source: String) -> [String] {
        return source.componentsSeparatedByString(separator)
    }
    return split
}

let commaSplitter = creatSplitter(",")
print(commaSplitter(data))

let semiColonSplitter = creatSplitter(";")
print(semiColonSplitter(data))
```

我们创建了一个产生分割字符串方法的工厂方法，输入分隔符号，返回一个该符号的分割方法，也就是我们先实现了函数的一部分，将函数从二元降为了一元，最主要的是，我们可以反复使用这一得到的方法，可能在该例中优点体现不太明显，但是如果逻辑更复杂，参数更多后，partical application带来的效率提高就非常可观了。

### A mild curry

使用curry也可以实现上例中的结果，如下，但是调用和工厂方法的写法都会改变。

```objectivec
func createSplitter(separator: String)(source: String) -> [String] {
    return source.componentsSeparatedByString(separator)
}

let commaSplitter = createSplitter(",")
print(commaSplitter(source: data))

let semiColonSplitter = createSplitter(";")
print(semiColonSplitter(source: data))
```

curry实现了相同的目的，但是它创建的方法包含了两组“分开的”参数，而当你输入第一个参数时，会返回一个函数，你可以继续输入第二个参数（第二个参数需要使用外部参数名，与一般函数一致）。

### A hotter curry

让我们进一步了解curry，将下面的三元函数进行curry化。

```objectivec
func addNumbers(one:Int, two:Int, three:Int) -> Int {
    return one + two + three
}
let sum = addNumbers(2, two: 5, three: 4)
print(sum)
```

```objectivec
func curryAddNumbers(one:Int)(two:Int)(three:Int) -> Int {
    return one+two+three
}
```

接着让我们分部对curry函数进行调用，每一步都会返回一个函数，最后一步返回最终结果。

```objectivec
let stepOne = curryAddNumbers(2)
let stepTwo = stepOne(two: 5)
let result = stepTwo(three: 4)
```

也可以像一般函数一样，一次性直接调用。

```objectivec
let result2 = curryAddNumbers(2)(two: 5)(three: 4)
```

当然，也可以在每一步添加多个参数。

```objectivec
func curryAddNumbers2(one:Int, two: Int)(three: Int) -> Int {
    return one+two+three
}
let result3 = curryAddNumbers2(2, two: 5)(three: 4)
```

### Practical curring

上面两例主要为了说明curry的使用，这里看下它在实际开发中的用法。

```objectivec
let text = "Swift"let paddedText = text.stringByPaddingToLength(10, withString: ".", startingAtIndex: 0)print(paddedText)
```

这是一个调用了NSString的填充字符串的方法（额外注意一点startingAtIndex:是指明要填充的字符串从哪位开始填充，所以不能超过withString:参数的长度）。我们在他基础之上要封装一个四元的curry函数。

```objectivec
func curriedPadding(startingAtIndex: Int, withString: String)
  (source: String, length: Int) -> String {
    return source.stringByPaddingToLength(length,
      withString: withString, startingAtIndex: startingAtIndex);
}
```

然后在它基础上创建一个只用点填充字符串的方法。

```objectivec
let dotPadding = curriedPadding(0, withString: ".")
let dotPadded = dotPadding(source: "Curry!", length: 10)
```










