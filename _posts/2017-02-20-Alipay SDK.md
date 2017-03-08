---
layout: post
title: Alipay SDK
date: 2017-02-20 14:36 +0800
categories: android sdk payment ali
---

#### 1.注册开发者帐号
可用手机号注册或淘宝帐号登录，创建应用、配置应用

#### 2.SDK接入
[SDK文档][Alipay-sdk]

若app仅用于测试且未上线前需使用支付宝提供的***沙箱环境***，扫码下载对应的[沙箱版钱包][Alipay-sandbox]

<<<<<<< HEAD
![沙箱版钱包二维码](./../css/pics/alipay/Alipay_sandbox.png)
=======
![沙箱版钱包二维码](/css/pics/alipay/Alipay_sandbox.png)
>>>>>>> 21f528e0f79137b1ab0d93dd6862b6a7b6b587d0

[签名工具下载][Alipay-Sign-Tool]

[Alipay Demo][Alipay-Demo]（其中包含下文所需jar）<a name="link_alipay_demo"></a>

- libs添加jar

```alipaySdk-20161222.jar```

- Mainifest.xml中添加权限和注册信息

```xml
<uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

```xml
<activity
            android:name="com.alipay.sdk.app.H5PayActivity"
            android:configChanges="orientation|keyboardHidden|navigation"
            android:exported="false"
            android:screenOrientation="behind" >
</activity>
<activity
            android:name="com.alipay.sdk.auth.AuthActivity"
            android:configChanges="orientation|keyboardHidden|navigation"
            android:exported="false"
            android:screenOrientation="behind" >
 </activity>
```

- java代码

见[Alipay Demo](#link_alipay_demo)。

需注意的是，在调用请求支付接口前需要添加以下代码进入沙箱测试环境：
```EnvUtils.setEnv(EnvUtils.EnvEnum.SANDBOX);```
否则会弹窗提示  <mark>系统繁忙，请稍后重试。(ALI40247)</mark>  (签名错误)

沙箱环境下的开发，有一个买家帐号和卖家帐号，均是自动生成以供测试。demo中的APPID应填写沙箱环境的APPID，而非创建应用的APPID（上线包用此）

- 混淆

```
-libraryjars libs/alipaySDK-20150602.jar
 
-keep class com.alipay.android.app.IAlixPay{*;}
-keep class com.alipay.android.app.IAlixPay$Stub{*;}
-keep class com.alipay.android.app.IRemoteServiceCallback{*;}
-keep class com.alipay.android.app.IRemoteServiceCallback$Stub{*;}
-keep class com.alipay.sdk.app.PayTask{ public *;}
-keep class com.alipay.sdk.app.AuthTask{ public *;}
```

#### 3.计费模式
费率按单笔计算；

一般行业费率：0.6%；特殊行业费率：1.2%

###### 特殊行业范围包括：手机、通讯设备销售；家用电器；数码产品及配件；休闲游戏；网络游戏点卡、渠道代理；游戏系统商；网游周边服务、交易平台；网游运营商（含网页游戏）。

[Alipay-sdk]: https://doc.open.alipay.com/docs/doc.htm?spm=a219a.7629140.0.0.YqhUS0&treeId=204&articleId=105051&docType=1
[Alipay-sandbox]: https://openhome.alipay.com/platform/appDaily.htm?tab=tool
[Alipay-Demo]: https://doc.open.alipay.com/doc2/detail.htm?treeId=54&articleId=104509&docType=1
[Alipay-Sign-Tool]: https://doc.open.alipay.com/docs/doc.htm?treeId=291&articleId=105971&docType=1
