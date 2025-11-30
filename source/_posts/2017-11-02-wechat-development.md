---
title: 微信开发四篇
date: 2017-11-02
categories:
  - Web
tags:
  - WeChat
excerpt: "微信开发四合一教程，涵盖微信网页授权、微信支付、网站应用微信登录（OAuth2.0）和微信 JS-SDK 集成。"
aiSummary: "本文是微信开发的综合性教程，涵盖了微信公众号平台和开放平台的四个核心功能模块：微信网页授权（OAuth2.0 获取用户信息）、微信支付（JSAPI 支付流程）、网站应用微信登录（第三方网站使用微信登录）、以及微信 JS-SDK（网页调用微信原生能力）。文章通过实际代码示例详细说明了各模块的开发流程、参数配置和常见问题处理，是微信开发入门和实践的实用参考。"
---



### 序言

> 上一篇微信相关的开发是[**《基于WEB微信通信实现智能聊天机器人》**](/posts/wechat-robot).
>
> 而怎么保持机器人稳定在线, 不被微信封号, 是一直遗留的问题…
>
> 网上一个稳定版本的IOS微信协议已经卖到2000! 所以无力研究了.
>
> 相比之下, 微信公众号开发和小程序更加火爆.

本篇将通过**微信公众平台**, **微信开放平台**, **微信商户平台**. 分为四篇来完成以下几个常用功能.



**第一篇: 微信网页授权**

**第二篇: 微信支付**

**第三篇: 网站应用微信登录**

**第四篇: 推送微信模版消息**

**相关文档** (必须了解)

- 微信公众平台技术文档: <https://mp.weixin.qq.com/wiki>
- 微信支付开发文档(公众号支付): <https://pay.weixin.qq.com/wiki/doc/api/index.html>
- 微信开放平台: <https://open.weixin.qq.com/>

**使用SDK** (开发效率事半功倍)

> - 微信支付、小程序、公众号&企业号开发Java SDK: <https://github.com/Wechat-Group/weixin-java-tools>
> - 可能是目前最好的支付SDK: <https://github.com/Pay-Group/best-pay-sdk>

<!-- more -->

### 你需要知道的事

通过阅读微信公众平台文档: 可以知道公众平台账号分为:

- 订阅号
- 服务号
- 企业号

每类账号的适用人群不同, 开放的接口权限不同, 申请门槛也不同.

**公众号接口权限说明**

> 官方参考: <http://kf.qq.com/faq/170104AJ3y26170104Yj673y.html>

可以看到, 微信授权和支付功能这两项权限只有**微信认证服务号**才有.

服务号一般人没有企业资质很难申请到, 更别说微信认证了.

好的一点是微信公众平台提供了一个测试号, 可以通过手机扫描二维码来获得测试号.

> 申请地址: <https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login>

注意: 测试公众号有很多体验接口, 包括网页授权获取(无支付权限).

### 你需要准备

> 微信开发需要手机电脑来回调试, 过程还是比较繁琐. 如何在本地方便的进行调试, 就要依赖一些东西.

1. 域名
   微信网页授权的回调地址不能是ip地址. 这一点在官方文档中有提到. 所以在回调到我们的应用程序接口时要支持
   `域名/接口路径` 的方式, 例如: `http://thank.mynatapp.cc/wechat/xxx`

2. 内网映射
   在手机上调试微信客户端时, 调用PC接口, 肯定无法用 `http://localhost:8888/wechat/xxx` 的方式了, 为了在本地方便的调试, 就需要用到内网穿透工具. 可以参考我的另一篇: [内网穿透介绍](/posts/intranet-penetration)

3. 抓包工具
   在手机上点击时, 无法像电脑浏览器中的F12一样看到详细的网络请求.

   所以使用fiddler或charles等抓包工具来抓取网络请求.

   手机Wi-Fi中的HTTP代理中填写主机的ip地址和8888端口(默认)即可.

4. 微信web开发者工具

   这是微信官方提供给开发者的工具, 很方便.

   不用下载抓包工具, 也不用设置手机代理.

   可以像电脑浏览器一样模拟手机发送请求. 支持公众号网页调试和小程序调试.

   ![微信开发-开发者工具](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E5%BE%AE%E4%BF%A1%E5%BC%80%E5%8F%91-%E5%BC%80%E5%8F%91%E8%80%85%E5%B7%A5%E5%85%B7.png)

   注意: 微信支付相关的调试在授权时需要在微信公众平台的账号中绑定开发者.

### 第一篇: 微信网页授权

网页授权是指用户在微信客户端访问第三方网页时, 公众号可通过授权机制来获取用户的基本信息. 从而实现业务逻辑.

> 授权流程: <https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140842>

通过了解授权流程, 可以看到. 网页授权流程主要分为四步：

1. 引导用户进入授权页面同意授权，获取code
2. 通过code换取网页授权access_token（与基础支持中的access_token不同）
3. 如果需要，开发者可以刷新网页授权access_token, 避免过期
4. 通过网页授权access_token和openid获取用户基本信息（支持UnionID机制）

主要来看前2步:

```
# 替换相关参数,相关参数说明在文档中有详细介绍...
https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=SCOPE&state=STATE#wechat_redirect

# 例如: 
https://open.weixin.qq.com/connect/oauth2/authorize?appid=YOUR_WECHAT_APP_ID&redirect_uri=http://thank.mynatapp.cc/sell/weixin/auth&response_type=code&scope=snsapi_base&state=STATE#wechat_redirect
```

在微信客户端访问该url后, 会进行跳转, `redirect_uri` 会重定向到我们的应用.

在跳转之前, 需要用户同意授权. 而用户同意授权的方式, 取决于上面的 `scope` 参数:

- snsapi_base: 静默授权, 微信端用户会在无感知的情况下同意授权, 但是拿到的信息也较少, 只有用户openid.

- snsapi_userinfo: 微信端会弹出授权页面点击确认. 这种情况下拿到的openid可以再继续拿到用户的更多信息

  如昵称, 性别, 所在地等.

当微信客户端用户同意授权后, 页面将跳转路径为: `redirect_uri/?code=CODE&state=STATE`

也就是携带着code参数进入我们的应用接口中, 从而在应用接口中获取到code.

> code作为换取access_token的票据，每次用户授权带上的code将不一样，code只能使用一次，5分钟未被使用自动过期。

##### 编码实现

```
@GetMapping("/auth")
public void auth(@RequestParam("code") String code){

    //第一步: 用户同意授权, 通过回调, 进入该方法, 获取到code
    log.info("-----进入auth方法: code={}", code);

    //第二步：通过code换取网页授权access_token
    String getTokenUrl = "https://api.weixin.qq.com/sns/oauth2/access_token?appid="
            +appid+"&secret="
            +appsecret+"&code="
            +code+"&grant_type=authorization_code";
    RestTemplate restTemplate = new RestTemplate();
    String response = restTemplate.getForObject(getTokenUrl, String.class);
    log.info("-----response={}", response );

    Map<String, Object> map = new Gson().fromJson(response, 
            new TypeToken<HashMap<String,Object>>(){}.getType());
    log.info("--- accessToken={}", map.get("access_token"));
    log.info("--- openid={}", map.get("openid"));
    log.info("--- scope={}", map.get("scope"));
}
```

授权过程是可以用手工编写代码来完成, 但比较繁琐. 而github上有很多现成的SDK提供, 所以不需要重复造轮子.

##### sdk实现

> Github地址: [微信支付、小程序、企业号和公众号（包括服务号和订阅号） Java SDK开发工具包](https://github.com/Wechat-Group/weixin-java-tools)

1. Maven依赖

   ```xml
   <dependency>
       <groupId>com.github.binarywang</groupId>
       <artifactId>weixin-java-mp</artifactId>
       <version>2.7.0</version>
   </dependency>
   ```

2. 完成微信公众号相关配置读取

   ```java
   @Component
   public class WechatMpConfig {
   
   	// 配置读取类
       @Autowired
       private WechatAccountConfig wechatAccountConfig;
   
       @Bean
       public WxMpService wxMpService(){
   
           WxMpService wxMpService = new WxMpServiceImpl();
           wxMpService.setWxMpConfigStorage(wxMpConfigStorage());
           return wxMpService;
       }
   
       @Bean
       public WxMpConfigStorage wxMpConfigStorage(){
   
           WxMpInMemoryConfigStorage wxMpConfigStorage = new WxMpInMemoryConfigStorage();
           wxMpConfigStorage.setAppId(wechatAccountConfig.getMpAppId());
           wxMpConfigStorage.setSecret(wechatAccountConfig.getMpAppSecret());
           return wxMpConfigStorage;
       }
   }
   ```

3. 找到该项目的github 开发wiki: 公众号开发 -> OAuth2网页授权

   授权过程分为这几步:

- 首先构造网页授权url，然后构成超链接让用户点击

  ```
  wxMpService.oauth2buildAuthorizationUrl(redirectURI, 
          WxConsts.OAUTH2_SCOPE_USER_INFO, 
          URLEncoder.encode(returnUrl));
  ```

  该方法返回一个url, 让微信客户端用户点击, 当用户同意授权, 会回调至设置的redirectURI. 并把code携带过去. 用户点击的页面如下:

  ![微信开发-用户点击授权](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E5%BE%AE%E4%BF%A1%E5%BC%80%E5%8F%91-%E7%94%A8%E6%88%B7%E7%82%B9%E5%87%BB%E6%8E%88%E6%9D%83.png)

  ```
  WxMpOAuth2AccessToken wxMpOAuth2AccessToken = wxMpService.oauth2getAccessToken(code);
  String openid = wxMpOAuth2AccessToken.getOpenId();
  ```

  再通过该方法获取openid. (除了openid以外, 还可以获得用户其它信息), 如下图:

  ```
  WxMpUser wxMpUser = this.wxMpService.oauth2getUserInfo(wxMpOAuth2AccessToken, null);
  ```

  ![微信开发-获取用户信息](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E5%BE%AE%E4%BF%A1%E5%BC%80%E5%8F%91-%E8%8E%B7%E5%8F%96%E7%94%A8%E6%88%B7%E4%BF%A1%E6%81%AF.png)


### 第二篇: 微信支付

> sdk: [Pay-Group/best-pay-sdk](https://github.com/Pay-Group/best-pay-sdk)
>
> 虽然微信官方有提供SDK和DEMO, 但是并不好用
>
> 这款SDK是在github中找的, 里面有很详细的视频说明, 而且还提供支付账号借用, 对于拿不到公众号认证资格的开发者来说很方便学习.

```
<!--微信支付相关SDK-->
<dependency>
  	<groupId>cn.springboot</groupId>
  	<artifactId>best-pay-sdk</artifactId>
  	<version>1.1.0</version>
</dependency>
```

![微信开发-支付相关](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E5%BE%AE%E4%BF%A1%E5%BC%80%E5%8F%91-%E6%94%AF%E4%BB%98%E7%9B%B8%E5%85%B3.png)

##### 支付流程

下面来描述一个最基本的支付流程, 包括以下几个环节:

- 统一下单
- 微信客户端H5支付
- 接收微信异步通知
- 退款

##### 1. 统一下单API

> 应用场景

除被扫支付场景以外，商户系统先调用该接口在微信支付服务后台生成预支付交易单，返回正确的预支付交易回话标识后再按扫码、JSAPI、APP等不同场景生成交易串调起支付。

1. 配置WxPayH5Config

   ```java
   @Component
   public class WechatPayConfig {
   
     	// 配置读取类
       @Autowired
       private WechatAccountConfig accountConfig;
   
       @Bean
       public BestPayServiceImpl bestPayService() {
           BestPayServiceImpl bestPayService = new BestPayServiceImpl();
           bestPayService.setWxPayH5Config(wxPayH5Config());
           return bestPayService;
       }
   
       @Bean
       public WxPayH5Config wxPayH5Config() {
           WxPayH5Config wxPayH5Config = new WxPayH5Config();
           wxPayH5Config.setAppId(accountConfig.getMpAppId());
           wxPayH5Config.setAppSecret(accountConfig.getMpAppSecret());
           wxPayH5Config.setMchId(accountConfig.getMchId());
           wxPayH5Config.setMchKey(accountConfig.getMchKey());
           wxPayH5Config.setKeyPath(accountConfig.getKeyPath());
           wxPayH5Config.setNotifyUrl(accountConfig.getNotifyUrl());
           return wxPayH5Config;
       }
   }
   ```

2. 设置发起支付的相关参数 `PayRequest` , 调用创建支付接口

   ```
   PayResponse payResponse = this.bestPayService.pay(payRequest);
   ```

   这样就完成了生成预付单. `payResponse` 返回结果中携带预付单信息(prepay_id及其他信息)

##### 2. 微信内H5调起支付

注意: 必须在微信客户端浏览器中

下单的返回对象 `payResponse` 会返回 `getBrandWCPayRequest` 中的参数

> 参数列表详细描述:<https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_7&index=6>

```java
function onBridgeReady(){
   WeixinJSBridge.invoke(
       'getBrandWCPayRequest', {
           "appId":"YOUR_WECHAT_APP_ID",     //公众号名称，由商户传入     
           "timeStamp":"1395712654",         //时间戳，自1970年以来的秒数     
           "nonceStr":"e61463f8efa94090b1f366cccfbbb444", //随机串     
           "package":"prepay_id=u802345jgfjsdfgsdg888",     
           "signType":"MD5",         //微信签名方式：     
           "paySign":"70EA570631E4BB79628FBCA90534C63FF7FADD89" //微信签名 
       },
       function(res){     
           if(res.err_msg == "get_brand_wcpay_request:ok" ) {}     // 使用以上方式判断前端返回,微信团队郑重提示：res.err_msg将在用户支付成功后返回    ok，但并不保证它绝对可靠。 
       }
   ); 
}
if (typeof WeixinJSBridge == "undefined"){
   if( document.addEventListener ){
       document.addEventListener('WeixinJSBridgeReady', onBridgeReady, false);
   }else if (document.attachEvent){
       document.attachEvent('WeixinJSBridgeReady', onBridgeReady); 
       document.attachEvent('onWeixinJSBridgeReady', onBridgeReady);
   }
}else{
   onBridgeReady();
}
```

此时在微信客户端生成JSAPI页面, 用户点击即可发起支付.

**关于支付成功的判断**

后台的商户系统通常会根据支付状态去修改商品的状态, 库存等.

如何判断用户是否支付成功呢?

在H5调起支付中, 网页支付接口中在完成后会返回 `err_msg` .

| 返回值                         | 描述               |
| ------------------------------ | ------------------ |
| get_brand_wcpay_request:ok     | 支付成功           |
| get_brand_wcpay_request:cancel | 支付过程中用户取消 |
| get_brand_wcpay_request:fail   | 支付失败           |

但是这种方式并不可靠, 微信团队也有说明.

所以, 可靠的方式是通过微信返回给我们的支付结果通知 .

##### 3. 支付结果通知API

- **微信异步通知**

  > 支付完成后，微信会把相关支付结果和用户信息发送给商户，商户需要接收处理，并返回应答。
  >
  > 注意微信的异步通知需要设置 notify_url , 否则无法接收.

  关于 `notify_url` : 微信没有设置白名单, 所以只要外网能访问到的地址即可.

  例如: <http://thank.mynatapp.cc/wechat/pay/notify>

  微信返回给商户系统的数据如下: (SUCCESS情况下)

  ```
  <xml>
    <appid><![CDATA[YOUR_WECHAT_APP_ID]]></appid>
    <attach><![CDATA[支付测试]]></attach>
    <bank_type><![CDATA[CFT]]></bank_type>
    <fee_type><![CDATA[CNY]]></fee_type>
    <is_subscribe><![CDATA[Y]]></is_subscribe>
    <mch_id><![CDATA[10000100]]></mch_id>
    <nonce_str><![CDATA[5d2b6c2a8db53831f7eda20af46e531c]]></nonce_str>
    <openid><![CDATA[oUpF8uMEb4qRXf22hE3X68TekukE]]></openid>
    <out_trade_no><![CDATA[1409811653]]></out_trade_no>
    <result_code><![CDATA[SUCCESS]]></result_code>
    <return_code><![CDATA[SUCCESS]]></return_code>
    <sign><![CDATA[B552ED6B279343CB493C5DD0D78AB241]]></sign>
    <sub_mch_id><![CDATA[10000100]]></sub_mch_id>
    <time_end><![CDATA[20140903131540]]></time_end>
    <total_fee>1</total_fee>
    <trade_type><![CDATA[JSAPI]]></trade_type>
    <transaction_id><![CDATA[1004400740201409030005092168]]></transaction_id>
  </xml>
  ```

  具体的参数信息在官方API中都有详细描述

  接受到微信异步通知后, 通过SDK来接收微信支付结果消息

  ```
  // 1. 验证签名 (SDK已做)
  
  // 2. 验证支付状态 (SDK已做)
  
  PayResponse payResponse = this.bestPayService.asyncNotify(notifyData);
  
  // 3. 商户系统自己做验证金额, 支付人等处理.
  
  // 4. 验证完成后即可修改商品和订单的相关状态
  
  // 5. 返回微信处理结果
  ```

- **商户同步返回处理结果**

  > 商户系统在接收到微信的异步通知后, 就可以进行商品状态/库存修改等操作了,
  >
  > 处理完成后需要告诉微信处理结果, 否则微信会一直发送异步通知过来.

  ```
  <xml>
    <return_code><![CDATA[SUCCESS]]></return_code>
    <return_msg><![CDATA[OK]]></return_msg>
  </xml>
  ```

##### 4. 申请退款API

> 当交易发生之后一段时间内，由于买家或者卖家的原因需要退款时，卖家可以通过退款接口将支付款退还给买家，微信支付将在收到退款请求并且验证成功之后，按照退款规则将支付款按原路退到买家帐号上。

与统一下单接口很类似, 不同的是申请退款请求需要双向证书, 开发者需要[微信商户平台](https://pay.weixin.qq.com)下载相关证书文件.

```
# 设置RefundRequest
RefundResponse refundResponse = this.bestPayService.refund(refundRequest);
```

### 第三篇: 微信登录功能

这里介绍网站应用的微信登录功能, 该功能是在微信开放平台下.

网站应用微信登录是基于OAuth2.0协议标准构建的微信OAuth2.0授权登录系统.

微信OAuth2.0授权登录是让微信用户以微信身份安全登录第三方应用或网站.

> 扫码登录体验: [一号店](https://passport.yhd.com/wechat/login.do)

**需要注意**!

- 微信登录相关的授权除了要有开放平台开发者账号, 还需要用户号有企业资质的认证. 普通账号无法授权!
- 开放平台的授权流程和公众平台类似, 但是 `AppID` 和 `AppSecret` 不同, 所以获得的 `openid` 也不同.
- 每个用户针对每个公众号会产生一个安全的 `OpenID` . 一个微信号在不同公众号下的 `openid` 是不同的

##### 授权流程

1. 第三方发起微信授权登录请求，微信用户允许授权第三方应用后，微信会拉起应用或重定向到第三方网站，并且带上授权临时票据code参数；
2. 通过code参数加上AppID和AppSecret等，通过API换取access_token；
3. 通过access_token进行接口调用，获取用户基本数据资源或帮助用户实现基本操作。

详细的流程参考[开发指南](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419316505&token=&lang=zh_CN), 这里我使用sdk方式:[best-pay-sdk](https://github.com/Pay-Group/best-pay-sdk)

##### sdk实现

首先完成微信开放平台相关配置, 配置 `AppID` 和 `AppSecret`

```java
@Component
public class WechatOpenConfig {

  	// 配置读取类
    @Autowired
    private WechatAccountConfig wechatAccountConfig;

    @Bean
    public WxMpService wxOpenService(){
        WxMpService wxOpenService = new WxMpServiceImpl();
        wxOpenService.setWxMpConfigStorage(wxOpenConfigStorage());
        return wxOpenService;
    }

    @Bean
    public WxMpConfigStorage wxOpenConfigStorage(){
        WxMpInMemoryConfigStorage wxMpInMemoryConfigStorage = new WxMpInMemoryConfigStorage();
        wxMpInMemoryConfigStorage.setAppId(wechatAccountConfig.getOpenAppId());
        wxMpInMemoryConfigStorage.setSecret(wechatAccountConfig.getOpenAppSecret());
        return wxMpInMemoryConfigStorage;
    }
}
```

**第一步: 请求code**

用户向第三方应用发起登录请求:

```java
@GetMapping("qrAuthorize")
public String qrAuthorize(@RequestParam("state") String state){

	String redirectUrl = this.projectUrlConfig.getWechatOpenAuthorize() 
							+ "/wechat/qrUserInfo";
	String qrPageUrl = wxOpenService.buildQrConnectUrl(redirectUrl, 
							WxConsts.QRCONNECT_SCOPE_SNSAPI_LOGIN, state);
	return "redirect:" + qrPageUrl;
}
```

跳转至微信登录的二维码界面, 等待用户扫码授权后, 会重定向到 `redirectUrl` , 并携带 `code` 和 `state` 参数.

`redirect_uri?code=CODE&state=STATE`

**第二步: 通过code换取access_token**

```java
@GetMapping("qrUserInfo")
public String qrUserInfo(@RequestParam("code") String code, 
						 @RequestParam("state") String state) {

	WxMpOAuth2AccessToken wxMpOAuth2AccessToken = new WxMpOAuth2AccessToken();
	try {
		wxMpOAuth2AccessToken = this.wxOpenService.oauth2getAccessToken(code);
	} catch (WxErrorException e) {
		log.error("[微信网页授权错误: {}]", e);
	}

	log.info("state={}", state);
	log.info("wxMpOAuth2AccessToken={}", wxMpOAuth2AccessToken);
	log.info("accessToken={}", wxMpOAuth2AccessToken.accessToken);
	log.info("openid={}", wxMpOAuth2AccessToken.getOpenId());
	return "redirect:";
}
```

获取到用户的 `openid` , 授权完成.

### 第四篇: 微信模版消息推送

这里用到微信公众平台中消息管理功能.

> 模板消息仅用于公众号向用户发送重要的服务通知，只能用于符合其要求的服务场景中，如信用卡刷卡
>
> 通知，商品购买成功通知等。不支持广告等营销类消息以及其它所有可能对用户造成骚扰的消息。
>
> 官方文档: <https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1433751277>

通过阅读文档: 这种容易产生骚扰, 欺骗功能的接口肯定还是需要公众账号, 认证服务号的.

还好的一点是测试号有**模版消息(业务通知)**的接口. 所以这里我用测试账号.

##### 1. 设置模版

在公众号管理中添加模版(测试号也类似)

![微信开发-添加模版](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E5%BE%AE%E4%BF%A1%E5%BC%80%E5%8F%91-%E6%B7%BB%E5%8A%A0%E6%A8%A1%E7%89%88.png)

- 模版标题: 随意填写

- 模版内容: 如果是公众号可以选择现有的模版, 测试号只能自己输入(需要遵循格式), 例如:

  ```
  {{first.DATA}}
  收款方:{{keyword1.DATA}}
  支付方:{{keyword2.DATA}}
  支付方式:{{keyword3.DATA}}
  交易状态:{{keyword4.DATA}} 
  {{remark.DATA}}
  ```

- 模版ID: 自动生成(用于接口调用)

##### 2. 发送模版消息

```
http请求方式: POST
https://api.weixin.qq.com/cgi-bin/message/template/send?access_token=ACCESS_TOKEN
```

post发送模版内容data数据

> 使用[sdk](https://github.com/Wechat-Group/weixin-java-tool) 实现

```java
@Autowired
private WxMpService wxMpService;

public void pushTemplateMessage() {

	String templateId = "f-wEkgooC6My0792dHeoEZ-Zd85cOXYJShMB45sCdV4";
	String openId = "ol4h7wot1fneGq1RN0LfWaNBtsYQ";

	WxMpTemplateMessage templateMessage = new WxMpTemplateMessage();
	templateMessage.setTemplateId(templateId);
	templateMessage.setToUser(openId);

	List<WxMpTemplateData> data = Arrays.asList(
			new WxMpTemplateData("first", "微信支付凭证哈哈哈！"),
			new WxMpTemplateData("keyword1", "谢谢谢", "#173177"),
			new WxMpTemplateData("keyword2", "可口可乐"),
			new WxMpTemplateData("keyword3", "支付宝转账"),
			new WxMpTemplateData("keyword4", "$10000 欠款未付", "#CD2626"),
			new WxMpTemplateData("remark", "欢迎再次购买！")
	);

	templateMessage.setData(data);

	try {
		wxMpService.getTemplateMsgService().sendTemplateMsg(templateMessage);
	} catch (WxErrorException e) {
		log.error("微信模版消息发送失败, {}", e);
	}
}
```

用了SDK, 就非常简单吧! 效果如图:

![微信开发-支付效果](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E5%BE%AE%E4%BF%A1%E5%BC%80%E5%8F%91-%E6%94%AF%E4%BB%98%E6%95%88%E6%9E%9C.jpg)