title: 一款Watch OS版微博
tags:
  - Watch OS
categories: []
---

###写在前面

手表买了将近快一年的时间了，每天常用的就是运动提醒了，尤其是间隔一小时站立及步数统计，对久坐人士有很好的督促作用。一直想着做一款手表App，最近终于有些时间就开始捣鼓了，大概前后花了两个周的空闲时间，基于Watch OS 1简单实现了一款手表版的微博,如有纰漏，还望指正。

###App Groups

首先了解下App Groups的概念，这是Apple在iOS8中引入的技术。旨在位于同一group的app之间共享一份数据读写空间，打破了沙盒技术对应用间通信与交互的限制。比如通知中心的Today Extension作为app功能的补充而存在，Watch App也类似于此。在开始之前需要先创建App Group。可以在Xcode的Capabilities中添加，或者在Apple developer 中心来添加，本文采用后一种方式给大家介绍。

![](http://7xkptx.com1.z0.glb.clouddn.com/34f4wv4wrb.png)

新生成的group id为"group.baidu.com.miniweibo"，用来关联Watch App与iPhone App，实现数据共享及进程通信。创建名为MiniWeibo的Project，对应生成MiniWeibo、MiniWeibo WatchKit 1 Extension和MiniWeibo WatchKit 1 App 三个target，然后在 Xcode 的 Capabilities 选项卡下分别打开主 target 与 WatchKit Extension target下的App Groups选项，刷新选中刚创建的group id即可。如下图所示：

![](http://7xkptx.com1.z0.glb.clouddn.com/43fwrgberh5.png)

选中后会自动添加名为MiniWeibo.entitlements与MiniWeibo WatchKit 1 Extension.entitlements的两个XML文件至工程文件中，当构建整个应用时，这两个entitlements文件也会提交给 `codesign` 作为应用所需要拥有哪些授权的参考，可以在 Xcode build setting 中的 code signing entitlements 中设置。entitlements文件内部格式如下：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.security.application-groups</key>
    <array>
    	<string>group.com.baidu.miniweibo</string>
    </array>
</dict>
</plist>
```

`    <key>com.apple.security.application-groups</key>`标识着此应用被添加至名为group.com.baidu.miniweibo的group中，为了与扩展 (extensions) 共享数据。另外还可以添加get-task-allow用来限定只能在用于开发的证书签名下运行，以及设置asp-environment的推送环境为development或者distribution。授权信息会被包含在应用的签名信息中，可以执行`$ codesign -d --entitlements - Example.app`，返回一个XML文件，查看签名信息中具体包含了什么授权信息，方便工程配置的查看与调试。

###数据共享与调试

App Groups共享数据可以创建group id为标记的NSUserDefaults，来写入和读取可plist化的数据对象：

``` objc
- (void)saveTextByNSUserDefaults
{
    NSUserDefaults *shared = [[NSUserDefaults alloc] initWithSuiteName:@"group.wangzz"];
    [shared setObject:_textField.text forKey:@"wangzz"];
    [shared synchronize];
}
```

或者通过NSFileManager的containerURLForSecurityApplicationGroupIdentifier方法，创建NSFileManager的单例，来管理App Group数据。File可以是普通的二进制文件，或者是sqlite文件。MiniWeibo采用的是基于sqlite的数据存储方式，来实现App间的通信：

``` objc
NSString *dataBasePath = [[[[NSFileManager defaultManager] containerURLForSecurityApplicationGroupIdentifier:@"group.com.baidu.miniweibo"] URLByAppendingPathComponent:@"miniweibo.db"] path];
_queue = [FMDatabaseQueue databaseQueueWithPath:dataBasePath];
```

sqlite内部的数据处理是采用第三方开源库[FMDB](https://github.com/ccgus/fmdb)来实现，详细了解请参考：https://github.com/ccgus/fmdb

大致实现流程可以描述为：每个可交互的 `WKInterfaceObject` 子类都对应了一个 action，来调用Watch Kit中openParentApplication方法，唤起iOS target中`application:handleWatchKitExtensionRequestreply:`代理方法，请求网络数据，需要持久化的通过sqlite存储共享，临时数据通过reply回调，传递给Watch Extension来刷新Watch App界面。

开发的过程中，需要在iPhone App 与 Watch App之间频繁进程切换，通过Xcode -> Debug -> Attach to Process -> [process name]，绑定iOS App 与 Watch Extension进行调试。

###工程架构

![](http://7xkptx.com1.z0.glb.clouddn.com/2fwehetjty.png)

Watch App是基于iOS App而存在的，WatchKit Extension主要负责逻辑部分，将随iOS App的主target被一同安装到iPhone中，通过UIApplicationDelegate及WKInterfaceController来调度手表上负责界面显示的WatchKit App。

![图片来自 http://tech.glowing.com/cn/make_apple_watch_app/](http://7xkptx.com1.z0.glb.clouddn.com/3fwregw5.png?imageView2/2/h/360)

####1，主要类

如同WKInterfaceController一样，WKInterfaceButton、WKInterfaceLabel、WKInterfaceImage等并非是UIView的子类，而是继承自NSObject，Watch App 中实际展现和渲染在屏幕上的 view 对于代码来说是非直接可见的，我们只能在 Extension target 中通过对应的代理对象对属性进行设置，由 WatchKit 传递给手表中的 Watch App 并进行界面刷新。

####2，Glance 和 Notification

GlanceController意为速览，算是Watch App的快捷显示及操作界面，如音乐App的播放控制，天气App的数据展示等。有些Glance界面是固定的，无需每次显示的时候刷新，只需在initWithContext中初始化即可；有些是需要动态展示，每次滑出Glance的时候都需要刷新界面，则需要调用呈现时的 `-willActivate` 方法，类似UIViewController中的viewWillAppear，相对的，`-didDeactivate`方法可以理解为viewDidDisappear，即界面即将消失。

当我们加入了一个新的通知界面之后，Xcode就会为我们生成这两个interface，分为Static Notification Interface Controller 和 Dynamic Notification Interface Controller两种。动态的Interface是可选的。当一个的通知到来之后，系统会优先去呈现动态的通知界面，当动态界面不可用或者低电量时才会去呈现静态的。动态通知使得我们可以提供一个更加丰富的体验给用户，如同其他Interface Controller一样，可以通过Outlet及IBAction来关联和定义，增强可操作性。通知流程如下图：

![](http://7xkptx.com1.z0.glb.clouddn.com/3df3wg45g.png)

``` objc
- (void)didReceiveRemoteNotification:(NSDictionary *)remoteNotification withCompletion:(void (^)(WKUserNotificationInterfaceType))completionHandler {
    // This method is called when a remote notification needs to be presented.
    // Implement it if you use a dynamic notification interface.
    // Populate your dynamic notification interface as quickly as possible.
    //
    // After populating your dynamic notification interface call the completion block.
    completionHandler(WKUserNotificationInterfaceTypeCustom);
}
```

​    completionHandler(WKUserNotificationInterfaceTypeCustom)中参数WKUserNotificationInterfaceTypeCustom决定了优先呈现Dynamic界面，如果是WKUserNotificationInterfaceTypeDefault则会显示Static通知界面。在调用completionHandler之前系统会短暂地显示Short-Look Interface，它不能滚动和定制，系统会显示App 的名字以及App 的图标Icon，以及一条自定义通知信息，同时在准备数据，构建Long-Look界面，然后系统会迅速的切换到The Long-Look Interface中，包含三个区域：顶部Sash、中间Content area及底部按钮区域，点击Sash、Content area都会直接进入Watch App，点击底部按钮区域可以触发Watch Kit与Watch Extension及iOS App交互。

准备自定义的NotificationInterface后，就可以开始测试了，模拟器可以直接选择Notification target编译即可。真机测试，可以使用Mac 端 apns小工具[SmartPush](https://github.com/shaojiankui/SmartPush)，进行开发和生产环境的调试。至此MiniWeibo工程的开发暂告一段落，附张截图吧：

![](http://7xkptx.com1.z0.glb.clouddn.com/1a0cef138909a32f8519deb24.png)

###开发中的几个Tips：

- 图片缓存

  使用`addCachedImageWithData:name:`来添加NSData缓存图片数据，相比较`addCachedImage:name:`方法避免了使用PND编码的过程，节省存储空间。每个Watch App大概有5mb的缓存分配，如果存储控件不足时，需要调用 `removeCachedImageWithName:`或者 `removeAllCachedImages`方法来清除数据。否则WatchKit将从最老的数据开始自动删除，为新数据腾出空间。


- WKInterfaceLabel高度自适应与滚动

  不同与UIKit，很多视图控件无法通过手写的方式来管理，需要在Interface.storyboard中设置参数。比如WKInterfaceLabel需要多行显示和滚动时，需要设置Label->Lines为0，Size->Width为Relative to Container，以及Size->Height为Size To Fit Content，意为宽度与父视图一样，高度为内容自适应，即可实现多行显示与滚动了。
- WKInterfaceGroup的使用

  WKInterfaceGroup继承自NSObject，可以理解为视图块，集中管理某个区域的视图显示，比如table cell。可以指定background colour、alpha及Radius等UIView的属性设置，在InterfaceController中使用频率很高。

###关于Watch OS 2

Watch OS 2引入了ClockKit与WatchConnectivity框架，不仅增强了表盘自定义控件的灵活性，数据共享操作改进很大，其中extension 是直接存在于手表中的，在Watch OS 1中通过app group管理的方式已经失效。另外HealthKit也得到增强，健康监护及运动统计类的App会大有施展空间。

####参考

1，https://onevcat.com/2014/08/notification-today-widget/ （[OneV's Den](https://onevcat.com/#blog)）

2，http://foggry.com/blog/2014/06/23/wwdc2014zhi-app-extensionsxue-xi-bi-ji/ （[王中周的技术博客](http://foggry.com/)）

3，http://objccn.io/issue-17-2/ （[objc中国](http://objccn.io/)）

4，[Apple Watch 文档](https://developer.apple.com/library/ios/documentation/General/Conceptual/WatchKitProgrammingGuide/BasicSupport.html)

















#### 