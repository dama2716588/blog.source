title: 聊一聊iOS后台任务
tags:
  - background
categories: []
date: 2016-01-13 20:43:00
---
iOS开发的过程中常常会有按下home键盘，进入后台，但不希望当前任务立即停止的情况，比如保存数据，断开链接，继续下载文件等，接下来就简单聊下iOS的后台任务。

#### 一，后台任务的分类

**程序的5个状态和对应的AppDelegate的7个方法** ：

- Not Running, 未运行
- Inactive,  非活动
- Active, 活动
- Background, 后台
- Suspend, 挂起

对应的方法分别是：

``` objc
// 进程启动但还没完成初始化，这个方法是iOS6之后才有的

- (BOOL)application:(UIApplication *)application willFinishLaunchingWithOptions:(NSDictionary *)launchOptions

// 进程启动基本完成      

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions

// 应用程序将要入非活动状态执行，在此期间，应用程序不接收消息或事件

- (void)applicationWillResignActive:(UIApplication *)application   

// 应用程序入活动状态，这个刚好跟上面那个方法相反

- (void)applicationDidBecomeActive:(UIApplication *)application    

// 程序被推送到后台，如果要设置后台继续运行，则在这个函数里面设置即可

- (void)applicationDidEnterBackground:(UIApplication *)application     

// 程序从后台将要回到前台

- (void)applicationWillEnterForeground:(UIApplication *)application   

// 程序将要退出

- (void)applicationWillTerminate:(UIApplication *)application   
```

在介绍iOS应用状态5种最基本的状态时，我们发现前台运行有两种状态，分别是Inactive和Active状态。大多数情况下，Inactive状态只是其它状态之间切换时短暂的停留状态，如前后台应用切换时，Inactive状态会在Active和Background之间短暂出现，比如App Switcher/回到原应用的操作等。

用户在按下home键后，app可做的事情有很多，比如听歌、打电话、下载电影、更新数据、定时任务等，可以大致分为两类，注册任务（耗时长）和非注册任务（耗时较短）。

#### 二，非注册任务的运行

非注册任务一般耗时较短，多用来保存数据或延迟执行某一命令等，通过系统API即可实现。可以先看一段官方实例代码：

``` objc
- (void)applicationDidEnterBackground:(UIApplication *)application
{
    bgTask = [application beginBackgroundTaskWithName:@"MyTask" expirationHandler:^{
        // Clean up any unfinished task business by marking where you
        // stopped or ending the task outright.
        [application endBackgroundTask:bgTask];
        bgTask = UIBackgroundTaskInvalid;
    }];

    // Start the long-running task and return immediately.
}
```

`beginBackgroundTaskWithName:expirationHandler:`方法标识了一个后台任务的开始，并用过超时处理的回调来结束此任务。那么，超时时间具体是多少？可以通过UIApplication的只读属性backgroundTimeRemaining来获取当前后台任务执行的剩余时间，它不是具体的数字（我执行的时间大概160秒），而是iOS根据当前系统环境综合考量后估算出来的。然后执行expirationHandler回调完成一个后台任务的执行周期。

#### 三，注册任务的流程

有些耗时较长的工作，则需要申请专门的权限来保证正常执行而不被挂起，只有少数几种类型被允许这个做。

- Apps that play audible content to the user while in the background, such as a music player app
- Apps that record audio content while in the background
- Apps that keep users informed of their location at all times, such as a navigation app
- Apps that support Voice over Internet Protocol (VoIP)
- Apps that need to download and process new content regularly
- Apps that receive regular updates from external accessories

申请使用以上场景的后台权限需要在Xcode->Capabilities->Background Mode中配置，如下图所示：

![](http://7xkptx.com1.z0.glb.clouddn.com/234f23g43g343.png)

勾选所需的模式后，会自动在app的info.plist文件中添加Required background modes一项，包含了所勾选的后台运行模式。如下所示：

``` objc
<key>UIBackgroundModes</key>
    <array>
		<string>fetch</string>
		<string>voip</string>
	</array>
```

以Background Fetch为例，具体说下任务流程。

首先需要在Xcode Capabilities 中开启Background fetch选项，在`didFinishLaunchingWithOptions`中设置下获取的时间间隔：

``` objc
[[UIApplication sharedApplication] setMinimumBackgroundFetchInterval:UIApplicationBackgroundFetchIntervalMinimum];
```

如果不对最小后台获取间隔进行设定的话，系统将使用默值`UIApplicationBackgroundFetchIntervalNever`，也就是永远不进行后台获取。而最小的时间间隔则有系统根据电量、网络状态、用户使用习惯等综合考量后来设定一定的差值，执行fetch任务，具体代码如下：

``` objc
//File: YourAppDelegate.m
-(void)application:(UIApplication *)application performFetchWithCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler
{
    UINavigationController *navigationController = (UINavigationController*)self.window.rootViewController;

    id fetchViewController = navigationController.topViewController;
    if ([fetchViewController respondsToSelector:@selector(fetchDataResult:)]) {
        [fetchViewController fetchDataResult:^(NSError *error, NSArray *results){
            if (!error) {
                if (results.count != 0) {
                    //Update UI with results.
                    //Tell system all done.
                    completionHandler(UIBackgroundFetchResultNewData);
                } else {
                    completionHandler(UIBackgroundFetchResultNoData);
                }
            } else {
                completionHandler(UIBackgroundFetchResultFailed);
            }
        }];
    } else {
        completionHandler(UIBackgroundFetchResultFailed);
    }
}
```

与fetch类型的是，后台任务同样可以满足远程通知唤醒app执行某一进程，结束后以本地通知的方式提醒用户，以静默、智能的方式带给用户使用上的绝佳体验，相似的场景还有后台电影、音乐的下载等，在此不多做介绍。

#### 四，后台任务的注意事项

##### 1，关于OpenGL ES

有些基于位置的app需要后台定时更新用户的当前位置，导致未知崩溃。原因就是Location update类型的后台任务在更新位置时，需要重新绘制MKMapView，调用了OpenGL ES，而OpenGL ES必须在程序Inactive以前关闭，不然会crash。如[官方文档](https://developer.apple.com/library/ios/documentation/3DDrawing/Conceptual/OpenGLES_ProgrammingGuide/ImplementingaMultitasking-awareOpenGLESApplication/ImplementingaMultitasking-awareOpenGLESApplication.html)描述：To summarize, your app needs to call the `glFinish` function to ensure that all previously submitted commands are drained from the command buffer and are executed by OpenGL ES. After it moves into the background, you must avoid all use of OpenGL ES until it moves back into the foreground.

可以在更新位置之前做一下app状态的检测：

``` objc
if( (appState != UIApplicationStateBackground) && (appState != UIApplicationStateInactive)) {
     // update location
}
```

##### 2，关于CAAnimation动画

![](http://7xkptx.com1.z0.glb.clouddn.com/ca_architecture_2x.png)
OpenGL提供了Core Animation的基础，它是底层的C接口，直接和iPhone，iPad的硬件通信，极少地抽象出来的方法。当app进入background模式时，所有基于Core Animation 的动画将自动停止，这是因为动画的渲染需要调用Open GL或者Core Graphics来实现UIKit层的变动，然后在applicationWillEnterForeground的时候重新启动即可。

参考：

http://onevcat.com/2013/08/ios7-background-multitask/

https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html