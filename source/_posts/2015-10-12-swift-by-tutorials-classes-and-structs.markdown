---
layout: post
title: "Swift by Tutorials--Classes and Structs"
date: 2015-10-12 15:26:48 +0800
comments: true
categories: iOS
published: true
---

继上两章对Swift的基本类型的学习，这一章开始介绍Swift中的Class和Struct，Class对所有面向对象的语言都不陌生，而Struct可能用的比较少，因为大部分用于C中，但Swift中的Struct与C还有很多不同，这章会一一介绍，而且同时会讲到Class与Struct之间的不同与使用场景，以及它们的扩展，也是这一章的重点。

##Getting started

###The class concept

1.类是通过总结一些对象的共同特点，定义基本类型，通过继承来创建具体使用的子类型，它拥有自己的数据和方法，可以视为数据的容器。  

##My first class

1.介绍示例项目的Class设计。  

###Creating the class

1.import语句用于导入Swift的库文件，记性好的话，之前Apple在OC中加入了@import来替代#import（如：@import Foundation;替代#import <Foundation/Foundation.h>），其实Swift中的import是和这个一致的。  
2.定义一个类如下即可，但是如果如下，有未初始化赋值的非optional变量，那么会提示你增加初始化方法。  

```objectivec
class Treasure {
	let what: String
	let latitude: Double
	let longitude:Double
}
init(what: String, latitude: Double, longitude: Double) {
	self.what = waht
	self.latitude = latitude
	self.longitude = longitude
}
```

###A struct-ural improvement

1.下一步的优化是把经纬度信息做成一个结构体，那么就涉及到了Swift中的Struct，Swift中Struct和Class一样，都可以存储数据和拥有自己的方法，但要记住Struct始终是一个数值型的容器，它的用途只是持有数据，不要让它承担更多的功能。  

```objectivec
struct GeoLocation {
	var latitude: Double
	var longitude: Double
}
```

2.在Swift中，在工程中的文件是相互自动import的，所以你不用再去手动导入，这一点在你开发library和framework也是一样的。  

###Reference types vs. value types

1.Swift中，Struct与Class的最大区别是，Class在本质上是指针引用类型，而Struct是值类型，在赋值过程中，Class传递的是指针，而Struct则会copy一份新值，从如下的例子即可看出。  

```objectivec
struct MyStruct {
	var foo: Double = 0.0
}
class MyClass {
	var foo: Double = 0.0
}
var structA = MyStruct()
var structB = structA
structB.foo = 1.0
print(structA.foo)
//0.0
print(structB.foo)
//1.0
var classA = MyClass()
var classB = classA
classB.foo = 1.0
print(classA.foo)
//1.0
print(classB.foo)
//0.0
```

2.需要说明的一点，Swift在copy一个Struct时是很智能的，只会在确定必要的时候copy，也就是说structB = structA并不会创建出拷贝，只有你开始改变其中一个值时，runtime会开始执行copy。  
3.另外关于let类型的Struct和Class还有一些细微的区别，对于二者的var实例，是没有区别的，都可以改变各自的属性或将自身赋值给其他实例，但对于let实例，Class依然可以修改自己的属性变量，但是不能将自己赋值给其他实例，而Struct既不能改变自己的属性变量，也不能将自己赋值给其他实例，这也是为什么Array和Dictionary是Struct而不是Class。  

```objectivec
struct MyStruct {
	var foo: Double = 0.0
}
class MyClass {
	var foo: Double = 0.0
}
var classA = MyClass()
let classB = MyClass()
classA.foo = 1.0
classB.foo = 1.0
classB = classA
//error
var structA = MyStruct()
let structB = MyStruct()
structA.foo = 1.0
structB.foo = 1.0
//error
structB = structA
//error
```

###Convenience initializers

1.在实例中其实还是不必要将GeoLocation暴露给使用者，可以直接给出更方便的初始化方法，这就用到了convenience initializer，在该初始化方法中跳转到了原初始化方法中，也称为designated initializer。  

```objectivec
convenience init(what: String, latitude: Double, longitude: Double) {
	let location = GeoLocation(latitude: latitude, longitude: longitude)
	self.init(what: what, location: location)
}
```

2.Struct不需要显式的初始化方法，Swift为你自动添加了，你只需要按Struct的属性顺序一一初始化就行了，这也是为什么上述方法中可以自动初始化GeoLocation。  

###Class inheritance

1.Swift中的继承是怎样的？如下例：  

```objectivec
class HistoryTreasure: Treasure {
	let yaer: Int
	init(what: String, year: Int, latitude: Double, longitude: Double) {
		self.year = year
		let location = GeoLocation(latitude: latitude, longitude: longitude)
		super.init(what: what, location: location)
	}
}
```

2.子类如果有自己新的属性时，需要自己的designated initializer，而子类的designated initializer必须引用父类的一个designated initializer（注意不可以是convenience initializer），所以和上一节比较相当于做了重复工作。  
3.这里有与OC不同的一点，在OC中子类的init()方法中，是先调用父类的init()，再进行子类属性的赋值，而在Swift中是最后调用父类的init()，因为在Swift中是initializer来初始化所有属性，最后交给父类来处理，父类的init()要放在最后，是因为它不知道子类中声明的新属性，必须在它之前把这些新属性初始化。  

<!--more-->

##Swift and MapKit

1.重写父类方法，Swift中需要在方法前加上override关键字，增加了可读性，同时如果你写的方法不存在于父类，那么编译器会报error通知你。  

```objectivec
override func viewDidLoad() {
	super.viewDidLoad()
}
```

###Class extensions and computed properties

1.示例中需要将Treasure类型的变量显示在MKMapView上，那么就要使Treasure遵循MKAnnotation协议，我们使用了extension来实现：  

```objectivec
import MapKit
extension Treasure: MKAnnotation {
	var coordinate: CLLocationCoordinate2D {
		return self.location.coordinate
	}
	var title: String {
		return self.what
	}
}
```

2.extension类似OC的category，都是扩展类的技术，但是优于category，因为extension不但可以添加方法，还可以添加新属性。  
3.这里添加的两个property有些特别，它们是Swift中的computed properties，每次访问它们都会执行后面的方法，用法与普通的properties是一致的。  

###Your first struct extension

1.Struct也可以添加extension，下例中的extension起到了分割代码，增加可读性的作用，这是extension的习惯用法。  

```objectivec
import MapKit
extension GeoLocation {
	var coordinate: CLLocationCoordinate2D {
		return CLLocationCoordinate2DMake(self.latitude, self.longitude)
	}
	var mapPoint: MKMapPoint {
		return MKMapPointForCoordinate(self.coordinate)
	}
}
```

###Inheriting from NSObject

1.类遵循MKAnnotation同时，也应该遵循NSObject协议，因为MKAnnotation继承自NSObject协议。  

###Pinning the map

1.在viewDidLoad()中添加以下代码，完成mapView上打点的工作。  

```objectivec
self.mapView.delegate = self
self.mapView.addAnnotations(self.treasures)
```

2.然后添加viewController的extension，并实现MKMapViewDelegate的方法：    

```objectivec
extension ViewController: MKMapViewDelegate {
	func mapView(mapView: MKMapView, viewForAnnotation annotation: MKAnnotation) -> MKAnnotationView
	{
		if let treasure = annotation as? Treasure {
			var view = mapView.dequeueReusableAnnotationViewWithIdentifier("pin") as! MKPinAnnotationView!
			if view = nil {
				view = MKPinAnnotationView(annotation: annotation, reuseIdentifier: "pin")
				view.canShowCallout = true
				view.animatesDrop = false
				view.calloutOffset = CGPoint(x: -5, y: 5)
				view.rightCalloutAccessoryView = UIButton(type: .DetailDisclosure) as UIView
			} else {
				view.annotation = annotation
			}
			return view
		}
		return nil
	}
}
```

3.在实现的mapView:viewForAnnotation方法中，annotation参数类型为MKAnnotation!，是一个隐式拆解的optional类型，但是我们还是进行了if/let的检查，因为这个方法是OC的API，是没有optional的，所以为了兼容Swift只能声明为这个类型，所以还需要显式拆解。  
4.同时除了检查是否为nil，我们还要注意传入的类型是否为Treasure类型，这儿用到了inline downcasting技术，也是Swift的一种简写语法，if let treasure = annotation as？ Treasure{}，如果annotation不是Treasure类型，那么也不会进入if，这是Swift中确保类型正确的一种技术。  
5.获取MKPinAnnotationView利用了一贯的复用技术，另外此处又一次用到了downcast技术，只不过因为返回的类型肯定可以确定都是MKPinAnnotationView，所以用了非optional版本。  

###The reduce algorithm

1.这一节是为了解决app载入后不能直接定位到目标位置，而要先定位自己的位置这个bug。  

```objectivec
let rectToDisplay = self.treasures.reduce(MKMapRectNull) {
  (mapRect: MKMapRect, treasure: Treasure) -> MKMapRect in
  let treasurePointRect = MKMapRect(origin: treasure.location.mapPoint, size: MKMapSize(width: 0, height: 0))
  return MKMapRectUnion(mapRect, treasurePointRect)
}
self.mapView.setVisibleMapRect(rectToDisplay, edgePadding: UIEdgeInsetsMake(74, 10, 10, 10), animated: false)
```

2.为了达到这一优化，实际就是要获取可以展示全部treasures的最小地图范围，然后在地图绘制这一区域。而输入是一个数组，需要一个它们逐个计算的结果，这里使用了函数式编程中的Reduce函数，这是处理这一问题的最佳方案，下面是Swift中reduce的原型，需要一个初始值initial，这里对应的是MKMapRectNull，是一个空区域，然后combine的方法是(mapRect: MKMapRect, treasure: Treasure) -> MKMapRect类型的方法，mapRect是每次执行后的返回值，初始值就是initial，而treasure是array每个元素的遍历，最后可以得到一个MKMapRect类型的区域，包含了所有元素的最小区域，然后setVisibleMapRect()，并加了边距来适应其他页面元素，最终达到了目的，Swift中的函数式编程还会在第七章继续讲解。  

```objectivec
reduce(initial: U, combine: (U, T) -> U) -> U
```

<!--more-->

##Polymorphism

1.现在又有新需求了，需要不同类型的Treasure在地图上显示Annotation颜色不同，这可以通过多态来实现，首先在父类中添加方法，再在子类中重写该方法：  

```objectivec
//in parent class
func pinColor() -> MKPinAnnotationColor  {
  return MKPinAnnotationColor.Red
}
//in subclass
override func pinColor() -> MKPinAnnotationColor  {
  return MKPinAnnotationColor.Purple
}
```

2.然后在绘制MKPinAnnotationView那儿调用该方法：  

```objectivec
view.pinColor = treasure.pinColor()
```

###Dynamic dispatch and final classes

1.对于上面的Treasure多态，runtime是怎么执行的呢？是靠dynamic dispatch实现的，这一技术其实在OC里用的很普遍，OC作为动态语言，可以在runtime修改所传递的消息，甚至消息的接收者，都是靠动态分发(详细过程可参照前一部的Effective Objective-C2.0的笔记)。  
2.Dynamic dispatch在Swift中依然存在，就是为了实现多态这类特性，不同于OC的消息分发机制，Swift的分发更像C++，它通过virtual tables(简称vtables)来实现。  
3.如上例，当编译器遇到pinColor()调用，因为Treasure有很多子类，它便会去使用vtable去查找，而如果是Treasure的子类调用pinColor()，编译器依然会去先去查找vtable，而不是直接去调用pinColor()方法，因为它并不知道有没有类继续在继承，虽然开发者知道它是没有子类的。  
4.所以通过告知编译器某些类没有子类，会提高app的效率，缩短运行时间，我们可以通过在类型前加final关键字来告知编译器这一信息。  

```objectivec
final class HistoryTreasure: Treasure
```

<!--more-->

##Adding annotations

1.继续，新的需求是用户点击每个annotation时弹出一个alertView来告知用户一些信息。因为每个alert提示的信息不同，所以打算将生成alert的任务交给treasure，然后viewController负责显示，采用的是利用protocol技术。  
2.首先在Treasure.swift文件中声明Alertable协议，然后Treasure的子类分别遵循并实现其中的方法。  

```objectivec
@objc protocol Alertable {
	fun alert() -> UIAlertController
}
```

```objectivec
extension HistoryTreasure: Alertable {  
  func alert() -> UIAlertController {
    let alert = UIAlertController(title: "History", message: "From \(self.year):\n\(self.what)", preferredStyle: UIAlertControllerStyle.Alert)
    return alert
  }
}
```

3.然后在viewController中实现MapView点击Annotation的委托方法。  

```objectivec
func mapView(mapView: MKMapView, annotationView view: MKAnnotationView, calloutAccessoryControlTapped control: UIControl) {
  if let treasure = view.annotation as? Treasure {
    if let alertable = treasure as? Alertable {
      let alert = alertable.alert()
      alert.addAction(UIAlertAction(title: "OK", style: UIAlertActionStyle.Default, handler: nil))
      self.presentViewController(alert, animated: true, completion: nil)
    }
  }
}
```

<!--more-->

##Sorting an array

1.新需求，用户可以在找到第一个treasure后能有选项可以使其找到下一个最近的treasure。首先我们给GeoLocation添加一个计算点之间的方法。  

```objectivec
func distanceBetween(other: GeoLocation) -> Double {
  let locationA = CLLocation(latitude: self.latitude, longitude: self.longitude)
  let locationB = CLLocation(latitude: other.latitude, longitude: other.longitude)
  return locationA.distanceFromLocation(locationB)
}
```

2.该方法是直接定义在struct的定义中的，Swift中的Struct可以定义方法，这也是它与C的Struct的最大区别，C中的Struct只能定义变量，导致与之相关的方法只能定义为全局方法，而如何按照类型归纳这些方法就需要开发者手工将其写到一个头文件中，这显然费时也不合理，而Swift真正实现了Struct中可以真正拥有自己的方法。  
3.像用户点击Annotation后弹出的alert再添加一个Find Nearest的选项，并实现找到离该点最近的Treasure。  

```objectivec
alert.addAction(UIAlertAction(title: "Find Nearest", style: UIAlertActionStyle.Default) { action in
  var sortedTreasures = self.treasures
  sortedTreasures.sortInPlace {
    let distanceA = treasure.location.distanceBetween($0.location)
    let distanceB = treasure.location.distanceBetween($1.location)
    return distanceA < distanceB
  }
  mapView.deselectAnnotation(treasure, animated: true)
  mapView.selectAnnotation(sortedTreasures[1], animated: true)
  })
```

4.上述代码的核心是sortedTreasures的排序，利用了sort()方法(Swift中改为sortInPlace())，$1和$2分别代表了传入方法的第一和第二参数，是简写形式，分别计算出两点距离当前treasure的距离，然后返回Bool告知是否已按照小大顺序排好，最后数组排序后，展示第二个元素，来展示最近的treasure。    

```objectivec
public mutating func sortInPlace(@noescape isOrderedBefore: (Self.Generator.Element, Self.Generator.Element) -> Bool)
```

<!--more-->

##Equality and operator overload

1.照例，新需求，需要标记用户发现treasure的路径并在用户在已发现的treasure上操作时提示用户。  
2.创建已找到Treasure的数组和要绘制的地图线，并实现MKMapViewDelegate的一个方法。  

```objectivec
private var foundLocations: [GeoLocation] = []
private var polyline: MKPolyline!
```

```objectivec
func mapView(mapView: MKMapView, rendererForOverlay overlay: MKOverlay) -> MKOverlayRenderer {
  if let polylineOverlay = overlay as? MKPolyline {
    let renderer = MKPolylineRenderer(polyline: polylineOverlay)
    renderer.strokeColor = UIColor.blueColor()
    return renderer
  }
  return nil
}
```

3.在上一节添加alert的位置再添加Found项，再创建markTreasureAsFound()方法来标记已找到的Treasure，并绘制新的MKPolyline。  

```objectivec
alert.addAction(UIAlertAction(title: "Found", style: UIAlertActionStyle.Default) { action in
  self.markTreasureAsFound(treasure)
  })
```

```objectivec
private func markTreasureAsFound(treasure: Treasure) {
  if let index = self.foundLocations.indexOf(treasure.location) {
    let alert = UIAlertController(title: "Oops!", message: "You've already found this treasure (at step \(index + 1))! Try again!", preferredStyle: .Alert)
    alert.addAction(UIAlertAction(title: "OK", style: .Default, handler: nil))
    self.presentViewController(alert, animated: true, completion: nil)
  } else {
    self.foundLocations.append(treasure.location)
    if self.polyline != nil {
      self.mapView.removeOverlay(self.polyline)
    }
    var coordinates = self.foundLocations.map { $0.coordinate }
    self.polyline = MKPolyline(coordinates: &coordinates, count: coordinates.count)
    self.mapView.addOverlay(self.polyline)
  }
}
```

4.首先利用find()函数（Swift2.0已弃用，改为collection的indexOf()方法）来获取目前位置是否已在foundLocations中，返回值为optional类型，所以需要if/let判断，这里就体现了这一技术的便利。  
5.在创建MKPolyline时，先对foundLocations使用了map方法，map如前面提到的reduce一样，也是函数式编程的一种，它的作用是从一个数组中获取另外一个数组，这里就是从foundLocations得到了由其中每一个location的coordinate组成的新数组，$0代表每一个location。  
6.这里需要实现find()方法（实际在Swift2.0已弃用），需要在GeoLocation中遵循Equatable协议，，因为find()中比较每个元素是通过==来比较的（类似OC中的isEqual()），而Class和Struc默认是不能用==比较的。这里用到了Swift的又一新特性，operator overload，既可以直接重写==这样的操作符，但需要注意下面对==的重写，并没有包含在extension中，因为operation overload都必须在定义在全局中，因为它本身并不属于某个类，它只是与一个类有关，基于要比较的的两个参数是这个类的实例。    

```objectivec
protocol Equatable {	func ==(lhs: Self, rhs: Self) -> Bool}
```

```objectivec
extension GeoLocation: Equatable {
}
func ==(lhs: GeoLocation, rhs: GeoLocation) -> Bool {
  return lhs.latitude == rhs.latitude && lhs.longitude == rhs.longitude
}
```

<!--more-->

##Access Control

1.目前为止所有变量和方法都是public的，但Swift提供了访问权限的关键字，包括：Public(所有代码均可访问)、Internal(只在该target(library或app)下的代码可以访问，是默认权限)、Private(只有该文件可以访问)。  
2.例如有一些类中的帮助方法，你不希望暴露在外，因为他们可以改变着一些不该暴露的内部状态信息。  
3.需要注意的是Unit test通常是另外一个Target，如果你的代码有部分需要单元测试，那么需要声明为public。  
4.与OC相比，OC是没有绝对的私有方法的，因为即使没有暴露在在头文件中，也可以通过runtime注入来访问私有方法。  

```objectivec
private func markTreasureAsFound(treasure: Treasure)
```

```objectivec
private var treasures: [Treasure] = []private var foundLocations: [GeoLocation] = [] private var polyline: MKPolyline!
```

5.internal一般是不会显式声明的，如果是一个library被多个app使用，你可能会将内部类声明为internal，这样就不会被其他app中的代码使用了。  
6.访问控制标志是表达你代码意图的很好的方式，可以使代码更易维护，也会减少Bug，
