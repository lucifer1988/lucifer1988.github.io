---
layout: post
title: "Effective Objective-C读书笔记5"
date: 2015-08-28 11:45:07 +0800
comments: true
categories: iOS
published: true
---

第五部分开始将讨论OC的核心技术，Reference Counting，也就是使用引用计数来进行内存管理，这一部分涉及到底层内存管理机制，ARC相关技术细节和在开发中涉及到内存管理需要注意的常见问题。

<!--more-->

##Item29 Understand Reference Counting

1. Reference Counting是OC管理内存的方式，当一个对象的RC为0时，将被释放，iOS没有垃圾回收机制。

###How Reference Counting Works

1. 在NSObject Protocol中有三个方法可以改变RC，*retain,release,autorelease*。
2. retainCount这个方法可以查看对象当前的RC值，但是并不推荐使用，Item36会讨论。
3. 对象之间往往是互相持有的，当该持有关系是*strong*时，被持有对象的RC加1，而持有链的最顶端是根对象，Mac OSX是NSApplication，iOS是UIApplication，都是应用创建的单例。
4. 举例一个NSMutableArray添加一个NSNumber，虽然在array添加number后，释放number，number的RC还是1，调用number理论上是可以的，但是并不推荐这么做，因为如果任何其他原因使number的RC为0，这一做法会导致崩溃。
5. 对象被释放后，它的内存将进入可用内存池，如果调用发生在内存被复写之前，不会发生错误，所以之类bug有时会很难复现，所以在MRC中往往在调用release后会赋值nil。

###Memory Management in Property Accessors

1. strong命名的property的setter方法，是新值retain，然后旧值release，然后赋值，这一顺序不能错，因为如果先release再retain，且正好两个对象是同一个，可能会导致对象提前释放，RC为0，成为野指针，再调用retain则会出错。想按照这个顺序来，必须判断新旧两值是否是同一对象。

###Autorelease Pools

1. 借助autorelease pool替代release操作的autorelease，常用于需要返回新建对象的方法，具体释放时间在下一次事件循环（Item34将讨论）。
2. 在直接调用该方法时不用额外的内存空间，直接调用即可。
3. 但是如果返回对象需要持有时，比如赋值给一个实例变量，它需要retain一次，然后使用完后手动release，所以可以将autorelease理解为延长对象的生命周期，可以至少保证到方法调用的边界。

###Retain Cycles

1. 循环引用一般是指两个或多个对象直接互相存在强引用，而导致RC都不能为0，所有对象都不能释放。
2. 在垃圾回收机制下，retain cycle会被定义为孤岛，而直接被全部释放，而在RC机制下，只能通过定义weak引用或者依靠外部帮助来使其中某个对象交出对其他对象的引用。

<!--more-->

##Item30 Use ARC to Make Reference Counting Easier

1. Clang编译器带来了一个静态分析器，可以定位出现RC问题的位置，例如没有添加release，除此之外，该分析器可以为你自动添加retain，release这些操作，这也就是ARC技术的基础。
2. 在ARC机制下，retain、release、autorelease、dealloc这些操作都是不允许的，因为这回影响编译器判断添加语句的位置。
3. 事实上，ARC并没有直接调用上述这些方法，而是调用了他们的C的替代方法，例如objc_retain，这么做的好处是，因为这些操作会频繁调用，使用C方法可以提高效率。这也是为什么不允许直接重写retain，release这些方法，因为这方法并不是直接调用的。

###Method-Naming Rules Applied by ARC

1. 内存管理指定方法名在OC里一直是惯例，而ARC将其加强了，含有以下名称的方法：*alloc、new、copy、mutableCopy*，所返回的对象的所有者为方法的调用者，而其他方法返回的对象为autorelease，会保持到方法调用边界。
2. 而ARC会依据方法名的开头添加响应的语句，例如上述四个关键字开头的，会直接返回，而一般方法，ARC会在返回对象前加上autorelease。
3. 而在调用这些方法时，第一类方法返回的对象，ARC会在方法结束前添加release方法，而第二类方法因为有autorelease，所以不会添加操作。
4. ARC通过命名规范来规范内存管理，再加上之前的命名空间，OC是少有的如此强调命名的语言之一。
5. ARC可以做一些无法手动完成的优化，比如，它将在编译期间取消多余的retain和release操作。
6. ARC在runtime也有进行优化，举例：EOCPerson的一般初始化方法返回的值（添加了autorelease），被赋值给一个对象的strong属性实例，按照之前的原则，需要在返回的对象加retain，这里的autorelease和retain看起来是多余的，ARC确实可以为了性能，直接去掉autorelease这个方法，所有返回的对象都为RC+1，但为了兼容MRC，ARC还是需要特殊处理。
7. ARC确实对这种现象做了处理，在返回对象之前它调用了*objc_autoreleaseReturnValue*，如果被发现该对象是需要retain的，则会添加一个flag，而不是调用autorelease。同样的，调用者也会调用*objc_retainAutoreleasedReturnValue*，而不是retain，该方法也会先检测flag，如果存在，则不会retain，这样提高了效率：  

```objectivec
//Within EOCPerson class
+(EOCPerson*)personWithName:(NSString*)name {
	EOCPerson *person = [[EOCPerson alloc] init];
	person.name = name;
	objc_autoreleaseReturnValue(person);
}
//Code using EOCPerson class
EOCPerson *tmp = [EOCPerson personWithName:@"Matt"];
_myPerson = objc_retainAutoreleasedReturnValue(tmp);

//objc_autoreleaseReturnValue
id objc_autoreleaseReturnValue(id object) {
	if(/*caller will retain object*/){
		set_flag(object);
		return object;//no autorelease
	} else {
		return [object autorelease];
	}
}

//objc_retainAutoreleasedReturnValue
id objc_retainAutoreleasedReturnValue(id object) {
	if(get_flag(object)){
		clear_flag(object);
		return object;//no retain
	} else {
		return [object retain];
	}
}
```

###Memory-Management Semantics of Variables

1. ARC也同时管理着本地变量和实例变量的内存，默认每个变量对于对象是strong引用。
2. 在setter方法中，ARC中直接对旧值赋值即可，ARC会自动添加正确的代码。
3. 声明实例变量时，也可以改变内存管理方式，__strong（默认，赋值将被retain）、__unsafe_retained（赋值同assign，但指针不会自动置空，可能出现野指针）、__weak（赋值同assign，对象被释放时，指针会自动置为nil，所以是安全的，iOS5后可用）、__autoreleasing（多用与方法的返回值）。
4. __weak用于本地变量时，常用于避免循环引用，比如在block中：  

```objectivec
NSURL *url = [NSURL URLWithString:@"http://www.example.com/"];
EOCNetworkRetcher *fetcher = [[EOCNetworkFetcher alloc] initWithURL:url];
EOCNetworkFetcher * __weak weakFetcher = fetcher;
[fetcher startWithCompletion:^(BOOL success){
	NSLog(@"Finished fetching from %@", weakFetcher.url);
}];
```

###ARC Handling of Instance Variables

1. 在ARC中，你一般不需要再重写dealloc方法，ARC借用Objective-C++的特性，Objective-C++对象在释放时会调用所有持有对象的析构方法，当编译器发现对象包含C++对象时，会生成*.cxx_destruct*方法，ARC借助这个方法，在其中执行清除内存的代码。
2. 但有时你仍需要重写dealloc方法，像CoreFoundation对象和堆上开辟的内存（如malloc），以及KVO、的解除，都需要手动释放，但注意*在ARC中，不需要在dealloc中写[super dealloc]*，因为ARC在*.cxx_destruct*中已经调用了这一方法。

###Overriding the Memory-Management Methods

1. 在MRC中，重写内存相关方法是允许的，比如单例常常重写release方法为一个空操作，这样单例就不会被释放。
2. 但在ARC中是不允许的，一是会导致ARC对对象周期的误判，二是ARC对内存管理做了深度的优化，当需要执行retain、release、autorelease时，ARC在OC的message dispatch做了优化处理，不能重写或调用这些方法则是该优化的前提。

<!--more-->

##Item31 Release References and Clean Up Observation State Only in dealloc

1.dealloc方法会在对象的引用计数为0时自动调用，但什么时候调用并不能保证，即使是在MRC中，手动控制release也一样，因为很多库会在你不知道的情况下修改对象，这会导致调用dealloc的时间发生变化。所以你千万不要手动去调用dealloc，runtime会在合适的时间调用。  
2.那么在ARC下，重写dealloc的话，一是用于释放CoreFoundation的对象，二是取消NSNotificationCenter中注册该对象的监听或KVO。  
3.如果你的类中使用到了文件描述集，sockets、或者开辟了大块儿内存，由于dealloc的调用时间不明，你可能在你不需要使用的时候即可释放这些内存，而不用等到dealloc触发，这样需要自定义一个清除方法，该方法必须在dealloc之前调用，不然就算异常了。  
4.清除资源需要另一个方法的原因是创建的对象并不是都会被调用dealloc，因为一部分对象在应用退出后台时并不会释放，它们只有在应用彻底被系统回收后才会释放，这是一种优化措施，但也会导致大量的资源被无故占用，所以在-(void)applicationWillTerminate:(UIApplication*)application中调用一些对象的clean方法是必要的。  
5.有时可以在dealloc中也可以去调用clean方法，这可以避免忘记调用clean，但最好还是手动去先执行clean，所以还是要提示下或者严重的话直接直接抛出异常：  

```objectivec
-(void)close {
	/*clean up resources*/
	_close = YES;
}

-(void)dealloc {
	if(!_closed) {
		NSLog(@"ERROR:close was not called before dealloc!");
		[self close];
	}
}
```

6.除了上述特例，一般是不允许在dealloc中调用类的其他实例方法，因为有可能导致方法执行前，该对象可能已经释放了。而且，dealloc方法是在导致对象最终释放的线程上执行的，所以需要在特定线程执行的方法在此调用，不能保证线程正确，即使是通过代码强制在某线程执行，也是不安全的，因为对象处于释放状态。
7.另外dealloc中也不可调用property的setter、getter方法，尤其是被重写的accessor，也有可能触发KVO的回调，导致未知的错误。

<!--more-->

##Item32 Beware of Memory Management with Exception-Safe Code

1. Exception是OC和C++中用于处理严重异常的对象，但有时你也需要通过代码处理这些异常，例如去注销一个KVO，但之前并没有注册过的情况。
2. 在try/catch中创建对象，并需要自己释放时，需要将释放代码写到finally中，这样才能保证无论是否异常都能保证对象释放。
3. 但在ARC中可以自动添加额外的处理代码，使用*-fobjc-arc-exceptions*这个flag来控制，但默认是关闭的，因为exception出现时application直接crash，资源也会回收，所以没必要再做处理，而且会带来性能问题，只有编译器处于Objective-C++时才会开启，因为OC++添加代码带来性能损耗没有ARC添加时那么大，另外OC++中Exception是被大量使用的。
4. 如果在ARC下需要单独处理exception，那么可以开启flag，但如果你有很多exception处理，那么你该考虑NSError了，如Item21所讲。

<!--more-->

##Item33 Use Weak References to Avoid Retain Cycles

1. 循环引用带来的问题主要是，引用环中的对象将不能再被调用，但也不能释放，从而导致内存泄露。
2. Java会有垃圾回收来解决这类问题，但iOS和Mac OS X 10.8之后是没有垃圾回收的，所以只能开发者自己去避免。
3. 使用unsafe_unretained可以避免这一问题，它类似assign，但assign一般用于数值型，而unsafe_unretained用于对象，但如字面意思一样，它不会因为所指向的对象被释放而置为空值，所以调用unsafe_unretained的对象，可能会因为所指对象不存在而崩溃，所以是不安全的。
4. 在ARC中我们常用的是weak字段，该字段与unsafe_unretained功能一致，但是它会在所指对象释放后自动指向nil，所以是安全的。
5. 关于循环引用，总的原则就是，如果你不持有一个对象，那么你就不该retain它（数组，集合不直接持有包含的对象，但是会retain它们，是个例外）。一般场景有，controller的UI控件（一般weak属性），一个对象的delegate属性（一般为weak属性）。

<!--more-->

##Item34 Use Autorelease Pool Blocks to Reduce High-Memory Waterline

1.*@autoreleasepool{}*这是OC中建立autorelease pool的方法，但我们一般不必去手动创建。  
2.main函数中的autorelease pool并不是必须的，只是UIApplicationMain()函数中需要autorelease的对象没有对应的pool，但它们在程序终止时时肯定会被释放的。  
3.autorelease pool可以嵌套，autorelease的对象总是被添加最里面的pool中。  
4.利用上述特性，我们可以对一些大量循环执行一个可能产生很多autorelease对象的方法做优化，如下EOCPerson的创建可能产生大量autorelease对象，这样产生的autorelease对象会在自己建的autorelease pool结束时释放，而不是长期存在于线程自己的autorelease pool，避免了出现应用内存陡升陡降这种“瀑布现象”：  

```objectivec
NSArray *databaseRecords = /*...*/;
NSMutableArray *people = [NSMutableArray new];
for(NSDictionary *record in databaseRecords){
	@autoreleasepool{
		EOCPerson *person = [[EOCPerson alloc] initWithRecord:record];
		[people addObject:person];
	}
}
```
		
5.autorelease pool可以被理解为放入了一个栈中，新建的pool会在最顶端，当它释放后会被移出栈，当一个对象调用了autorelease，它将被添加到最顶端的autorelease pool。  
6.使用autorelease pool来优化瀑布现象并不是必要的，这取决你的应用，如果确实导致了问题，那么去使用它，如果不必要使用，那么就不要添加多于的autorelease pool。  
7.ARC之前使用autorelease pool是使用NSAutoreleasePool，因为它属于重量级对象，所以一般是隔段时间进行释放：  

```objectivec
NSArray *databaseRecords = /*...*/;
NSMutableArray *people = [NSMutableArray new];
int i=0;
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
for(NSDictionary *record in databaseRecords){
	EOCPerson *person = [[EOCPerson alloc] initWithRecord:record];
	[people addObject:person];
	//Drain the pool only every 10 cycles
	if(++i == 10){
		[pool drain];
	}
}
//Also drain at the end in case the loop is not multiple of 10
[pool drain];
```

8.推荐使用新语法@autoreleasepool，更加轻量，而且一个重要特性，NSAutoreleasePool中创建的autoreleased对象在执行drain之后还能使用，这可能造成崩溃，且很多时候难以发现，而使用@autoreleasepool，这类代码不会被编过，也就及早避免了这一问题。  

<!--more-->

##Item35 Use Zombies to Help Debug Memory-Management Problems

1.内存问题一般很难处理，原因是被释放的那块内存不一定就很快被重写，或者正好被一个同类的对象重写，这样有时不会导致崩溃，有时却会，所以开发者有时会无从下手。  
2.Cocoa的Zombies特性会帮助我们解决这一问题，当该模式启用，所有被释放的对象会转化为NSZombie对象，其占用过的内存也不会被重用，当该对象收到消息时，会抛出异常，告知开发者所收到的消息，原来的对象类型这些信息。  

```objectivec
void PrintClassInfo(id obj){
	Class cls = object_getClass(obj);
	Class superCls = class_getSuperclass(cls);
	NSLog(@"===%s:%s===",class_getName(cls),class_getName(superCls));
}
int main(int argc, char *argv[]){
	EOCClass *obj = [[EOCClass alloc] init];
	NSLog(@"Before release:");
	printClassInfo(obj);
	[obj release];
	NSLog(@"After release");
	PrintClassInfo(obj);
}
//result
//Before release:
//===EOCClass:NSObject===
//After release:
//===_NSZombie_EOCClass:nil===
``` 

3.通过上述手段我们得知obj在dealloc后变为了_NSZombie_EOCClass，但并没有它的父类，实际上，它是通过对原类型的类名修改，然后对_NSZombie_类型执行objc_duplicateClass()，完全拷贝zombie类并使用新类名（也可使用继承，但不如copy效率），制造出obj对应的zombie类，然后用objc_setClass()修改obj的isa指针，改变其类型，这一切都是利用runtime完成的（通过method swizzles对dealloc方法替换）。  
4.由于_NSZombie_没有实现任何方法，所以向它或者它的copy类型发送任何消息，会直接进入forwarding mechanism，在寻求转发时如果发现类型名以_NSZombie_开头，那么直接抛出异常，并打印出message、原类型这些信息。  

<!--more-->

##Item36 Avoid Using retainCount

1. *retainCount*是NSObject Protocol的一个方法，用于返回对象目前的引用计数值，在ARC中已经弃用，但即使在MRC中，任然应该避免使用它。
2. 原因一是*retainCount*返回的是实时的count值，也就是说像autorelease这样将要发生的count减少的情况，不会在该方法反映出来，所以依据该值去执行一些改变count的方法，往往会出问题。
3. 有时retainCount会返回一个极大的值，这是NSString或NSNumber直接设置常量时，系统会将其作为一个单例的常量，而不是去创建一个对应的对象，这些对象的ratainCount是不会改变的，但只是对一些特例的优化。




