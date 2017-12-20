---
layout: post
title: Sina Weibo SDK
date: 2017-12-20 18:05
categories: android login weibo
---

## Sina Weibo SDK

[官网][office_link]

### SDK功能特性（3.0）

#### 1.关系升级

可以获取同样使用该应用的好友，可额外获取30%的关系链（未使用新版SDK3.0的应用，只能获取同授权该应用的好友关系，这点与facebook和twitter的授权机制相同）

![](https://github.com/ysuiboli/ysuiboli.github.io/blob/master/css/pics/sina_weibo/weibo_img1.jpg)

#### 2.授权升级

手机中安装了新浪微博使用客户端登录，仅一步授权页面，中间没有加载过程；没有安装的，使用H5页授权。

#### 3.SSO授权

#### 4.下载推荐

#### 5.微博原生分享

提供原生、H5、API分享方式可供选择

#### 6.LinkCard解析

什么是LinkCard：在微博消息流内,分享一条链接,该链接将解析为包含一个对象数据的特殊短链,且该对象数据可以在微博消息流内显示并交互,这种形态就是微博消息流LinkCard解析。

#### 7.社会化组件

微博关注和评论功能

#### 8.短信注册服务

#### 9.特殊接口权限

* 用户信息
* 用户、内容关系
* 用户内容
* 授权时效

### SDK接入流程

![](https://github.com/ysuiboli/ysuiboli.github.io/blob/master/css/pics/sina_weibo/weibo_img2.jpg)


### 网站配置

#### 1.填写基本信息后，并配置授权回调链接后，可以使用授权登录

![](https://github.com/ysuiboli/ysuiboli.github.io/blob/master/css/pics/sina_weibo/weibo_web_config1.png)

![](https://github.com/ysuiboli/ysuiboli.github.io/blob/master/css/pics/sina_weibo/weibo_web_config2.png)

#### 2.添加测试账户后，可正常调用Sina Weibo API

![](https://github.com/ysuiboli/ysuiboli.github.io/blob/master/css/pics/sina_weibo/weibo_web_config3.png)

#### 3.完善图1中基本信息，app上国内主流应用市场，通过审核后，非测试用户也可以正常使用Sina Weibo API功能。

#### 4.调用API错误码，参考链接[Error Code][Error_Code]

### 代码集成

#### Step1.

build.gradle(project)中加入：

```gradle
maven { url "https://dl.bintray.com/thelasterstar/maven/" }
```

#### Step2.

build.gradle(module)中加入：

```gradle
compile 'com.sina.weibo.sdk:core:4.1.4:openDefaultRelease@aar'
```

#### Step3.

授权前先调用：

```java
WbSdk.install(App.getContext(), new AuthInfo(App.getContext(), Constants.WB_APP_KEY, Constants.WB_REDIRECT_URL, Constants.WB_SCOPE));
```

3个键值定义：

```java
public static final String WB_APP_KEY = "xxxxx";
    /**
     * 当前 DEMO 应用的回调页，第三方应用可以使用自己的回调页。
     *
     * <p>
     * 注：关于授权回调页对移动客户端应用来说对用户是不可见的，所以定义为何种形式都将不影响，
     * 但是没有定义将无法使用 SDK 认证登录。
     * 建议使用默认回调页：https://api.weibo.com/oauth2/default.html
     * </p>
     */
    public static final String WB_REDIRECT_URL = "http://www.xxxxx.net";
    /**
     * Scope 是 OAuth2.0 授权机制中 authorize 接口的一个参数。通过 Scope，平台将开放更多的微博
     * 核心功能给开发者，同时也加强用户隐私保护，提升了用户体验，用户在新 OAuth2.0 授权页中有权利
     * 选择赋予应用的功能。
     *
     * 我们通过新浪微博开放平台-->管理中心-->我的应用-->接口管理处，能看到我们目前已有哪些接口的
     * 使用权限，高级权限需要进行申请。
     *
     * 目前 Scope 支持传入多个 Scope 权限，用逗号分隔。
     *
     * 有关哪些 OpenAPI 需要权限申请，请查看：http://open.weibo.com/wiki/%E5%BE%AE%E5%8D%9AAPI
     * 关于 Scope 概念及注意事项，请查看：http://open.weibo.com/wiki/Scope
     */
//    public static final String WB_SCOPE = "email" +
//            ",direct_messages_read" +
//            ",direct_messages_write" +
//            ",friendships_groups_read" +
//            ",friendships_groups_write" +
//            ",statuses_to_me_read," +
//            "follow_app_official_microblog" +
//            ",invitation_write";
    public static final String WB_SCOPE = "all";

```

### 登录

#### 登录代码接入

```java
/** 封装了 "access_token"，"expires_in"，"refresh_token"，并提供了他们的管理功能  */
private Oauth2AccessToken mAccessToken;
/** 注意：SsoHandler 仅当 SDK 支持 SSO 时有效 */
private SsoHandler mSsoHandler;

//登录代码
mSsoHandler = new SsoHandler(activity);
mSsoHandler.authorize(new SelfWbAuthLister());

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data){
    super.onActivityResult(requestCode, resultCode, data);
    if(mSsoHandler != null){
        mSsoHandler.authorizeCallBack(requestCode, resultCode, data);
    }
}

//授权回调代码
private class SelfWbAuthLister implements WbAuthListener {

    @Override
    public void onSuccess(Oauth2AccessToken oauth2AccessToken) {
    	//授权成功
        if(activity == null) return;
        activity.runOnUiThread(() -> {
            mAccessToken = oauth2AccessToken;
            if(mAccessToken.isSessionValid()){
                //保存SP到token
                AccessTokenKeeper.writeAccessToken(App.getContext(), mAccessToken);
            }
        });
    }

    @Override
    public void cancel() {
        //取消授权
    }

    @Override
    public void onFailure(WbConnectErrorMessage wbConnectErrorMessage) {
    	//授权失败
    }
}

//退出登录
AccessTokenKeeper.clear(App.getContext());

```

#### API接口调用（获取个人资料、…)

参考链接[微博API][weibo_api]




[office_link]:http://open.weibo.com/wiki/移动客户端接入
[Error_Code]:http://open.weibo.com/wiki/Error_code
[weibo_api]:http://open.weibo.com/wiki/微博API
