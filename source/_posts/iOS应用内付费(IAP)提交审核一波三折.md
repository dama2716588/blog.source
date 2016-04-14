title: iOS应用内付费(IAP)提交审核一波三折
tags:
  - IAP
categories:
  - App Store
date: 2015-09-19 15:00:00
---
App Store的审核众所周知是一个耗时耗力的活儿，尤其是第一个版本的提交，如果App又包含的内付费的功能，那就更需要耐心了。为期一个半月的审核终于通过，简单记录下期间的经验，希望能帮助到大家。

#### 首先汇总下被拒的官方回复：

- 2.2 - Apps that exhibit bugs will be rejected
- 2.9 - Apps that are "demo", "trial", or "test" versions will be rejected. Beta Apps may only be submitted through TestFlight and must follow the TestFlight guidelines
- 3.3 - Apps with names, descriptions, screenshots, or previews not relevant to the content and functionality of the App will be rejected
- 11.4 - Apps that use IAP to purchase credits or other currencies must consume those credits within the App
- 14.1 - Any App that is defamatory, offensive, mean-spirited, or likely to place the targeted individual or group in harm's way will be rejected
- 14.3 - Apps that display user generated content must include a method for filtering objectionable material, a mechanism for users to flag offensive content, and the ability to block abusive users from the service
- 17.2 - Apps that require users to share personal information, such as email address and date of birth, in order to function will be rejected

#### 针对以上问题的分析与解决：

- 2.2，苹果测试人员在审核期间会使用最新的iOS系统来测试应用，但不一定是最新的硬件。所以在提审之前要保证App在最新的系统下运行流畅，如果是iPhone的版本，也需要确保在iPad上不出问题才行。当时我们提交的时候官方最新的iOS版本是8.4.1，我们以为在8.4.0上运行没问题就好，忽略了iOS小版本的某些特性差异。
  
- 2.9，由于我们的App是做了日文的本地化，在首页banner图片上有细小的测试字样，提审的时候没有去除，苹果对这方面的审核非常严格，会认为你的产品当前仍处于测试版，所以不能上架。提醒开发者留意这些细节。
  
- 3.3，此项是关于iTunes Connect 中App介绍说明的，建议说明文案宁少勿多，不要涉及等等、更多字样，只列举明确包含的功能即可。
  
- 11.4，苹果关于IAP的规则是虚拟货币不可以在app内流通且只能在平台内消费，更不允许有送礼+分成等方式，关于收入的分成十分严格，请开发者谨慎对待。在产品初期制定好使用规则，不然以后改动成本巨大。
  
- 14.1/14.3，建议开发者对应用的评级有一个合理的定位，尤其是UGC、视频方面的App，在不清楚的情况下级别勾选越高越对审核有帮助。详细请参考[官方审核指南](https://developer.apple.com/app-store/review/guidelines/)。如果是UGC内容要有非常明显的举报入口，及一定范围内的敏感词过滤功能。
  
- 17.2，在App集成第三方登录时会经常遇到，苹果建议开发者有自己的帐号系统，如果是使用Facebook/Twitter/Weibo/Weixin做认证，除了拉取用户个人资料和分享，App必须包含显著的FB和TW特定账户功能。特定功能比如同步Feed至第三方系统，获取粉丝及关注列表等。针对此条款我们专门做了申诉，大家可以作为参考：
  
  Sorry but I am afraid you misunderstood the function of our application. Actually wedo not force users to share personal information in order to function, we justpull our users’ profile information when they login from Facebook and Twitter.And for the account-based features from Facebook and Twitter, users can share streaming to The-third Party platform, andafter sharing success, there will be link on the-third Party platform  which can click and jump to our app. Userscan also share their clips to Facebook and Twitter and these message will besynchronized. we made screenshots to explain this functionality of ourapplication. I hope it works to help you know more about our app. 
  
  另外还配了应用内使用Facebook功能的配图，比如分享、同步Feed，最后才审核通过。下图为数次被拒的原因。

![Purchases](http://7xkptx.com1.z0.glb.clouddn.com/43g34h6.png)

#### 重点说一下IAP审核遇到的问题

- 先说购买凭证的验证，在苹果审核期间只会再Sandbox环境购买，所以购买凭证需要链接苹果测试服务器（[https://sandbox.itunes.apple.com/verifyReceipt](https://sandbox.itunes.apple.com/verifyReceipt) ）来验证，等审核通过，后端部署到苹果正式服务器（[https://buy.itunes.apple.com/verifyReceipt](https://buy.itunes.apple.com/verifyReceipt)）即可。我们在这方面犯了一个错误就是在期间后段链接的是苹果的测试服务器造成购买失败，应用被拒绝。
- 如果应用被拒了一次，再次提交时的如果IAP商品的状态为Developer Action Needed（如下图所示），需要手动刷新下，编辑下商品名称，加个空格即可。然后状态会变为正在等待审核，再上传二进制文件。

![Purchases](http://7xkptx.com1.z0.glb.clouddn.com/f43thj4b53h235g5b.png)



#### One more thing - 合理申诉

- 如果应用被拒后，第一时间先确认是由于二进制文件的问题，还是文案的描述问题，或者是苹果审核团队的疑问未得到解答。如果是二进制文件被拒，则需要修复问题后重新打包上传，等待审核结果。


- 如果是提审文案或者配图的问题只需要修改下再次提交审核即可，无需二次打包。
- 还有一种情况是审核被拒，可以不做任何改动，直接申诉。在iTunes Connect解决方案中心会收到来自苹果审核团队的站内信，只需要详细逐条回复即可。最好绘声绘色，图文并茂，一般24小时内会得到苹果的二次确认。如果审核人员认可了你的申诉，那么你的App很快就会进入In Review的状态，离上架就只有一步之遥了。心酸经历附个图：

![Purchases](http://7xkptx.com1.z0.glb.clouddn.com/f2f32g43gwg.png)

#####相关引用：
https://blog.coding.net/blog/ios-testFlight
http://blog.devtang.com/blog/2013/04/07/tricks-in-iap/