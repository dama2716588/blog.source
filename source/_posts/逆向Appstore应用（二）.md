title: 逆向Appstore应用（二）

---
###准备工作

代码签名 (code signing) 对一个App来讲至关重要，是iOS系统安全的重要组成部分，决定了App的哪些功能是被授权或者禁止的。尽管各种证书、配置文件等让初学这头痛不已，但确实给用户带来了极大的安全保障。

文件及环境：

- 一个苹果的认证部门 Apple Worldwide Developer Relations CA颁发的Certificate
- 基于该Certificate生成的mobileprovision文件
- 系统环境 OSX 10.11.5

###证书及文件

####Certificate

Certificate是如何生成的呢？

首先需要开发者生成一个CertificateSigningRequest.certSigningRequest文件，具体步骤为钥匙串访问⟶证书助理⟶从证书颁发机构请求证书，按照提示填写相关信息，保存到磁盘即可。

等到CertificateSigningRequest.certSigningRequest后，通过   ` openssl asn1parse -i -in CertificateSigningRequest.certSigningRequest`查看如下：

![](http://7xkptx.com1.z0.glb.clouddn.com/f43h5j65k87k.png)

这个文件包含：
1，申请者信息
2，申请者公钥，此信息是申请者使用的私钥对应的公钥
3，摘要算法（SHA）和公钥加密算法（RSA）

此文件包含了我的信息，使用了sha1摘要算法和RSA公钥加密算法。苹果的Meber Center在拿到certSigningRequest文件后，将信息记录下来，并签发出相关的证书（Certificate），car证书包含哪些信息呢？又是如何使用certSigningRequest文件中的公钥呢？我们用openssl来看一下证书的内容：

`openssl x509 -inform der -in ios_distribution.cer -noout -text`

``` objc
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            05:33:03:2c:57:2a:ad:6c
        Signature Algorithm: sha1WithRSAEncryption
        Issuer: C=US, O=Apple Inc., OU=Apple Worldwide Developer Relations, CN=Apple Worldwide Developer Relations Certification Authority
        Validity
            Not Before: Jul 10 07:45:50 2014 GMT
            Not After : Jul  9 07:45:50 2017 GMT
        Subject:......
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
            RSA Public Key: (2048 bit)
                Modulus (2048 bit):
    Signature Algorithm: sha1WithRSAEncryption
        a8:b8:b5:ea:74:e9:06:d0:52:90:21:10:10:f1:f6:ce:1f:e7:
        73:08:4f:ca:02:f8:73:66:06:36:48:5b:46:7d:ac:be:bd:c4:
        ......
        d5:46:0a:7b         
```

`Data`域即为证书的实际内容，与`Data`域平级的`Signature Algorithm`实际就是苹果的CA（Certificate Authority）的公钥，Data域包含我的苹果账号信息，其中最为重要的是我的公钥（Subject Public Key Info），这个公钥与我本机的私钥是对应的。当我们双击安装完证书后，`KeyChain会自动将这对密钥关联起来`，所以在KeyChain中可以看到类似的效果：

![](http://7xkptx.com1.z0.glb.clouddn.com/vw3g54gd34f.png)

![](http://7xkptx.com1.z0.glb.clouddn.com/ef3egfwergf3.png)

后续在程序上真机的过程中，会使用这个私钥，对代码进行签名，而公钥会附带在`mobileprovision`文件中，打包进app。那么mobileprovision又从哪里来？有什么作用呢？

####mobileprovision

首先来看一张图：

![](http://7xkptx.com1.z0.glb.clouddn.com/fefegewr4f45f43f.png)

在Apple Developer Center通过之前生成的Certificate来生成mobileprovision配置文件，它将授权和沙盒联系了起来，可以用于让应用在你的开发设备上可以被运行和调试，也可以用于内部测试 (ad-hoc) 或者企业级应用的发布。配置文件并不是一个 plist 文件，它是一个根据密码讯息语法 (Cryptographic Message Syntax) 加密的文件（下文中会简称 CMS）。security 也可以解码这个 CMS 格式，那么我们就用 security 来看看一个 mobileprovision 文件内部是什么样子：
``` objc
$ security cms -D -i example.mobileprovision
```

你会得到一个 XML 格式的 plist 文件内容输出，DeveloperCertificates 这项，这一项是一个列表，包含了可以为使用这个配置文件的应用签名的所有证书。如果你用了一个不在这个列表中的证书进行签名，无论这个证书是否有效，这个应用都无法运行。ProvisionedDevices，在这一项里包含了所有可以用于测试的设备列表。因为配置文件需要被苹果签名，所以每次你添加了新的设备进去就要重新下载新的配置文件。具体如下：

``` objc
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  	<key>DeveloperCertificates</key>
       	<array>
       		<data>MIIF0DCCBLi......Oo7Clog==</data>
        </array>
  <key>ProvisionsAllDevices</key>
  <true/>
  <key>TeamIdentifier</key>
       	<array>
       		<string>C789GLWV85</string>
       	</array>
  <key>Version</key>
       	<integer>1</integer>
</dict>
</plist>
```

所以，证书（及其对应的私钥）和配置文件（mobileprovision）是签名和打包的两个必要文件，如果要重新签名一个App，就需要在这两个上面动手脚了。

首先来了解一个已经签名了的App包含的内容，`$ codesign -vv -d Example.app` 会列出一些有关 `Example.app` 的签名信息：

``` objc
Executable=/Users/pandora/Desktop/shell/gcdsample/Payload/GCDSample.app/GCDSample
Identifier=com.baidu.GCDSample
Format=app bundle with Mach-O universal (armv7 arm64)
CodeDirectory v=20200 size=851 flags=0x0(none) hashes=19+5 location=embedded
Signature size=4700
Authority=iPhone Developer: 张三 (AFCH46B9XZ)
Authority=Apple Worldwide Developer Relations Certification Authority
Authority=Apple Root CA
Signed Time=Jul 18, 2016, 17:23:01
Info.plist entries=26
TeamIdentifier=RYRPKMVKDL
Sealed Resources version=2 rules=12 files=7
Internal requirements count=1 size=180
```

`Authority=iPhone Developer: 张三 (AFCH46B9XZ)`就是我的证书，iPhone Developer是标示开发使用，如果是发布证书的话则标示为 iPhone Distribution。是由`Apple Worldwide Developer Relations Certification Authority` 设置了签名，依此类推这个证书则是被 `Apple Root CA` 设置了签名。`Identifier` 是我在 Xcode 中设置的 bundle identifier。`TeamIdentifier` 用于标识我的工作组（系统会用这个来判断应用是否是由同一个开发者发布），可以通过私钥共享的方式将存储在keychain中的证书对应的私钥导出为p12文件，其他团队成员将此文件导入自己电脑，然后配合从member center下载对应的mobileprovision文件，就可以进行真机开发以及打包发布了。值得一提的是在新发布的Xcode8中新增了支持自动管理证书和自定义管理证书，自动管理证书将会根据你所选择的开发者账号自动为你处理配置文件和签名证书的设置，并且只在必要时进行提醒。这为开发者节省了不少时间和经历。

上面提到证书（及其对应的私钥）和配置文件（mobileprovision）是签名和打包的两个必要文件，那么，mobileprovision在哪里呢？

App在签名的过程中会在程序包中新建一个叫做`_CodeSignatue/CodeResources` 的文件，这个文件中存储了被签名的程序包中所有文件的签名。你可以自己去查看这个签名列表文件，它仅仅是一个 plist 格式文件。这个列表文件中不光包含了文件和它们的签名的列表，还包含了一系列规则，这些规则决定了哪些资源文件应当被设置签名。在 `CodeResources` 文件中会有4个不同区域，其中的 `rules` 和 `files` 是为老版本准备的，而 `files2` 和 `rules2`是为新的第二版的代码签名准备的。最主要的区别是在新版本中你无法再将某些资源文件排除在代码签名之外，在过去（OS X 10.9.5 之前）你是可以的，只要在被设置签名的程序包中添加一个名为 `ResourceRules.plist` 的文件，这个文件会规定哪些资源文件在检查代码签名是否完好时应该被忽略。但是在新版本的代码签名中，这种做法不再有效。所有的代码文件和资源文件都必须设置签名，不再可以有例外。在新版本的代码签名规定中，一个程序包中的可执行程序包，例如扩展 (extension)，是一个独立的需要设置签名的个体，在检查签名是否完整时应当被单独对待。

所以，比如微信这种多target的App，在做重签名的时候，不仅是主工程，还包括watch app及其他extension，都需要重新签名才可以。

以开源中国为例，先从Appstore下载ipa文件，首先执行`$ unzip oschina.ipa`，解压ipa包，进入Payload文件夹内，找到iosapp.app包，`$ codesign -vv -d oschina.app` 会列出一些有关 `Example.app` 的签名信息，通过`security`命令产看keychain中已经安装的证书文件`$ security find-identity -p codesigning`，显示结果如下：

``` objc
Policy: Code Signing
  Matching identities
  1) C552814957BC5691121564774AC86E036B9E2AEE "iPhone Developer: abc (WGDSKET7K5)" (CSSMERR_TP_CERT_EXPIRED)
  2) 630648B3BF32E6D349EDE08C4517CAFA9B12FD6B "iPhone Developer: def (UX59QF88DD)" (CSSMERR_TP_CERT_EXPIRED)
  
  Valid identities only
  1) 32F000F9C845C6626E8E18919E58C386065C5D16 "iPhone Distribution: abc Co.,Ltd."
  2) 38A279804C22853C3F2575FD06BF26E225C08569 "iPhone Distribution: def Co., Ltd."
```

Matching identities下显示所有已经安装的证书，Valid identities only代表当前可用的证书。

### 真正的开始

由于Appstore的的应用都是经过DRM加密的，如果想重签名App，需要将从Appstore下载的ipa文件解密，否则就算签名成功，安装成功，app还是会闪退。通过[逆向Appstore应用（一）](http://dama2716588.github.io/2016/07/20/%E9%80%86%E5%90%91Appstore%E5%BA%94%E7%94%A8%EF%BC%88%E4%B8%80%EF%BC%89/)中的方法，可以拿到解密后开源中国iosapp.decrypted文件，替换Payload目录下ios.app内的名为iosapp的二进制文件，此时就可以得到解密后的iosapp.app文件了。在Payload目录下执行：
``` objc
$ codesign -s "iPhone Distribution: abc" iosapp.app
```

提示：iosapp.app: is already signed，说明此app文件已经被签名过了，需要加上-f参数：

```
$ codesign -f -s "iPhone Distribution: abc" iosapp.app
```
显示：iosapp.app: replacing existing signature，签名成功，执行`codesign -vv -d`查看签名信息如下：

``` objc
➜  Payload codesign -vv -d iosapp.app
Executable=/Users/pandora/Desktop/shell/oschina/Payload/iosapp.app/iosapp
Identifier=net.oschina.iosapp
Format=app bundle with Mach-O universal (armv7 arm64)
CodeDirectory v=20200 size=64082 flags=0x0(none) hashes=1997+3 location=embedded
Signature size=4758
Authority=iPhone Distribution: abc.
Authority=Apple Worldwide Developer Relations Certification Authority
Authority=Apple Root CA
Signed Time=Sep 6, 2016, 18:08:09
Info.plist entries=36
TeamIdentifier=C789GLWV85
Sealed Resources version=2 rules=12 files=473
Internal requirements count=1 size=212
```

然后，压缩已经签名的Payload文件夹为zip格式，然后修改zip为ipa格式即可得到解密后的ipa文件了。使用[同步推](http://tui.tongbu.com/)安装提示ApplicationVerificationFail，提示认证失败，这是因为配置文件（.mobileprovision）认证失败，决定了某一个应用是否能够在某一个特定的设备上运行，接下来需要给替换开源中国的mobileprovision文件。找到证书"iPhone Distribution: abc"对应的配置文件A.mobileprovision，修改名称为：embedded.mobileprovision，copy至iosapp.app包内，重新压缩转换为ipa文件，安装时依然提示：ApplicationVerificationFail。

这是因为缺少了entitlements文件，它决定了哪些系统资源在什么情况下允许被一个应用使用。简单的说它就是一个沙盒的配置列表，上面列出了哪些行为被允许，哪些会被拒绝，Xcode 会将这个文件作为 `--entitlements` 参数的内容传给 `codesign` 。在 Xcode 的 Capabilities 选项卡下选择一些选项之后， Xcode 会自动生成一个 `.entitlements` 文件，然后在需要的时候往里面添加条目。比如将应用添加进了一个 App Group (比如说为了与extensions 共享数据，`com.apple.security.application-groups`)， 或者开启了推送功能 (`aps-environment`)，如果有将它连接到调试器的需求，这就需要将 `get-task-allow` 设为 `true`等等。

可以通过如下命令查看entitlements文件：
``` objc
codesign -d --entitlements - /Users/pandora/Desktop/shell/oschina/开源中国\ 3.7.1/Payload/iosapp.app
```
得到的信息如下：

``` objc
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
       	<dict>
       		<key>keychain-access-groups</key>
       		<array>
       			<string>WSUBE85MHP.net.oschina.iosapp</string>
       		</array>

       		<key>com.apple.developer.team-identifier</key>
       		<string>WSUBE85MHP</string>

       		<key>application-identifier</key>
       		<string>WSUBE85MHP.net.oschina.iosapp</string>

       	</dict>
</plist>
```

所以，重新签名一个app时，除了替换Certificate证书与mobileprovision配置文件外，还需要生成对应的entitlements.plist文件，分别执行以下两个命令：

```
$ security cms -D -i "extracted/Payload/$APPLICATION/embedded.mobileprovision" > t_entitlements_full.plist
$ /usr/libexec/PlistBuddy -x -c 'Print:Entitlements' t_entitlements_full.plist > t_entitlements.plist
```
t_entitlements.plist内容如下：
```objc
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>application-identifier</key>
	<string>C789GLWV85.com.baidu.abc</string>
	<key>aps-environment</key>
	<string>production</string>
	<key>com.apple.developer.associated-domains</key>
	<string>*</string>
	<key>com.apple.developer.team-identifier</key>
	<string>C789GLWV85</string>
	<key>com.apple.security.application-groups</key>
	<array>
		<string>group.com.baidu.abc</string>
	</array>
	<key>get-task-allow</key>
	<false/>
	<key>keychain-access-groups</key>
	<array>
		<string>C789GLWV85.*</string>
	</array>
</dict>
</plist>
```

得到t_entitlements.plist后，把它作为参数传递给codesign，重新签名app文件：

``` objc
codesign -f -s "iPhone Distribution: abc" /Users/pandora/Desktop/shell/oschina/Payload/iosapp.app/ --entitlements t_entitlements.plist
```

最后，压缩Payload文件夹，转化为ipa格式的文件，终于安装成功！本次重签名未更改bundleID，安装的时候会覆盖之前从Appstore下载的应用。如果想更改应用bundleID，可以使用[sigh](https://github.com/fastlane-old/sigh)来重签名应用。

####参考：

[ObjC中国](https://objccn.io/issue-17-2/)

[iOS重签名探索](http://www.jianshu.com/p/6b659c669338)

[漫谈iOS程序的证书和签名机制](http://www.pchou.info/ios/2015/12/14/ios-certification-and-code-sign.html)

[iOS 冰与火之歌番外篇 - App Hook 答疑以及 iOS 9 砸壳](http://gold.xitu.io/entry/56ee241c731956005d22ac8e)





