---
title: 基于 WEB 微信通信实现智能聊天机器人
date: 2017-04-10
categories:
  - Web
tags:
  - WeChat
excerpt: "基于微信 Web 协议实现智能聊天机器人，支持群聊、图灵机器人 API 对接、天气查询等智能对话功能。"
aiSummary: "本文记录了基于微信 Web 版通信协议实现智能聊天机器人的完整过程。作者参考开源项目 wechat-robot，修复了原版 bug 并进行功能增强，实现了类似 QQ 小冰的微信机器人。核心功能包括：模拟微信客户端登录（扫码）、消息监听与自动回复、群聊 @ 机器人触发、接入图灵机器人 API 实现语义理解和智能对话、天气查询等插件扩展。是微信个人号自动化的技术探索。"
---


### 背景

> 之前在 QQ 群里看到一个叫“QQ 小冰”的机器人，只要在群里 `@` 她就会出来聊天。除了讲笑话、查天气，语义理解也非常智能（类似美拍的小冰）。
> 顺着这个思路，我调研了市面上的相关机器人，包括微软小冰、茉莉机器人、图灵机器人等，收费的果断跳过。

开源地址：[wechat-robot (GitHub)](https://github.com/thxiex/wechat-robot)

关于具体实现，我发现现有的很多机器人都有 API，提供第三方接入，微信和 QQ 也支持。常见的方式是通过微信公众号接入或关注机器人好友。但我想要的是一个能像真实好友一样出现在对话列表里、且支持群聊的机器人，而不是藏在订阅号列表中的公众号。此外，作为一个开发者，我更希望能自由地为这个机器人编写定制化功能。

<!-- more -->

### 参考资料

- [挖掘微信Web版通信的全过程](http://www.tanhao.me/talk/1466.html/)
- [Python版本 WeixinBot](https://github.com/Urinx/WeixinBot)
- [Java版本 wechat-robot](https://github.com/biezhi/wechat-robot)

对比网上的开源实现，Python 版本的生态和功能普遍比 Java 版本完善。
在这个项目中，我主要参考了上述的 Java 版本。由于原版本存在部分 bug，我在修复问题的基础上，根据自己的需求进行了一些功能增强。

### 开发日志与功能增强

在修复原 Java 版本的部分 bug 之外，我根据自己的想法加入了以下功能：

**修复 Bug**：
1. 对群聊中的消息判断不准确 (`WechatServiceImpl --> handleMsg()`) 

**新增功能**：
1. 机器人接口替换为图灵机器人（原来是茉莉机器人）
2. 支持群聊中被 @ 时回复消息
3. 增加给特定用户定时发送问候语的功能
4. 在定时发送功能中增加 API 调用：
   - 金山 API（获取每日一句英语）
   - 茉莉机器人（获取当天当地天气信息）
5. 增加 Emoji 表情，并支持随机发送
6. 程序处理“图灵机器人”消息内容的水印
7. 增加消息防撤回（识别撤回消息并保存到消息字典）
8. 增加语义处理（趣味回答、口头禅等）
9. 完善控制台和记录文件的 LOGGER 日志，方便日后维护及调试
10. 调用 API 异常处理（例如茉莉机器人的接口有时很不稳定，增加备用接口处理异常以不影响主体功能）

**TODO**：
1. 增加发送图片和语音的功能
2. 研究如何不依赖手机端，并在程序出现异常后重新选择线路
3. 增强程序稳定性



### 执行流程

其实这里与机器人的对话并不是难得, 因为已经有现成的API提供

主要是需要研究微信 Web 协议与API

```text
       +--------------+     +---------------+   +---------------+
       |              |     |               |   |               |
       |   Get UUID   |     |  Get Contact  |   | Status Notify |
       |              |     |               |   |               |
       +-------+------+     +-------^-------+   +-------^-------+
               |                    |                   |
               |                    +-------+  +--------+
               |                            |  |
       +-------v------+               +-----+--+------+      +--------------+
       |              |               |               |      |              |
       |  Get QRCode  |               |  Weixin Init  +------>  Sync Check  <----+
       |              |               |               |      |              |    |
       +-------+------+               +-------^-------+      +-------+------+    |
               |                              |                      |           |
               |                              |                      +-----------+
               |                              |                      |
       +-------v------+               +-------+--------+     +-------v-------+
       |              | Confirm Login |                |     |               |
+------>    Login     +---------------> New Login Page |     |  Weixin Sync  |
|      |              |               |                |     |               |
|      +------+-------+               +----------------+     +---------------+
|             |
|QRCode Scaned|
+-------------+
```



### API

#### 获取会话ID

> Get UUID

```http
URL: https://login.wx.qq.com/jslogin
请求方式: GET
参数: 
	a. appid: YOUR_WECHAT_APP_ID(固定字符串)
	b. fun: new(固定值)
	c. lang: zh_CN(固定值)
	d. _: 1491804797(13位毫秒时间戳)
返回数据(String):
window.QRLogin.code = 200; window.QRLogin.uuid = "[UUID]"
状态码code=200表示成功
```



#### 显示二维码图片

> Get QRCode

```http
URL: https://login.weixin.qq.com/qrcode/[UUID] (上一步获取到的返回值window.QRLogin.uuid)
请求方式: GET
返回数据: 二维码
```



#### 手机端扫描二维码等待确认登录

```http
URL: https://login.weixin.qq.com/cgi-bin/mmwebwx-bin/login
请求方式: GET
参数: 
	a. uuid: [UUID](前面获取到的UUID)
	b. tip: 1 (1-未扫描  0-已扫描)
	d. _: 1491804797(13位毫秒时间戳)
返回数据(String):
window.code=xxx(408 登陆超时, 201 扫描成功但未确认, 200 确认登录)
由于该请求需要用户在手机端连续做几个操作, 所以代码里要轮询来实现. 直到返回结果为200.
获取到以下URL后需要继续访问当前链接获取wxuin和wxsid
window.redirect_uri="https://wx.qq.com/cgi-bin/mmwebwx-bin/webwxnewloginpage?ticket=xxx&uuid=xxx&lang=xxx&scan=xxx";
```



#### 后续流程简述

大致步骤如下：

1. 初始化微信, 开启状态通知, 保存个人信息, 登录信息, 并将联系人列表和群组列表保存下来.

2. 然后选择同步线路, 轮询进行消息检查. 

   获取到最新消息后调用机器人API(这里我用的是图灵机器人)获得回答结果.

3. 然后调用消息发送API, 完成消息发送.

相关的通信过程和API网上有很多. 在开头参考中有推荐



### 附注

为了方便开发, 加几个附注:

#### 1: 同步状态
在同步消息检查的API中:<https://webpush2.weixin.qq.com/cgi-bin/mmwebwx-bin/synccheck>

为了模拟实时消息的更新, 在程序中轮询2秒检查一次, 此接口的返回值如下:

```json
window.synccheck={retcode:"xxx",selector:"xxx"}
第一步判断: retcode
	0-正常
	1100-失败/登出微信
第二步判断: selector
	0 正常
	2/6 新的消息
	7 进入/离开聊天界面
```

所以当`selector=2/6`时, 我们就可以进行消息处理.

这里selector有个很奇怪的返回值, 就是`3`! 

我翻阅各种API也没找到为什么有时会返回`3`导致程序死掉



#### 2: 消息账户类型

在发送消息之前, 需要获取同步消息.
URL: <https://wx.qq.com/cgi-bin/mmwebwx-bin/webwxsync?sid=xxx&skey=xxx&pass_ticket=xxx>
返回值包括了消息发送方, 接收方, 消息内容, 消息类型.
消息来源的账号类型大致有这几类:
来自个人: 以@开头
来自群聊: 以@@开头
来自公众号/服务号: 以@开头，VerifyFlag & 8 != 0
来自特殊账号:

```java
// 特殊用户 须过滤
("newsapp", "fmessage", "filehelper", "weibo", "qqmail", "fmessage", "tmessage", "qmessage",
 "qqsync", "floatbottle", "lbsapp", "shakeapp", "medianote", "qqfriend", "readerapp", "blogapp",
 "facebookapp", "masssendapp", "meishiapp", "feedsapp", "voip", "blogappweixin", "weixin", "wxitil",
"brandsessionholder", "weixinreminder", "wxid_novlwrv3lqwv11", "gh_22b87fa7cb3c", "officialaccounts",
"notification_messages", "wxid_novlwrv3lqwv11", "gh_22b87fa7cb3c",  "userexperience_alarm");
```



#### 3: 消息类型

| MsgType | 说明               |
| ------- | ------------------ |
| 1       | 文本消息           |
| 3       | 图片消息           |
| 34      | 语音消息           |
| 37      | VERIFYMSG          |
| 40      | POSSIBLEFRIEND_MSG |
| 42      | 共享名片           |
| 43      | 视频通话消息       |
| 47      | 动画表情           |
| 48      | 位置消息           |
| 49      | 分享链接           |
| 50      | VOIPMSG            |
| 51      | 微信初始化消息     |
| 52      | VOIPNOTIFY         |
| 53      | VOIPINVITE         |
| 62      | 小视频             |
| 9999    | SYSNOTICE          |
| 10000   | 系统消息           |
| 10002   | 撤回消息           |



### 接入图灵机器人

关于图灵机器人的调用，前往[官网](http://www.turingapi.com/)注册账号并获取 API Key 即可快速接入，非常方便。



### 效果演示

在调试功能时, 可以加上log, 查看同步连接信息和消息
![小豆丁-后台日志](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E5%B0%8F%E8%B1%86%E4%B8%81-%E5%90%8E%E5%8F%B0%E6%97%A5%E5%BF%97.png)

附上几张和机器人的聊天:

![小豆丁-聊天截图1](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E5%B0%8F%E8%B1%86%E4%B8%81-%E8%81%8A%E5%A4%A9%E6%88%AA%E5%9B%BE1.jpg)

![小豆丁-聊天截图2](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E5%B0%8F%E8%B1%86%E4%B8%81-%E8%81%8A%E5%A4%A9%E6%88%AA%E5%9B%BE2.jpg)

![小豆丁-群聊](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E5%B0%8F%E8%B1%86%E4%B8%81-%E7%BE%A4%E8%81%8A.jpg)