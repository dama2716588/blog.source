title: 使用Github发布iOS应用（OTA）
---
**OTA** (Over The Air)是苹果很早就支持的功能，目的是让企业用户脱离Appstore通过网页来发布app，想要实现OTA发布，首页要购买一个企业证书，价格是$299/年。申请企业证书：<https://developer.apple.com/programs/enterprise/>

#####发布流程如下：

**1，使用Xcode生成ipa安装包**

- 首先在TARGETS的General配置中，将Bundle Identifier设置为该企业帐号对应的App ID，如com.baidu.XXX；
- 然后在Build Setting的Code signing一栏，选择企业帐号对应的Distribution证书；
- 接下来选择Xcode->Product->Archive，即可开始打包了。
- 生成xcarchive，接下来Export时，需要选择Save for Enterprise Deployment一项，点击next即可导出ipa文件了。

![Enterprise](http://7xkptx.com1.z0.glb.clouddn.com/giter5ty546j65ge.png)

**2，将ipa、plist、html文件、icon上传至新建的根目录**

首先新建一个空白的github仓库，将打包好的ipa文件及空白install.plist、install.html文件及app Icon图片上传至仓库根目录。这样就有了每个文件在github上的链接地址。

然后配置install.plist文件如下：

``` 
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>items</key>
    <array>
        <dict>
            <key>assets</key>   
                              
          <array>
              <dict>
                  <key>kind</key>
                  <string>software-package</string>
                  <key>url</key>
                  <string>https://***/XXX.ipa</string>
              </dict> 
                             
              <dict>
        		    <key>kind</key>
        			<string>display-image</string>
        			<key>needs-shine</key>
        			<true/>
        			<key>url</key>
        			<string>https://***/Icon57.png</string>
        		</dict>

        		<dict>
        			<key>kind</key>
        			<string>full-size-image</string>
        			<key>needs-shine</key>
        			<true/>
        			<key>url</key>
        			<string>https://***/Icon57.png</string>
        		</dict>
           </array>
            
            <key>metadata</key>
            
            <dict>
                <key>bundle-identifier</key>
                <string>com.baidu.***</string>
                <key>bundle-version</key>
                <string>1.0.0</string>
                <key>kind</key>
                <string>software</string>
                <key>title</key>
                <string>Hiclub</string>
            </dict>
            
        </dict>
    </array>
</dict>
</plist>
```
pist文件中需要注意：

- software-package对应的链接即github上ipa文件所在地址
- display-image为安装过程中iphone桌面上显示的图标
- bundle-identifier为打包ipa中所配置的id
- bundle-version为当前app版本

**3，配置install.html文件，用来生成安装页**

```
<!DOCTYPE HTML>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
        <title>Install</title>
    </head>
    <body>
        <p align=center>
          <font size="10">
            <a style="color:#69DEDA" href="itms-services://?action=download-manifest&url=***/install.plist">点击安装</a>
          </font>
        </p>
    </body>
</html>
```
install.plist即为该文件在github上的链接地址，然后将全部文件push到github。
 
**4，生成在线安装链接** 

![htmlpreview](http://7xkptx.com1.z0.glb.clouddn.com/gitdcehe7654grvew.png)
由于github上的安装页面install.html无法直接预览，通过一个非常使用的小工具[htmlpreview](http://htmlpreview.github.io/)，将install.html原链接转换为可预览的链接，如上图所示。最后用iPhone Safari 打开新生成的链接就可以安装了。 
 