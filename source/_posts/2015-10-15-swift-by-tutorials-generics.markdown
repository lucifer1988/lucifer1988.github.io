---
layout: post
title: "Swift by Tutorials--Generics"
date: 2015-10-15 11:05:56 +0800
comments: true
categories: iOS
published: true
---

经过前三章，基本对Swift的基本语法有了较全面的介绍，接下来会分别就Swift比较重要的几个技术要点介绍，这一节将讨论一个比较流行的语言特性，generics，即泛型。对于类型安全的编程语言，希望代码可以在一个场景下运行，但又想要在其他场景中也可以是合法的，比如对于一个加法函数，Int和Float类型的函数形式是一样的，只是变量类型不同，在强调类型的语言中，你必须分开定义这两个方法。很多语言为这一问题提供了解决方案，C++是使用了模板，而Swift、Java、C#是使用了泛型，也就是这一章的主角，配合主题这一章将创建一个Flickr照片搜素App来实践这一技术。  

<!--more-->

##Introducing generics

1.泛型是什么？举例来说Array和Dictionary就是类型安全的泛型应用实例。在OC中Array和Dictionary是可以存放不同类型的对象的，当然这有时是提供了很多方便，但当你去使用一个Array或Dictionary时，你如何知道其中的类型？只能通过文档或其他代码，而且没有任何办法去控制在runtime中出现数据异常。  
2.而Swift中对Array和Dictionary中类型是固定的，编译器会完成类型检查，而你的代码本身也对自己做了注释，比较下处理点击的方法在OC和Swift中的区别，在OC中调用这一方法，你一般是需要将set中的对象转化为UITouch类型，而Swift不仅省去你这一操作，代码可读性也更优。  

```objectivec
//in OC
-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event;
//in Swift
func touchesBegan(touches: [UITouch]!, withEvent event: UIEvent!)
```

2.所以说泛型就是类似Array这样，所有的Array运作方式都是一致的，都是将数据存在一张有序的表中，但泛型的Array将值的类型作为了参数，也就是不论Array中类型如何，都可以执行Array的方法。  

<!--more-->

##Generics in action

1.本章的实例项目是要从Flickr的搜索接口获取对应图片，并进行展示的一个App，其中网络访问部分大神已经写好了。  

<!--more-->

##Ordered dictionaries

1.第一个需求就是希望将用户最近搜索的图片放在前面，那么如果正常来讲，我们要用Array来存数据源，而不是Dictionary或Set，因为后两者是无序的，而这里为了应用泛型，打算自己创建一个有序的Dictionary，实际上就是想把key作为顺序。  

###The initial data structure

1.首先声明有序字典为Struct类型，并声明它的泛型类型参数，KeyType和ValueType并不是真实的类型，只是两个你用来替代类型的参数，一般用T来表示，如果单词表示的话用驼峰式大写首字母来表示。  

```objectivec
struct OrderedDictionary<KeyType, ValueType> {
}
```

2.创建一个有序字典最简单的方法是，在内部同时维护一个Dictionary和Array，这里使用了typealias分别给[KeyType]和[KeyType: ValueType]做了类型名替代，这样下面声明Array和Dictionary就可以直接用替代类型名来定义，同理这个也可以用在替换函数类型和闭包等比较长的类型的替换。    

```objectivec
typealias ArrayType = [KeyType]
typealias DictionaryType = [KeyType: ValueType]

var array = ArrayType()
var dictionary = DictionaryType()
```

3.对比Dictionary的定义，KeyType需要遵循Hashable协议，因为Dictionary需要对key做hash，所以在定义泛型那儿要加上遵循该协议。  

```objectivec
struct OrderedDictionary<KeyType: Hashable, ValueType>
```

###Keys, values and all that jazz

1.第一个要加入的方法是insert方法，因为是有序字典，所以有所不同。  

```objectivec
mutating func insert(value: ValueType, forKey key: KeyType, atIndex index: Int) -> ValueType? {
  var adjustedIndex = index
  
  let existingValue = self.dictionary[key]
  if existingValue != nil {
    let existingIndex = self.array.indexOf(key)!
    if existingIndex < index {
      adjustedIndex--
    }
    self.array.removeAtIndex(existingIndex)
  }
  
  self.array.insert(key, atIndex:adjustedIndex)
  self.dictionary[key] = value
  
  return existingValue
}
```

2.有几点需要说明，首先，该方法前的mutating关键字，因为Struct默认是不可变的，也就是是说你不能在实例方法中修改struct的成员变量，加上mutating是为了告诉编译器该方法可以修改struct成员变量，使编译器在适当的时候对struct做copy操作(前面说过，实际上是copy-on-write)，同时也增加了可读性。  
3.然后是remove方法，这里先对index是否越界做了判断，可以像OC中使用Assertions断言，也可以如下使用precondition，如果失败，会退出App。    

```objectivec
mutating func removeAtIndex(index: Int) -> (KeyType, ValueType) {
  precondition(index < self.array.count, "Index out-of-bounds")
  let key = self.array.removeAtIndex(index)
  let value = self.dictionary.removeValueForKey(key)!
  return (key, value)
}
```

4.这里在结束后会返回一个元组类型的删除值，使之与Swift的Array和Dictionary的remove方法保持一致。   

###Accessing values

1.上一节为有序字典添加了写入的方法，接下来添加一些读取的方法，首先是获取count的方法，如下，使用了前面提到的computed property技术。    

```objectivec
var count: Int {
  return self.array.count
}
```

2.在Swift中我们一般使用subscript来访问变量，类似dictionary[1]，一般是见于Array和Dictionary，不过我们计划在我们的有序字典也加入这一特性。  

```objectivec
subscript(key: KeyType) -> ValueType? {
  get {
    return self.dictionary[key]
  }
  set {
    if let index = self.array.indexOf(key) {
    } else {
      self.array.append(key)
    }
    self.dictionary[key] = newValue
  }
}
```

3.上述代码就是如何在自己的Struct中加入subscript行为，类似computed property，subscript有两个闭包，分别是getter和setter。  
4.因为这是一个有序数组，我们打算让他支持通过index来访问，需要注意的是：一，无论setter，getter都需要判断index是否越界；二，setter中输入的值newValue是一个元组类型，所以需要用let (key, value) = newValue将键值取出来。    

```objectivec
subscript(index: Int) -> (KeyType, ValueType) {
  get {
    precondition(index < self.array.count, "Index out-of-bounds")
    let key = self.array[index]
    let value = self.dictionary[key]!
    return (key, value)
  }
  set {
    precondition(index < self.array.count, "Index out-of-bounds")
    let (key, value) = newValue
    let originalKey = self.array[index]
    self.dictionary[originalKey] = nil
    self.array[index] = key
    self.dictionary[key] = value
  }
}
```

5.这里可能有个疑问，就是如果使用者使用Int作为KeyType呢？因为Int也遵循hashable，完全可以作为key，那么编译器如何判断该用哪组方法呢？遇到这种情况，setter方法当然没问题，因为赋值也不同，那么getter方法只能在取值时就声明返回值的类型，这样编译器会通过这个类型选择使用哪个方法。  

```objectivec
var dict = OrderedDictionary<Int, String>()
dict.insert("dog", forKey: 1, atIndex: 0)
dict.insert("cat", forKey: 2, atIndex: 1)
print(dict.array.description + " : " + dict.dictionary.description)
//"[1, 2] : [2: "cat", 1: "dog"]"
var byIndex: (Int, String) = dict[0]
print(byIndex)
//"(1, "dog")"
var byKey: String? = dict[2]
print(byKey)
//"Optional("cat")"
```

6.在使用type interface时，编译器需要明确知道返回值的类型，如果出现上述相同方法，返回值类型不同的情况，必须caller指明类型，否则编译器是不会知道该返回那个值的。  

<!--more-->

##Aside: Assertions & preconditions

1.assertions和precondition都是判断程序是否能继续执行时的判断条件，不同的是，assertion是不会在release build时编译的，而precondition可以；assertion是被用于在开发时获取bug，而precondition是用于当一个条件不满足时，抛出严重异常的。  
2.assertion的一个使用场景是有多个创建view的方法共同来构建页面，但其中一些依赖于另一些完成，这时要使用assertion。  

```objectivec
private func configureTableView() { 
	self.tableView = UITableView(frame: CGRectZero) 
	self.tableView.delegate = self 
	self.tableView.dataSource = self 
	self.view.addSubview(self.tableView)}private func configureHeader() {	assert(self.tableView != nil)	let headerView = UIView(frame: CGRectMake(0, 0, 320, 50))	headerView.backgroundColor = UIColor.clearColor()	let label = UILabel(frame: CGRectZero) 	label.text = "My Table" 	label.sizeToFit()	label.frame = CGRectMake(0, 0, label.bounds.size.width, label.bounds.size.height) headerView.addSubview(label)	self.tableView.tableHeaderView = headerView }
```

3.关于assertion一个有趣的现象是编译器允许在release build时假设assertion是true，有时也会导致一些bug，如下，输入0时，在debug下没问题，会触发断言；而在release中，编译器自动认为assertion是true，然后optimizer就会跳过if，直接进入>0的分支。  

```objectivec
func foo(value: Int) { 
	assert(value > 0)	if value > 0 {		print("Yes, it's greater than zero")	} else { 
		print("Nope")	} }
```

4.再来看下precondition，它和assertion做的是一样的工作，但是可以在release下运行，使用它是为了确保一些必要的条件，如下例，数组越界即使不加precondition，也会crash，但是通过precondition，可以获取到自定义的log信息，方便调试。    

```objectivec
func fetchPeopleBetweenIndexes(start: Int, end: Int) -> [Person] { 
	precondition(start < end)	precondition(start >= 0)	precondition(end <= self.people.count)	return Array(self.people[start..<end]) 
}
```

5.一般的经验是，在你release时可以跳过，但是希望在debug阶段获取失败信息时使用assertion；而在将会导致数据损坏或者其他严重问题前，使用precondition。同时在你开发一些第三方库时，在容易出现数据输入错误这些位置可以使用precondition来提示开发者。  

<!--more-->

##Adding image search

1.创建App的数据源，使用的就是之前自定义的有序字典，你可能注意到使用了Flickr.photo，Photo是一个定义在Flickr中的类，这样的机制非常有用，在保持类名尽量短的基础上实现了命名空间，在Flickr类中，可以单独使用Photo类。  

```objectivec
var searches = OrderedDictionary<String, [Flickr.Photo]>()
```

2.然后实现tableView的委托和数据源协议。    

```objectivec
func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
  return self.searches.count
}

func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
  let cell = tableView.dequeueReusableCellWithIdentifier("Cell", forIndexPath: indexPath) as UITableViewCell
  let (term, photos) = self.searches[indexPath.row]
  cell.textLabel!.text = "\(term) (\(photos.count))"
  return cell
}
```

3.然后是UISearchBarDelegate，这里调用Flickr的search方法是使用了Trailing Closures技术，即如果closure作为一个方法最后一个变量，那么可以写到所调用方法的外面（后面），如下：    

```objectivec
func searchBarSearchButtonClicked(searchBar: UISearchBar!) {
  searchBar.resignFirstResponder()
  let searchTerm = searchBar.text
  Flickr.search(searchTerm!) {
    switch ($0) {
    case .Error:
      break
    case .Results(let results):
      self.searches.insert(results, forKey: searchTerm!, atIndex: 0)
      self.tableView.reloadData()
    }
  }
}
```

```objectivec
func someFunctionThatTakesAClosure(closure: () -> Void) {
    // function body goes here
}
// here's how you call this function without using a trailing closure:
someFunctionThatTakesAClosure({
    // closure's body goes here
})
// here's how you call this function with a trailing closure instead:
someFunctionThatTakesAClosure() {
    // trailing closure's body goes here
}
```

###Show me the photos!

1.这一节来完成详情页，先是在prepareForSegue方法中设置DetailViewController。  

```objectivec
override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
  if segue.identifier == "showDetail" {
    if let indexPath = self.tableView.indexPathForSelectedRow {
      let (_, photos) = self.searches[indexPath.row]
      (segue.destinationViewController as! DetailViewController).photos = photos
    }
  }
}
```

###Deleting searches

1.为搜索页添加滑动删除功能。  

```objectivec
self.navigationItem.leftBarButtonItem = self.editButtonItem()
```

```objectivec
override func setEditing(editing: Bool, animated: Bool)  {
  super.setEditing(editing, animated: animated)
  self.tableView.setEditing(editing, animated: animated)
}
```

```objectivec
func tableView(tableView: UITableView, canEditRowAtIndexPath indexPath: NSIndexPath) -> Bool {
  return true
}
```

```objectivec
func tableView(tableView: UITableView, commitEditingStyle editingStyle: UITableViewCellEditingStyle, forRowAtIndexPath indexPath: NSIndexPath) {
  if editingStyle == .Delete {
    self.searches.removeAtIndex(indexPath.row)
    tableView.deleteRowsAtIndexPaths([indexPath], withRowAnimation: .Fade)
  }
}
```

<!--more-->

## Generic functions and protocols

1.这一节介绍泛型的函数和协议，之前一直使用的find方法就是一个泛型方法，这是一个全局方法，泛型参数C定义了domain参数，也间接定义了value参数的类型，且返回值也和C有关：  

```objectivec
func find<C: Collection where C.GeneratorType.Element: Equatable> (domain: C, value: C.GeneratorType.Element) -> C.IndexType?
```

2.我们之前定义的有序字典的insert方法中这么使用了find()，没有指出C，其实这又是type interface的体现，通过第一个参数推断出了C的类型。  

```objectivec
let existingIndex = find(self.array, key)!
```

3.那么GeneratorType是什么？上述的Collection协议，同时也遵从于SequenceType协议，如下，要实现该协议，必须有个typealias名为Generator，且遵从于GeneratorType协议，同时实现generate()方法返回Generator类型。  

```objectivec
protocol SequenceType {	typealias Generator : GeneratorType 
	public func generate() -> Self.Generator}
```

4.那么在自定义的有序数组上实验下SeqenceType，首先定义typealias名为Generator，使用AnyGenerator这个泛型类。  

```objectivec
extension OrderedDictionary: SequenceType {
  typealias Generator = AnyGenerator<(KeyType, ValueType)>
  func generate() -> AnyGenerator<(KeyType, ValueType)> {
    var index = 0
    return anyGenerator {
      if index < self.array.count {
        let key = self.array[index++]
        return (key, self.dictionary[key]!)
      } else {
        return nil
      }
    }
  }
}
```

5.而在实现generate()方法中，通过调用了anyGenerator方法，该方法只有一个closure参数，所以使用了Trailing Closures技术，这个closure会在每次调用next()时调用，在closure中，你完成自己的遍历方法。  

```objectivec
public func anyGenerator<Element>(body: () -> Element?) -> AnyGenerator<Element>
```

6.实现了SequenceType Protocol，你可以使用for-in来遍历字典，其实typealias Generator = AnyGenerator<(KeyType, ValueType)>这句可以删除，因为Swift从func generate() -> AnyGenerator<(KeyType, ValueType)>()返回值推断出了该类型。  
7.实际上SequenceType Protocol就是一个泛型协议，只不过因为protocol不能使用<>关键字，而像Java和C#是可以的，原因也很简单，Protocols本身就是定义给class或struct实现的，这本身就是带有泛型的性质，Swift的思想就是protocol定义接口，而class和struct定义类型、泛型或其他。  



