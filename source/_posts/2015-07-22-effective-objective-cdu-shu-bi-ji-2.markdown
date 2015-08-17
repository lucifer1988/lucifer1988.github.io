---
layout: post
title: "Effective Objective-C读书笔记2"
date: 2015-07-22 17:09:55 +0800
comments: true
categories: iOS
published: false
---

继续上一篇，这篇的主题是Objects，Messaging，and the Runtime。

<!--more-->

##Item6 Understand Properties

1. 在C++和Java中常使用@public和@private来声明实例变量，但在OC中由于对象是在编译期间定义的，所以按照这种定义方法，在新增变量后会导致访问偏移量出错，除非重新编译，但是这样就失去了动态语言的优势。
2. OC的解决方案是将实例变量作为可存储内存偏移量的类对象，这同时可以将实例变量定义到实现文件中，从而实现隐藏。Apple鼓励使用存取方法而不是直接访问实例变量，也是为了解决这一问题，@property就是为了方便提供getter和setter方法。
3. OC中的点方法类似C中访问结构体的成员，但其实是编译器转化为了对应的getter方法。

###Property Attributes

1. 主要说下有关内存管理的property属性，主要有assign，strong，weak，unsafe_unretained，copy。
2. assign：主要用于标量的property属性，简单的赋值操作，引用计数不变。
3. strong：声明的是持有关系，新值会被retain，旧值release，引用计数加1。
4. weak：声明的是非持有关系，与assign类似，如果指向的对象被释放，该值也会被释放。
5. unsafe_unretained：可以理解为针对对象的assign属性，但是所指向的对象被释放后，该值不会被释放，所以容易造成野指针，一般很少用到它。
6. copy：与strong类似，只不过所赋值的引用计数不变，旧值会被赋给一个所赋值copy的引用计数为1的对象，一般用于不可变对象，可能被赋可变对象的值时，这样可确保旧值改变时，不可变对象不发生变化。
7. get=<>：可以自定义getter方法的名字，一般用于布尔型property，一般getter方法以is开头。
8. 额外1：如果对一不可变对象复制，copy是指针复制（浅拷贝）和mutableCopy就是对象复制（深拷贝）。如果是对可变对象复制，都是深拷贝，但是copy返回的对象是不可变的。
9. 额外2：不要在init（包括自定义的初始化方法）和dealloc中使用setter和getter方法。
10. atomic用以确保线程安全，但是iOS平台的property基本都是nonatomic的，主要是因为性能问题，而且atomic也并非完全是线程安全的（例如一个线程频繁访问一个对象时，另一线程同时在写入，前一线程也会拿到不同的值），而在Mac OS X就不存在这个性能瓶颈了。

<!--more-->

##Item7 Access Instance Variables Primarliy Directly When Accessing Them Internally

1. 本章讨论的是如何在内部使用实例变量，有两种方式，一是使用生成的存取方法，二是直接使用实例变量。
2. 优缺点如下：
   1. 直接访问对象，速度会快，绕开了OC的method dispatch，编译器会直接访问存储对象的内存。
   2. 直接访问对象会绕开与内存相关的setter方法，例如你设置的copy型的setter，只会按照retain来执行。
   3. 直接访问对象不会触发KVO。
   4. 使用存取方法会使调试变得简单，你可以在getter/setter添加断点。
3. 比较推荐的做法是，在存对象的时候使用setter方法，而在读取对象时直接读取，这样既享受了快速读取，也可以利用property控制保存对象。
4. 但是这么做还是有一些需要注意的地方：
   1. 在初始化方法中，一定要使用直接赋值的方法，主要是因为怕子类复写了对象的setter方法，而导致异常，如果一个对象声明在了父类的内部，而子类不能直接访问它，你也不能直接访问读取该变量，这种情况只能通过setter赋值
   2.如果实例变量使用了延时加载，那么读取一定要使用getter方法，不然这个对象永远都不会有值。
   
<!--more-->

##Item8 Understand Object Equality

1. 比较两个对象，不使用==，那样只会比较指针的值，而一般使用*isEqual:*，如果对象有自己的专有比较方法，例如*isEqualToString:*，优先使用这些方法，速度会快些。
2. *-(NSUIntegetr)hash;*是一个与比较对象息息相关的方法，hash相同的对象不一定相同，而相同的对象hash值一定相同。
3. 所以自定义对象重写*isEqual:*方法，也一定要重写hash方法，共有三种方案：
   1. 返回一个常数，这个方案优点是使用单个对象时快，但是如果把大量对象放入同一集合，由于hash值相同，集合会挨个检查这些对象是否真的相同，从而导致向一个集合添加大量对象时就会很慢；
   2. 使用一个拼接的唯一字符串，然后进行hash，这个方案避免了上面的问题，但是出现了单个对象需要生成一个字符串，从而影响了速度的问题；
   3. 先取一系列变量的hash值，再将其异或，这个方案算是为了避免上述问题的折衷方案。
   
###Class-Specific Equality Methods

1. 自定义类可以通过重写*isEqual:*方法，在方法里判断如果是同一类型，就调用上面的比较方法，如不是就调用父类的*isEqual:*方法，这样可以实现子类也可以与父类进行比较。

###Deep versus Shallow Equality

1. 有时你并不需要判断对象的所有信息是否相同，比如来自数据库的信息，可能只通过判断id就可以进行判断，所谓的浅比较就是这样。

###Equality of Mutable Classes in Containers

1. 这一部分主要讲的是，向集合添加可变对象，然后改变该对象，是有可能让集合出现重复对象的，这点值得关注。

<!--more-->

##Item9 Use the Class Cluster Pattern to Hide Implementation Detail

1. 类簇是OC中很重要的一个设计模式，例如UIButton的创建，类簇解决的问题是需要统一创建同一基本类型的不同对象，而同时避免暴露子类和父类内部复杂的switch语句。

###Creating a Class Cluster

1. 创建类簇的思路：一个基类，一些空方法，一个创建对象的工厂方法，继承的子类对空方法重写。这样的类簇有个缺点就是用户可能以为自己使用的类就是那个基类，而不知道其实是它的子类。

###Class Clusters in Cocoa

1. 由于很多Cocoa类都是使用了类簇模式，所以类似*[maybeAnArray class] == [NSArray class]*这样的校验类型的方法是不会返回正确值的，而要使用*[maybeAnArray isKindOfClass:[NSArray class]]*。
2. 添加一个类簇的子类而不去改写其基类的工厂方法，对于NSArray是可以的，但是有三点要求：1、必须是该类簇基类的子类；2、该子类必须定义自己的存储空间，也就是说内部要有一个NSArray的对象来实现数据的存储；3、子类必须重写类簇文档中规定重写的方法。

<!--more-->

##Item10 Use Associated Objects to Attach Custom Data to Existing Classes

1. 有时为了为一个类绑定一些信息，而又不方便添加多余的property或者继承这个类，可以考虑使用*association*，类似字典型的键值读取，也可以设置内存管理策略，但是需要注意绑定的key必须是唯一的指针，而不只是值相同，所以一般使用全局的静态变量作为key。

###An Example of Using Associated Objects

1. 通过使用*Associated Objects*实现了UIAlertView的回调Block化，使得代码的可读性更好，也更方便。
2. *Associated Objects*提供了一个将对象之间互相绑定的方法，但是并不推荐大范围使用该方法，因为会导致调试变的更难。 

<!--more-->

##Item11 Understand the Role of objec_msgSend

1. OC利用动态绑定成为了真正的动态语言，OC中传递消息最终被转化为函数*void objc_msgSend(id self, SEL cmd, ...)*，例如：*id returnValue = [someObkect messageName:parameter];*转化后，*id returnValue = objc_msgSend(someObject, @sleector(messageName:), parameter);*。
2. *objc_msgSend*执行的顺序是先在接受者实现的方法中找符合的方法来执行，如没有，向继承链上方逐级寻找符合的实现方法。 
3. *objc_msgSend*会为每个类缓存一张查找表，来加速这一过程，但即使如此，还是比在C中直接调用静态调用函数慢，但这常常不是应用的瓶颈，这样来换取程序的灵活性还是值得的。
4. *objc_msgSend*是针对确定消息的处理，下面还有一些处理个别案例的方法。
5. *objc_msgSend_stret*用于处理用户返回适用于CPU寄存器的结构体的消息（不太懂）。
6. *objc_msgSend_fpret*用于处理返回浮点值的消息，一些结构需要在函数调用时对浮点数寄存器特殊处理，所以这是该方法存在的意义（不太懂）。
7. *objc_msgSendSuper*直接把消息转发给父类执行，类的所有方法都是一个个类似*<return_type> Class_selector(id self, SEL _CMD, ...)*这样的原型，这些方法指针存在该类的一个查找表中等待调用，该原型与*objc_msgSend*是相同的，也就实现了[尾部递归调用](http://www.ruanyifeng.com/blog/2015/04/tail-call.html)的可能，这样会实现调用栈的空间复杂度保持O(1)，不会产生溢出。

<!--more-->

##Item12 Understand Message Forwarding

1. 转发路径是为了处理接受者无法处理消息的情况，分为两条路径：1、*dynamic method resolution*期望接收者自己在runtime添加处理方法；2、*full forwarding mechanism*到了这一步，runtime得知接收者是不可能对消息做出响应了，所以要求接收者自己处理该消息，又分为两步：(1)询问是否有其他对象可以接收消息，如果有则转发给该对象；(2)如果也没有替代的接收者，那么将使用*NSInvocation*来对消息进行封装，然后交给原接收者去处理。[这儿也做了详细解释](http://bugly.qq.com/blog/?p=64)。

###Dynamic Method Resolution

1. *+(BOOL)resolveInstanceMethod:(SEL)selector*用于表明类有无实例方法可处理该消息，可以说是给予该类的第二次机会。
2. 这类方法是存在的，例如CoreData的@dynamic的property的accessing方法，而*+(BOOL)resolveInstanceMethod:(SEL)selector*对其的处理是判断是否是@dynamic property，如果是，向该类添加预备好的getter，setter方法，已确保类可以响应该消息。

###Replacement Receiver

1. *-(id)forwardTargetForSelector:(SEL)selector*用于返回可以替代原接收者的对象（如果其存在的话），这其实提供了一些多继承的特性，即原类内部可以有其他对象来实现这一方法。但是无法对消息进行修改，只是转发，如需修改消息，则需要采取最后一步。

###Full Forwarding Mechanism

1. -(void)forwardInvocation:(NSInvocation*)invocation用于转发接收到的NSInvocation消息，可以进行简单转发，但这和上述的方法没有区别，而更为常见的用途是修改消息，如增加参数或者改变方法等。
2. 重写该方法时需要调用父类的相同方法来处理改invocation，这样会最终调用NSObject的*doesNotRecognizeSelector*，最终抛出异常，但如果你不希望程序崩溃，就不要去调用父类的方法。

###The Full Picture

1. 具体图表见[这儿](http://bugly.qq.com/blog/?p=64)。
2. 解决的代价是越来越高的，所以最好在第一阶段解决这一问题。

###Full Example of Dynamic Method Resolution

1. 举例说明，将一个model中的所有对象都存在一个dictionary中，而这些对象申明为@dynamic，在*+(BOOL)resolveInstanceMethod:(SEL)selector*中根据selector的信息对相应的对象动态添加setter，getter方法，大幅减少代码量，但缺点是想特殊处理某个对象，就变得比较麻烦了。

<!--more-->

##Item13 Consider Method Swizzling to Debug Opaque Methods

1. *Method Swizzling*主要用于不知道类的源码，且不用继承、重写，即可为原方法添加hook的手段（其实是在runtime中先交换，再执行一次原方法而已-_-）。
2. 通过添加一个类的分类，在分类添加一个方法，在这个方法中进行递归调用，然后与目标方法进行交换，这时再执行原方法时，会依次执行这两个方法。[另外一篇文章也有说明](http://blog.csdn.net/yiyaaixuexi/article/details/9374411)。

<!--more-->

##Item14 Understand What a Class Object Is

1. Class本身也是一个结构体指针，叫objc_class，Class也有一个Class类型的isa指针，说明Class本身也是一个OC对象，他的类型叫做metaclass，Class有Class类型的super_class指针，用来指向他的父类Class。

###Inspecting the Class Hierarchy

1. *isMemberOfClass:*用于判断是否属于该类，*isKindOfClass:*用于判断是否属于该类或者该类的子类。原理还是利用上述的Class的isa和super_class指针。
2. 内省（自我类型检查）是OC中的重要技术，应用也很广泛，除了上述方法，也可利用*[object class] == [EOCSomeClass class]*来判断，之所以这么写是合理的，是因为每个class的Class类型是一个单例对象，所以可以直接比较指针。
3. 但是还是推荐使用默认的类型检测方法，因为这样可以利用消息转发技术，如果一个对象的所有方法都是代理对象执行的，那么调用class方法只会返回代理对象的类型，而调用*isKindOfClass:*方法，代理会把消息转给被代理的对象，会得到正确的类型。
