---
layout: post
title: "Effective Objective-C读书笔记3"
date: 2015-08-17 14:15:03 +0800
comments: true
categories: iOS
published: true
---

第二部分主要讲了一些OC底层的运作机制，这一部分开始，主要涉及实践部分，第三部分的主题是：Interface and API Design。

<!--more-->

##Item15 Use Prefix Names to Avoid Namespace Clashes

1. OC是没有内建的命名空间的，所以必须采取措施避免这一问题。
2. 解决方案是自己在所有类都添加自定义的前缀，一般是项目名缩写，但推荐使用三个字母，因为两个字母被苹果使用，所以重名的概率大。
3. 在.m文件中的纯C函数和全局变量也有可能出现重名，所以定义时需格外注意，也要加上前缀。
4. 还有一种可能，你自己封装的类库A和应用使用了同一类库B，而应用也使用了你的类库A，这样的话，只能手动将你自己使用的类库B的所有加上类库A的前缀，虽然麻烦，但是如果是大工程的话，必须这么做。

<!--more-->

##Item16 Have a Designated Initializer

1. 一个类可能有很多初始化方法，但其中基本初始化方法只需有一个，其他初始化方法只是参数变化，这样保证数据在一个方法中赋值，便于维护。
2. 为了避免用户使用原始的*init:*方法而出现错误，该类中应该重写*init:*方法，可以做一个默认的赋值，或者直接抛出异常。
3. 继承一个拥有基本初始化方法的类，子类的初始化方法要调用父类的基本初始化方法，而且需要重写父类的基本初始化方法，与上一条的理由一致。
4. 有时可能需要两个基本初始化方法，特例比如遵循NSCoding的类，要有一个-(id)initWithCoder:(NSCoder*)decoder的初始化方法，而该类的子类也必须重写initWithCoder:，同时调用父类initWithCoder:。

##Item17 Implement the description Method

1.重写对象的-(NSString*)description方法，可以获得更多的实用信息，默认的只是类名和指针地址，这也是NSObject协议的其中一项。  
2.这是一种将NSDictionary特性结合起来的description写法。  

```objectivec
-(NSString *)description {
	return [NSString stringWithFormat:@"<%@:%p,%@>",
	[self class],
	self,
	@{@"title": _title,
	@"latitude": @(_latitude),
	@"longitude": @(_longitude)}
	];
}
```  

3.LLDB中的*po*命令会执行print-object函数，它返回的是NSObject协议的另一方法-(NSString *)debugDescription，而这一方法默认返回的是description的结果，如果需要隐藏部分信息，可以分别重写这两个方法，OC默认类型很多就是这么干的，例如NSArray。  

<!--more-->

##Item18 Prefer Immutable Objects

1. 设计类的时候，其中的property除非必须可变，都应设计为不可变只读类型，之前Item8也讨论过类似问题，一个可变集合加入两个可变数组，然后设法改变数组，可能会出现集合中有相同数组，而不会报错的问题。
2. 解决这个问题的设计是在.h文件中设置property为readonly，而在.m文件中添加匿名分类，重新定义相同的property为readwrite，这样实现了对外只读，而内部可以进行修改。
3. 如果需要对外提供修改变量的方法，也不建议直接把可变变量暴露，而是对外还是暴露只读变量，内部再定义一个可变的内部变量，外部的只读变量的getter方法返回内部可变变量的copy，而同时添加增删的外部方法来操作内部变量。

<!--more-->

##Item19 Use Clear and Consistent Naming

1. OC命名方式是尽量详细，多使用一些介词，表明方法功能，同时使用驼峰命名法。

###Method Naming

1. 如果一个方法返回了一个新对象，那么方法一般以该对象的类型开头。
2. 一个参数前需要加一个名词来描述他的类型。
3. 一个方法描述对一个对象进行操作时，需要包含一个动词，然后每个参数前依旧需要名词描述。
4. 避免使用缩写，而使用全称，例如：*str*和*string*。
5. *Boolean*类型的property的getter方法用*is*前缀，返回*Boolean*的方法应该以*has*或*is*作为前缀。
6. 保留*get*关键字，在方法并无返回值，但是通过传入的参数，进行值的返回时使用get，比如：-(void)getCharacters:(unichar *)buffer range:(NSRange)aRange。

###Class and Protocol Naming

1. 主要是注意你继承的类要以其类名结尾，但前缀不要，要加上自己的前缀，协议要以Delegate结尾。

<!--more-->

##Item20 Prefix Private Method Names

1. 用特殊前缀标记类的私有方法，会在调试时更加方便，Matt的方式是在方法前加*p_*前缀，例如：*-(void)p_privateMethod*，当然你最好定义自己的方式。
2. Apple的方式是在方法前加*_*来标识私有方法，但不推荐开发者这么做，因为如果你继承了Cocoa的类，很容易覆盖原来的私有方法。

<!--more-->

##Item21 Understand the Objective-C Error Model

1.抛出exception后，本来将要释放的对象将得不到释放，所以会造成内存泄露，所以抛异常时一定是非常严重的错误出现的场景。  
2.场景一是基类的一些必须被子类重写的方法可以抛出异常已告知开发者去重写，因为OC没有基类的特殊概念。

```objectivec
-(void)mustOverrideMethod {
	NSString *reason = [NSString stringWithFormat:@"%@ must be overridden", NSStringRromSelector(_cmd)];
	@throw[NSException exceptionWithName:NSInternalInconsistencyException reason:reason userInfo:nil];
}
```

3.而处理一般的异常OC通常使用NSError，该类包含以下信息：  
1）Error domain(String):表明错误发生的域，一般是自定义的全局变量，例如：*NSURLErrorDomain*。
2）Error code(Integer):表明特定域的错误码，参考HTTP的状态码。  
3）Userinfo(Dictionary):额外的信息，包括本地化描述信息和导致该错误的原因。  
4.NSError的一些使用场景：  
1）被用于Delegate中，例如：-(void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error。  
2）用于返回型参数，参照Item19，类似：  
```objectivec
//-(BOOL)doSomething:(NSError**)error
NSError *error = nil;
BOOL ret = [object doSomething:&error];
if(error) {
	//There was an error
}
```  
5.上述方法传入的是NSError**类型，开启ARC时该类型会转化为NSError* __autoreleasing*类型，该对象会在方法执行后自动释放，这么做，是因为doSomething:不能确定调用者会不会对NSError释放，大部分方法return的对象也是一样会添加autorelease（除了new，alloc，copy，mutableCopy等）。  
6.doSomething的内部实现：  

```objectivec
-(BOOL)doSomething:(NSError**)error {
	//Do something that may cause an error
	if(/*there was an error*/) {
		//有必要检查error，有可能传入nil值
		if(error) {
			//Pass error through the out-parameter
			*error = [NSError errorWithDomain:domain code:code userInfo:userInfo];
		}
		return NO;
	} else {
		return YES;
	}
}
```  

<!--more-->

##Item22 Understand the NSCopying Protocol

1.想要让自定义对象实现copy功能，必须遵循NSCopying协议，其中只有一个方法需要重写：-(id)copyWithZone:(NSZone*)zone。  
2.一个重写copyWithZone:方法的例子，_friends是内部变量，所以使用了copy->_friends：

```objectivec
-(id)copyWithZone:(NSZone *)zone {
	EOCPerson *copy = [[[self class] allocWithZone:zone] initWithFirstName:_firstName andLastName:_lastName];
	copy->_friends = [_friends mutableCopy];
	return copy;
}
```

3.关于这儿是否需要对_friends进行copy的讨论，作者认为如果原变量是可变的，是需要深拷贝的，而如果原变量是不可变的，则直接进行指针赋值即可，这样可以省一部分内存。  
4.如果你的类有mutable和immutable两个类型，那么应该分别遵循NSMutableCopying和NSCopying协议，分别返回可变和不可变的copy。  
5.采取这种方式的好处是可以提供一个可变与不可变类型的转换，而且采用copy，immutableCopy，mutableCopy三个方法的缺陷是我们很难判断将要复制的对象是不是可变的。  
6.接下来讨论的是深拷贝和浅拷贝的问题，OC默认的Copy协议支持的都是浅拷贝，也就是指针拷贝，但是一些类的初始化方法提供了深拷贝，例如NSSet的：-(id)initWithSet:(NSArray*)array copyItems:(BOOL)copyItems。所以如果你需要进行深拷贝，必须自己定义：

```objectivec
-(id)deepCopy {
	EOCPerson *copy = [[[self class] alloc] initWithFirstName:_firstName andLastName:_lastName];
	copy->_friends = [[NSMutableSet alloc] initWithSet:_friends copyItems:YES];
	return copy;
}
```

	