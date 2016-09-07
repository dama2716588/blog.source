title: 逆向Appstore应用（一）
tags:
  - ipa
categories:
  - App Store
date: 2016-07-20 19:39:00
---
###写在前面

最近在做一些iOS启动速度优化和App瘦身相关的工作，有时候需要横向对比一下竞品的数据，免不了要对一些商店的App在许可范围内做一些手脚，下面以[开源中国](http://git.oschina.net/oschina/iphone-app)的iPhone客户端为例给大家简单介绍下操作步骤。

当前环境：

- 一台越狱iPhone，系统为iOS8.3
- mac OS X EI 10.11.5
- iTunes 12.4

### 通过iTunes获取ipa

![](http://7xkptx.com1.z0.glb.clouddn.com/t34hy45u76.png)

点击下载，然后到我的Apps中找到刚刚下载的开源中国.ipa文件，将后缀.ipa修改为.zip，直接解压即可。找到Payload文件夹下的iosapp包文件，查看包内容，就是开发者上传Appstore的全部内容了。里面包含有png、xib等资源文件，以及本地化语言包，签名包以及二进制文件。

###使用工具解密二进制文件

![配图来自ifanr](http://7xkptx.com1.z0.glb.clouddn.com/old-dbd-magenta.jpg)

App Store上的应用都使用了DRM(Digital Rights Management)数字版权加密保护技术，想更多了解DMG历史请戳[这里](http://www.ifanr.com/473806)。首先得破解加密的可执行文件，可以通过编译[dumpdecrypted](https://github.com/stefanesser/dumpdecrypted)来解密，原理是让app预先加载一个解密的dumpdecrypted.dylib，然后在程序运行后，将代码动态解密，最后在内存中dump出来整个程序。（如果没有越狱设备，可以通过第三方市场下载未加密的ipa文件，跳过此步骤）。

![](http://7xkptx.com1.z0.glb.clouddn.com/05ce2ce5f74e040661b53791b.png)

找到刚才解压好的开源中国二进制，名为iosapp，运行：

``` objc
otool -l iosapp_本地路径 | grep crypt
# 结果为：
     cryptoff 16384
    cryptsize 5619712
      cryptid 1
     cryptoff 16384
    cryptsize 6144000
      cryptid 1
```

cryptid 1代表加密，cryptid 0代表未加密。两个分别对应着armv7和arm64，也就是它们都有加密。接下来要开始编译dumpdecrypted.dylib文件了，首先clone [dumpdecrypted](https://github.com/stefanesser/dumpdecrypted.git)到桌面，cd到dump decrypted目录，执行以下命令，即可生成dumpdecrypted.dylib文件了。

``` objc
 make
`xcrun --sdk iphoneos --find gcc` -Os  -Wimplicit -isysroot `xcrun --sdk iphoneos --show-sdk-path` -F`xcrun --sdk iphoneos --show-sdk-path`/System/Library/Frameworks -F`xcrun --sdk iphoneos --show-sdk-path`/System/Library/PrivateFrameworks -arch armv7 -arch armv7s -arch arm64 -c -o dumpdecrypted.o dumpdecrypted.c
```

编译后的目录如下：

![](http://7xkptx.com1.z0.glb.clouddn.com/e3fef6ddadbd919bbee9602a5.png)

如何使用dumpdecrypted.dylib文件来破解spa包中的二进制文件呢？

首先需要在已经越狱的iPhone中下载几个插件：

- OpenSSH，用来建立与iPhone的ssh链接，传递文件用；
- adv-cmds，在iPhone上使用*ps命令*查看进程，方便找到当前运行的App在iPhone中的文件目录；
- iFile，在iPhone上建立一个web server，方便与mac传递数据；

通过Cydia安装好插件后，就要开始解密iosapp这个文件了。在iPhone设置中查找当前链接网络的ip地址，比如是：192.168.0.100，打开mac命令行，建立ssh链接：

``` shell
ssh root@192.168.0.100
```

此时提示输入密码，默认为：alpine，无需修改。连接成功后，cd 到 /var/mobile/Containers/Bundle/Application/ 目录下，可能看到所有安装的App文件夹，在iPhone中打开开源中国App，保持前台运行。然后在mac终端执行以下命令：

``` shell
ps -e | grep var
```

结果如下：

``` objc
  143 ??         0:04.00 /usr/libexec/pkd -d/var/db/PlugInKit-Annotations
  398 ??         0:01.08 /private/var/db/stash/_.T1qNfY/Applications/MobileSafari.app/webbookmarksd
  529 ??         0:04.78 /private/var/db/stash/_.T1qNfY/Applications/Stocks.app/PlugIns/StocksWidget.appex/StocksWidget
  530 ??         0:01.77 /private/var/db/stash/_.T1qNfY/Applications/MobileCal.app/PlugIns/CalendarWidget.appex/CalendarWidget
 3490 ??         0:14.24 /var/mobile/Containers/Bundle/Application/7738D5DE-B38D-4BF1-9551-BF671978045F/CocoaPods.app/CocoaPods
 3622 ??         0:08.91 /var/mobile/Containers/Bundle/Application/B590E6C8-D5FA-4697-AF4B-EF717ADBCB90/BaiduBoxApp.app/BaiduBoxApp
 4328 ??         0:06.65 /var/mobile/Containers/Bundle/Application/86E38E63-5FF3-4EF5-AB07-D13C4CB70168/iosapp.app/iosapp
 4350 ??         0:01.47 /var/mobile/Containers/Bundle/Application/FBA32574-0453-4BB6-9F46-A87757F22325/WeChat.app/WeChat
 4394 ttys000    0:00.01 grep var
```

由此可见开源中国的App运行在/var/mobile/Containers/Bundle/Application/86E38E63-5FF3-4EF5-AB07-D13C4CB70168/iosapp.app/目录下，每个app目录下都有一个Info.plist。我们可以通过这个Info.plist得到app的Bundle ID：
``` objc
cat /var/mobile/Containers/Bundle/Application/86E38E63-5FF3-4EF5-AB07-D13C4CB70168/iosapp.app/Info.plist
```
得到开源中的的Bundle ID为net.oschina.iosapp，由于App的运行会受到沙盒的限制，因此dump出来的app只能保存在data目录下，这里我们可以通过Bundle ID和一个private API得到data目录的位置：
``` objc
    Class cls = NSClassFromString(@"LSApplicationWorkspace");
    id s = [(id)cls performSelector:NSSelectorFromString(@"defaultWorkspace")];
    NSArray *arr = [s performSelector:NSSelectorFromString(@"allInstalledApplications")];
    Class Proxy  = NSClassFromString(@"LSApplicationProxy");
    
    for (id proxy in arr) {
        id bundleID = [proxy valueForKey:@"applicationIdentifier"];
        if (bundleID && [bundleID isEqualToString:@"net.oschina.iosapp"]) {
            id dataUrl = [proxy valueForKey:@"dataContainerURL"];
            NSLog(@"dataUrl : %@",dataUrl);
        }
    }
```
打印出来的data url为`file:///private/var/mobile/Containers/Data/Application/6C425F0E-217C-4051-A131-735F3A816D89`，接下来在mac终端，cd到dumpdecrypted.dylib所在目录，将dumpdecrypted.dylib拷贝到开源中国的Documents目录下:

``` objective-c
scp dumpdecrypted.dylib root@172.24.64.228:/private/var/mobile/Containers/Data/Application/6C425F0E-217C-4051-A131-735F3A816D89/Documents/dumpdecrypted.dylib
```

传输成功显示：

``` sh
dumpdecrypted.dylib   100%  193KB 192.9KB/s   00:00
```
然后进入iPhone中开源中国所在根目录的Documents下，执行`DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib`相关的命令，即将开始编译：

``` objc
DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib /var/mobile/Containers/Bundle/Application/7202242C-DABA-4520-8D3D-81448728F587/iosapp.app/iosapp
```
成功后，会在Documents目录生成名为iosapp.decrypted的文件，至此就算是破解成功了，如下图所示：
![](http://7xkptx.com1.z0.glb.clouddn.com/f3fh389h4f.png)

如果显示`dumpdecrypted.dylib: stat() failed with errno=1` 即“Operation not permitted”，表示操作权限不够，可以将dumpdecrypted.dylib文件sip至手机的usr/lib目录下，然后再执行`DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib`命令编译，此时会在usr/lib目录下生成iosapp.decrypted文件。

![](http://7xkptx.com1.z0.glb.clouddn.com/9518200bbe399f475dea89e86.png)
砸壳后的app只能解密砸壳运行时的架构，可以看到在arm64架构下显示cryptid为0，已经成功解密。

###解密后的文件如何使用？

在得到iosapp.decrypted后，真正的逆向App工作才刚刚开始。未完待续...




####参考：

http://www.ifanr.com/473806

http://www.liuchendi.com/2015/12/23/iOS/24_dumpdecrypted/