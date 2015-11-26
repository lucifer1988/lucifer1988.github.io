---
layout: post
title: "Swift by Tutorials--Swift and Cocoa"
date: 2015-11-25 11:20:10 +0800
comments: true
categories: iOS
published: true
---
Swift是一门新推出的语言，但是核心框架还是Cocoa，这与OC是一致的，Cocoa的Foundation和UIKit框架对于开发应用仍是最重要的。这一章将创建一个应用，主要介绍Swift一Cocoa直接的交互，同时了解Cocoa的设计模式如何在Swift中体现出来。

<!--more-->

## Getting started

作者提供了一个starter project，添加了一个viewcontroller和之前介绍的JSON.swift（帮助解析JSON的工具类），另外还有facebook的SDK，所以接下来会介绍如何在Swift中混用OC代码。

## Bridging Swift and Objectivec-C

通过**bridging**技术，可以让我们在Swift和OC间相互调用。

### Swift bridging header

新建一个OC的.h文件，如：CafeHunter-ObjCBridging.h，然后在**Build Settings**中找到**Objective-C Bridging Header**，填写该头文件的路径，如：CafeHunter/CafeHunter-ObjCBridging.h，然后可以在该头文件中添加所需的OC头文件，如下，这样就可以在Swift中使用OC的类和方法了。

```objectivec
#import <FacebookSDK/FacebookSDK.h>
```

```objectivec
FBSettings.setDefaultAppID("INSERT_YOUR_FB_APP_ID")
```

### Objective-C compatibility header

那么如何在OC中使用Swift代码呢？还记得刚才在**Build Settings**中找到的**Objective-C Bridging Header**上面的**Install Objective-C Compatibility Header**吗？该项就是控制Swift向OC转换的开关，默认是打开的。

在report navigation的Build，中可以找到** Copy CafeHunter-Swift.h**的信息，可以双击打开该文件，你会发现你用Swift中创建的继承于OC的类和方法都在这里有对应的OC代码，而且实际上还有用@objec标记的代码，也有对应的转化。需要注意的两点是：

1. Swift中的私有方法是没有转化的，因为理论上外部不可能使用私有方法，但私有方法仍会注册在runtime中。
2. 转化的代码每个类之前会有一行类似**SWIFT_CLASS("_TtC10CafeHunter11AppDelegate")**的代码，实际上是Swift的压缩命名，为每个类添加了命名空间，也是该类在runtime中实际的名字，即使不同库中有相同名类，编译器也会通过该类名，准确找到对应的类。

## Adding the UI

添加一个FaceBook的登录View和MKMapView到Storybiard，然后关联到代码，如下。

```objectivec
@IBOutlet weak var mapView: MKMapView!
@IBOutlet weak var loginView: FBLoginView!
```

有几点说明：

1. outlet的类型是optional的，而且是隐式拆解的，这是因为如果不这么设置，编译器会发现这些变量没有在初始化中赋值，从而报错，所以这是为了避免编译器报错的手段，但我们知道它会在IB中初始化，所以采用了隐式拆解。使用outlet时不用再去拆解，但同时需要注意在view加载之前调用它们会导致崩溃，这点需要谨记。
2. outlet被加了weak关键字，这和OC中是一致的，是因为viewController的view对outlet是有强引用的，所以不必再添加额外的强引用。

### Showing the user's location

在Swift中长把协议的实现放在单独的extension中，这样可以将代码分组，但依然可以访问原类的变量和方法。

```objectivec
extension ViewController: MKMapViewDelegate {
  
  func mapView(mapView: MKMapView, didFailToLocateUserWithError error: NSError) {
    print(error)
    let alert = UIAlertController(title: "Error", message: "Failed to obtain location!", preferredStyle: .Alert)
    alert.addAction(UIAlertAction(title: "OK", style: .Default, handler: nil))
    self.presentViewController(alert, animated: true, completion: nil)
  }
  
  func mapView(mapView: MKMapView, didUpdateUserLocation userLocation: MKUserLocation) {
    let newLocation = userLocation.location!
    let distance = self.lastLoction?.distanceFromLocation(newLocation)
    if distance == nil || distance > searchDistance {
      self.lastLoction = newLocation
      self.centerMapOnLocation(newLocation)
      self.fetchCafesAroundLocation(newLocation)
    }
  }
}
```

本节其他部分都是一些业务逻辑方面的内容，数不赘述。

## Fetching data

前面的开发进行到了最后一步，就是将用户位置附近的咖啡馆找出来，并在地图上展示，这依赖于一个FaceBook的接口访问，同时本地需要定义咖啡馆的模型。

### Building the data model

先定义Cafe的model，首先你会想到Cafe在这里是一个纯数据类型，你可能会使用struct，因为Swift中struct也是model的选择之一。

但同时，你希望Cafe可以直接被显示到地图上，那么它就必须遵循MKAnnotation协议，这时，编译器便会报错，因为MKAnnotation是OC的协议，Cafe遵循该协议，必须可以被转化为OC代码，但struct在OC中只是一个C的数据类型，无法转化，所以这里必须声明Cafe为class，且必须继承自NSObject，因为MKAnnotation也遵循NSObject协议。最后，别忘了在init()结尾，添加**super.init()**，这样才能调到NSObject的初始化方法。

```objectivec
import Foundation
import MapKit

class Cafe: NSObject {
  let fbid: String
  let name: String
  let location: CLLocationCoordinate2D
  let street: String
  let city: String
  let zip: String
  
  init(fbid: String, name: String, location: CLLocationCoordinate2D, street: String, city: String, zip: String) {    self.fbid = fbid
    self.name = name
    self.location = location
    self.street = street
    self.city = city
    self.zip = zip
    super.init()  }
}

extension Cafe: MKAnnotation {
  var title: String? {
    return name
  }
  var coordinate: CLLocationCoordinate2D {
      return location
  }
}
```

### Fetching from Facebook

下面将从Facebook的接口获取数据。

```objectivec
var urlString = "https://graph.facebook.com/v2.0/search/"
urlString += "?access_token="
urlString += "\(FBSession.activeSession().accessTokenData.accessToken)"
urlString += "&type=place"
urlString += "&q=cafe"
urlString += "&center=\(location.coordinate.latitude),"
urlString += "\(location.coordinate.longitude)"
urlString += "&distance=\(Int(searchDistance))"

let url = NSURL(string: urlString)!
print("Request URL: \(url)")

let request = NSURLRequest(URL: url)
NSURLConnection.sendAsynchronousRequest(request, queue: NSOperationQueue.mainQueue()) { (response: NSURLResponse?, data: NSData?, error: NSError?) -> Void in
    if error != nil {
      let alert = UIAlertController(title: "Oops!", message: "An error occured", preferredStyle: .Alert)
      alert.addAction(UIAlertAction( title: "OK", style: .Default, handler: nil))
      self.presentViewController(alert, animated: true, completion: nil)
      return
    }
    
    let jsonObject: AnyObject! = try? NSJSONSerialization.JSONObjectWithData(data!, options: NSJSONReadingOptions(rawValue: 0))
    
    if let jsonObject = jsonObject as? [String:AnyObject] {
        if let data = JSONValue.fromObject(jsonObject)?["data"]?.array {
            var cafes: [Cafe] = []
            for cafeJSON in data {
            if let cafeJSON = cafeJSON.object {
                if let cafe = Cafe.fromJSON(cafeJSON) {
                    cafes.append(cafe)
                }
            }
        }
        
        self.mapView.removeAnnotations(self.cafes)
        self.cafes = cafes
        self.mapView.addAnnotations(cafes)
        }
    }
}
```

1. 首先拼接要访问的URL，采用了String拼接，而不是stringWithFormat:，这样代码比较清晰
2. 然后将string转化为NSURL，虽然NSURL的参数需要NSString，但是使用String也没问题，这是因为Swift对它们进行了无缝的桥接。
3. 使用了NSURLConnection的异步请求，使用closure处理回调。
4. JSON的反序列化，目前Swift2.0采用了try/catch这样的写法，而弃用了之前OC的传入NSError**参数的做法，代码更简洁。
5. NSJSONSerialization的JSONObjectWithData()方法实际返回的是NSDictionary，但是我们使用时是转化成[NSObject:AnyObject]，这也是Swift的隐式转换，还有NSArray和[AnyObject]。
6. 具体的JSON对象的解析，使用了JSON.swift库，关于这个库的原理在之前讲过，主要是利用enum的新特性，为每一种JSON元素类型提供了type。

### Parsing the JSON data

让我们在Cafe类中添加一个直接从JSON初始化的方法，在取JSON每一个元素时，一定要注意使用optional类型，这样可以保证不会因为某个值不存在而崩溃。

```objectivec
class func fromJSON(json: [String:JSONValue]) -> Cafe? {
   let fbid = json["id"]?.string
   let name = json["name"]?.string
   let latitude = json["location"]?["latitude"]?.double
   let longitude = json["location"]?["longitude"]?.double
   
   if fbid != nil && name != nil && latitude != nil && longitude != nil {
       var street: String
       if let maybeStreet = json["location"]?["street"]?.string {
           street = maybeStreet
       } else {
           street = ""
       }
       
       var city: String
       if let maybeCity = json["location"]?["city"]?.string {
           city = maybeCity
       } else {
           city = ""
       }
       
       var zip: String
       if let maybeZip = json["location"]?["zip"]?.string {
           zip = maybeZip
       } else {
           zip = ""
       }
       
       let location = CLLocationCoordinate2D(latitude: latitude!, longitude: longitude!)
       return Cafe(fbid: fbid!, name: name!, location: location, street: street, city: city, zip: zip)
   }
   return nil
}
```

从OC的角度考虑，我们可能会问，为什么不写一个secondary初始化方法？这是由于Swift决定的，某个类初始化方法是不能返回nil值的，如果可以，那么所有的Swift类型都是optional类型了，那么区分optional就没有意义，所以我们一定要注意，像构建这类可能返回nil的初始化方法，最好采用该类型的工厂方法来实现。

## Selectors

这一小节想给app加一个刷新按钮，引出OC中的target/selector模式在Swift中如何使用的问题。

OC中的方法调用是动态分发的，即方法的调用者（target）和方法名（SEL）都是可以在runtime动态分发的，而且你可以在runtime中修改这些值，而编译器不会在意该方法有没有实现。而在Swift中，所有的方法调用在编译期间就会决定，不再采用动态分发。那么如何解决Swift中调用原OC中需要target/SEL的方法，就需要得到解决。

Swift给出的方案是使用一个结构体Selector，Selector遵循了StringLiteralConvertible协议，其内部使用时可以直接从String转换，而使用者只需传入一个String即可。

```objectivec
self.navigationItem.leftBarButtonItem = UIBarButtonItem(barButtonSystemItem: .Refresh, target: self, action: "refresh:")
```

```objectivec
public convenience init(barButtonSystemItem systemItem: UIBarButtonSystemItem, target: AnyObject?, action: Selector)
```

你可以尝试不去实现refresh:方法，而编译器也不会报错，而在执行时才会报错，这就是因为它的真正实现还是使用了OC的动态分发，只是做了Swift的桥接。

## Protocols and delegates

接下来会为每个cafe创建一个detail view，并定义一个protocol，使用委托模式，并确保可以桥接到OC。

### Creating the detail view

创建对应的viewController，并在storyboard上定义好xib。

这里声明了一个Cafe变量作为数据源，如下，与以往不同的是，添加了didSet()方法，该方法会在cafe被赋值之后会调用，类似OC中的KVO，你可以为实现变量的didSet和willSet，分别在赋值前和复制后调用。

```objectivec
var cafe: Cafe? {
  didSet {
    self.setupWithCafe()
  }
}
```

在Cafe类添加一个pictureURL变量，是一个computed property，提供了返回值。

```objectivec
var pictureURL: NSURL {
return NSURL(string: "http://graph.facebook.com/place/picture?id=\(self.fbid)&type=large")!
}
```

接下来是setupWithCafe()方法，由于该方法放在了cafe的didSet中调用了，而且存在很多outlet存在，需要判断这些outlet是否加载完成，所以先进行了self.isViewLoaded()判断。

```objectivec
private func setupWithCafe() {
  if !self.isViewLoaded() {
    return
  }
  
  if let cafe = self.cafe {
    self.title = cafe.name
    
    self.nameLabel.text = cafe.name
    self.streetLabel.text = cafe.street
    self.cityLabel.text = cafe.city
    self.zipLabel.text = cafe.zip
    
    let request = NSURLRequest(URL: cafe.pictureURL)
    NSURLConnection.sendAsynchronousRequest(request, queue: NSOperationQueue.mainQueue()) {
      (response: NSURLResponse?, data: NSData?, error: NSError?) -> Void in
      let image = UIImage(data: data!)
      self.imageView.image = image
    }
  }
}
```

### Wiring up the detail view

在ViewController中，通过实现mapView(mapView: MKMapView!, viewForAnnotation annotation: MKAnnotation!) -> MKAnnotationView!和mapView(mapView: MKMapView!, annotationView view: MKAnnotationView!, calloutAccessoryControlTapped control: UIControl!)两个MKMapViewDelegate的方法，完成了通过点击大头针，弹出详情按钮，再进入详情页的功能。

这里还想将ViewController作为CafeViewController的委托，在CafeViewController点击返回时调用。

```objectivec
@objc protocol CafeViewControllerDelegate {
  optional func cafeViewControllerDidFinish(viewController: CafeViewController)
}
```

这里在声明protocol前加了**@objc**关键字是因为该protocol有optional方法，因为这样可以使Swift添加多个runtime检测，来检测protocol的一致性和是否实现了optional类型的方法。同时这也限制了，protocol的实现者必须是class类型，因为runtime对@obj的检测需要对象为class。而你也可以通过在protocol后加限制来实现。

```objectivec
@objc protocol CafeViewControllerDelegate: class {
  optional func cafeViewControllerDidFinish(viewController: CafeViewController) 
}
```

然后在CafeViewController添加delegate变量，与OC一样，使用weak属性，需要注意的是声明为optional类型，因为不是一定有委托对象的。

```objectivec
weak var delegate: CafeViewControllerDelegate?
```

然后是delegate的调用，这里使用了optional chain，delegate的存在与cafeViewControllerDidFinish()方法的实现与否都是不确定的。

```objectivec
@IBAction private func back(sender: AnyObject) { 
	self.delegate?.cafeViewControllerDidFinish?(self)}
```

最后，是在ViewController中的delegate实现。

```objectivec
extension ViewController: CafeViewControllerDelegate {
  func cafeViewControllerDidFinish(viewController: CafeViewController) {
    self.dismissViewControllerAnimated(true, completion: nil)
  } 
}
```





 






