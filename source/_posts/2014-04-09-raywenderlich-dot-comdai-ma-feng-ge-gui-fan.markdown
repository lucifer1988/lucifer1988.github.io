---
layout: post
title: "raywenderlich.com代码风格规范"
date: 2014-04-09 14:36:55 +0800
comments: true
categories: iOS 
description: "raywenderlich.com代码风格规范"
keywords: objective-c,raywenderlich.com,
published: true
---

<img src="/images/rayWenderlich_icon.png">

[raywenderlich.com](http://www.raywenderlich.com)对于搞iOS开发的人来说不会陌生（如果你经常关注一些技术博客的话），它原本只是Ray Wenderlich的个人博客，但通过不断聚集优秀的开发者参与到其站点的技术博客撰写，包括了应用开发和游戏开发的各个方面，同时将这些技术博客整理成书，作为开发教程出售（貌似最近还出视频教程了，又想法圈钱了···），这样raywenderlich.com渐渐发展成了一个iOS开发社区，其优质的文章和对文章本地化的重视，使得其影响力逐渐向全球扩展。本文是对其最近公布的自家的Objective-C代码风格规范的一些整理，原文地址在[这里](https://github.com/raywenderlich/objective-c-style-guide)。

<!--more-->

##目录

* [语言](#语言)
* [代码结构](#代码结构)
* [空格](#空格)
* [注释](#注释)
* [命名](#命名)
* [下划线](#下划线)
* [方法](#方法)
* [变量](#变量)
* [属性](#属性)
* [点表达式](#点表达式)
* [文字量](#文字量)
* [常量](#常量)
* [枚举类型](#枚举类型)
* [分支语句](#分支语句)
* [私有属性](#私有属性)
* [布尔型](#布尔型)
* [条件语句](#条件语句)
* [三元运算符](#三元运算符)
* [Init方法](#Init方法)
* [构造类方法](#构造类方法)
* [CGRect相关函数](#CGRect相关函数)
* [愉快路径](#愉快路径)
* [异常处理](#异常处理)
* [单例](#单例)
* [换行](#换行)
* [笑脸（你没看错，这也有规范）](#笑脸)
* [Xcode工程](#Xcode工程)

## <a name="语言"></a>语言

推荐使用美英，主要体现在命名时使用美英单词。

**推荐：**

```objc
UIColor *myColor = [UIColor whiteColor];
```

**不推荐：**

```objc
UIColor *myColour = [UIColor whiteColor];
```

## <a name="代码结构"></a>代码结构
统一使用`#pragma mark -`组织代码结构。

```objc
#pragma mark - Lifecycle

- (instancetype)init {}
- (void)dealloc {}
- (void)viewDidLoad {}
- (void)viewWillAppear:(BOOL)animated {}
- (void)didReceiveMemoryWarning {}

#pragma mark - Custom Accessors

- (void)setCustomProperty:(id)value {}
- (id)customProperty {}

#pragma mark - IBActions

- (IBAction)submitData:(id)sender {}

#pragma mark - Public

- (void)publicMethod {}

#pragma mark - Private

- (void)privateMethod {}

#pragma mark - Protocol conformance
#pragma mark - UITextFieldDelegate
#pragma mark - UITableViewDataSource
#pragma mark - UITableViewDelegate

#pragma mark - NSCopying

- (id)copyWithZone:(NSZone *)zone {}

#pragma mark - NSObject

- (NSString *)description {}
```

## <a name="空格"></a>空格
* 使用2个空格缩进（理由是可以在保持打印空白的基础上，尽量减少换行的可能性），不要使用tab缩进，可以在Xcode的preference下进行修改（默认是4个空格）；
* 方法中的`{}`和在`if`/`else`/`switch`/`while`等语法中出现的`{}`在本行开始，而结束于新一行。

**推荐：**

```objc
if (user.isHappy) {
  //Do something
} else {
  //Do something else
}
```

**不推荐：**

```objc
if (user.isHappy)
{
    //Do something
}
else {
    //Do something else
}
```

* 方法之间应该有且只有一空行来保持代码结构清晰，方法中的空白行用来划分功能，但是经常你可能需要将它们重构为新的方法；
* 推荐使用自动提示的语法结构，不过`@sythesize`和`@dyamic`可以声明在新行；
* `冒号对齐`这样的方法调用结构要避免，假如方法中出现3个以上的冒号，这样的调用结构会使代码可读性很差。千万别在含有block的方法中去对齐冒号，这样Xcode的缩进会使其非法。

**推荐：**

```objc
// blocks are easily readable
[UIView animateWithDuration:1.0 animations:^{
  // something
} completion:^(BOOL finished) {
  // something
}];
```

**不推荐：**

```objc
// colon-aligning makes the block indentation hard to read
[UIView animateWithDuration:1.0
                 animations:^{
                     // something
                 }
                 completion:^(BOOL finished) {
                     // something
                 }];
```

## <a name="注释"></a>注释
注释主要用来解释这段代码为什么存在于此，注意所有注释需要及时更新或删除。

大段的注释要避免，有必要可以加到单独的文档，注释需要一些简短的说明。PS：不包括那些为生成文档而做的注释（一些工具可以通过代码中的注释生成文档）。

## <a name="命名"></a>命名
Apple的命名习惯是尽可能详细，尤其是和内存管理相关的。

长的，描述性的方法和变量名命名是被推荐的。

**推荐：**

```objc
UIButton *settingsButton;
```

**不推荐：**

```objc
UIButton *setBut;
```

类名和常量名必须要有三个字母的命名前缀（主要为了避免和Apple大多数两个前缀的命名冲突，比如UIButton，CAAnimation，CGRect等），不过Core Data的实体命名可以省略这些前缀。比如raywenderlich.com的命名前缀为RTW。

常量需要驼峰型命名，所有单词首字母大写，且使用相关类名作为前缀。

**推荐：**

```objc
static NSTimeInterval const RWTTutorialViewControllerNavigationFadeAnimationDuration = 0.3;
```

**不推荐：**

```objc
static NSTimeInterval const fadetime = 1.7;
```

property使用驼峰形命名，保证开头单词小写，且使用Apple的自动合成规则，除了特殊情况，不要手动声明`@synthesize`。

**推荐：**

```objc
@property (strong, nonatomic) NSString *descriptiveVariableName;
```

**不推荐：**

```objc
id varnm;
```

## <a name="下划线"></a>下划线

当使用property时，实例变量的读写要使用`self.`，这样的话所有的property可以清楚地区分出来。

一个例外：在初始化方法中，需要的实例变量要直接使用`_variableName`型，为了避免调用getter/setter方法可能出现的循环引用。

临时变量不能使用下划线。

## <a name="方法"></a>方法

在方法声明时，方法类型（-/+）后要有一个空格。方法段之间也要有一个空格。在每个变量前要加一个描述性的词语用来描述这个变量。

不要在用于描述的词语中加入“and”，具体例子见下：

**推荐：**

```objc
- (void)setExampleText:(NSString *)text image:(UIImage *)image;
- (void)sendAction:(SEL)aSelector to:(id)anObject forAllCells:(BOOL)flag;
- (id)viewWithTag:(NSInteger)tag;
- (instancetype)initWithWidth:(CGFloat)width height:(CGFloat)height;
```

**不推荐：**

```objc
-(void)setT:(NSString *)text i:(UIImage *)image;
- (void)sendAction:(SEL)aSelector :(id)anObject :(BOOL)flag;
- (id)taggedView:(NSInteger)tag;
- (instancetype)initWithWidth:(CGFloat)width andHeight:(CGFloat)height;
- (instancetype)initWith:(int)width and:(int)height;  // Never do this.
```

## <a name="变量"></a>变量

变量命名尽量使用描述性的词语。单个字母的命名除了在`for()`循环中，其他地方都是不允许的。

星号声明了指向变量的指针，例如：`NSString *text`，不是`NSString* text`或`NSString * text`，除了在声明常量的时候（例如：`NSString *const text`）。

如果可能的话，使用[私有属性](#私有属性)，而不是实例变量，尽管使用实例变量也是对的，不过这样的协定可以保持代码的一致性。

只有在初始化方法（如`init`，`initWithCoder:`等），`dealloc`和setter/getter方法中使用下划线加变量名的读写方法，更多关于这一情况的说明见[Apple相关文档](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmPractical.html#//apple_ref/doc/uid/TP40004447-SW6)

**推荐：**

```objc
@interface RWTTutorial : NSObject

@property (strong, nonatomic) NSString *tutorialName;

@end
```

**不推荐：**

```objc
@interface RWTTutorial : NSObject {
  NSString *tutorialName;
}
```

## <a name="属性"></a>属性

属性需要清楚地列出，属性的property类型顺序应该是先内存相关，再原子性相关，这与从IB中自动关联的属性的顺序是一致的。

**推荐：**

```objc
@property (weak, nonatomic) IBOutlet UIView *containerView;
@property (strong, nonatomic) NSString *tutorialName;
```

**不推荐：**

```objc
@property (nonatomic, weak) IBOutlet UIView *containerView;
@property (nonatomic) NSString *tutorialName;
```

带有不可变性质的属性（比如：NSString）推荐使用`copy`而不是`strong`，理由是其他人可能传入一个其对应的可变实例（比如：NSMutableString），而你可能不会注意到。

**推荐：**

```objc
@property (copy, nonatomic) NSString *tutorialName;
```

**不推荐：**

```objc
@property (strong, nonatomic) NSString *tutorialName;
```

## <a name="点表达式"></a>点表达式

读写property时应一直使用点表达式，这使代码变得简洁，而`[]`表达式用于其他所有的实例中。

**推荐：**

```objc
NSInteger arrayCount = [self.array count];
view.backgroundColor = [UIColor orangeColor];
[UIApplication sharedApplication].delegate;
```

**不推荐：**

```objc
NSInteger arrayCount = self.array.count;
[view setBackgroundColor:[UIColor orangeColor]];
UIApplication.sharedApplication.delegate;
```

## <a name="文字量"></a>文字量

`NSString`，`NSDictionary`，`NSArry`，`NSNumber`如果可能的话，尽量使用它们的不可变实例。`NSArray`和`NSDictionary`不能存在`nil`值，否则会引起崩溃。

**推荐：**

```objc
NSArray *names = @[@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul"];
NSDictionary *productManagers = @{@"iPhone": @"Kate", @"iPad": @"Kamal", @"Mobile Web": @"Bill"};
NSNumber *shouldUseLiterals = @YES;
NSNumber *buildingStreetNumber = @10018;
```

**不推荐：**

```objc
NSArray *names = [NSArray arrayWithObjects:@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul", nil];
NSDictionary *productManagers = [NSDictionary dictionaryWithObjectsAndKeys: @"Kate", @"iPhone", @"Kamal", @"iPad", @"Bill", @"Mobile Web", nil];
NSNumber *shouldUseLiterals = [NSNumber numberWithBool:YES];
NSNumber *buildingStreetNumber = [NSNumber numberWithInteger:10018];
```

## <a name="常量"></a>常量

常量推荐内联的字符串或数字，推荐定义为`static`变量，除非要作为宏，不然不要使用`#define`。

**推荐：**

```objc
static NSString * const RWTAboutViewControllerCompanyName = @"RayWenderlich.com";

static CGFloat const RWTImageThumbnailHeight = 50.0;
```

**不推荐：**

```objc
#define CompanyName @"RayWenderlich.com"

#define thumbnailHeight 2
```

## <a name="枚举类型"></a>枚举类型

当使用枚举型时，推荐使用新的固定基础类型定义，理由是有更强的类型检查和代码完成功能。SDK现在包含了一个宏来确保和推广使用固定基础类型：`NS_ENUM()`。

**例如：**

```objc
typedef NS_ENUM(NSInteger, RWTLeftMenuTopItemType) {
  RWTLeftMenuTopItemMain,
  RWTLeftMenuTopItemShows,
  RWTLeftMenuTopItemSchedule
};
```

也可以做明确的赋值

```objc
typedef NS_ENUM(NSInteger, RWTGlobalConstants) {
  RWTPinSizeMin = 1,
  RWTPinSizeMax = 5,
  RWTPinCountMin = 100,
  RWTPinCountMax = 500,
};
```

旧式的k型常量定义只被用于书写CoreFoundation的C代码中。

**不推荐：**

```objc
enum GlobalConstants {
  kMaxPinSize = 5,
  kMaxPinCount = 500,
};
```

## <a name="分支语句"></a>分支语句

花括号对于分支语句并不是必须的，除非编译器强制使用。
当一个分支包含多于一条语句，花括号需要添加。

```objc
switch (condition) {
  case 1:
    // ...
    break;
  case 2: {
    // ...
    // Multi-line example using braces
    break;
  }
  case 3:
    // ...
    break;
  default: 
    // ...
    break;
}
```

有时会出现一段代码被多个分支使用，这时就相当于“穿过”。一个“穿过”就是移除这一分支的‘break’语句，使代码继续执行下一分支。“穿过”这种情况需要在代码中进行注释。

```objc
switch (condition) {
  case 1:
    // ** fall-through! **
  case 2:
    // code executed for values 1 and 2
    break;
  default: 
    // ...
    break;
}
```

当在`switch`语句中使用枚举类型，`default`是不需要的：

```objc
RWTLeftMenuTopItemType menuType = RWTLeftMenuTopItemMain;
switch (menuType) {
  case RWTLeftMenuTopItemMain:
    // ...
    break;
  case RWTLeftMenuTopItemShows:
    // ...
    break;
  case RWTLeftMenuTopItemSchedule:
    // ...
    break;
}
```

## <a name="私有变量"></a>私有变量

私有变量应该定义到`.m`文件中的类扩展（匿名分类）中，命名的分类（如`RWTPrivate`，`private`）是不允许使用的，除非是在做另一个类的拓展。匿名分类可以暴露和共享于与`+Private.h`文件的命名惯例测试中。

**例如：**

```objc
@interface RWTDetailViewController ()

@property (strong, nonatomic) GADBannerView *googleAdView;
@property (strong, nonatomic) ADBannerView *iAdView;
@property (strong, nonatomic) UIWebView *adXWebView;

@end
```

## <a name="布尔型"></a>布尔型

Objective-C使用`YES`和`NO`。因此`true`和`false`只用于CoreFoundation，C和C++中。由于`nil`意味着`NO`，拿`nil`来做比较条件是不允许的，永远不要直接拿`YES`来比较，因为`YES`被定义为1，而一个布尔型可以支持到8比特。

这是为了文件间更多的一致性和更好的可读性。

**推荐：**

```objc
if (someObject) {}
if (![anotherObject boolValue]) {}
```

**不推荐：**

```objc
if (someObject == nil) {}
if ([anotherObject boolValue] == NO) {}
if (isAwesome == YES) {} // Never do this.
if (isAwesome == true) {} // Never do this.
```

如果布尔型被命名为一个形容词，属性名可以省略`is`，但是getter方法要保持这一命名。

```objc
@property (assign, getter=isEditable) BOOL editable;
```

这一部分来自[Cocoa Naming Guidelines](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingIvarsAndTypes.html#//apple_ref/doc/uid/20001284-BAJGIIJE)。

## <a name="条件语句"></a>条件语句

条件语句主体任何时候都要使用花括号，即使是只有一条语句也需要。这是为了避免错误。这些错误包括添加下一条语句，以为这条语句位于if主体中。另一个更危险的缺点是，if主体内的语句被注释掉，这时下一条语句无意中成为了if语句中的一部分。除此之外，这样的风格与其他条件语句格式保持了一致，便于查找。

**推荐：**

```objc
if (!error) {
  return success;
}
```

**不推荐：**

```objc
if (!error)
  return success;
```

或

```objc
if (!error) return success;
```

## <a name="三元运算符"></a>三元运算符

三元运算符`?:`，只有在可以提高可读性和简洁性时才可使。单一的判断条件一般可以使用，当执行多个判断条件时推荐使用`if`语句提高可读性，或者将条件重构为实例变量。总的来说，使用三元运算符的最好时机是决定如何给一个变量赋值的时候。

非布尔类型变量需要比较时，要添加`()`提高可读性。如果是布尔类型，则不需要。

**推荐：**

```objc
NSInteger value = 5;
result = (value != 0) ? x : y;

BOOL isHorizontal = YES;
result = isHorizontal ? x : y;
```

**不推荐：**

```objc
result = a > b ? x = c > d ? c : d : y;
```

## <a name="Init方法"></a>Init方法

Init方法遵守Apple生成的代码模板，`instancetype`应取代`id`作为返回值。

```objc
- (instancetype)init {
  self = [super init];
  if (self) {
    // ...
  }
  return self;
}
```

关于`instancetype`参照[Class Constructor Methods](#class-constructor-methods)。

## <a name="构造类方法"></a>构造类方法

当类构造方法使用时，同样需要注意返回值使用`instancetype`，而不是`id`，这样可确保编译器得知正确的结果类型。

```objc
@interface Airplane
+ (instancetype)airplaneWithType:(RWTAirplaneType)type;
@end
```

更多关于`instancetype`在[NSHipster.com](http://nshipster.com/instancetype/)。

## <a name="CGRect相关函数"></a>CGRect相关函数

当读取一个`CGRect`的`x`、`y`、`width`、`height`时，要使用`CGGeometry`相关的函数，而不是直接读取结构体。参照Apple的相关文档：

>All functions described in this reference that take CGRect data structures as inputs implicitly standardize those rectangles before calculating their results. For this reason, your applications should avoid directly reading and writing the data stored in the CGRect data structure. Instead, use the functions described here to manipulate rectangles and to retrieve their characteristics.

**推荐：**

```objc
CGRect frame = self.view.frame;

CGFloat x = CGRectGetMinX(frame);
CGFloat y = CGRectGetMinY(frame);
CGFloat width = CGRectGetWidth(frame);
CGFloat height = CGRectGetHeight(frame);
CGRect frame = CGRectMake(0.0, 0.0, width, height);
```

**不推荐：**

```objc
CGRect frame = self.view.frame;

CGFloat x = frame.origin.x;
CGFloat y = frame.origin.y;
CGFloat width = frame.size.width;
CGFloat height = frame.size.height;
CGRect frame = (CGRect){ .origin = CGPointZero, .size = frame.size };
```

## <a name="愉快路径"></a>愉快路径

进行条件编程时，左边缘的代码应该是愉快路径。也就是说，不要嵌套`if`语句。多个`return`是允许的。

**推荐：**

```objc
- (void)someMethod {
  if (![someOther boolValue]) {
	return;
  }

  //Do something important
}
```

**不推荐：**

```objc
- (void)someMethod {
  if ([someOther boolValue]) {
    //Do something important
  }
}
```

## <a name="异常处理"></a>异常处理

当方法通过引用的方式返回一个错误参数，使用返回值进行判断，而不是那个错误参数。

**推荐：**

```objc
NSError *error;
if (![self trySomethingWithError:&error]) {
  // Handle Error
}
```

**不推荐：**

```objc
NSError *error;
[self trySomethingWithError:&error];
if (error) {
  // Handle Error
}
```

一些Apple的API在成功的情况下向错误参数写入一些垃圾值，所以通过错误参数来判断会带来不良的影响。

## <a name="单例"></a>单例

单例对象生成共享实例时要注意线程安全。

```objc
+ (instancetype)sharedInstance {
  static id sharedInstance = nil;

  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    sharedInstance = [[self alloc] init];
  });

  return sharedInstance;
}
```

这将避免一些[多线程下的应用崩溃](http://cocoasamurai.blogspot.com/2011/04/singletons-your-doing-them-wrong.html)。

## <a name="换行"></a>换行

换行是一个重要的部分，本代码风格规则着重打印和在线的可读性。

例如：

```objc
self.productsRequest = [[SKProductsRequest alloc] initWithProductIdentifiers:productIdentifiers];
```

长代码的话，可以进行换行，在第二行开头遵照“空白”一节的规定，需要空两个空格。

```objc
self.productsRequest = [[SKProductsRequest alloc] 
  initWithProductIdentifiers:productIdentifiers];
```

## <a name="笑脸"></a>笑脸

笑脸是raywenderlich.com站点代码风格的显著特征。使用正确的笑脸表现编程时巨大的喜悦和激动是很重要的。使用方括号笑脸，是因为它是使用ASCII能获得的最大的笑脸···。使用以圆括号结尾的笑脸会出现一个半心形，所以不被推荐···。

**推荐：**

```objc
:]
```

**不推荐：**

```objc
:)
``` 

## <a name="Xcode工程"></a>Xcode工程

物理文件结构和Xcode工程文件结构要保持一致。Xcode项目中的新建分组都要对应文件系统中的文件夹。代码不但要按类型，也要按特征来分组，保证结构更清晰。

可能的话，尽量打开"Treat Warnings as Errors"选项，如果你想忽略一个类型的警告，请查看[这里](http://clang.llvm.org/docs/UsersManual.html#controlling-diagnostics-via-pragmas)。

## 其他Objective-C的代码风格规范

* [Robots & Pencils](https://github.com/RobotsAndPencils/objective-c-style-guide)
* [New York Times](https://github.com/NYTimes/objective-c-style-guide)
* [Google](http://google-styleguide.googlecode.com/svn/trunk/objcguide.xml)
* [GitHub](https://github.com/github/objective-c-conventions)
* [Adium](https://trac.adium.im/wiki/CodingStyle)
* [Sam Soffes](https://gist.github.com/soffes/812796)
* [CocoaDevCentral](http://cocoadevcentral.com/articles/000082.php)
* [Luke Redpath](http://lukeredpath.co.uk/blog/my-objective-c-style-guide.html)
* [Marcus Zarra](http://www.cimgf.com/zds-code-style-guide/)


