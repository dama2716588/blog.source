title: APNS测试与部署
tags:
  - 推送
categories: []
date: 2015-11-09 15:27:00
---
**APNS**即Apple Push Notification Service，中文翻译为苹果推送通知服务。特点是稳定、方便，不足是没有送达结果的统计，所以衍生了针对此服务的第三方推送。比如[极光推送](https://developer.apple.com/app-store/review/guidelines/)、[leancloud](https://leancloud.cn/docs/ios_push_cert.html)等，很大程度上减少了服务端的开发量。本文主要介绍APNS的开发调试及部署上线的流程。客户端准备工作如下：

### 创建Certificates

进入[苹果开发者中心](https://developer.apple.com/account/ios/identifiers/bundle/bundleList.action)，打开App IDs，找到Xcode工程对应的Bundle ID，即可看到Push Notifications选项开发与生产配置分别为Configurable，点击Edit，进入下一步Create Certificate，如下图所示。

![](http://7xkptx.com1.z0.glb.clouddn.com/234gwhh54.png)

![](http://7xkptx.com1.z0.glb.clouddn.com/e2f3gwerh65.png)

生成Cer文件的过程中需要本地生成一个.certSigningRequest文件上传

![](http://7xkptx.com1.z0.glb.clouddn.com/1ntf3qwf34.png)

### 如何生成 Certificate Signing Request

打开mac系统中的Keychain，在证书助理中选择从证书颁发机构请求证书，填写邮箱保存本地即可。如下图：

![](http://7xkptx.com1.z0.glb.clouddn.com/1f4h4e5j6ju6.png)

![](http://7xkptx.com1.z0.glb.clouddn.com/2fwgrrh5.png)

生成CSR文件后上传，即可生成Developerment版的cer证书，下载证书到本地，双击安装到钥匙串中，然后打开钥匙串找到刚在安装的cer证书，点击导出，选择个人信息交换(.p12)格式。

![](http://7xkptx.com1.z0.glb.clouddn.com/4bertnr6jutyk.png)

完成上述操作后，打开终端，进入p12文件所在文件夹，执行以下命令，生成服务端push所用的pem证书就可以了。

``` bash
openssl pkcs12 -in XXX.p12 -out XXX.pem -nodes
```

查看证书有效期：
``` bash
openssl x509 -in xxx.pem -noout -dates
```

返回结果：
``` bash
notBefore=Nov  6 07:55:33 2015 GMT
notAfter=Nov  5 07:55:33 2016 GMT
```

连接APNS测试证书是否合法：
``` bash
// Development 环境
openssl s_client -connect gateway.sandbox.push.apple.com:2195 -cert xxx.pem -key xxx.pem 
// Distribution 环境
openssl s_client -connect gateway.push.apple.com:2195 -cert xxx.pem -key xxx.pem

```
合法返回结果：
``` bash
Protocol  : TLSv1
Cipher    : AES256-SHA
Session-ID:
Session-ID-ctx:
Master-Key: 30AF233C50CBEB51B7358BA47E6B4D556CC962BC288F6D51E68300D86400F927925077B5B90C4938B189146E0A4897B2
Key-Arg   : None
Start Time: 1446972326
Timeout   : 300 (sec)
Verify return code: 0 (ok)
```

### 如何测试
Developer环境下的测试推荐一个mac上的app，[Cocoa-APNS-Test](https://github.com/Zambiorix/Cocoa-APNS-Test)，部署简单方便。Production环境下的测试则需要Adhoc证书的支持了，具体操作请参考[这里](https://segmentfault.com/a/1190000000624185)。