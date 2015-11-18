---
layout: post
title: "CALayer&amp;CoreAnimation"
date: 2015-11-09 10:18:03 +0800
comments: true
categories: iOS
published: true
---

这两天打算系统的整理一下CALayer和CoreAnimation相关的知识，之前开发中只是在具体场景中使用时才会去找相应的解决方法，而没有系统的进行整理，所以打算开一篇专门介绍。这一篇[博客](http://www.cnblogs.com/kenshincui/p/3972100.html)也做了详尽的介绍，本文的思路也与其基本一致，另外会记录自己的一些理解。

<!--more-->

## CALayer相关

CALayer包含在QuartzCore框架中。它与UIView的区别是，UIView是UIResponder的子类，是可以接受事件并做出响应的，而CALayer是NSObject的子类，它只是用来展示内容的类。而UIView有一个layer属性就是用来绘制其图像的，而复杂的图层结构在CALayer中是以树的形式存储的，而CoreAnimation中的动画都是在CALayer上进行操作的，可能我们在开发中大部分是直接在UIView组织动画，而这些其实是CoreAnimation在UIView层的封装，本质还是对其layer属性进行操作。

<img src="/images/CALayerTree.png">

### CALayer属性

下表列出了CALayer常用的一些属性：

| 属性 | 说明 | 是否支持隐式动画 |
| ------------ | ------------- |:------------:|
| anchorPoint | 锚点，CGPoint类型，默认为(0.5,0.5)，可以理解为layer的重心，始终与position位置重合 | 是 
| backgroundColor | 背景色  | 是 
| borderColor | 边框颜色  | 是 
| borderWidth | 边框宽度  | 是 
| bounds | 范围大小  | 是 
| contents | id类型，layer显示内容，通常是CGImageRef类型 | 是 
| contentsRect | layer内容的位置和范围  | 是 
| cornerRadius | 圆角半径 | 是 
| doubleSided | 图层背面是否显示，默认YES  | 否 
| frame | layer的位置和范围，由于不支持隐式动画，所以改变layer的位置和大小，通常使用修改bounds和position来替代 | 否 
| hidden | 是否隐藏 | 是 
| mask | 图层蒙版 | 是 
| maskToBounds | 子图层是否剪切图层边界，默认NO | 是 
| opacity | 透明度，类似UIView的alpha | 是 
| position | layer中心位置，类似UIView的center | 是 
| shadowColor | 阴影颜色 | 是 
| shadowOffset | 阴影偏移量 | 是 
| shadowOpacity | 阴影透明度，默认为0，所以设置阴影时必须设置此值 | 是 
| shadowPath| 阴影形状 | 是 
| shadowRadius | 阴影模糊半径 | 是 
| sublayers | 子图层数组 | 是 
| sublayerTransform | 子图层形变 | 是 
| transform | 图层形变 | 是 

  
### 隐式动画

CALayer很多属性在修改时就能形成动画，被称为隐式动画，但是需要注意UIView的根视图layer的属性修改并不会形成动画，因为根图层一般是作为容器来使用，修改它的属性可能会直接影响子图层。另外，UIView的根图层是由系统创建的，而无法重新创建，但可以添加子图层。

隐式动画的本质是这些属性的变化默认实现了CABasicAnimation，具体见[Apple文档](https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/AnimatableProperties/AnimatableProperties.html#//apple_ref/doc/uid/TP40004514-CH11-SW2)，具体示例见[AnimationCase](https://github.com/lucifer1988/AnimationCase)中的AnimatablePropertiesTest。

```objectivec
@interface ACAnimatablePropertiesViewController () {
    CALayer *layer;
}

@end

static CGFloat width = 150.0;

@implementation ACAnimatablePropertiesViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    layer = [[CALayer alloc] init];
    layer.bounds = CGRectMake(0, 0, width, width);
    layer.position = self.view.center;
    layer.backgroundColor = [UIColor blueColor].CGColor;
    layer.cornerRadius = width/2;
    
    layer.shadowColor = [UIColor grayColor].CGColor;
    layer.shadowOffset = CGSizeMake(2, 3);
    layer.shadowOpacity = 0.9;
    
    layer.borderWidth = 2.0;
    layer.borderColor = [UIColor greenColor].CGColor;
    
    [self.view.layer addSublayer:layer];
}

- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    UITouch *touch = [touches anyObject];
    CGFloat randWidth = arc4random()%200+100;
    layer.bounds = CGRectMake(0, 0, randWidth, randWidth);
    layer.cornerRadius = randWidth/2;
    layer.position = [touch locationInView:self.view];
    layer.backgroundColor = [UIColor colorWithRed:arc4random()%225/225.0 green:arc4random()%225/225.0 blue:arc4random()%225/225.0 alpha:1.0].CGColor;
    layer.borderColor = [UIColor colorWithRed:arc4random()%225/225.0 green:arc4random()%225/225.0 blue:arc4random()%225/225.0 alpha:1.0].CGColor;
}

@end
```

[AnchorPoint](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CALayer_class/index.html#//apple_ref/occ/instp/CALayer/anchorPoint)的作用：图层的锚点，范围在（0~1,0~1）表示在x、y轴的比例，这个点永远可以同position重合，当图层中心点固定后，调整anchorPoint即可达到调整图层显示位置的作用（因为它永远和position重合），类似旋转动画中，改变anchorPoint的值可以改变旋转的中心位置，详见[AnimationCase](https://github.com/lucifer1988/AnimationCase)中的AnchorPointTest。

```objectivec
@interface ACAnchorPointViewController () <UITextFieldDelegate> {
    CALayer *layer;
}

@property (weak, nonatomic) IBOutlet UITextField *xField;
@property (weak, nonatomic) IBOutlet UITextField *yField;

@end

@implementation ACAnchorPointViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    layer = [[CALayer alloc] init];
    layer.backgroundColor = [UIColor lightGrayColor].CGColor;
    layer.position = self.view.center;
    layer.bounds = CGRectMake(0, 0, 150.0, 150.0);
    
    CABasicAnimation *rotationAnimation = [CABasicAnimation animationWithKeyPath:@"transform.rotation.z"];
    rotationAnimation.duration = 2;
    rotationAnimation.repeatCount = HUGE_VALF;
    rotationAnimation.removedOnCompletion = NO;
    rotationAnimation.fromValue = [NSNumber numberWithFloat:0];
    rotationAnimation.toValue = [NSNumber numberWithFloat:3.1415926*2];
    [layer addAnimation:rotationAnimation forKey:@"rotationTransform"];
    
    [self.view.layer addSublayer:layer];
}

#pragma mark - UITextFieldDelegate

- (BOOL)textFieldShouldReturn:(UITextField *)textField {
    double value = textField.text.length ? textField.text.doubleValue : 0.5;
    if (value < 0 || value > 1) {
        UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"输入值必须介于0到1" message:nil preferredStyle:UIAlertControllerStyleAlert];
        [alert addAction:[UIAlertAction actionWithTitle:@"我知道了" style:UIAlertActionStyleCancel handler:nil]];
        [self presentViewController:alert animated:YES completion:nil];
        return NO;
    }
    if (textField == self.xField) {
        CGPoint point = layer.anchorPoint;
        point.x = value;
        layer.anchorPoint = point;
    } else {
        CGPoint point = layer.anchorPoint;
        point.y = value;
        layer.anchorPoint = point;
    }
    [textField resignFirstResponder];
    return YES;
}

@end
```

### CALayer绘图

CALayer的绘图方法主要有两种，不过都需要调用layer的setNeedDisplay方法（需要注意的是必须是layer调用，而不是UIView，因为UIView也有一个setNeedDisplay方法）。

* 通过实现代理方法**drawLayer:inContext:**来绘制
* 通过自定义图层**drawInContext:**来绘制

#### 实现代理方法绘制layer

通过代理方法进行图层绘图只要指定图层的代理，然后在代理对象中重写-(void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx方法即可。需要注意这个方法虽然是代理方法但是不用手动实现CALayerDelegate，因为CALayer定义中给NSObject做了分类扩展，所有的NSObject都包含这个方法。另外设置完代理后必须要调用图层的setNeedDisplay方法，否则绘制的内容无法显示。

使用代理方法绘制图形、图像时在drawLayer:inContext:方法中可以通过事件参数获得绘制的图层和图形上下文。在这个方法中绘图时所有的位置都是相对于图层而言的，图形上下文指的也是当前图层的图形上下文，详见[AnimationCase](https://github.com/lucifer1988/AnimationCase)中的DrawLayerByDelegateTest。

```objectivec
@interface ACDrawLayerByDelegateViewController () {
    CALayer *layer;
}

@end

static CGFloat width = 150.0;

@implementation ACDrawLayerByDelegateViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    layer = [[CALayer alloc] init];
    layer.bounds = CGRectMake(0, 0, width, width);
    layer.position = self.view.center;
    layer.backgroundColor = [UIColor blueColor].CGColor;
    layer.cornerRadius = width/2;
    layer.masksToBounds = YES;
    
    layer.borderWidth = 2.0;
    layer.borderColor = [UIColor grayColor].CGColor;
    
    layer.delegate = self;
    
    [self.view.layer addSublayer:layer];
    
    [layer setNeedsDisplay];
}

- (void)dealloc {
    layer.delegate = nil;
}

#pragma mark - layer delegate

- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx {
    CGContextSaveGState(ctx);
    
    CGContextScaleCTM(ctx, 1, -1);
    CGContextTranslateCTM(ctx, 0, -width);
    
    UIImage *image = [UIImage imageNamed:@"avatar.png"];
    CGContextDrawImage(ctx, CGRectMake(0, 0, width, width), image.CGImage);
    
    CGContextRestoreGState(ctx);
}

@end
```

另外需要注意的是上面代码中绘制图片圆形裁切效果时如果不设置masksToBounds是无法显示圆形，但是对于其他图形却没有这个限制。原因就是当绘制一张图片到图层上的时候会重新创建一个图层添加到当前图层，这样一来如果设置了圆角之后虽然底图层有圆角效果，但是子图层还是矩形，只有设置了masksToBounds为YES让子图层按底图层剪切才能显示圆角效果。同样的，有些朋友经常在网上提问说为什么使用UIImageView的layer设置圆角后图片无法显示圆角，只有设置masksToBounds才能出现效果，也是类似的问题。

#### 自定义layer来绘制

在自定义图层中绘图时只要自己编写一个类继承于CALayer然后在drawInContext:中绘图即可。同前面在代理方法绘图一样，要显示图层中绘制的内容也要调用图层的setNeedDisplay方法，否则drawInContext方法将不会调用，详见[AnimationCase](https://github.com/lucifer1988/AnimationCase)中的DrawLayerByCustomTest。

```objectivec
@implementation ACCustomLayer

- (void)drawInContext:(CGContextRef)ctx {
    CGContextSetRGBFillColor(ctx, 135.0/255.0, 232.0/255.0, 84.0/255.0, 1);
    CGContextSetRGBStrokeColor(ctx, 135.0/255.0, 232.0/255.0, 84.0/255.0, 1);
    CGPoint center = CGPointMake(width/2, width/2);
    CGContextMoveToPoint(ctx, center.x, center.y - 60.0);
    for(int i = 1; i < 5; ++i) {
        CGFloat x = 60.0 * sinf(i * 4.0 * M_PI / 5.0);
        CGFloat y = 60.0 * cosf(i * 4.0 * M_PI / 5.0);
        CGContextAddLineToPoint(ctx, center.x - x, center.y - y);
        NSLog(@"x:%f, y:%f", center.x - x, center.y - y);
    }
    
    CGContextClosePath(ctx);
    
    CGContextDrawPath(ctx, kCGPathFillStroke);
}

@end
```

```objectivec
@interface ACDrawLayerCustomViewController ()

@end

@implementation ACDrawLayerCustomViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    ACCustomLayer *layer = [[ACCustomLayer alloc] init];
    layer.bounds = CGRectMake(0, 0, width, width);
    layer.position = self.view.center;
    layer.backgroundColor = [UIColor blueColor].CGColor;
    
    [self.view.layer addSublayer:layer];
    [layer setNeedsDisplay];
}

@end
```

### 带阴影效果的圆形图片裁切

如果设置了masksToBounds=YES之后确实可以显示图片圆角效果，但遗憾的是设置了这个属性之后就无法设置阴影效果。因为masksToBounds=YES就意味着外边框不能显示，而阴影恰恰作为外边框绘制的，这样两个设置就产生了矛盾。要解决这个问题不妨换个思路:使用两个大小一样的图层，下面的图层负责绘制阴影，上面的图层用来显示图片。

```objectivec
@interface ACCircleAvatarWithShadowViewController () {
    CALayer *layer;
}

@end

static CGFloat width = 150.0;

@implementation ACCircleAvatarWithShadowViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    CALayer *shadowLayer = [[CALayer alloc] init];
    shadowLayer.bounds = CGRectMake(0, 0, width, width);
    shadowLayer.position = self.view.center;
    shadowLayer.backgroundColor = [UIColor whiteColor].CGColor;
    shadowLayer.cornerRadius = width/2;
    shadowLayer.shadowOffset = CGSizeMake(2, 2);
    shadowLayer.shadowColor = [UIColor grayColor].CGColor;
    shadowLayer.shadowOpacity = 1.0;
    
    [self.view.layer addSublayer:shadowLayer];
    
    layer = [[CALayer alloc] init];
    layer.bounds = CGRectMake(0, 0, width, width);
    layer.position = self.view.center;
    layer.backgroundColor = [UIColor blueColor].CGColor;
    layer.cornerRadius = width/2;
    layer.masksToBounds = YES;
    
    layer.borderWidth = 2.0;
    layer.borderColor = [UIColor grayColor].CGColor;
    
    layer.delegate = self;
    
    [self.view.layer addSublayer:layer];
    
    [layer setNeedsDisplay];
}

- (void)dealloc
{
    layer.delegate = nil;
}

#pragma mark - layer delegate

- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx {
    CGContextSaveGState(ctx);
    
    CGContextScaleCTM(ctx, 1, -1);
    CGContextTranslateCTM(ctx, 0, -width);
    
    UIImage *image = [UIImage imageNamed:@"avatar.png"];
    CGContextDrawImage(ctx, CGRectMake(0, 0, width, width), image.CGImage);
    
    CGContextRestoreGState(ctx);
}

@end
```

### CALayer形变

从上面代码中大家不难发现使用Core Graphics绘制图片时会倒立显示，对图层的图形上下文进行了反转。可以控制图层直接旋转而不用借助于图形上下文的形变操作。对于上面的程序，只需要设置图层的transform属性即可。需要注意的是transform是CATransform3D类型，形变可以在三个维度上进行，而且都有对应的形变设置方法（如：CATransform3DMakeTranslation()、CATransform3DMakeScale()、CATransform3DMakeRotation()）。

事实上如果仅仅就显示一张图片在图层中当然没有必要那么麻烦，直接设置图层contents就可以了，不牵涉到绘图也就没有倒立的问题了。

```objectivec
//Plan B
CALayer *anotherlayer = [[CALayer alloc] init];
anotherlayer.bounds = CGRectMake(0, 0, width, width);
center = self.view.center;
center.y += width;
anotherlayer.position = center;
anotherlayer.backgroundColor = [UIColor blueColor].CGColor;
anotherlayer.cornerRadius = width/2;
anotherlayer.masksToBounds = YES;
anotherlayer.borderColor = [UIColor grayColor].CGColor;
anotherlayer.borderWidth = 2.0;
UIImage *image=[UIImage imageNamed:@"avatar.png"];
[anotherlayer setContents:(id)image.CGImage];

[self.view.layer addSublayer:anotherlayer];
```

在动画开发中形变往往不是直接设置transform，而是通过keyPath进行设置。这种方法设置形变的本质和前面没有区别，只是利用了KVC可以动态修改其属性值而已，但是这种方式在动画中确实很常用的，因为它可以很方便的将几种形变组合到一起使用。key path的所有设置类型，见[Apple文档](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/Key-ValueCodingExtensions/Key-ValueCodingExtensions.html#//apple_ref/doc/uid/TP40004514-CH12-SW1)，同样是解决动画旋转问题，只要将前面的旋转代码改为下面的代码即可：

```objectivec
[layer setValue:@M_PI forKeyPath:@"transform.rotation.x"];
```

## CoreAnimation相关

在iOS中CoreAnimation分为几类：基础动画、关键帧动画、动画组、转场动画。各个类的关系大致如下：

<img src="/images/CAAnimationTree.png">

* CAAnimation：核心动画的基础类，不能直接使用，负责动画运行时间、速度的控制，本身实现了CAMediaTiming协议。
* CAPropertyAnimation：属性动画的基类（通过属性进行动画设置，注意是可动画属性），不能直接使用。
* CAAnimationGroup：动画组，动画组是一种组合模式设计，可以通过动画组来进行所有动画行为的统一控制，组中所有动画效果可以并发执行。
* CATransition：转场动画，主要通过滤镜进行动画效果设置。
* CABasicAnimation：基础动画，通过属性修改进行动画参数控制，只有初始状态和结束状态。
* CAKeyframeAnimation：关键帧动画，同样是通过属性进行动画参数控制，但是同基础动画不同的是它可以有多个状态控制。

基础动画、关键帧动画都属于属性动画，就是通过修改属性值产生动画效果，开发人员只需要设置初始值和结束值，中间的过程动画（又叫“补间动画”）由系统自动计算产生。和基础动画不同的是关键帧动画可以设置多个属性值，每两个属性中间的补间动画由系统自动完成，因此从这个角度而言基础动画又可以看成是有两个关键帧的关键帧动画。

### 基础动画

在开发过程中很多情况下通过基础动画就可以满足开发需求，使用的UIView代码块制作的动画也是基础动画（在iOS7中UIView也对关键帧动画进行了封装），只是UIView装饰方法隐藏了更多的细节。如果不使用UIView封装的方法，动画创建一般分为以下几步：

1. 初始化动画并设置动画属性

2. 设置动画属性初始值（可以省略）、结束值以及其他动画属性

3. 给图层添加动画

#### 移动动画

下面以一个移动动画为例进行演示，在这个例子中点击屏幕哪个位置落花将飞向哪里：

```objectivec
@interface ACMoveAnimationViewController () {
    CALayer *petalLayer;
}

@end

@implementation ACMoveAnimationViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    UIImage *backgroundImage = [UIImage imageNamed:@"background.jpg"];
    self.view.layer.contents = (id)backgroundImage.CGImage;
    
    petalLayer = [[CALayer alloc] init];
    petalLayer.bounds = CGRectMake(0, 0, 10, 20);
    petalLayer.position = CGPointMake(50, 150);
    petalLayer.contents = (id)[UIImage imageNamed:@"petal.png"].CGImage;
    
    [self.view.layer addSublayer:petalLayer];
}

- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    UITouch *touch = touches.anyObject;
    [self animationMoveTo:[touch locationInView:self.view]];
}

#pragma mark - Animation

- (void)animationMoveTo:(CGPoint)location {
    CABasicAnimation *moveAnimation = [CABasicAnimation animationWithKeyPath:@"position"];
    moveAnimation.toValue = [NSValue valueWithCGPoint:location];
    moveAnimation.duration = 5.0;
    moveAnimation.repeatCount = 0;
    moveAnimation.removedOnCompletion = YES;
    [petalLayer addAnimation:moveAnimation forKey:@"petalLayer_moveAnimation"];
}

@end
```

#### 完整移动动画

上面实现了一个基本动画效果，但是这个动画存在一个问题：动画结束后动画图层回到了原来的位置。

图层动画的本质就是将图层内部的内容转化为位图经硬件操作形成一种动画效果，其实图层本身并没有任何的变化。上面的动画中图层并没有因为动画效果而改变它的位置（对于缩放动画其大小也是不会改变的），所以动画完成之后图层还是在原来的显示位置没有任何变化，如果这个图层在一个UIView中你会发现在UIView移动过程中你要触发UIView的点击事件也只能点击原来的位置（即使它已经运动到了别的位置），因为它的位置从来没有变过。

通过给动画设置一个代理去监听动画的开始和结束事件，在动画开始前给动画添加一个自定义属性“petalLayer_moveAnimation_destination”存储动画终点位置，然后在动画结束后设置动画的位置为终点位置。在**- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag**中设置代码关闭了position的隐式动画，否则会导致两次移动，另外添加了使花瓣匀速移动的时间计算，和防止多次点击的flag控制。

```objectivec
@interface ACMoveAnimationFullEditionViewController () {
    CALayer *petalLayer;
    BOOL isMoving;
}

@end

@implementation ACMoveAnimationFullEditionViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    isMoving = NO;
    UIImage *backgroundImage = [UIImage imageNamed:@"background.jpg"];
    self.view.layer.contents = (id)backgroundImage.CGImage;
    
    petalLayer = [[CALayer alloc] init];
    petalLayer.bounds = CGRectMake(0, 0, 10, 20);
    petalLayer.position = CGPointMake(50, 150);
    petalLayer.contents = (id)[UIImage imageNamed:@"petal.png"].CGImage;
    
    [self.view.layer addSublayer:petalLayer];
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    if (isMoving) {
        return;
    }
    
    UITouch *touch = touches.anyObject;
    [self animationMoveTo:[touch locationInView:self.view]];
}

#pragma mark - Animation

- (void)animationMoveTo:(CGPoint)location {
    CABasicAnimation *moveAnimation = [CABasicAnimation animationWithKeyPath:@"position"];
    moveAnimation.toValue = [NSValue valueWithCGPoint:location];
    moveAnimation.duration = [self petalMoveDuring:location];
    moveAnimation.repeatCount = 0;
    moveAnimation.removedOnCompletion = YES;
    
    moveAnimation.delegate = self;
    [moveAnimation setValue:[NSValue valueWithCGPoint:location] forKey:@"petalLayer_moveAnimation_destination"];
    
    [petalLayer addAnimation:moveAnimation forKey:@"petalLayer_moveAnimation"];
}

- (NSTimeInterval)petalMoveDuring:(CGPoint)destination {
    CGFloat deltaX = destination.x - petalLayer.position.x;
    CGFloat deltaY = destination.y - petalLayer.position.y;
    CGFloat distance = sqrt(deltaX*deltaX + deltaY*deltaY);
    return (NSTimeInterval)distance/50.0;
}

#pragma mark - Animation Delegate

- (void)animationDidStart:(CAAnimation *)anim {
    isMoving = YES;
}

- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag {
    [CATransaction begin];
    //禁用隐式动画
    [CATransaction setDisableActions:YES];
    petalLayer.position=[[anim valueForKey:@"petalLayer_moveAnimation_destination"] CGPointValue];
    [CATransaction commit];
    isMoving = NO;
}

@end
```

#### 旋转动画

图层的形变都是基于锚点进行的。例如旋转，旋转的中心点就是图层的锚点，这在最开始的锚点例子中也有介绍。

需要注意的是只给移动动画设置了代理，在旋转动画中并没有设置代理，否则代理方法会执行两遍。由于旋转动画会无限循环执行（上面设置了重复次数无穷大），并且两个动画的执行时间没有必然的关系，这样一来移动停止后可能还在旋转，为了让移动动画停止后旋转动画停止就需要使用到动画的暂停和恢复方法。

核心动画的运行有一个媒体时间的概念，假设将一个旋转动画设置旋转一周用时60秒的话，那么当动画旋转90度后媒体时间就是15秒。如果此时要将动画暂停只需要让媒体时间偏移量设置为15秒即可，并把动画运行速度设置为0使其停止运动。类似的，如果又过了60秒后需要恢复动画（此时媒体时间为75秒），这时只要将动画开始开始时间设置为当前媒体时间75秒减去暂停时的时间（也就是之前定格动画时的偏移量）15秒（开始时间=75-15=60秒），那么动画就会重新计算60秒后的状态再开始运行，与此同时将偏移量重新设置为0并且把运行速度设置1。这个过程中真正起到暂停动画和恢复动画的其实是动画速度的调整，媒体时间偏移量以及恢复时的开始时间设置主要为了让动画更加连贯。

```objectivec
- (void)animationRotate {
    CABasicAnimation *rotateAnimation = [CABasicAnimation animationWithKeyPath:@"transform.rotation.z"];
    rotateAnimation.duration = 6;
    rotateAnimation.repeatCount = HUGE_VALF;
    rotateAnimation.removedOnCompletion = NO;
    rotateAnimation.autoreverses = YES;
    rotateAnimation.toValue = [NSNumber numberWithFloat:M_PI_2*3];
    [petalLayer addAnimation:rotateAnimation forKey:@"petalLayer_rotationAnimation"];
}

- (void)animationPause {
    //取得指定图层动画的媒体时间，后面参数用于指定子图层，这里不需要
    CFTimeInterval interval = [petalLayer convertTime:CACurrentMediaTime() fromLayer:nil];
    //设置时间偏移量，保证暂停时停留在旋转的位置
    [petalLayer setTimeOffset:interval];
    //速度设置为0，暂停动画
    petalLayer.speed = 0;
}

- (void)animationResume {
    //获得暂停的时间
    CFTimeInterval beginTime = CACurrentMediaTime() - petalLayer.timeOffset;
    //设置偏移量
    petalLayer.timeOffset = 0;
    //设置开始时间
    petalLayer.beginTime = beginTime;
    //设置动画速度，开始运动
    petalLayer.speed = 1.0;
}
```

### 关键帧动画

熟悉flash开发的朋友对于关键帧动画应该不陌生，这种动画方式在flash开发中经常用到。关键帧动画就是在动画控制过程中开发者指定主要的动画状态，至于各个状态间动画如何进行则由系统自动运算补充（每两个关键帧之间系统形成的动画称为“补间动画”），这种动画的好处就是开发者不用逐个控制每个动画帧，而只要关心几个关键帧的状态即可。

关键帧动画开发分为两种形式：一种是通过设置不同的属性值进行关键帧控制，另一种是通过绘制路径进行关键帧控制。后者优先级高于前者，如果设置了路径则属性值就不再起作用。

#### 设置属性值的关键帧控制

对于前面的落花动画效果而言其实落花的过程并不自然，很显然实际生活中它不可能沿着直线下落，这里我们不妨通过关键帧动画的values属性控制它在下落过程中的属性。

```objectivec
@interface ACKeyFrameAnimationViewController () {
    CALayer *petalLayer;
}

@end

@implementation ACKeyFrameAnimationViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    UIImage *backgroundImage = [UIImage imageNamed:@"background.jpg"];
    self.view.layer.contents = (id)backgroundImage.CGImage;
    
    petalLayer = [[CALayer alloc] init];
    petalLayer.bounds = CGRectMake(0, 0, 10, 20);
    petalLayer.position = CGPointMake(50, 150);
    petalLayer.contents = (id)[UIImage imageNamed:@"petal.png"].CGImage;
    
    [self.view.layer addSublayer:petalLayer];
    
    [self addKeyframeMoveAnimation];
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

#pragma mark - animation

- (void)addKeyframeMoveAnimation {
    CAKeyframeAnimation *keyframeAnimation = [CAKeyframeAnimation animationWithKeyPath:@"position"];
    //关键帧动画的初始值不能省略
    NSValue *key1 = [NSValue valueWithCGPoint:petalLayer.position];
    NSValue *key2 = [NSValue valueWithCGPoint:CGPointMake(80, 320)];
    NSValue *key3 = [NSValue valueWithCGPoint:CGPointMake(45, 400)];
    NSValue *key4 = [NSValue valueWithCGPoint:CGPointMake(55, 500)];
    NSArray *values = @[key1, key2, key3, key4];
    
    keyframeAnimation.values = values;
    keyframeAnimation.duration = 8.0;
    keyframeAnimation.beginTime = CACurrentMediaTime()+2;
    keyframeAnimation.delegate = self;
    [keyframeAnimation setValue:key4 forKey:@"petalLayer_keyframeAnimation_destination"];
    
    [petalLayer addAnimation:keyframeAnimation forKey:@"petalLayer_keyframeAnimation_position"];
}

#pragma mark - animation delegate

- (void)animationDidStart:(CAAnimation *)anim {
}

- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag {
    [CATransaction begin];
    [CATransaction setDisableActions:YES];
    petalLayer.position = [[anim valueForKey:@"petalLayer_keyframeAnimation_destination"] CGPointValue];
    [CATransaction commit];
}

@end
```

#### 通过设置路径的关键帧控制

上面的方式固然比前面使用基础动画效果要好一些，但其实还是存在问题，那就是落花飞落的路径是直线的，当然这个直线是根据程序中设置的四个关键帧自动形成的，那么如何让它沿着曲线飘落呢？这就是第二种类型的关键帧动画，通过描绘路径进行关键帧动画控制。假设让落花沿着一条贝塞尔曲线飘落：

```objectivec
- (void)addKeyframeMoveAnimation {
    CAKeyframeAnimation *keyframeAnimation = [CAKeyframeAnimation animationWithKeyPath:@"position"];
    
    CGMutablePathRef path = CGPathCreateMutable();
    CGPathMoveToPoint(path, NULL, petalLayer.position.x, petalLayer.position.y);
    CGPathAddCurveToPoint(path, NULL, 300, 250, -100, 450, 100, 550);
    
    keyframeAnimation.path = path;
    CGPathRelease(path);
    keyframeAnimation.duration = 8.0;
    keyframeAnimation.beginTime = CACurrentMediaTime()+2;
    keyframeAnimation.delegate = self;
    [keyframeAnimation setValue:[NSValue valueWithCGPoint:CGPointMake(100, 550)] forKey:@"petalLayer_keyframeAnimation_destination"];
    
    [petalLayer addAnimation:keyframeAnimation forKey:@"petalLayer_keyframeAnimation_position"];
}
```

#### 关键帧动画的其他重要属性

* keyTimes：各个关键帧的时间控制。前面使用values设置了四个关键帧，默认情况下每两帧之间的间隔为:8/(4-1)秒。如果想要控制动画从第一帧到第二针占用时间4秒，从第二帧到第三帧时间为2秒，而从第三帧到第四帧时间2秒的话，就可以通过keyTimes进行设置。keyTimes中存储的是时间占用比例点，此时可以设置keyTimes的值为0.0，0.5，0.75，1.0（当然必须转换为NSNumber），也就是说1到2帧运行到总时间的50%，2到3帧运行到总时间的75%，3到4帧运行到8秒结束。
* caculationMode：动画计算模式。还拿上面keyValues动画举例，之所以1到2帧能形成连贯性动画而不是直接从第1帧经过8/3秒到第2帧是因为动画模式是连续的（值为kCAAnimationLinear，这是计算模式的默认值）；而如果指定了动画模式为kCAAnimationDiscrete离散的那么你会看到动画从第1帧经过8/3秒直接到第2帧，中间没有任何过渡。其他动画模式还有：kCAAnimationPaced（均匀执行，会忽略keyTimes）、kCAAnimationCubic（平滑执行，对于位置变动关键帧动画运行轨迹更平滑）、kCAAnimationCubicPaced（平滑均匀执行）。

<img src="/images/caculationMode.png">

### 组合动画

实际开发中一个物体的运动往往是复合运动，单一属性的运动情况比较少，但恰恰属性动画每次进行动画设置时一次只能设置一个属性进行动画控制(不管是基础动画还是关键帧动画都是如此)，这样一来要做一个复合运动的动画就必须创建多个属性动画进行组合。对于一两种动画的组合或许处理起来还比较容易，但是对于更多动画的组合控制往往会变得很麻烦，动画组的产生就是基于这样一种情况而产生的。动画组是一系列动画的组合，凡是添加到动画组中的动画都受控于动画组，这样一来各类动画公共的行为就可以统一进行控制而不必单独设置，而且放到动画组中的各个动画可以并发执行，共同构建出复杂的动画效果。

动画组使用起来并不复杂，首先单独创建单个动画（可以是基础动画也可以是关键帧动画），然后将基础动画添加到动画组，最后将动画组添加到图层即可。

前面关键帧动画部分，路径动画看起来效果虽然很流畅，但是落花本身的旋转运动没有了，这里不妨将基础动画部分的旋转动画和路径关键帧动画进行组合使得整个动画看起来更加的和谐、顺畅。

```objectivec
@interface ACGroupAnimationViewController () {
    CALayer *petalLayer;
}

@end

@implementation ACGroupAnimationViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    UIImage *backgroundImage = [UIImage imageNamed:@"background.jpg"];
    self.view.layer.contents = (id)backgroundImage.CGImage;
    
    petalLayer = [[CALayer alloc] init];
    petalLayer.bounds = CGRectMake(0, 0, 10, 20);
    petalLayer.position = CGPointMake(100, 150);
    petalLayer.contents = (id)[UIImage imageNamed:@"petal.png"].CGImage;
    
    [self.view.layer addSublayer:petalLayer];
    
    [self addAnimationGroup];
}

- (void)addAnimationGroup {
    CAAnimationGroup *animationGroup = [CAAnimationGroup animation];
    
    CABasicAnimation *rotateAnimation = [self rotateAnimation];
    CAKeyframeAnimation *keyframeMoveAnimation = [self keyframeMoveAnimation];
    animationGroup.animations = @[rotateAnimation, keyframeMoveAnimation];
    
    animationGroup.delegate = self;
    animationGroup.duration = 8.0;
    animationGroup.beginTime = CACurrentMediaTime()+2;
    
    [petalLayer addAnimation:animationGroup forKey:nil];
}

- (CAKeyframeAnimation *)keyframeMoveAnimation {
    CAKeyframeAnimation *keyframeAnimation = [CAKeyframeAnimation animationWithKeyPath:@"position"];
    
    CGMutablePathRef path = CGPathCreateMutable();
    CGPathMoveToPoint(path, NULL, petalLayer.position.x, petalLayer.position.y);
    CGPathAddCurveToPoint(path, NULL, 300, 250, -100, 450, 100, 550);
    
    keyframeAnimation.path = path;
    CGPathRelease(path);
    [keyframeAnimation setValue:[NSValue valueWithCGPoint:CGPointMake(100, 550)] forKey:@"petalLayer_keyframeAnimation_destination"];
    return keyframeAnimation;
}

- (CABasicAnimation *)rotateAnimation {
    CABasicAnimation *rotateAnimation = [CABasicAnimation animationWithKeyPath:@"transform.rotation.z"];
    rotateAnimation.repeatCount = HUGE_VALF;
    rotateAnimation.removedOnCompletion = NO;
    rotateAnimation.autoreverses = YES;
    rotateAnimation.toValue = [NSNumber numberWithFloat:M_PI_2*3];
    [rotateAnimation setValue:[NSNumber numberWithFloat:M_PI_2*3] forKey:@"rotateAnimation_toValue"];
    
    return rotateAnimation;
}

#pragma mark - animation delegate

- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag {
    CAAnimationGroup *animationGroup = (CAAnimationGroup *)anim;
    CABasicAnimation *rotateAnimation = (CABasicAnimation *)animationGroup.animations[0];
    CAKeyframeAnimation *keyframeMoveAnimation = (CAKeyframeAnimation *)animationGroup.animations[1];
    CGFloat toValue = [[rotateAnimation valueForKey:@"rotateAnimation_toValue"] floatValue];
    CGPoint toPoint = [[keyframeMoveAnimation valueForKey:@"petalLayer_keyframeAnimation_destination"] CGPointValue];
    
    [CATransaction begin];
    [CATransaction setDisableActions:YES];
    petalLayer.position = toPoint;
    petalLayer.transform = CATransform3DMakeRotation(toValue, 0, 0, 1);
    [CATransaction commit];
}

@end
```

### 转场动画

转场动画就是从一个场景以动画的形式过渡到另一个场景。转场动画的使用一般分为以下几个步骤：

1. 创建转场动画
2. 设置转场类型、子类型（可选）及其他属性
3. 设置转场后的新视图并添加动画到图层

Apple公开的只有四个转场动画类型：

1. fade(kCATransitionFade，支持方向)
2. moveIn(kCATransitionMoveIn，支持方向)
3. push(kCATransitionPush，支持方向)
4. reveal(kCATransitionReveal，支持方向)

另外还有一些私有API可以使用，但是只能用字符串来设置：

1. cube(支持方向)
2. oglFlip(支持方向)
3. suckEffect(不支持方向)
4. rippleEffect(不支持方向)
5. pageCurl(支持方向)
6. pageUnCurl(支持方向)
7. cameraIrisHollowOpen(不支持方向)
8. cameraIrisHollowClose(不支持方向)

支持方向的类型还可以选择四个subtype：

1. kCATransitionFromRight
2. kCATransitionFromLeft
3. kCATransitionFromTop
4. kCATransitionFromBottom

下面代码展示了全部效果：

```objectivec
@interface ACTransitionAnimationViewController () {
    UIImageView *imageView;
    NSArray *transitionTypes;
}

@end

static int transitionTypeIndex;

@implementation ACTransitionAnimationViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    transitionTypes = @[@"fade", @"moveIn", @"push", @"reveal", @"cube", @"oglFlip", @"suckEffect", @"rippleEffect", @"pageCurl", @"pageUnCurl", @"cameraIrisHollowOpen", @"cameraIrisHollowClose"];
    transitionTypeIndex = 0;
    
    imageView = [[UIImageView alloc] initWithFrame:[UIScreen mainScreen].bounds];
    [imageView setImage:[UIImage imageNamed:@"background.jpg"]];
    [self.view addSubview:imageView];
    
    UISwipeGestureRecognizer *swipeGesture = [[UISwipeGestureRecognizer alloc] initWithTarget:self action:@selector(swipe:)];
    swipeGesture.direction = UISwipeGestureRecognizerDirectionLeft;
    [self.view addGestureRecognizer:swipeGesture];
}

- (void)swipe:(UISwipeGestureRecognizer *)gestureRecognizer {
    if (transitionTypeIndex > 11) {
        transitionTypeIndex = 0;
    }
    CATransition *transition = [[CATransition alloc] init];
    transition.type = transitionTypes[transitionTypeIndex];
    transition.subtype = kCATransitionFromRight;
    transition.duration = 1.0;
    [imageView.layer addAnimation:transition forKey:@"imageView_transition"];
    
    transitionTypeIndex++;
}

@end
```

### 逐帧动画

前面介绍了核心动画中大部分动画类型，但是做过动画处理的朋友都知道，在动画制作中还有一种动画类型“逐帧动画”。说到逐帧动画相信很多朋友第一个想到的就是UIImageView，通过设置UIImageView的animationImages属性，然后调用它的startAnimating方法去播放这组图片。当然这种方法在某些场景下是可以达到逐帧的动画效果，但是它也存在着很大的性能问题，并且这种方法一旦设置完图片中间的过程就无法控制了。

虽然在CoreAnimation中没有直接提供逐帧动画类型，但是却提供了用于完成逐帧动画的相关对象CADisplayLink。CADisplayLink是一个计时器，但是同NSTimer不同的是，CADisplayLink的刷新周期同屏幕完全一致。例如在iOS中屏幕刷新周期是60次/秒，CADisplayLink刷新周期同屏幕刷新一致也是60次/秒，这样一来使用它完成的逐帧动画（又称为“时钟动画”）完全感觉不到动画的停滞情况。

要将CADisplayLink加入到主线程的Runloop，它的时钟周期就和主运行循环保持一致，而主运行循环周期就是屏幕刷新周期。在CADisplayLink加入到主运行循环队列后就会循环调用目标方法，在这个方法中更新视图内容就可以完成逐帧动画。

当然这里不得不强调的是逐帧动画性能势必较低，但是对于一些事物的运动又不得不选择使用逐帧动画，例如人的运动，这是一个高度复杂的运动，基本动画、关键帧动画是不可能解决的。所大家一定要注意在循环方法中尽可能的降低算法复杂度，同时保证循环过程中内存峰值尽可能低。下面以一个鱼的运动为例为大家演示一下逐帧动画。

```objectivec
@interface ACFrameByFrameAnimationViewController () {
    UIImageView *imageView;
    CALayer *layer;
    NSMutableArray *imageArray;
}

@end

static int frameIndex = 0;

@implementation ACFrameByFrameAnimationViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    UIImage *backgroundImage = [UIImage imageNamed:@"background1.png"];
    self.view.layer.contents = (id)backgroundImage.CGImage;
    
    imageArray = [@[] mutableCopy];
    for (int i=0; i<10; i++) {
        UIImage *image = [UIImage imageNamed:[NSString stringWithFormat:@"fish%d.png", i]];
        [imageArray addObject:image];
    }
    
    //PlanA use UIImageView's AnimationImages
//    imageView = [[UIImageView alloc] initWithFrame:CGRectMake(0, 0, 87, 32)];
//    imageView.center = self.view.center;
//    imageView.animationDuration = imageArray.count/30;
//    [imageView setAnimationImages:imageArray];
//    [self.view addSubview:imageView];
//    [imageView startAnimating];
    
    //PlanB use CADisplayLink
    layer = [[CALayer alloc] init];
    layer.bounds = CGRectMake(0, 0, 87, 32);
    layer.position = self.view.center;
    [self.view.layer addSublayer:layer];
    
    CADisplayLink *displayLink = [CADisplayLink displayLinkWithTarget:self selector:@selector(nextFrame)];
    [displayLink addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSDefaultRunLoopMode];
}

//60fps
- (void)nextFrame {
    static int i = 0;
    //30fps
    if (++i%2 == 0) {
        UIImage *image = imageArray[frameIndex];
        layer.contents = (id)image.CGImage;
        frameIndex = (frameIndex+1)%10;
    }
}

@end
```

## UIView动画封装

其实UIView本身对于基本动画和关键帧动画、转场动画都有相应的封装，在对动画细节没有特殊要求的情况下使用起来也要简单的多。可以说在日常开发中90%以上的情况使用UIView的动画封装方法都可以搞定，因此在熟悉了核心动画的原理之后还是有必要给大家简单介绍一下UIView中各类动画使用方法的。

### 基本动画

#### 移动动画

下面是之前移动动画在UIView层的封装，使用了block封装，也可以选用之前的beginAnimations单独设置，效果是一样的。

```objectivec
@interface ACViewMoveAnimationViewController () {
    UIImageView *petalView;
}

@end

@implementation ACViewMoveAnimationViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    UIImage *backgroundImage = [UIImage imageNamed:@"background.jpg"];
    self.view.layer.contents = (id)backgroundImage.CGImage;
    
    petalView = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"petal.png"]];
    petalView.center = CGPointMake(50, 150);
    [self.view addSubview:petalView];
}

- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    UITouch *touch = touches.anyObject;
    CGPoint location = [touch locationInView:self.view];
    //方法1：block方式
    /*开始动画，UIView的动画方法执行完后动画会停留在重点位置，而不需要进行任何特殊处理
     duration:执行时间
     delay:延迟时间
     options:动画设置，例如自动恢复、匀速运动等
     completion:动画完成回调方法
     */
    [UIView animateWithDuration:[self petalMoveDuring:[touch locationInView:self.view]] delay:0.0 options:UIViewAnimationOptionCurveLinear animations:^{
        petalView.center = location;
    } completion:nil];
    //方法2：静态方法
    //开始动画
    //[UIView beginAnimations:@"KCBasicAnimation" context:nil];
    //[UIView setAnimationDuration:5.0];
    //[UIView setAnimationDelay:1.0];//设置延迟
    //[UIView setAnimationRepeatAutoreverses:NO];//是否回复
    //[UIView setAnimationRepeatCount:10];//重复次数
    //[UIView setAnimationStartDate:(NSDate *)];//设置动画开始运行的时间
    //[UIView setAnimationDelegate:self];//设置代理
    //[UIView setAnimationWillStartSelector:(SEL)];//设置动画开始运动的执行方法
    //[UIView setAnimationDidStopSelector:(SEL)];//设置动画运行结束后的执行方法
    //petalView.center = location;
    //开始动画
    //[UIView commitAnimations];
}

- (NSTimeInterval)petalMoveDuring:(CGPoint)destination {
    CGFloat deltaX = destination.x - petalView.center.x;
    CGFloat deltaY = destination.y - petalView.center.y;
    CGFloat distance = sqrt(deltaX*deltaX + deltaY*deltaY);
    NSLog(@"%f", distance);
    return (NSTimeInterval)distance/50.0;
}

@end
```

#### 弹簧动画效果

iOS7新加入了一个弹性动画的接口，具体实现见代码。

```objectivec
@interface ACViewSpringAnimationViewController () {
    UIImageView *ballView;
}

@end

@implementation ACViewSpringAnimationViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    ballView = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"ball"]];
    ballView.center = self.view.center;
    [self.view addSubview:ballView];
}

- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    UITouch *touch = touches.anyObject;
    CGPoint location = [touch locationInView:self.view];
    /*创建弹性动画
     damping:阻尼，范围0-1，阻尼越接近于0，弹性效果越明显
     velocity:弹性复位的速度
     */
    [UIView animateWithDuration:[self petalMoveDuring:location] delay:0 usingSpringWithDamping:0.5 initialSpringVelocity:0 options:UIViewAnimationOptionCurveLinear animations:^{
        ballView.center = location;
    } completion:nil];
}

- (NSTimeInterval)petalMoveDuring:(CGPoint)destination {
    CGFloat deltaX = destination.x - ballView.center.x;
    CGFloat deltaY = destination.y - ballView.center.y;
    CGFloat distance = sqrt(deltaX*deltaX + deltaY*deltaY);
    NSLog(@"%f", distance);
    return (NSTimeInterval)distance/150.0;
}

@end
```

#### UIView动画设置参数

在动画方法中有一个NS_OPTION参数，UIViewAnimationOptions类型，它是一个枚举类型，动画参数分为三类，对应CoreAnimation的各类设置，可以组合使用：

1. 常规动画属性设置（可以同时选择多个进行设置）
	* UIViewAnimationOptionLayoutSubviews：动画过程中保证子视图跟随运动。
	* UIViewAnimationOptionAllowUserInteraction：动画过程中允许用户交互。
	* UIViewAnimationOptionBeginFromCurrentState：所有视图从当前状态开始运行。
	* UIViewAnimationOptionRepeat：重复运行动画。
	* UIViewAnimationOptionAutoreverse ：动画运行到结束点后仍然以动画方式回到初始点。
	* UIViewAnimationOptionOverrideInheritedDuration：忽略嵌套动画时间设置。
	* UIViewAnimationOptionOverrideInheritedCurve：忽略嵌套动画速度设置。
	* UIViewAnimationOptionAllowAnimatedContent：动画过程中重绘视图（注意仅仅适用于转场动画）。 
	* UIViewAnimationOptionShowHideTransitionViews：视图切换时直接隐藏旧视图、显示新视图，而不是将旧视图从父视图移除（仅仅适用于转场动画）
	* UIViewAnimationOptionOverrideInheritedOptions ：不继承父动画设置或动画类型。
2. 动画速度控制（可从其中选择一个设置）
	* UIViewAnimationOptionCurveEaseInOut：动画先缓慢，然后逐渐加速。
	* UIViewAnimationOptionCurveEaseIn ：动画逐渐变慢。
	* UIViewAnimationOptionCurveEaseOut：动画逐渐加速。
	* UIViewAnimationOptionCurveLinear ：动画匀速执行，默认值。
3. 转场类型（仅适用于转场动画设置，可以从中选择一个进行设置，基本动画、关键帧动画不需要设置）
	* UIViewAnimationOptionTransitionNone：没有转场动画效果。
	* UIViewAnimationOptionTransitionFlipFromLeft ：从左侧翻转效果。
	* UIViewAnimationOptionTransitionFlipFromRight：从右侧翻转效果。
	* UIViewAnimationOptionTransitionCurlUp：向后翻页的动画过渡效果。   
	* UIViewAnimationOptionTransitionCurlDown ：向前翻页的动画过渡效果。   
	* UIViewAnimationOptionTransitionCrossDissolve：旧视图溶解消失显示下一个新视图的效果。   
	* UIViewAnimationOptionTransitionFlipFromTop ：从上方翻转效果。   
	* UIViewAnimationOptionTransitionFlipFromBottom：从底部翻转效果。

### UIView关键帧动画

从iOS7开始UIView动画中封装了关键帧动画，下面就来看一下如何使用UIView封装方法进行关键帧动画控制，这里实现前面关键帧动画部分对于落花的控制。

```objectivec
@interface ACViewKeyframeAnimationViewController () {
    UIImageView *petalView;
}

@end

@implementation ACViewKeyframeAnimationViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    self.view.layer.contents = (id)[UIImage imageNamed:@"background.jpg"].CGImage;
    
    petalView = [[UIImageView alloc] init];
    petalView.frame = CGRectMake(50, 150, 0, 0);
    petalView.image = [UIImage imageNamed:@"petal"];
    [petalView sizeToFit];
    [self.view addSubview:petalView];
    
    [self addKeyframeAnimation];
}

- (void)addKeyframeAnimation {
    [UIView animateKeyframesWithDuration:5.0 delay:0.0 options:UIViewKeyframeAnimationOptionCalculationModeLinear animations:^{
        [UIView addKeyframeWithRelativeStartTime:0.0 relativeDuration:0.5 animations:^{
            petalView.center = CGPointMake(80.0, 220.0);
        }];
        [UIView addKeyframeWithRelativeStartTime:0.5 relativeDuration:0.25 animations:^{
            petalView.center = CGPointMake(45.0, 300.0);
        }];
        [UIView addKeyframeWithRelativeStartTime:0.75 relativeDuration:0.25 animations:^{
            petalView.center = CGPointMake(55.0, 400.0);
        }];
    } completion:nil];
}

@end
```

#### UIView关键帧动画设置参数

对于关键帧动画也有一些动画参数设置options，UIViewKeyframeAnimationOptions类型，和上面基本动画参数设置有些差别，关键帧动画设置参数分为两类，可以组合使用：

1. 常规动画属性设置（可以同时选择多个进行设置）

	* UIViewAnimationOptionLayoutSubviews：动画过程中保证子视图跟随运动。
	* UIViewAnimationOptionAllowUserInteraction：动画过程中允许用户交互。
	* UIViewAnimationOptionBeginFromCurrentState：所有视图从当前状态开始运行。
	* UIViewAnimationOptionRepeat：重复运行动画。
	* UIViewAnimationOptionAutoreverse：动画运行到结束点后仍然以动画方式回到初始点。
	* UIViewAnimationOptionOverrideInheritedDuration：忽略嵌套动画时间设置。
	* UIViewAnimationOptionOverrideInheritedOptions：不继承父动画设置或动画类型。

2. 动画模式设置（同前面关键帧动画动画模式一一对应，可以从其中选择一个进行设置）

	* UIViewKeyframeAnimationOptionCalculationModeLinear：连续运算模式。
	* UIViewKeyframeAnimationOptionCalculationModeDiscrete：离散运算模式。
	* UIViewKeyframeAnimationOptionCalculationModePaced：均匀执行运算模式。
	* UIViewKeyframeAnimationOptionCalculationModeCubic：平滑运算模式。
	* UIViewKeyframeAnimationOptionCalculationModeCubicPaced：平滑均匀运算模式。

注意：前面说过关键帧动画有两种形式，上面演示的是属性值关键帧动画，路径关键帧动画目前UIView还不支持。

### UIView转场动画

从iOS4.0开始，UIView直接封装了转场动画。

```objectivec
@interface ACViewTransitionAnimationViewController () {
    UIImageView *imageView;
    NSArray *transitionTypes;
}

@end

static int transitionTypeIndex;

@implementation ACViewTransitionAnimationViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    transitionTypes = @[[NSNumber numberWithUnsignedInteger:UIViewAnimationOptionTransitionFlipFromRight], [NSNumber numberWithUnsignedInteger:UIViewAnimationOptionTransitionCurlUp], [NSNumber numberWithUnsignedInteger:UIViewAnimationOptionTransitionCrossDissolve], [NSNumber numberWithUnsignedInteger:UIViewAnimationOptionTransitionFlipFromBottom]];
    transitionTypeIndex = 0;
    
    imageView = [[UIImageView alloc] initWithFrame:[UIScreen mainScreen].bounds];
    [imageView setImage:[UIImage imageNamed:@"background.jpg"]];
    [self.view addSubview:imageView];
    
    UISwipeGestureRecognizer *swipeGesture = [[UISwipeGestureRecognizer alloc] initWithTarget:self action:@selector(swipe:)];
    swipeGesture.direction = UISwipeGestureRecognizerDirectionLeft;
    [self.view addGestureRecognizer:swipeGesture];
}

- (void)swipe:(UISwipeGestureRecognizer *)gestureRecognizer {
    if (transitionTypeIndex > 3) {
        transitionTypeIndex = 0;
    }
    
    UIViewAnimationOptions option;
    option = [transitionTypes[transitionTypeIndex] unsignedIntegerValue];
    option = option|UIViewAnimationOptionCurveLinear;
    [UIView transitionWithView:imageView duration:1.0 options:option animations:^{
        [imageView setImage:[UIImage imageNamed:@"background.jpg"]];
    } completion:nil];
    
    transitionTypeIndex++;
}

@end
```

如果有两个完全不同的视图，并且每个视图布局都很复杂，此时要在这两个视图之间进行转场可以使用以下方法进行两个视图间的转场，需要注意的是默认情况下转出的视图会从父视图移除，转入后重新添加，可以通过UIViewAnimationOptionShowHideTransitionViews参数设置，设置此参数后转出的视图会隐藏（不会移除）转入后再显示。

```objectivec
+(void)transitionFromView:(UIView *)fromView toView:(UIView *)toView duration:(NSTimeInterval)duration options:(UIViewAnimationOptions)options completion:(void (^)(BOOL finished))completion;
```

注意：转场动画设置参数完全同基本动画参数设置；同直接使用转场动画不同的是使用UIView的装饰方法进行转场动画其动画效果较少，因为这里无法直接使用私有API。


















