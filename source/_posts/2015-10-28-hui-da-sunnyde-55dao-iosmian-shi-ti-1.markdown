---
layout: post
title: "回答Sunny的55道iOS面试题1"
date: 2015-10-28 15:32:57 +0800
comments: true
categories: iOS
published: true
---

百度知道的[微博@我就叫Sunny怎么了](http://weibo.com/u/1364395395)出了55道iOS的面试题，很多题目是很有深度，现在自己尝试解一下这些题，并付上大神给出的答案。

<!--more-->

###1. 风格纠错题
![enter image description here](http://i.imgur.com/O7Zev94.png)

**我的解答：**

```objectivec
typedef NS_ENUM(NSInteger, CCUserSex) {
	CCUserSexMan = 0,
	CCUserSexWoman
}
@interface CCUser : NSObject
@property(nonatomic, copy)NSString *name;
@property(nonatomic, assign)NSUInteger age;
@property(nonatomic, assign)CCUserSex sex;
+(instancetype)userWithName:(NSString *)name age:(NSUInteger)age;
-(instancetype)initWithName:(NSString *)name age:(NSUInteger)age;

-(void)login;
```

**大神解答：**

```objectivec
// .h文件
// http://weibo.com/luohanchenyilong/
// https://github.com/ChenYilong
// 修改完的代码，这是第一种修改方法，后面会给出第二种修改方法

typedef NS_ENUM(NSInteger, CYLSex) {
    CYLSexMan,
    CYLSexWoman
};

@interface CYLUser : NSObject<NSCopying>

@property (nonatomic, readonly, copy) NSString *name;
@property (nonatomic, readonly, assign) NSUInteger age;
@property (nonatomic, readonly, assign) CYLSex sex;

- (instancetype)initWithName:(NSString *)name age:(NSUInteger)age sex:(CYLSex)sex;
+ (instancetype)userWithName:(NSString *)name age:(NSUInteger)age sex:(CYLSex)sex;

@end
```
 
 详解：
 
 1.enum建议使用NS_ENUM和NS_OPTIONS来定义，这是Apple推荐的枚举定义宏，主要是可以通过它来指定type的类型。  
 2.数值型变量尽量使用Foundation的数据类型，而不要使用c的类型，主要是为了适配不同处理器而考虑的。  
 3.大的工程必须在类前加前缀作为命名空间。  
 4.命名习惯，通常情况下，即使有类似 withA:withB: 的命名需求，也通常是使用withA:andB: 这种命名，用来表示方法执行了两个相对独立的操作（从设计上来说，这时候也可以拆分成两个独立的方法），它不应该用作阐明有多个参数，比如下面的：  
 
```objectivec
 //错误，不要使用"and"来连接参数
- (int)runModalForDirectory:(NSString *)path andFile:(NSString *)name andTypes:(NSArray *)fileTypes;
//错误，不要使用"and"来阐明有多个参数
- (instancetype)initWithName:(CGFloat)name andAge:(CGFloat)age;
//正确，使用"and"来表示两个相对独立的操作
- (BOOL)openFile:(NSString *)fullPath withApplication:(NSString *)appName andDeactivate:(BOOL)flag;
```
 
 5.一些有可变类型的类型，如NSString、NSArray、NSDictionary，经常使用copy来命名property，为了避免将可变类型赋给当前对象，然后在外部改变数值，而对当前类型产生影响。  
 6.OC有designated和secondary初始化方法的概念。 designated初始化方法是提供所有的参数，secondary初始化方法是一个或多个，并且提供一个或者更多的默认参数来调用designated初始化方法的初始化方法。子类在继承父类时，可以添加新的secondary初始化方法，但必须在内部调用父类的designated初始化方法，而且也必须重写父类暴露的secondary初始化方法。  
 
###2. 什么情况使用 weak 关键字，相比 assign 有什么不同？
 
 **我的解答：**  
weak关键字一般用于xib导出的IBOutlet类型控件的变量声明，和delegate的声明，主要用于避免循环引用。相比assign，weak也是不会改变对象的retain count，但是weak只能用于object的property声明，而assign一般用于数值型变量的property声明。  
 
 **大神解答：**  
  
 1. 在 ARC 中,在有可能出现循环引用的时候,往往要通过让其中一端使用 weak 来解决,比如: delegate 代理属性  
 2. 自身已经对它进行一次强引用,没有必要再强引用一次,此时也会使用 weak,自定义 IBOutlet 控件属性一般也使用 weak；当然，也可以使用strong。

不同点：
 
 1. `weak` 此特质表明该属性定义了一种“非拥有关系” (nonowning relationship)。为这种属性设置新值时，设置方法既不保留新值，也不释放旧值。此特质同assign类似，
然而在属性所指的对象遭到摧毁时，属性值也会清空(nil out)。
而 `assign` 的“设置方法”只会执行针对“纯量类型” (scalar type，例如 CGFloat 或 
NSlnteger 等)的简单赋值操作。

 2. assigin 可以用非 OC 对象,而 weak 必须用于 OC 对象
 
###3. 怎么用 copy 关键字？

**我的解答：**  

1. copy一般用于property时是用于NSString、NSArray、NSDictionary这些有可变类型，原因为了避免将可变类型赋给当前对象，然后在外部改变数值，而对当前类型产生影响。
2. 然后也经常适用于block类型的property赋值，原因是block声明时是被存放在栈中，如果不使用copy对其操作，将其拷贝到堆上，那么该block会在其声明的作用域失效时被释放。

**大神解答：**  

 1. NSString、NSArray、NSDictionary 等等经常使用copy关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary；
 2. block 也经常使用 copy 关键字，具体原因见[官方文档：***Objects Use Properties to Keep Track of Blocks***](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithBlocks/WorkingwithBlocks.html#//apple_ref/doc/uid/TP40011210-CH8-SW12)：

  block 使用 copy 是从 MRC 遗留下来的“传统”,在 MRC 中,方法内部的 block 是在栈区的,使用 copy 可以把它放到堆区.在 ARC 中写不写都行：对于 block 使用 copy 还是 strong 效果是一样的，但写上 copy 也无伤大雅，还能时刻提醒我们：编译器自动对 block 进行了 copy 操作。如果不写 copy ，该类的调用者有可能会忘记或者根本不知道“编译器会自动对 block 进行了 copy 操作”，他们有可能会在调用之前自行拷贝属性值。这种操作多余而低效。
  
![enter image description here](http://i.imgur.com/VlVKl8L.png)

下面做下解释：
 copy 此特质所表达的所属关系与 strong 类似。然而设置方法并不保留新值，而是将其“拷贝” (copy)。
当属性类型为 NSString 时，经常用此特质来保护其封装性，因为传递给设置方法的新值有可能指向一个 NSMutableString 类的实例。这个类是 NSString 的子类，表示一种可修改其值的字符串，此时若是不拷贝字符串，那么设置完属性之后，字符串的值就可能会在对象不知情的情况下遭人更改。所以，这时就要拷贝一份“不可变” (immutable)的字符串，确保对象中的字符串值不会无意间变动。只要实现属性所用的对象是“可变的” (mutable)，就应该在设置新属性值时拷贝一份。


> 用 `@property` 声明 NSString、NSArray、NSDictionary 经常使用 copy 关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary，他们之间可能进行赋值操作，为确保对象中的字符串值不会无意间变动，应该在设置新属性值时拷贝一份。

###4. 这个写法会出什么问题： `@property (copy) NSMutableArray *array;`

**我的解答：**  

对NSMutableArray执行copy操作，会返回NSArray类型的对象，那么给array赋值一个NSMutabeArray后，再对其经行修改操作，会出现错误。

**大神解答：**

两个问题：1、添加,删除,修改数组内的元素的时候,程序会因为找不到对应的方法而崩溃.因为 copy 就是复制一个不可变 NSArray 的对象；2、使用了 atomic 属性会严重影响性能 ； 

比如下面的代码就会发生崩溃
 
```objectivec
// .h文件
// http://weibo.com/luohanchenyilong/
// https://github.com/ChenYilong
// 下面的代码就会发生崩溃

@property (nonatomic, copy) NSMutableArray *mutableArray;
```

```objectivec
// .m文件
// http://weibo.com/luohanchenyilong/
// https://github.com/ChenYilong
// 下面的代码就会发生崩溃

NSMutableArray *array = [NSMutableArray arrayWithObjects:@1,@2,nil];
self.mutableArray = array;
[self.mutableArray removeObjectAtIndex:0];
```

接下来就会崩溃：
 
```objectivec
 -[__NSArrayI removeObjectAtIndex:]: unrecognized selector sent to instance 0x7fcd1bc30460
```

第2条原因，如下：

> 该属性使用了同步锁，会在创建时生成一些额外的代码用于帮助编写多线程程序，这会带来性能问题，通过声明 nonatomic 可以节省这些虽然很小但是不必要额外开销。

在默认情况下，由编译器所合成的方法会通过锁定机制确保其原子性(atomicity)。如果属性具备 nonatomic 特质，则不使用同步锁。请注意，尽管没有名为“atomic”的特质(如果某属性不具备 nonatomic 特质，那它就是“原子的”(atomic))。

在iOS开发中，你会发现，几乎所有属性都声明为 nonatomic。

一般情况下并不要求属性必须是“原子的”，因为这并不能保证“线程安全” ( thread safety)，若要实现“线程安全”的操作，还需采用更为深层的锁定机制才行。例如，一个线程在连续多次读取某属性值的过程中有别的线程在同时改写该值，那么即便将属性声明为 atomic，也还是会读到不同的属性值。

因此，开发iOS程序时一般都会使用 nonatomic 属性。但是在开发 Mac OS X 程序时，使用
 atomic 属性通常都不会有性能瓶颈。

###5. 如何让自己的类用 copy 修饰符？如何重写带 copy 关键字的 setter？

**我的解答：**  

1.要让自己的类实现NSCopying协议的方法，也就是**- (id)copyWithZone:(NSZone *)zone;**方法。  
2.重写copy的setter：  

```objectivec
-(void)setCopyProperty:(id)copyProperty{
	_copyProperty = [copyProperty copy];
}
```

**大神解答：**

> 若想令自己所写的对象具有拷贝功能，则需实现 NSCopying 协议。如果自定义的对象分为可变版本与不可变版本，那么就要同时实现 `NSCopying` 与 `NSMutableCopying` 协议。




具体步骤：

 1. 需声明该类遵从 NSCopying 协议
 2. 实现 NSCopying 协议。该协议只有一个方法: 

```objectivec
- (id)copyWithZone:(NSZone *)zone;
```
注意：一提到让自己的类用 copy 修饰符，我们总是想覆写copy方法，其实真正需要实现的却是 “copyWithZone” 方法。

以第一题的代码为例：
   

```objectivec
	// .h文件
	// http://weibo.com/luohanchenyilong/
	// https://github.com/ChenYilong
	// 修改完的代码

	typedef NS_ENUM(NSInteger, CYLSex) {
	    CYLSexMan,
	    CYLSexWoman
	};

	@interface CYLUser : NSObject<NSCopying>

	@property (nonatomic, readonly, copy) NSString *name;
	@property (nonatomic, readonly, assign) NSUInteger age;
	@property (nonatomic, readonly, assign) CYLSex sex;

	- (instancetype)initWithName:(NSString *)name age:(NSUInteger)age sex:(CYLSex)sex;
	+ (instancetype)userWithName:(NSString *)name age:(NSUInteger)age sex:(CYLSex)sex;

	@end
```


然后实现协议中规定的方法：

 
```objectivec
- (id)copyWithZone:(NSZone *)zone {
	CYLUser *copy = [[[self class] allocWithZone:zone] 
		             initWithName:_name
 							      age:_age
						          sex:_sex];
	return copy;
}
```
但在实际的项目中，不可能这么简单，遇到更复杂一点，比如类对象中的数据结构可能并未在初始化方法中设置好，需要另行设置。举个例子，假如 CYLUser 中含有一个数组，与其他 CYLUser 对象建立或解除朋友关系的那些方法都需要操作这个数组。那么在这种情况下，你得把这个包含朋友对象的数组也一并拷贝过来。下面列出了实现此功能所需的全部代码:

```objectivec
// .h文件
// http://weibo.com/luohanchenyilong/
// https://github.com/ChenYilong
// 以第一题《风格纠错题》里的代码为例

typedef NS_ENUM(NSInteger, CYLSex) {
    CYLSexMan,
    CYLSexWoman
};

@interface CYLUser : NSObject<NSCopying>

@property (nonatomic, readonly, copy) NSString *name;
@property (nonatomic, readonly, assign) NSUInteger age;
@property (nonatomic, readonly, assign) CYLSex sex;

- (instancetype)initWithName:(NSString *)name age:(NSUInteger)age sex:(CYLSex)sex;
+ (instancetype)userWithName:(NSString *)name age:(NSUInteger)age sex:(CYLSex)sex;
- (void)addFriend:(CYLUser *)user;
- (void)removeFriend:(CYLUser *)user;

@end
```

// .m文件



```objectivec
// .m文件
// http://weibo.com/luohanchenyilong/
// https://github.com/ChenYilong
//

@implementation CYLUser {
    NSMutableSet *_friends;
}

- (void)setName:(NSString *)name {
    _name = [name copy];
}

- (instancetype)initWithName:(NSString *)name
                         age:(NSUInteger)age
                         sex:(CYLSex)sex {
    if(self = [super init]) {
        _name = [name copy];
        _age = age;
        _sex = sex;
        _friends = [[NSMutableSet alloc] init];
    }
    return self;
}

- (void)addFriend:(CYLUser *)user {
    [_friends addObject:user];
}

- (void)removeFriend:(CYLUser *)user {
    [_friends removeObject:person];
}

- (id)copyWithZone:(NSZone *)zone {
    CYLUser *copy = [[[self class] allocWithZone:zone]
                     initWithName:_name
                     age:_age
                     sex:_sex];
    copy->_friends = [_friends mutableCopy];
    return copy;
}

- (id)deepCopy {
    CYLUser *copy = [[[self class] allocWithZone:zone]
                     initWithName:_name
                     age:_age
                     sex:_sex];
    copy->_friends = [[NSMutableSet alloc] initWithSet:_friends
                                             copyItems:YES];
    return copy;
}

@end

```

以上做法能满足基本的需求，但是也有缺陷：

> 如果你所写的对象需要深拷贝，那么可考虑新增一个专门执行深拷贝的方法。

【注：深浅拷贝的概念，在下文中有介绍，详见下文的：***用@property声明的 NSString（或NSArray，NSDictionary）经常使用 copy 关键字，为什么？如果改用 strong 关键字，可能造成什么问题？***】

在例子中，存放朋友对象的 set 是用 “copyWithZone:” 方法来拷贝的，这种浅拷贝方式不会逐个复制 set 中的元素。若需要深拷贝的话，则可像下面这样，编写一个专供深拷贝所用的方法:
	

```objectivec
- (id)deepCopy {
    CYLUser *copy = [[[self class] allocWithZone:zone]
                     initWithName:_name
                     age:_age
                     sex:_sex];
    copy->_friends = [[NSMutableSet alloc] initWithSet:_friends
                                             copyItems:YES];
    return copy;
}

```

至于***如何重写带 copy 关键字的 setter***这个问题，


如果抛开本例来回答的话，如下：


 
```objectivec
- (void)setName:(NSString *)name {
    //[_name release];
    _name = [name copy];
}
```

不过也有争议，有人说“苹果如果像下面这样干，是不是效率会高一些？”


```objectivec
- (void)setName:(NSString *)name {
    if (_name != name) {
        //[_name release];//MRC
        _name = [name copy];
    }
}
```



这样真得高效吗？不见得！这种写法“看上去很美、很合理”，但在实际开发中，它更像下图里的做法：

![enter image description here](http://i.imgur.com/UwV9oSn.jpeg)

克强总理这样评价你的代码风格：

![enter image description here](http://i.imgur.com/N77Lkic.png)

我和总理的意见基本一致：


> 老百姓 copy 一下，咋就这么难？




你可能会说：

 
之所以在这里做`if判断` 这个操作：是因为一个 if 可能避免一个耗时的copy，还是很划算的。
(在刚刚讲的：《如何让自己的类用 copy 修饰符？》里的那种复杂的copy，我们可以称之为 “耗时的copy”，但是对 NSString 的 copy 还称不上。)


但是你有没有考虑过代价：


> 你每次调用 `setX:` 都会做 if 判断，这会让 `setX:` 变慢，如果你在 `setX:`写了一串复杂的 `if+elseif+elseif+...` 判断，将会更慢。

要回答“哪个效率会高一些？”这个问题，不能脱离实际开发，就算 copy 操作十分耗时，if 判断也不见得一定会更快，除非你把一个“ @property他当前的值 ”赋给了他自己，代码看起来就像：

```objectivec
[a setX:x1];
[a setX:x1];    //你确定你要这么干？与其在setter中判断，为什么不把代码写好？
```

或者


```objectivec
[a setX:[a x]];   //队友咆哮道：你在干嘛？！！
```

> 不要在 setter 里进行像 `if(_obj != newObj)` 这样的判断。（该观点参考链接：[ ***How To Write Cocoa Object Setters： Principle 3: Only Optimize After You Measure*** ](http://vgable.com/blog/tag/autorelease/)
）


什么情况会在 copy setter 里做 if 判断？
例如，车速可能就有最高速的限制，车速也不可能出现负值，如果车子的最高速为300，则 setter 的方法就要改写成这样：

 
```objectivec
-(void)setSpeed:(int)_speed{
    if(_speed < 0) speed = 0;
    if(_speed > 300) speed = 300;
    _speed = speed;
}
```



回到这个题目，如果单单就上文的代码而言，我们不需要也不能重写 name 的 setter ：由于是 name 是只读属性，所以编译器不会为其创建对应的“设置方法”，用初始化方法设置好属性值之后，就不能再改变了。（ 在本例中，之所以还要声明属性的“内存管理语义”--copy，是因为：如果不写 copy，该类的调用者就不知道初始化方法里会拷贝这些属性，他们有可能会在调用初始化方法之前自行拷贝属性值。这种操作多余而低效）。

那如何确保 name 被 copy？在初始化方法(initializer)中做：

```objectivec
	- (instancetype)initWithName:(NSString *)name 
								 age:(NSUInteger)age 
								 sex:(CYLSex)sex {
	     if(self = [super init]) {
	     	_name = [name copy];
	     	_age = age;
	     	_sex = sex;
	     	_friends = [[NSMutableSet alloc] init];
	     }
	     return self;
	}

```

###6. @property 的本质是什么？ivar、getter、setter 是如何生成并添加到这个类中的

**我的解答：**

1. @property本质就是自动为变量创建对应的setter，getter的快捷语法。  
2. 额，不会。

**大神解答：**

**@property 的本质是什么？**

> @property = ivar + getter + setter;

下面解释下：

> “属性” (property)有两大概念：ivar（实例变量）、存取方法（access method ＝ getter + setter）。

“属性” (property)作为 objectivec 的一项特性，主要的作用就在于封装对象中的数据。 objectivec 对象通常会把其所需要的数据保存为各种实例变量。实例变量一般通过“存取方法”(access method)来访问。其中，“获取方法” (getter)用于读取变量值，而“设置方法” (setter)用于写入变量值。这个概念已经定型，并且经由“属性”这一特性而成为 `objectivec 2.0` 的一部分。
而在正规的 objectivec 编码风格中，存取方法有着严格的命名规范。
正因为有了这种严格的命名规范，所以 objectivec 这门语言才能根据名称自动创建出存取方法。其实也可以把属性当做一种关键字，其表示:

> 编译器会自动写出一套存取方法，用以访问给定类型中具有给定名称的变量。
所以你也可以这么说：

> @property = getter + setter;

例如下面这个类：



```objectivec
@interface Person : NSObject
@property NSString *firstName;
@property NSString *lastName;
@end
```


上述代码写出来的类与下面这种写法等效：



```objectivec
@interface Person : NSObject
- (NSString *)firstName;
- (void)setFirstName:(NSString *)firstName;
- (NSString *)lastName;
- (void)setLastName:(NSString *)lastName;
@end
```





**ivar、getter、setter 是如何生成并添加到这个类中的?**

> “自动合成”( autosynthesis)

完成属性定义后，编译器会自动编写访问这些属性所需的方法，此过程叫做“自动合成”(autosynthesis)。需要强调的是，这个过程由编译
器在编译期执行，所以编辑器里看不到这些“合成方法”(synthesized method)的源代码。除了生成方法代码 getter、setter 之外，编译器还要自动向类中添加适当类型的实例变量，并且在属性名前面加下划线，以此作为实例变量的名字。在前例中，会生成两个实例变量，其名称分别为
 `_firstName` 与 `_lastName`。也可以在类的实现代码里通过
 `@synthesize` 语法来指定实例变量的名字.



```objectivec
@implementation Person
@synthesize firstName = _myFirstName;
@synthesize lastName = _myLastName;
@end
```

我为了搞清属性是怎么实现的,曾经反编译过相关的代码,他大致生成了五个东西

 1. `OBJC_IVAR_$类名$属性名称` ：该属性的“偏移量” (offset)，这个偏移量是“硬编码” (hardcode)，表示该变量距离存放对象的内存区域的起始地址有多远。
 2. setter 与 getter 方法对应的实现函数
 2. `ivar_list` ：成员变量列表
 2. `method_list` ：方法列表
 2. `prop_list` ：属性列表


也就是说我们每次在增加一个属性,系统都会在 `ivar_list` 中添加一个成员变量的描述,在 `method_list` 中增加 setter 与 getter 方法的描述,在属性列表中增加一个属性的描述,然后计算该属性在对象中的偏移量,然后给出 setter 与 getter 方法对应的实现,在 setter 方法中从偏移量的位置开始赋值,在 getter 方法中从偏移量开始取值,为了能够读取正确字节数,系统对象偏移量的指针类型进行了类型强转.

###7. @protocol 和 category 中如何使用 @property

**我的解答：**

理论上说@protocol和category是不能使用@property，主要是声明property后，不能自动合成setter、getter方法，所以无法使用变量，不过确实存在需要时，可以使用associated objects技术为其绑定对象，作为实例变量的代替。  

**大神解答：**

 1. 在 protocol 中使用 property 只会生成 setter 和 getter 方法声明,我们使用属性的目的,是希望遵守我协议的对象能实现该属性
 2. category 使用 @property 也是只会生成 setter 和 getter 方法的声明,如果我们真的需要给 category 增加属性的实现,需要借助于运行时的两个函数：

  1. `objc_setAssociatedObject`
  2. `objc_getAssociatedObject`
  
###8. runtime 如何实现 weak 属性

**我的解答：**

额，不会。

**大神解答：**

要实现 weak 属性，首先要搞清楚 weak 属性的特点：

> weak 此特质表明该属性定义了一种“非拥有关系” (nonowning relationship)。为这种属性设置新值时，设置方法既不保留新值，也不释放旧值。此特质同 assign 类似， 然而在属性所指的对象遭到摧毁时，属性值也会清空(nil out)。

那么 runtime 如何实现 weak 变量的自动置nil？


> runtime 对注册的类， 会进行布局，对于 weak 对象会放入一个 hash 表中。 用 weak 指向的对象内存地址作为 key，当此对象的引用计数为0的时候会 dealloc，假如 weak 指向的对象内存地址是a，那么就会以a为键， 在这个 weak 表中搜索，找到所有以a为键的 weak 对象，从而设置为 nil。

###9. @property中有哪些属性关键字？/ @property 后面可以有哪些修饰符？

**我的解答：**

有atomic、nonatomic、retain、assign、copy、strong（ARC）、weak（ARC）、readonly、readwrite，另外有指定setter和getter方法的关键字setter=setXXX、getter=xxx，最后一个在BOOL型对象时常用getter=isXXX来替代getter方法。

**大神解答：**

属性可以拥有的特质分为四类:
 
 1. 原子性--- `nonatomic` 特质

    在默认情况下，由编译器合成的方法会通过锁定机制确保其原子性(atomicity)。如果属性具备 nonatomic 特质，则不使用同步锁。请注意，尽管没有名为“atomic”的特质(如果某属性不具备 nonatomic 特质，那它就是“原子的” ( atomic) )，但是仍然可以在属性特质中写明这一点，编译器不会报错。若是自己定义存取方法，那么就应该遵从与属性特质相符的原子性。

 2. 读/写权限---`readwrite(读写)`、`readonly (只读)`
 3. 内存管理语义---`assign`、`strong`、 `weak`、`unsafe_unretained`、`copy`
 4. 方法名---`getter=<name>` 、`setter=<name>`
   
  `getter=<name>`的样式：


```objectivec
        @property (nonatomic, getter=isOn) BOOL on;
```
 <p><del>（ `setter=<name>`这种不常用，也不推荐使用。故不在这里给出写法。）
</del></p>


 `setter=<name>`一般用在特殊的情境下，比如：


在数据反序列化、转模型的过程中，服务器返回的字段如果以 `init` 开头，所以你需要定义一个 `init` 开头的属性，但默认生成的 `setter` 与 `getter` 方法也会以 `init` 开头，而编译器会把所有以 `init` 开头的方法当成初始化方法，而初始化方法只能返回 self 类型，因此编译器会报错。

这时你就可以使用下面的方式来避免编译器报错：


```objectivec
@property(nonatomic, strong, getter=p_initBy, setter=setP_initBy:)NSString *initBy;

```


另外也可以用关键字进行特殊说明，来避免编译器报错：

```objectivec
@property(nonatomic, readwrite, copy, null_resettable) NSString *initBy;
- (NSString *)initBy __attribute__((objc_method_family(none)));
```

 3. 不常用的：`nonnull`,`null_resettable`,`nullable`

###10. weak属性需要在dealloc中置nil么？

**我的解答：**

不需要，weak属性的变量，在其所指向内存释放后，会自动指向nil，成为空指针。

**大神解答：**

不需要。


> 在ARC环境无论是强指针还是弱指针都无需在 dealloc 设置为 nil ， ARC 会自动帮我们处理

即便是编译器不帮我们做这些，weak也不需要在 dealloc 中置nil：

正如上文的：***runtime 如何实现 weak 属性*** 中提到的：

我们模拟下 weak 的 setter 方法，应该如下：

```objectivec
- (void)setObject:(NSObject *)object
{
    objc_setAssociatedObject(self, "object", object, OBJC_ASSOCIATION_ASSIGN);
    [object cyl_runAtDealloc:^{
        _object = nil;
    }];
}
```

也即:

> 在属性所指的对象遭到摧毁时，属性值也会清空(nil out)。

###11. @synthesize和@dynamic分别有什么作用？

**我的解答：**

@synthesize用于合成声明的property的实例变量和其setter、getter方法，并可以在其后面声明对应的内部变量，目前已不需要手写@synthesize操作，系统会对property执行autosynthesize操作。而@dynamic声明的变量，是告诉系统该变量没有显式的setter、getter方法，但是会在运行时为其添加setter/getter方法，常用于CoreData变量的声明，不是很常用。

**大神解答：**

 1. @property有两个对应的词，一个是 @synthesize，一个是 @dynamic。如果 @synthesize和 @dynamic都没写，那么默认的就是`@syntheszie var = _var;`
 2. @synthesize 的语义是如果你没有手动实现 setter 方法和 getter 方法，那么编译器会自动为你加上这两个方法。
 3. @dynamic 告诉编译器：属性的 setter 与 getter 方法由用户自己实现，不自动生成。（当然对于 readonly 的属性只需提供 getter 即可）。假如一个属性被声明为 @dynamic var，然后你没有提供 @setter方法和 @getter 方法，编译的时候没问题，但是当程序运行到 `instance.var = someVar`，由于缺 setter 方法会导致程序崩溃；或者当运行到 `someVar = var` 时，由于缺 getter 方法同样会导致崩溃。编译时没问题，运行时才执行相应的方法，这就是所谓的动态绑定。
 
###12. ARC下，不显式指定任何属性关键字时，默认的关键字都有哪些？

**我的解答：**

默认添加的关键字有atomic，readwrite，assign。

**大神解答：**

1. 对应基本数据类型默认关键字是  
 atomic,readwrite,assign  
2. 对于普通的 objectivec 对象  
 atomic,readwrite,strong

参考链接：

 1. [ ***objectivec ARC: strong vs retain and weak vs assign*** ](http://stackoverflow.com/a/15541801/3395008)

 2. [ ***Variable property attributes or Modifiers in iOS*** ](http://rdcworld-iphone.blogspot.in/2012/12/variable-property-attributes-or.html)
 
###13. 用@property声明的NSString（或NSArray，NSDictionary）经常使用copy关键字，为什么？如果改用strong关键字，可能造成什么问题？

**我的解答：**

上面答过。

**大神解答：**

1. 因为父类指针可以指向子类对象,使用 copy 的目的是为了让本对象的属性不受外界影响,使用 copy 无论给我传入是一个可变对象还是不可对象,我本身持有的就是一个不可变的副本.
 2. 如果我们使用是 strong ,那么这个属性就有可能指向一个可变对象,如果这个可变对象在外部被修改了,那么会影响该属性.

 copy 此特质所表达的所属关系与 strong 类似。然而设置方法并不保留新值，而是将其“拷贝” (copy)。
当属性类型为 NSString 时，经常用此特质来保护其封装性，因为传递给设置方法的新值有可能指向一个 NSMutableString 类的实例。这个类是 NSString 的子类，表示一种可修改其值的字符串，此时若是不拷贝字符串，那么设置完属性之后，字符串的值就可能会在对象不知情的情况下遭人更改。所以，这时就要拷贝一份“不可变” (immutable)的字符串，确保对象中的字符串值不会无意间变动。只要实现属性所用的对象是“可变的” (mutable)，就应该在设置新属性值时拷贝一份。


举例说明：

定义一个以 strong 修饰的 array：

```objectivec
@property (nonatomic ,readwrite, strong) NSArray *array;
```

然后进行下面的操作：

```objectivec
    NSMutableArray *mutableArray = [[NSMutableArray alloc] init];
    NSArray *array = @[ @1, @2, @3, @4 ];
    self.array = mutableArray;
    [mutableArray removeAllObjects];;
    NSLog(@"%@",self.array);
    
    [mutableArray addObjectsFromArray:array];
    self.array = [mutableArray copy];
    [mutableArray removeAllObjects];;
    NSLog(@"%@",self.array);
```

打印结果如下所示：

```objectivec
2015-09-27 19:10:32.523 CYLArrayCopyDmo[10681:713670] (
)
2015-09-27 19:10:32.524 CYLArrayCopyDmo[10681:713670] (
    1,
    2,
    3,
    4
)
```

（详见仓库内附录的 Demo。）


为了理解这种做法，首先要知道，两种情况：


 1. 对非集合类对象的 copy 与 mutableCopy 操作；
 2. 对集合类对象的 copy 与 mutableCopy 操作。

####1. 对非集合类对象的copy操作：

在非集合类对象中：对 immutable 对象进行 copy 操作，是指针复制，mutableCopy 操作时内容复制；对 mutable 对象进行 copy 和 mutableCopy 都是内容复制。用代码简单表示如下：

 - [immutableObject copy] // 浅复制
 - [immutableObject mutableCopy] //深复制
 - [mutableObject copy] //深复制
 - [mutableObject mutableCopy] //深复制
	
比如以下代码：


```objectivec
NSMutableString *string = [NSMutableString stringWithString:@"origin"];//copy
NSString *stringCopy = [string copy];
```


查看内存，会发现 string、stringCopy 内存地址都不一样，说明此时都是做内容拷贝、深拷贝。即使你进行如下操作：


```objectivec
[string appendString:@"origion!"]
```

stringCopy 的值也不会因此改变，但是如果不使用 copy，stringCopy 的值就会被改变。
  集合类对象以此类推。
所以，

> 用 @property 声明 NSString、NSArray、NSDictionary 经常使用 copy 关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary，他们之间可能进行赋值操作，为确保对象中的字符串值不会无意间变动，应该在设置新属性值时拷贝一份。

####2、集合类对象的copy与mutableCopy

集合类对象是指 NSArray、NSDictionary、NSSet ... 之类的对象。下面先看集合类immutable对象使用 copy 和 mutableCopy 的一个例子：

```objectivec
NSArray *array = @[@[@"a", @"b"], @[@"c", @"d"]];
NSArray *copyArray = [array copy];
NSMutableArray *mCopyArray = [array mutableCopy];
```

查看内容，可以看到 copyArray 和 array 的地址是一样的，而 mCopyArray 和 array 的地址是不同的。说明 copy 操作进行了指针拷贝，mutableCopy 进行了内容拷贝。但需要强调的是：此处的内容拷贝，仅仅是拷贝 array 这个对象，array 集合内部的元素仍然是指针拷贝。这和上面的非集合 immutable 对象的拷贝还是挺相似的，那么mutable对象的拷贝会不会类似呢？我们继续往下，看 mutable 对象拷贝的例子：


```objectivec
NSMutableArray *array = [NSMutableArray arrayWithObjects:[NSMutableString stringWithString:@"a"],@"b",@"c",nil];
NSArray *copyArray = [array copy];
NSMutableArray *mCopyArray = [array mutableCopy];
```


查看内存，如我们所料，copyArray、mCopyArray和 array 的内存地址都不一样，说明 copyArray、mCopyArray 都对 array 进行了内容拷贝。同样，我们可以得出结论：

在集合类对象中，对 immutable 对象进行 copy，是指针复制， mutableCopy 是内容复制；对 mutable 对象进行 copy 和 mutableCopy 都是内容复制。但是：集合对象的内容复制仅限于对象本身，对象元素仍然是指针复制。用代码简单表示如下：


```objectivec
[immutableObject copy] // 浅复制
[immutableObject mutableCopy] //单层深复制
[mutableObject copy] //单层深复制
[mutableObject mutableCopy] //单层深复制
```


这个代码结论和非集合类的非常相似。

参考链接：[iOS 集合的深复制与浅复制](https://www.zybuluo.com/MicroCai/note/50592)

###14. @synthesize合成实例变量的规则是什么？假如property名为foo，存在一个名为`_foo`的实例变量，那么还会自动合成新变量么？

**我的解答：**

额，规则不知道，感觉不会合成新变量。

**大神解答：**

在回答之前先说明下一个概念：

> 实例变量 = 成员变量 ＝ ivar

这些说法，笔者下文中，可能都会用到，指的是一个东西。


正如
[Apple官方文档 ***You Can Customize Synthesized Instance Variable Names***](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/EncapsulatingData/EncapsulatingData.html#//apple_ref/doc/uid/TP40011210-CH5-SW6) 所说：
![enter image description here](http://i.imgur.com/D6d0zGJ.png)

如果使用了属性的话，那么编译器就会自动编写访问属性所需的方法，此过程叫做“自动合成”( auto synthesis)。需要强调的是，这个过程由编译器在编译期执行，所以编辑器里看不到这些“合成方法” (synthesized method)的源代码。除了生成方法代码之外，编译器还要自动向类中添加适当类型的实例变量，并且在属性名前面加下划线，以此作为实例变量的名字。

 
```objectivec
@interface CYLPerson : NSObject 
@property NSString *firstName; 
@property NSString *lastName; 
@end
```


在上例中，会生成两个实例变量，其名称分别为
 `_firstName` 与 `_lastName`。也可以在类的实现代码里通过 `@synthesize` 语法来指定实例变量的名字:
 
```objectivec
@implementation CYLPerson 
@synthesize firstName = _myFirstName; 
@synthesize lastName = _myLastName; 
@end 
```



上述语法会将生成的实例变量命名为 `_myFirstName` 与 `_myLastName` ，而不再使用默认的名字。一般情况下无须修改默认的实例变量名，但是如果你不喜欢以下划线来命名实例变量，那么可以用这个办法将其改为自己想要的名字。笔者还是推荐使用默认的命名方案，因为如果所有人都坚持这套方案，那么写出来的代码大家都能看得懂。

总结下 @synthesize 合成实例变量的规则，有以下几点：


 1. 如果指定了成员变量的名称,会生成一个指定的名称的成员变量,

 2. 如果这个成员已经存在了就不再生成了.
 2. 如果是 `@synthesize foo;` 还会生成一个名称为foo的成员变量，也就是说：

 > 如果没有指定成员变量的名称会自动生成一个属性同名的成员变量,



 2. 如果是 `@synthesize foo = _foo;` 就不会生成成员变量了.

假如 property 名为 foo，存在一个名为 `_foo` 的实例变量，那么还会自动合成新变量么？
不会。如下图：

![enter image description here](http://i.imgur.com/t28ge4W.png)

###15. 在有了自动合成属性实例变量之后，@synthesize还有哪些使用场景？

**我的解答：**

额，为了指定内部变量的名称会用到，其他呢？为啥就和@synthesize干上了？

**大神解答：**

回答这个问题前，我们要搞清楚一个问题，什么情况下不会autosynthesis（自动合成）？

 1. 同时重写了 setter 和 getter 时
 2. 重写了只读属性的 getter 时
 3. 使用了 @dynamic 时
 4. 在 @protocol 中定义的所有属性
 5. 在 category 中定义的所有属性
 6. 重载的属性 
 
当你在子类中重载了父类中的属性，你必须 使用 `@synthesize` 来手动合成ivar。

除了后三条，对其他几个我们可以总结出一个规律：当你想手动管理 @property 的所有内容时，你就会尝试通过实现 @property 的所有“存取方法”（the accessor methods）或者使用 `@dynamic` 来达到这个目的，这时编译器就会认为你打算手动管理 @property，于是编译器就禁用了 autosynthesis（自动合成）。

因为有了 autosynthesis（自动合成），大部分开发者已经习惯不去手动定义ivar，而是依赖于 autosynthesis（自动合成），但是一旦你需要使用ivar，而 autosynthesis（自动合成）又失效了，如果不去手动定义ivar，那么你就得借助 `@synthesize` 来手动合成 ivar。

其实，`@synthesize` 语法还有一个应用场景，但是不太建议大家使用：

可以在类的实现代码里通过 `@synthesize` 语法来指定实例变量的名字:
 
```objectivec
@implementation CYLPerson 
@synthesize firstName = _myFirstName; 
@synthesize lastName = _myLastName; 
@end 
```



上述语法会将生成的实例变量命名为 `_myFirstName` 与 `_myLastName`，而不再使用默认的名字。一般情况下无须修改默认的实例变量名，但是如果你不喜欢以下划线来命名实例变量，那么可以用这个办法将其改为自己想要的名字。笔者还是推荐使用默认的命名方案，因为如果所有人都坚持这套方案，那么写出来的代码大家都能看得懂。



举例说明：应用场景：


```objectivec

//
// .m文件
// http://weibo.com/luohanchenyilong/ (微博@iOS程序犭袁)
// https://github.com/ChenYilong
// 打开第14行和第17行中任意一行，就可编译成功

@import Foundation;

@interface CYLObject : NSObject
@property (nonatomic, copy) NSString *title;
@end

@implementation CYLObject {
    //    NSString *_title;
}

//@synthesize title = _title;

- (instancetype)init
{
    self = [super init];
    if (self) {
        _title = @"微博@iOS程序犭袁";
    }
    return self;
}

- (NSString *)title {
    return _title;
}

- (void)setTitle:(NSString *)title {
    _title = [title copy];
}

@end
```

结果编译器报错：
![enter image description here](http://i.imgur.com/fAEGHIo.png)

当你同时重写了 setter 和 getter 时，系统就不会生成 ivar（实例变量/成员变量）。这时候有两种选择：

 1. 要么如第14行：手动创建 ivar
 2. 要么如第17行：使用`@synthesize foo = _foo;` ，关联 @property 与 ivar。

更多信息，请戳- 》[ ***When should I use @synthesize explicitly?*** ](http://stackoverflow.com/a/19821816/3395008)

###16. objc中向一个nil对象发送消息将会发生什么？

**我的解答：**

返回nil。

**大神解答：**

在 objectivec 中向 nil 发送消息是完全有效的——只是在运行时不会有任何作用:

 1. 如果一个方法返回值是一个对象，那么发送给nil的消息将返回0(nil)。例如：  

 
```objectivec
Person * motherInlaw = [[aPerson spouse] mother];
```


 如果 spouse 对象为 nil，那么发送给 nil 的消息 mother 也将返回 nil。
 2. 如果方法返回值为指针类型，其指针大小为小于或者等于sizeof(void*)，float，double，long double 或者 long long 的整型标量，发送给 nil 的消息将返回0。
 2. 如果方法返回值为结构体,发送给 nil 的消息将返回0。结构体中各个字段的值将都是0。
 2. 如果方法的返回值不是上述提到的几种情况，那么发送给 nil 的消息的返回值将是未定义的。

具体原因如下：


> objc是动态语言，每个方法在运行时会被动态转为消息发送，即：objc_msgSend(receiver, selector)。


那么，为了方便理解这个内容，还是贴一个objc的源代码：


 
```objectivec
// runtime.h（类在runtime中的定义）
// http://weibo.com/luohanchenyilong/
// https://github.com/ChenYilong

struct objc_class {
  Class isa OBJC_ISA_AVAILABILITY; //isa指针指向Meta Class，因为Objc的类的本身也是一个Object，为了处理这个关系，runtime就创造了Meta Class，当给类发送[NSObject alloc]这样消息时，实际上是把这个消息发给了Class Object
  #if !__OBJC2__
  Class super_class OBJC2_UNAVAILABLE; // 父类
  const char *name OBJC2_UNAVAILABLE; // 类名
  long version OBJC2_UNAVAILABLE; // 类的版本信息，默认为0
  long info OBJC2_UNAVAILABLE; // 类信息，供运行期使用的一些位标识
  long instance_size OBJC2_UNAVAILABLE; // 该类的实例变量大小
  struct objc_ivar_list *ivars OBJC2_UNAVAILABLE; // 该类的成员变量链表
  struct objc_method_list **methodLists OBJC2_UNAVAILABLE; // 方法定义的链表
  struct objc_cache *cache OBJC2_UNAVAILABLE; // 方法缓存，对象接到一个消息会根据isa指针查找消息对象，这时会在method Lists中遍历，如果cache了，常用的方法调用时就能够提高调用的效率。
  struct objc_protocol_list *protocols OBJC2_UNAVAILABLE; // 协议链表
  #endif
  } OBJC2_UNAVAILABLE;
```

objc在向一个对象发送消息时，runtime库会根据对象的isa指针找到该对象实际所属的类，然后在该类中的方法列表以及其父类方法列表中寻找方法运行，然后在发送消息的时候，objc_msgSend方法不会返回值，所谓的返回内容都是具体调用时执行的。
那么，回到本题，如果向一个nil对象发送消息，首先在寻找对象的isa指针时就是0地址返回了，所以不会出现任何错误。

###17. objc中向一个对象发送消息[obj foo]和`objc_msgSend()`函数之间有什么关系？

**我的解答：**

编译期间[obj foo]会被编译为C代码，也就是objc_msgSend()方法，消息的接收者和消息名都会作为参数传到该方法中。

**大神解答：**

具体原因同上题：该方法编译之后就是`objc_msgSend()`函数调用.

我们用 clang 分析下，clang 提供一个命令，可以将objectivec的源码改写成C++语言，借此可以研究下[obj foo]和`objc_msgSend()`函数之间有什么关系。

以下面的代码为例，由于 clang 后的代码达到了10万多行，为了便于区分，添加了一个叫 iOSinit 方法，

```objectivec
//
//  main.m
//  http://weibo.com/luohanchenyilong/
//  https://github.com/ChenYilong
//  Copyright (c) 2015年 微博@iOS程序犭袁. All rights reserved.
//


#import "CYLTest.h"

int main(int argc, char * argv[]) {
    @autoreleasepool {
        CYLTest *test = [[CYLTest alloc] init];
        [test performSelector:(@selector(iOSinit))];
        return 0;
    }
}
```

在终端中输入

```objectivec
clang -rewrite-objc main.m
```
就可以生成一个`main.cpp`的文件，在最低端（10万4千行左右）

![enter image description here](http://i.imgur.com/eAH5YWn.png)

我们可以看到大概是这样的：

 
```objectivec
((void ()(id, SEL))(void )objc_msgSend)((id)obj, sel_registerName("foo"));
```

也就是说：

>  [obj foo];在objc动态编译时，会被转意为：`objc_msgSend(obj, @selector(foo));`。

###18. 什么时候会报unrecognized selector的异常？

**我的解答：**

当向对象发送消息时，对象无法响应该消息，且在系统的转发后也无法响应时，最后会报错unrecognized selector。

**大神解答：**

简单来说：


> 当调用该对象上某个方法,而该对象上没有实现这个方法的时候，
可以通过“消息转发”进行解决。



简单的流程如下，在上一题中也提到过：


> objc是动态语言，每个方法在运行时会被动态转为消息发送，即：objc_msgSend(receiver, selector)。


objc在向一个对象发送消息时，runtime库会根据对象的isa指针找到该对象实际所属的类，然后在该类中的方法列表以及其父类方法列表中寻找方法运行，如果，在最顶层的父类中依然找不到相应的方法时，程序在运行时会挂掉并抛出异常unrecognized selector sent to XXX 。但是在这之前，objc的运行时会给出三次拯救程序崩溃的机会：


 1. Method resolution

 objc运行时会调用`+resolveInstanceMethod:`或者 `+resolveClassMethod:`，让你有机会提供一个函数实现。如果你添加了函数，那运行时系统就会重新启动一次消息发送的过程，否则 ，运行时就会移到下一步，消息转发（Message Forwarding）。

 2. Fast forwarding

 如果目标对象实现了`-forwardingTargetForSelector:`，Runtime 这时就会调用这个方法，给你把这个消息转发给其他对象的机会。
只要这个方法返回的不是nil和self，整个消息发送的过程就会被重启，当然发送的对象会变成你返回的那个对象。否则，就会继续Normal Fowarding。
这里叫Fast，只是为了区别下一步的转发机制。因为这一步不会创建任何新的对象，但下一步转发会创建一个NSInvocation对象，所以相对更快点。
 3. Normal forwarding

 这一步是Runtime最后一次给你挽救的机会。首先它会发送`-methodSignatureForSelector:`消息获得函数的参数和返回值类型。如果`-methodSignatureForSelector:`返回nil，Runtime则会发出`-doesNotRecognizeSelector:`消息，程序这时也就挂掉了。如果返回了一个函数签名，Runtime就会创建一个NSInvocation对象并发送`-forwardInvocation:`消息给目标对象。

为了能更清晰地理解这些方法的作用，git仓库里也给出了一个Demo，名称叫“ `_objc_msgForward_demo` ”,可运行起来看看。

###19. 一个objc对象如何进行内存布局？（考虑有父类的情况）

**我的解答：**

额，不好描述。

**大神解答：**

 - 所有父类的成员变量和自己的成员变量都会存放在该对象所对应的存储空间中.
 - 每一个对象内部都有一个isa指针,指向他的类对象,类对象中存放着本对象的


  1. 对象方法列表（对象能够接收的消息列表，保存在它所对应的类对象中）
  2. 成员变量的列表,
  2. 属性列表,

 它内部也有一个isa指针指向元对象(meta class),元对象内部存放的是类方法列表,类对象内部还有一个superclass的指针,指向他的父类对象。

每个 objectivec 对象都有相同的结构，如下图所示：

 ![enter image description here](http://i.imgur.com/7mJlUj1.png)

翻译过来就是

|  objectivec 对象的结构图 | 
 ------------- |
 ISA指针 |
 根类的实例变量 |
 倒数第二层父类的实例变量 |
 ... |
 父类的实例变量 |
 类的实例变量 | 


 - 根对象就是NSobject，它的superclass指针指向nil

 - 类对象既然称为对象，那它也是一个实例。类对象中也有一个isa指针指向它的元类(meta class)，即类对象是元类的实例。元类内部存放的是类方法列表，根元类的isa指针指向自己，superclass指针指向NSObject类。



如图:
![enter image description here](http://i.imgur.com/w6tzFxz.png)

###20. 一个objc对象的isa的指针指向什么？有什么作用？

**我的解答：**

isa指针指向objc所属的类型的结构体，用于找到该类的方法列表。

**大神解答：**

指向他的类对象,从而可以找到对象上的方法。

###21. 下面的代码输出什么？

```objectivec
	@implementation Son : Father
	- (id)init
	{
	    self = [super init];
	    if (self) {
	        NSLog(@"%@", NSStringFromClass([self class]));
	        NSLog(@"%@", NSStringFromClass([super class]));
	    }
	    return self;
	}
	@end
```

**我的解答：**

son，son

**大神解答：**

都输出 Son

	NSStringFromClass([self class]) = Son
	NSStringFromClass([super class]) = Son
 


这个题目主要是考察关于 objectivec 中对 self 和 super 的理解。
 

我们都知道：self 是类的隐藏参数，指向当前调用方法的这个类的实例。那 super 呢？

很多人会想当然的认为“ super 和 self 类似，应该是指向父类的指针吧！”。这是很普遍的一个误区。其实 super 是一个 Magic Keyword， 它本质是一个编译器标示符，和 self 是指向的同一个消息接受者！他们两个的不同点在于：super 会告诉编译器，调用 class 这个方法时，要去父类的方法，而不是本类里的。


上面的例子不管调用`[self class]`还是`[super class]`，接受消息的对象都是当前 `Son ＊xxx` 这个对象。

当使用 self 调用方法时，会从当前类的方法列表中开始找，如果没有，就从父类中再找；而当使用 super 时，则从父类的方法列表中开始找。然后调用父类的这个方法。


这也就是为什么说“不推荐在 init 方法中使用点语法”，如果想访问实例变量 iVar 应该使用下划线（ `_iVar` ），而非点语法（ `self.iVar` ）。

点语法（ `self.iVar` ）的坏处就是子类有可能覆写 setter 。假设 Person 有一个子类叫 ChenPerson，这个子类专门表示那些姓“陈”的人。该子类可能会覆写 lastName 属性所对应的设置方法：

```objectivec
//
//  ChenPerson.m
//  
//
//  Created by https://github.com/ChenYilong on 15/8/30.
//  Copyright (c) 2015年 http://weibo.com/luohanchenyilong/ 微博@iOS程序犭袁. All rights reserved.
//

#import "ChenPerson.h"

@implementation ChenPerson

@synthesize lastName = _lastName;

- (instancetype)init
{
    self = [super init];
    if (self) {
        NSLog(@"🔴类名与方法名：%s（在第%d行），描述：%@", __PRETTY_FUNCTION__, __LINE__, NSStringFromClass([self class]));
        NSLog(@"🔴类名与方法名：%s（在第%d行），描述：%@", __PRETTY_FUNCTION__, __LINE__, NSStringFromClass([super class]));
    }
    return self;
}

- (void)setLastName:(NSString*)lastName
{
    //设置方法一：如果setter采用是这种方式，就可能引起崩溃
//    if (![lastName isEqualToString:@"陈"])
//    {
//        [NSException raise:NSInvalidArgumentException format:@"姓不是陈"];
//    }
//    _lastName = lastName;
    
    //设置方法二：如果setter采用是这种方式，就不会引起崩溃
    _lastName = @"陈";
    NSLog(@"🔴类名与方法名：%s（在第%d行），描述：%@", __PRETTY_FUNCTION__, __LINE__, @"会调用这个方法,想一下为什么？");

}

@end
```

在基类 Person 的默认初始化方法中，可能会将姓氏设为空字符串。此时若使用点语法（ `self.lastName` ）也即 setter 设置方法，那么调用将会是子类的设置方法，如果在刚刚的 setter 代码中采用设置方法一，那么就会抛出异常，


为了方便采用打印的方式展示，究竟发生了什么，我们使用设置方法二。


如果基类的代码是这样的：


```objectivec
//
//  Person.m
//  nil对象调用点语法
//
//  Created by https://github.com/ChenYilong on 15/8/29.
//  Copyright (c) 2015年 http://weibo.com/luohanchenyilong/ 微博@iOS程序犭袁. All rights reserved.
//  

#import "Person.h"

@implementation Person

- (instancetype)init
{
    self = [super init];
    if (self) {
        self.lastName = @"";
        //NSLog(@"🔴类名与方法名：%s（在第%d行），描述：%@", __PRETTY_FUNCTION__, __LINE__, NSStringFromClass([self class]));
        //NSLog(@"🔴类名与方法名：%s（在第%d行），描述：%@", __PRETTY_FUNCTION__, __LINE__, self.lastName);
    }
    return self;
}

- (void)setLastName:(NSString*)lastName
{
    NSLog(@"🔴类名与方法名：%s（在第%d行），描述：%@", __PRETTY_FUNCTION__, __LINE__, @"根本不会调用这个方法");
    _lastName = @"炎黄";
}

@end
```

那么打印结果将会是这样的：

```objectivec
 🔴类名与方法名：-[ChenPerson setLastName:]（在第36行），描述：会调用这个方法,想一下为什么？
 🔴类名与方法名：-[ChenPerson init]（在第19行），描述：ChenPerson
 🔴类名与方法名：-[ChenPerson init]（在第20行），描述：ChenPerson
```

我在仓库里也给出了一个相应的 Demo（名字叫：Demo_21题_下面的代码输出什么）。有兴趣可以跑起来看一下，主要看下他是怎么打印的，思考下为什么这么打印。


接下来让我们利用 runtime 的相关知识来验证一下 super 关键字的本质，使用clang重写命令:


```objectivec
	$ clang -rewrite-objc test.m
```

将这道题目中给出的代码被转化为:


```objectivec
    NSLog((NSString *)&__NSConstantStringImpl__var_folders_gm_0jk35cwn1d3326x0061qym280000gn_T_main_a5cecc_mi_0, NSStringFromClass(((Class (*)(id, SEL))(void *)objc_msgSend)((id)self, sel_registerName("class"))));

    NSLog((NSString *)&__NSConstantStringImpl__var_folders_gm_0jk35cwn1d3326x0061qym280000gn_T_main_a5cecc_mi_1, NSStringFromClass(((Class (*)(__rw_objc_super *, SEL))(void *)objc_msgSendSuper)((__rw_objc_super){ (id)self, (id)class_getSuperclass(objc_getClass("Son")) }, sel_registerName("class"))));
```

从上面的代码中，我们可以发现在调用 [self class] 时，会转化成 `objc_msgSend`函数。看下函数定义：


```objectivec
	id objc_msgSend(id self, SEL op, ...)
```
我们把 self 做为第一个参数传递进去。

而在调用 [super class]时，会转化成 `objc_msgSendSuper`函数。看下函数定义:


```objectivec
	id objc_msgSendSuper(struct objc_super *super, SEL op, ...)
```

第一个参数是 `objc_super` 这样一个结构体，其定义如下:


```objectivec
struct objc_super {
	   __unsafe_unretained id receiver;
	   __unsafe_unretained Class super_class;
};
```

结构体有两个成员，第一个成员是 receiver, 类似于上面的 `objc_msgSend`函数第一个参数self 。第二个成员是记录当前类的父类是什么。

所以，当调用 ［self class] 时，实际先调用的是 `objc_msgSend`函数，第一个参数是 Son当前的这个实例，然后在 Son 这个类里面去找 - (Class)class这个方法，没有，去父类 Father里找，也没有，最后在 NSObject类中发现这个方法。而 - (Class)class的实现就是返回self的类别，故上述输出结果为 Son。

objc Runtime开源代码对- (Class)class方法的实现:


```objectivec
- (Class)class {
    return object_getClass(self);
}
```

而当调用 `[super class]`时，会转换成`objc_msgSendSuper函数`。第一步先构造 `objc_super` 结构体，结构体第一个成员就是 `self` 。
第二个成员是 `(id)class_getSuperclass(objc_getClass(“Son”))` , 实际该函数输出结果为 Father。

第二步是去 Father这个类里去找 `- (Class)class`，没有，然后去NSObject类去找，找到了。最后内部是使用 `objc_msgSend(objc_super->receiver, @selector(class))`去调用，

此时已经和`[self class]`调用相同了，故上述输出结果仍然返回 Son。


参考链接：[微博@Chun_iOS](http://weibo.com/junbbcom)的博文[刨根问底objectivec Runtime（1）－ Self & Super](http://chun.tips/blog/2014/11/05/bao-gen-wen-di-objective%5Bnil%5Dc-runtime(1)%5Bnil%5D-self-and-super/)

###22. runtime如何通过selector找到对应的IMP地址？（分别考虑类方法和实例方法）

**我的解答：**

从类型的结构体中找到对应的实例方法列表中按照selector的名称去查找对应的方法，而类方法列表存储在类型的类型，即meta class当中。

**大神解答：**

每一个类对象中都一个方法列表,方法列表中记录着方法的名称,方法实现,以及参数类型,其实selector本质就是方法名称,通过这个方法名称就可以在方法列表中找到对应的方法实现.

###23. 使用runtime Associate方法关联的对象，需要在主对象dealloc的时候释放么？

**我的解答：**

不需要，原因不清楚。

**大神解答：**

 - 在ARC下不需要。
 - <p><del> 在MRC中,对于使用retain或copy策略的需要 。</del></p>在MRC下也不需要

> 无论在MRC下还是ARC下均不需要。


[ ***2011年版本的Apple API 官方文档 - Associative References***  ](https://web.archive.org/web/20120818164935/http://developer.apple.com/library/ios/#/web/20120820002100/http://developer.apple.com/library/ios/documentation/cocoa/conceptual/objectivec/Chapters/ocAssociativeReferences.html) 一节中有一个MRC环境下的例子：


 
```objectivec
// 在MRC下，使用runtime Associate方法关联的对象，不需要在主对象dealloc的时候释放
// http://weibo.com/luohanchenyilong/ (微博@iOS程序犭袁)
// https://github.com/ChenYilong
// 摘自2011年版本的Apple API 官方文档 - Associative References 

static char overviewKey;
 
NSArray *array =
    [[NSArray alloc] initWithObjects:@"One", @"Two", @"Three", nil];
// For the purposes of illustration, use initWithFormat: to ensure
// the string can be deallocated
NSString *overview =
    [[NSString alloc] initWithFormat:@"%@", @"First three numbers"];
 
objc_setAssociatedObject (
    array,
    &overviewKey,
    overview,
    OBJC_ASSOCIATION_RETAIN
);
 
[overview release];
// (1) overview valid
[array release];
// (2) overview invalid
```
文档指出 

> At point 1, the string `overview` is still valid because the `OBJC_ASSOCIATION_RETAIN` policy specifies that the array retains the associated object. When the array is deallocated, however (at point 2), `overview` is released and so in this case also deallocated.

我们可以看到，在`[array release];`之后，overview就会被release释放掉了。

既然会被销毁，那么具体在什么时间点？


> 根据[ ***WWDC 2011, Session 322 (第36分22秒)*** ](https://developer.apple.com/videos/wwdc/2011/#322-video)中发布的内存销毁时间表，被关联的对象在生命周期内要比对象本身释放的晚很多。它们会在被 NSObject -dealloc 调用的 object_dispose() 方法中释放。

对象的内存销毁时间表，分四个步骤：

	// 对象的内存销毁时间表
	// http://weibo.com/luohanchenyilong/ (微博@iOS程序犭袁)
	// https://github.com/ChenYilong
    // 根据 WWDC 2011, Session 322 (36分22秒)中发布的内存销毁时间表 

     1. 调用 -release ：引用计数变为零
         * 对象正在被销毁，生命周期即将结束.
         * 不能再有新的 __weak 弱引用， 否则将指向 nil.
         * 调用 [self dealloc] 
     2. 父类 调用 -dealloc
         * 继承关系中最底层的父类 在调用 -dealloc
         * 如果是 MRC 代码 则会手动释放实例变量们（iVars）
         * 继承关系中每一层的父类 都在调用 -dealloc
     3. NSObject 调 -dealloc
         * 只做一件事：调用 objectivec runtime 中的 object_dispose() 方法
     4. 调用 object_dispose()
         * 为 C++ 的实例变量们（iVars）调用 destructors 
         * 为 ARC 状态下的 实例变量们（iVars） 调用 -release 
         * 解除所有使用 runtime Associate方法关联的对象
         * 解除所有 __weak 引用
         * 调用 free()


对象的内存销毁时间表：[参考链接](http://stackoverflow.com/a/10843510/3395008)。

###24. objc中的类方法和实例方法有什么本质区别和联系？

**我的解答：**

1. 类方法是类型调用的，而实例方法是类的实例来调用的。
2. 类方法中不能使用实例方法和实例变量，而实例方法中可以使用类方法。

**大神解答：**

类方法：

 1. 类方法是属于类对象的
 2. 类方法只能通过类对象调用
 2. 类方法中的self是类对象
 2. 类方法可以调用其他的类方法
 2. 类方法中不能访问成员变量
 2. 类方法中不定直接调用对象方法

实例方法：

 1. 实例方法是属于实例对象的
 2. 实例方法只能通过实例对象调用
 2. 实例方法中的self是实例对象
 2. 实例方法中可以访问成员变量
 2. 实例方法中直接调用实例方法
 2. 实例方法中也可以调用类方法(通过类名)



