title: iOS面试总结
tags: []
categories: []
date: 2015-12-24 18:14:00
---
写在前面：iOS开发者水平良莠不齐，个人比较看中知识的全面性、编码规范性及学习能力，最近在面试iOS开发有些心得想和大家分享，整理如下：

### 一，基础篇

#### 1，关键字

`strong` 该属性值对应 __strong 关键字，即该属性所声明的变量将成为对象的持有者；_

`weak` 该属性对应 __weak 关键字，与 __weak 定义的变量一致，该属性所声明的变量将没有对象的所有权，并且当对象被破弃之后，对象将被自动赋值nil，delegate 和 Outlet 应该用 weak 属性来声明；

`copy` 与 strong 的区别是声明变量是拷贝对象的持有者；

`assign` 一般Scalar Varible用该属性声明，比如,int, BOOL；

`static` 类全局变量，只是在编译时候进行初始化，对于static变量，无论是定义在方法体里面 还是在方法体外面其作用域都一样；

`extern` extern的原理很简单，就是告诉编译器：“你现在编译的文件中，有一个变量虽然没有在本文件中定义，但是它是在别的文件中定义的全局变量，你要放行！”，比如 `extern NSString *aaa`;

`const` 

- const把对象转换为一个常量，不可修改，比如`const int bufSize = 512`；
- const 对象默认为当前文件的局部变量，在全局作用域声明的const变量是定义该对象的文件的局部变量，不能被其它文件访问，可以加extern解决；

`synchronised` 通过对一段代码的使用进行加锁。其他试图执行该段代码的线程都会被阻塞，直到加锁线程退出执行该段被保护的代码段，也就是说`@synchronized()`代码块中的最后一条语句已经被执行完毕的时候；

#### 2，应用程序的状态

![](http://7xkptx.com1.z0.glb.clouddn.com/1348823833_6296.png)

主要考察对app当前状态及生命周期的了解程度。具体如下：

- `application:willFinishLaunchingWithOptions:` - 这个方法是你在启动时的第一次机会来执行代码
- `application:didFinishLaunchingWithOptions:` - 这个方法允许你在显示app给用户之前执行最后的初始化操作
- `applicationDidBecomeActive:` - app已经切换到active状态后需要执行的操作
- `applicationWillResignActive:` - app将要从前台切换到后台时需要执行的操作
- `applicationDidEnterBackground:` - app已经进入后台后需要执行的操作
- `applicationWillEnterForeground:` - app将要从后台切换到前台需要执行的操作，但app还不是active状态
- `applicationWillTerminate:` - app将要结束时需要执行的操作

#### 3，**viewController的生命周期**

**单个**：
- initWithCoder:(NSCoder *)aDecoder：（如果使用storyboard或者xib）    
- loadView：加载view    
- viewDidLoad：view加载完毕    
- viewWillAppear：控制器的view将要显示    
- viewWillLayoutSubviews：控制器的view将要布局子控件    
- viewDidLayoutSubviews：控制器的view布局子控件完成      
  这期间系统可能会多次调用viewWillLayoutSubviews、viewDidLayoutSubviews 俩个方法  
- viewDidAppear:控制器的view完全显示    
- viewWillDisappear：控制器的view即将消失的时候    
  这期间系统也会调用viewWillLayoutSubviews 、viewDidLayoutSubviews 两个方法    
- viewDidDisappear：控制器的view完全消失的时候

**多个跳转**：
- 当我们点击push的时候首先会加载下一个界面然后才会调用界面的消失方法
- initWithCoder:(NSCoder *)aDecoder：`ViewController2` (如果用xib创建的情况下）
- loadView：`ViewController2`
- viewDidLoad：`ViewController2`
- viewWillDisappear：**ViewController1** 将要消失
- viewWillAppear：`ViewController2` 将要出现
- viewWillLayoutSubviews `ViewController2`
- viewDidLayoutSubviews `ViewController2`
- viewWillLayoutSubviews:**ViewController1**
- viewDidLayoutSubviews:**ViewController1**
- viewDidDisappear:**ViewController1** 完全消失
- viewDidAppear:`ViewController2` 完全出现

#### 4，OC设计模式

MVC、 delegate、 通知、 KVO、 KVC、 单例、 工厂模式等。

需要分别描述下各自的使用场景，比如通知适合一对多？iOS 代理为啥要用weak修饰? iOS系统单例有哪些？

#### 5，__block和__weak修饰符的区别

- __block不管是ARC还是MRC模式下都可以使用，可以修饰对象，还可以修饰基本数据类型；
- __weak只能在ARC模式下使用，也只能修饰对象，不能修饰基本数据类型；
- __block对象可以在block中被重新赋值，__weak不可以；

#### 6，Objective-C中类别和类扩展的区别

Class extension常常被误解为一个匿名的category。它们的语法的确很相似。虽然都可以用来为一个现有的类添加方法和属性，但它们的目的和行为却是不同的，category和extensions的不同在于后者可以添加属性；另外类扩展添加的方法是必须要实现的；可以运行时给category通过`objc_setAssociatedObject`、`objc_getAssociatedObject`添加和读取属性。

#### 7，copy的使用

用@property声明的NSString（或NSArray，NSDictionary）经常使用copy关键字，为什么？如果改用strong关键字，可能造成什么问题？

- 因为父类指针可以指向子类对象,使用copy的目的是为了让本对象的属性不受外界影响,使用copy无论给我传入是一个可变对象还是不可对象,我本身持有的就是一个不可变的副本.
- 如果我们使用是strong,那么这个属性就有可能指向一个可变对象,如果这个可变对象在外部被修改了,那么会影响该属性.

#### 8，LLVM 与 Clang

Clang 是一个 C++ 编写、基于 LLVM、发布于 LLVM BSD 许可证下的。C/C++/Objective C/Objective C++ 编译器，其目标（之一）就是超越 GCC。

Apple 使用 LLVM 在不支持全部 OpenGL 特性的 GPU (Intel 低端显卡) 上生成代码 (JIT)，令程序仍然能够正常运行。之后 LLVM 与 GCC 的集成过程引发了一些不快，GCC 系统庞大而笨重，而 Apple 大量使用的 Objective-C 在 GCC 中优先级很低。此外 GCC 作为一个纯粹的编译系统，与 IDE 配合很差。加之许可证方面的要求，Apple 无法使用修改版的 GCC 而闭源。于是 Apple 决定从零开始写 C family 的前端，也就是基于 LLVM 的 Clang 了。

Clang 的特性：

- 快，通过编译 OS X 上几乎包含了所有 C 头文件的 carbon.h 的测试，包括预处理 (Preprocess)，语法 (lex)，解析 (parse)，语义分析 (Semantic Analysis)，抽象语法树生成 (Abstract Syntax Tree) 的时间，Clang 是 Apple GCC 4.0 的 2.5x 快。(2007-7-25)   
- 内存占用小：Clang 内存占用是源码的 130%，Apple GCC 则超过 10x。      
- GCC 兼容性。   
- 设计清晰简单，容易理解，易于扩展增强。与代码基础古老的 GCC 相比，学习曲线平缓。   
- 基于库的模块化设计，易于 IDE 集成及其他用途的重用。

#### 9，BAD_ACCESS如何调试

BAD_ACCESS的出现是因为访问了野指针，比如对一个已经释放的对象执行了release、访问已经释放对象的成员变量或者发消息。

- 重写object的respondsToSelector方法，现实出现EXEC_BAD_ACCESS前访问的最后一个object;
- 设置 Scheme Zombie 模式；
- 设置全局断点快速定位问题代码所在行；
- Xcode 7 已经集成了BAD_ACCESS捕获功能：**Address Sanitizer**。 用法如下：在配置中勾选✅Enable Address Sanitizer；

### 二，实战篇

#### 10，以下代码运行结果如何?

``` objc
- (void)viewDidLoad
{
    [super viewDidLoad];
    NSLog(@"1");
    dispatch_sync(dispatch_get_main_queue(), ^{
        NSLog(@"2");
    });
    NSLog(@"3");
}
```

结果：死锁

原因：

- dispatch_sync在等待block语句执行完成，而block语句需要在主线程里执行，所以dispatch_sync如果在主线程调用就会造成死锁；
- dispatch_sync是同步的，本身就会阻塞当前线程，也即主线程。而又往主线程里塞进去一个block，所以就会发生死锁；
- MainThread等待dispatch_sync，dispatch_sync等待block，block等待 mainquen, mainquen等待MainThread，而MainThread等待dispatch_sync。这样就形成了一个死循环；

#### 11，HitTest方法

![](http://7xkptx.com1.z0.glb.clouddn.com/d43g45h47j.png)

场景：

View A位于上方，View B位于下方。View A上有Button 2 ，View B上有Button 1，如何穿透View A ，点击让Button 2响应？

`-(UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event`

- 我们都知道，一个屏幕事件由响应链一步步传下去。这个函数返回的view就是可以让你决定在这个point的事件，你用来接收事件的view。当然，如果这个point不在你的view的范围，返回nil；
- 如果hitTest返回的view不为空，则会把hitTest返回的view作为第一响应者
- 如果hitTest返回的view为空，调用次序是从subview top到bottom，包括view本身，知道找到响应者为止。

代码如下：

``` objc
View A:
-(id)hitTest:(CGPoint)point withEvent:(UIEvent *)event
{
    UIView *hitView = [super hitTest:point withEvent:event];
    if (hitView == self)
    {
        return nil;
    }
    else
    {
        return hitView;
    }
}
```



``` objc
View B:
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event
{
    // 当touch point是在self.buttonFirst上，则hitTest返回self.buttonFirst
    CGPoint btnPointInA = [self.buttonFirst convertPoint:point fromView:self];
    if ([self.buttonFirst pointInside:btnPointInA withEvent:event]) {
        return self.buttonFirst;
    }
    // 否则，返回默认处理
    return [super hitTest:point withEvent:event];
}
```

#### 12，GCD同步

如何用GCD同步若干个异步调用？（如根据若干个url异步加载多张图片，然后在都下载完成后合成一张整图）

``` objc
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_t group = dispatch_group_create();
dispatch_group_async(group, queue, ^{ /*加载图片1 */ });
dispatch_group_async(group, queue, ^{ /*加载图片2 */ });
dispatch_group_async(group, queue, ^{ /*加载图片3 */ }); 
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        // 合并图片
});
```

#### 13，NSOperation的特点

**依赖：**顾名思义，第一个方法用于添加依赖，第二个方法则用于移除依赖。需要特别注意的是，用`addDependency:`方法添加的依赖关系是单向的，比如`[A addDependency:B];`，表示 A 依赖 B，B 并不依赖 A ；

**暂停：**如果我们想要暂停和恢复执行 operation queue 中的 operation ，可以通过调用 operation queue 的 setSuspended: 方法来实现这个目的。不过需要注意的是，暂停执行 operation queue 并不能使正在执行的 operation 暂停执行，而只是简单地暂停调度新的 operation 。另外，我们**并不能单独地暂停执行一个 operation** ，除非直接 cancel 掉；

**优先级：** 我们只能够在执行一个 operation 或将其添加到 operation queue 前，通过 operation 的 `setThreadPriority:` 方法来修改它的线程优先级。当 operation 开始执行时，NSOperation 类中默认的 `start` 方法会使用我们指定的值来修改当前线程的优先级。另外，我们指定的这个线程优先级只会影响 `main` 方法执行时所在线程的优先级。所有其它的代码，包括 operation 的 completion block 所在的线程会一直以默认的线程优先级执行。因此，当我们自定义一个并发的 operation 类时，我们也需要在 `start` 方法中根据指定的值自行修改线程的优先级。

#### 14, UITextView代理方法的使用

``` objc
- (BOOL)textViewShouldBeginEditing:(UITextView *)textView;
- (BOOL)textViewShouldEndEditing:(UITextView *)textView;
- (void)textViewDidBeginEditing:(UITextView *)textView;
- (void)textViewDidEndEditing:(UITextView *)textView;

- (BOOL)textView:(UITextView *)textView shouldChangeTextInRange:(NSRange)range replacementText:(NSString *)text;
- (void)textViewDidChange:(UITextView *)textView;
- (void)textViewDidChangeSelection:(UITextView *)textView;
```

#### 15，如何把Model转换为字典

通过runtime的方式：

- 首先，可以通过`class_copyPropertyList` 和 `protocol_copyPropertyList`方法来获取类的属性；  
  
- 比如获取某个类（obj）的属性列表：  

``` objc  
unsigned int propsCount;
objc_property_t *props = class_copyPropertyList([obj class], &propsCount);
```
  
- 通过property_getName方法就可以得到某个类属性的名字了
  
``` objc
  unsigned int propsCount;
  objc_property_t *props = class_copyPropertyList([obj class], &propsCount);
  for(int i = 0;i < propsCount; i++){    
    objc_property_t prop = props[i];    
    NSString *propName = [NSString stringWithUTF8String:property_getName(prop)];
  }
```
  
- 得到类属性的名称后，就可以知道该属性对应的类型了，如果是Object-C class，直接判断数据类型即可，比如NSString、NSArray、NSDictionary等。如果该属性的值对应的是派生类，则需要回到上一步重新解析，直到遍历完为止

#### 16，常用的SVN/Git操作

- SVN 分支与tag  
  
  [SVN](https://www.baidu.com/s?wd=SVN&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1Y3PA7bP1mvrj99uH61rHT10ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6K1TL0qnfK1TL0z5HD0IgF_5y9YIZ0lQzqlpA-bmyt8mh7GuZR8mvqVQL7dugPYpyq8Q1n1Pjc1nWRvPs)官方推荐在一个版本库的根目录下先建立trunk、branches、tags这三个文件夹，其中trunk是开发主干，存放日常开发的内容；branches存放各分支的内容，比如为不同客户定制的不同版本；tags存放某个版本状态的标签，比如验收测试版、1.0.3版等。tags中的内容是存放不再修改的，tags通常只给管理员开放写权限。
  
- SVN 回滚  
  
  1). 改动没有被提交:直接svn revert something就行了；当something为目录时，需要加上参数-R(Recursive,递归)，否则只会将something这个目录的改动。   
  
  2). 改动已经被提交:可以使用svn diff -r HEAD:2500 [something]，此处的something可以是文件、目录或整个项目。如果需要回滚到版本号2500：

``` bash
svn merge -r HEAD:2500 something
```

#### 17，APNS 、IAP、itms-*services*协议等

- 询问关于推送、应用内付费以及企业帐号发布等知识；
- 对AppFlyer、Adhoc、iTunes connect等了解使用情况。

#### 18，算法题

1). 四个人夜间要过一座桥，每人走路速度不一样，过桥需要时间分别是1，2，5，10分钟。现在只有一只手电筒在过桥时必须带，同时只能两人过，如何安排能够让四人最快速度过桥？

2). 25匹马赛跑，每次只能跑5匹，最快能赛几次找出跑得最快的3匹马？

#### 19，编码规范

不规范：
``` objc
typedef enum {
    UserSex_Man,
    UserSex_Woman,
}UserSex;

@interface testA : NSObject

@property(nonatomic, strong) NSString *name;
@property (assign, nonatomic) int age;
@property (nonatomic, assign) UserSex sex;

-(id)initUserModelWithUserName: (NSString*)name withAge:(int)age;


- (void)doLogIn;

@end
```

修改后：
``` objc
typedef NS_ENUM(NSInteger, TSTUserSexType) {
    TSTUserSexTypeMan,
    TSTUserSexTypeWoman,
};

@interface testA : NSObject

@property (nonatomic, strong) NSString       *name;
@property (nonatomic ,assign) int            age;
@property (nonatomic, assign) TSTUserSexType sex;

- (instancetype)initUserModelWithUserName:(NSString*)name withUserAge:(int)age;

- (void)doLoginWithSuccess:(void(^)(id response))success
                   failure:(void(^)(NSError *error))failure;

@end
```

### 三、高级篇

#### 20，Autorelease对象什么时候释放？

对于每一个Runloop， 系统会隐式创建一个Autorelease pool，这样所有的release pool会构成一个象CallStack一样的一个栈式结构，在每一个Runloop结束时，当前栈顶的Autorelease pool会被销毁，这样这个pool里的每个Object会被release。那什么是一个Runloop呢？ 一个UI事件，Timer call， delegate call， 都会是一个新的Runloop。

Autorelease对象是在当前的runloop迭代结束时释放的，而它能够释放的原因是系统在每个runloop迭代中都加入了自动释放池Push和Pop。

#### 21，深入理解runloop

一般来讲，一个线程一次只能执行一个任务，执行完成后线程就会退出。如果我们需要一个机制，让线程能随时处理事件但并不退出，这种模型通常被称作 [Event Loop](http://en.wikipedia.org/wiki/Event_loop)。 Event Loop 在很多系统和框架里都有实现，比如 Node.js 的事件处理，比如 Windows 程序的消息循环，再比如 OSX/iOS 里的 RunLoop。实现这种模型的关键点在于：如何管理事件/消息，如何让线程在没有处理消息时休眠以避免资源占用、在有消息到来时立刻被唤醒。

RunLoop 实际上就是一个对象，这个对象管理了其需要处理的事件和消息，并提供了一个入口函数来执行上面 Event Loop 的逻辑。线程执行了这个函数后，就会一直处于这个函数内部 "接受消息->等待->处理" 的循环中，直到这个循环结束（比如传入 quit 的消息），函数返回。

OSX/iOS 系统中，提供了两个这样的对象：NSRunLoop 和 CFRunLoopRef。CFRunLoopRef 是在 CoreFoundation 框架内的，它提供了纯 C 函数的 API，所有这些 API 都是线程安全的。NSRunLoop 是基于 CFRunLoopRef 的封装，提供了面向对象的 API，但是这些 API 不是线程安全的。

问题一：NSURLConnection或NSStream指定RunLoop Mode的原因？
问题二：为何NSTimer在界面滚动时无响应？

参考回答一：
如果是在主线程，那么在滚动ScrollView或者TableView时，主线程的Run Loop会运行在UITrackingRunLoopMode模式，那么NSURLConnection或者NSStream的回调就无法运行，设置为NSRunLoopCommonModes，都可以保证NSURLConnection或者NSStream的回调可以被调用。
参考回答二：
当用户触摸界面时，主线程的run loop不再对timer事件进行处理。解决办法如下：
`[[NSRunLoop currentRunLoop] addTimer:timer forMode:UITrackingRunLoopMode];`

参考：http://blog.ibireme.com/2015/05/18/runloop/

#### 22，runtime的理解与使用

场景一：运行时给category添加属性，比如`objc_getAssociatedObject`、`objc_setAssociatedObject`；
场景二：动态获取类属性名称，比如`class_copyPropertyList`；
场景三：消息转发：
第一步：动态方法解析
对象在接收到未知的消息时，首先会调用所属类的类方法 +resolveInstanceMethod: 或者 +resolveClassMethod:，前者处理实例方法调用，后者处理类方法调用。我们可以它们里面用 class_addMethod() 加入异常处理的方法，不过前提是我们以及实现了处理方法。
第二步：备用接收者
如果在第一步还是无法处理消息，则 Runtime 会继续调以下方法：
``` bash
- (id)forwardingTargetForSelector:(SEL)aSelector
```
如果一个对象实现了这个方法，并返回一个非 nil 的结果，则这个对象会作为消息的新接收者，且消息会被分发到这个对象。当然这个对象不能是 self 自身，否则就会出现无限循环。当然，如果我们没有指定相应的对象来处理 aSelector，则应该调用父类的实现来返回结果。
第三步：完整转发
如果第二步：备用接收者还是未能处理好消息，那么接下来只有启用完整的消息转发机制了，这时候会调用以下方法：
``` bash
- (void)forwardInvocation:(NSInvocation *)anInvocation
```
运行时系统会在这一步给消息接收者最后一次机会将消息转发给其它对象。对象会创建一个表示消息的 NSInvocation 对象，把与尚未处理的消息有关的全部细节都封装在 anInvocation 中，包括：selector、目标(target)和参数。我们可以在 -forwardInvocation: 方法中选择将消息转发给其它对象。完整实例如下：
ViewController示例代码：

``` objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wundeclared-selector"
    
    [self performSelector:@selector(unknownMethod)];
    [self performSelector:@selector(unknownMethod2)];
    [self performSelector:@selector(unknownMethod3)];
    
}

+ (BOOL)resolveInstanceMethod:(SEL)sel {
    NSString *selectorString = NSStringFromSelector(sel);
    if ([selectorString isEqualToString:@"unknownMethod"]) {
        class_addMethod(self.class, @selector(unknownMethod), (IMP) dealWithExceptionForUnknownMethod, "v@:");
    }
    return [super resolveInstanceMethod:sel];
}

// Deal with unknownMethod.
void dealWithExceptionForUnknownMethod(id self, SEL _cmd) {
    NSLog(@"%@, %p", self, _cmd); // Print: <ViewController: 0x7ff96be33e60>, 0x1078259fc
}

// Deal with unknownMethod2.
- (id)forwardingTargetForSelector:(SEL)aSelector {
    NSString *selectorString = NSStringFromSelector(aSelector);
    if ([selectorString isEqualToString:@"unknownMethod2"]) {
        return [[RuntimeMethodHelper alloc] init];
    }
    return [super forwardingTargetForSelector:aSelector];
}

// Deal with unknownMethod3.
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    NSMethodSignature *signature = [super methodSignatureForSelector:aSelector];
    if (!signature) {
        if ([RuntimeMethodHelper instancesRespondToSelector:aSelector]) {
            signature = [RuntimeMethodHelper instanceMethodSignatureForSelector:aSelector];
        }
    }
    return signature;
}

- (void)forwardInvocation:(NSInvocation *)anInvocation {
    if ([RuntimeMethodHelper instancesRespondToSelector:anInvocation.selector]) {
        [anInvocation invokeWithTarget:[[RuntimeMethodHelper alloc] init]];
    }
}

```
RuntimeMethodHelper示例代码：
``` objc
/*RuntimeMethodHelper.h*/
@interface RuntimeMethodHelper : NSObject
- (void)unknownMethod2;
- (void)unknownMethod3;
@end

/*RuntimeMethodHelper.m*/
@implementation RuntimeMethodHelper
- (void)unknownMethod2 {
    NSLog(@"%@, %p", self, _cmd); // Print: <RuntimeMethodHelper: 0x7fb61042f410>, 0x10170d99a
}

- (void)unknownMethod3 {
    NSLog(@"%@, %p", self, _cmd); // Print: <RuntimeMethodHelper: 0x7f814b498ee0>, 0x102d79929
}
@end
```
场景四：动态创建类和对象，例如：
``` objc
// 创建类实例
id class_createInstance ( Class cls, size_t extraBytes );
// 在指定位置创建类实例
id objc_constructInstance ( Class cls, void *bytes );
// 销毁类实例
void * objc_destructInstance ( id obj );
```
场景五：IOS中如何Hook消息？
`class_replaceMethod`使用该函数可以在运行时动态替换某个类的函数实现，截获系统类的某个实例函数。
``` objc
IMP orginIMP;
NSString * MyUppercaseString(id SELF, SEL _cmd)
{
    NSLog(@"begin uppercaseString");
    NSString *str = orginIMP (SELF, _cmd);（3）
    NSLog(@"end uppercaseString");
    return str;
}
-（void）testReplaceMethod
{
      Class strcls = [NSString class];
      SEL  oriUppercaseString = @selector(uppercaseString);
      orginIMP = [NSStringinstanceMethodForSelector:oriUppercaseString];  （1）  
      IMP imp2 = class_replaceMethod(strcls,oriUppercaseString,(IMP)MyUppercaseString,NULL);（2）
      NSString *s = "hello world";
      NSLog(@"%@",[s uppercaseString]];
}
/*
执行结果为:
begin uppercaseString
end uppercaseString
HELLO WORLD
这段代码的作用就是
（1）得到uppercaseString这个函数的函数指针存到变量orginIMP中
（2）将NSString类中的uppercaseString函数的实现替换为自己定义的MyUppercaseString
（3）这样每次对NSString调用uppercaseString的时候，都会打印出log来
*/
```
场景六：Method Swizzling
例如，我们想跟踪在程序中每一个view controller展示给用户的次数，在每个view controller的viewDidAppear中添加跟踪代码，但是这太过麻烦。这种情况下，我们就可以使用Method Swizzling。示例代码如下：
``` objc
#import <objc/runtime.h> 
 
@implementation UIViewController (Tracking) 
 
+ (void)load { 
    static dispatch_once_t onceToken; 
    dispatch_once(&onceToken, ^{ 
        Class aClass = [self class]; 
 
        SEL originalSelector = @selector(viewWillAppear:); 
        SEL swizzledSelector = @selector(xxx_viewWillAppear:); 
 
        Method originalMethod = class_getInstanceMethod(aClass, originalSelector); 
        Method swizzledMethod = class_getInstanceMethod(aClass, swizzledSelector); 
        
        // When swizzling a class method, use the following:
        // Class aClass = object_getClass((id)self);
        // ...
        // Method originalMethod = class_getClassMethod(aClass, originalSelector);
        // Method swizzledMethod = class_getClassMethod(aClass, swizzledSelector);
 
        BOOL didAddMethod = 
            class_addMethod(aClass, 
                originalSelector, 
                method_getImplementation(swizzledMethod), 
                method_getTypeEncoding(swizzledMethod)); 
 
        if (didAddMethod) { 
            class_replaceMethod(aClass, 
                swizzledSelector, 
                method_getImplementation(originalMethod), 
                method_getTypeEncoding(originalMethod)); 
        } else { 
            method_exchangeImplementations(originalMethod, swizzledMethod); 
        } 
    }); 
} 
 
#pragma mark - Method Swizzling 
 
- (void)xxx_viewWillAppear:(BOOL)animated { 
    [self xxx_viewWillAppear:animated]; 
    NSLog(@"viewWillAppear: %@", self); 
} 
 
@end
```
#### 23，lib库的编写与使用

- 如何保证lib库中category文件的正常读取？  
- 如何保证lib对armv7s的支持？  
  Build Setting -> Architectures, 添加 $(ARCHS_STANDARD)和   armv7s
- 如何合并不同平台的lib库？
- iOS 第三方库冲突的如何处理？可以对lib库内的文件修改重修打包吗 ？  
实例，比如libHChatSDK.a中包含了JSONKit库，现有工程中同样包含了JSONKit库，这样在当前工程中加入libHChatSDK.a时会引起duplicate symbols for architecture armv7的编译错误，那么我们可以重新编辑libHChatSDK.a，删除JSONKit.o然后再打包合并即可，具体命令如下：
```bash
# 1,新建armv7s目录
mkdir ~/desktop/armv7s
# 2,分离出armv7s平台
lipo  libHChatSDK.a  -thin  armv7s  -output  libHChatSDK-armv7s.a
# 3,解压 libHChatSDK.a
cd armv7s && ar xv libHChatSDK-armv7s.a
# 4,查看库中所包含的文件列表
ar -t armv7/libHChatSDK-armv7s.a
# 5,找到JSONKit.o并删除
rm JSONKit.o
# 6,重新打包libHChatSDK-armv7s.a
ar rcs libHChatSDK-armv7s.a armv7s/*.o
# 7,重复第3步，确认JSONKit已经被删除
# 8,按照1-7分别生成armv7平台、arm64平台及模拟器使用的x86_64、i386平台
# 9,最后用lipo -c 命令合并各个平台即可
```
常用的lib命令：
```bash
# 用 lipo info 查看静态库支持的平台
$ lipo libname.a -info
# 用 lipo remove 参数来删除平台
lipo libname.a -remove x86_64 -output libname1.a
# 用 lipo create 将两个不同平台的库合并到一起
$ lipo -create libname1.a libname2.a -output libname3.a
# 用 lipo thin 参数来分离平台
lipo libname.a -thin armv7 -output armv7/libname-armv7.a
```
#### 24，ARM64与ARMv7

Arm处理器，因为其低功耗和小尺寸而闻名，几乎所有的手机处理器都基于arm，其在嵌入式系统中的应用非常广泛，它的性能在同等功耗产品中也很出色。

Armv6、armv7、armv7s、arm64都是arm处理器的指令集，所有指令集原则上都是向下兼容的，如iPhone4S的CPU默认指令集为armv7指令集，但它同时也兼容armv6指令集，只是使用armv6指令集时无法充分发挥其性能，即无法使用armv7指令集中的新特性，同理，iPhone5的处理器标配armv7s指令集，同时也支持armv7指令集，只是无法进行相关的性能优化，从而导致程序的执行效率没那么高。 

需要注意的是iOS模拟器没有运行arm指令集，编译运行的是x86指令集，所以，只有在iOS设备上，才会执行设备对应的arm指令集。

iOS设备与ARM平台分布如下图：

![](http://7xkptx.com1.z0.glb.clouddn.com/armv7s.png)



#### 参考：

http://www.jianshu.com/p/2e7ae4457083

http://www.iswifting.com/2015/07/26/71/

http://blog.leichunfeng.com/blog/2015/05/31/objective-c-autorelease-pool-implementation-principle/