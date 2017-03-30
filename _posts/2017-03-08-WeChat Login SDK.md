---
layout: post
title: WeChat Login SDK
date: 2017-03-08 20:28 +0800
categories: android sdk login
---

#### 1.申请AppID
在[微信开放平台][wechat-console]创建应用

7个工作日内审核通过，第一次申请比较久，之后1～2天能申请好

填写项目信息时有一项需注意 <mark>应用签名</mark>, 此签名需下载微信的签名工具 [签名工具][wechat-signation]

###### 注：不同于微信的分享到朋友、分享到朋友圈，登录和支付功能需要申请开发者资质且需缴纳费用，目前是300元/年审核认证费。

#### 2.SDK 接入（AS）
- module下的build.gradle中添加：

```
dependencies {
    compile 'com.tencent.mm.opensdk:wechat-sdk-android-with-mta:1.0.2'
}
```
或

```
dependencies {
    compile 'com.tencent.mm.opensdk:wechat-sdk-android-without-mta:1.0.2'
}
```
(其中，前者包含统计功能)

- AndroidManifest.xml中添加如下权限

```
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

- 包名目录下建包<mark>wxapi</mark>，并在该包下新增WXEntryActivity类继承Actvity，并实现  接口用以接收分享回调信息。

- Android.xml中添加

```xml
<activity
            android:name=".wxapi.WXEntryActivity"
            android:exported="true"
            android:screenOrientation="portrait"
            android:theme="@android:style/Theme.Translucent.NoTitleBar" />
```

- 代码实现

类WXEntryActivity.java

```java
public class WXEntryActivity extends Activity implements IWXAPIEventHandler {

    public static final String APP_ID = "待应用审核通过填入生成的APP_ID";
    private IWXAPI iwxapi;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_wxentry);

        iwxapi = WXAPIFactory.createWXAPI(this, APP_ID, true);
        iwxapi.handleIntent(getIntent(), this);
    }

    @Override
    public void onReq(BaseReq baseReq) {
        Log.e("yc", "onReq:"+baseReq.toString());
    }

    @Override
    public void onResp(BaseResp baseResp) {
        Log.e("yc", "onResp:errCode:"+baseResp.errCode+" type:"+baseResp.getType()+" errStr:"+baseResp.errStr);
        Bundle bundle = new Bundle();
        switch (baseResp.errCode){
            case BaseResp.ErrCode.ERR_OK:
                if(baseResp.getType() == 1) {
                    //登录授权成功
                    Message msg = Message.obtain();
                    msg.obj = (SendAuth.Resp)baseResp; //包含access_token、openid等信息
                    WeChatLoginBiz.handler.sendMessage(msg);
                }else if(baseResp.getType() == 2) {
                    //分享成功
                    Toast.makeText(App.getContext(), "分享成功", Toast.LENGTH_SHORT).show();
                }
                break;
            case BaseResp.ErrCode.ERR_USER_CANCEL:
                if(baseResp.getType() == 1){
                    //登录用户未授权
                }else if(baseResp.getType() == 2) {
                    //分享取消
                    Toast.makeText(App.getContext(), "取消了分享", Toast.LENGTH_SHORT).show();
                }
                break;
            case BaseResp.ErrCode.ERR_AUTH_DENIED:
                //拒绝

                break;
        }
        finish();
    }
}
```

业务处理类WeChatShareBiz.java

```java
public class WeChatLoginBiz {

    private IWXAPI iwxapi;
    private static WeChatLoginListener listener;
    public static Handler handler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            SendAuth.Resp resp = (SendAuth.Resp) msg.obj;
            String code = resp.code;
            getWeChatToken(code);
        }
    };

    public WeChatLoginBiz(Activity context){
        iwxapi = WXAPIFactory.createWXAPI(context, WXEntryActivity.APP_ID);
        iwxapi.registerApp(WXEntryActivity.APP_ID);
    }

    public void login(){
        requestForCode();
    }

	//1.用于弹出授权登录页并获取返回码
    protected void requestForCode(){
        SendAuth.Req req = new SendAuth.Req();
        req.scope = "snsapi_userinfo"; //作用域：获取用户信息
        req.state = "xxx"; //授权后原样返回自己应用，用于安全考虑
        iwxapi.sendReq(req);
    }

	/** 2.获取token 
	* url https://api.weixin.qq.com/sns/oauth2/access_token?appid=%s&secret=%s&code=%s&grant_type=authorization_code
	* @param code 弹出授权登录页返回码
	*/
    private static void getWeChatToken(String code) {
        WebApis.getWCToken(code, new WebApis.BeanCallBack<WeChatToken>() {
            @Override
            public void onSucess(WeChatToken weChatToken) {
                String token = weChatToken.getAccess_token();
                String openId = weChatToken.getOpenid();
                getUserInfo(token, openId);
            }

            @Override
            public void onFail(int status, String message) {

            }
        });
    }

	/**3.获取用户信息
	* url: https://api.weixin.qq.com/sns/userinfo?access_token=%s&openid=%s
	* @param token 2中获取到的access_token
	* @param openid 授权用户唯一标识
	*/
    private static void getUserInfo(String token, String openId){
        WebApis.getWCProfile(token, openId, new WebApis.BeanCallBack<WeChatProfile>() {
            @Override
            public void onSucess(WeChatProfile weChatProfile) {
                //未找到微信退出登录方法，自己标记微信登录成功
                SPUtil.put(App.getContext(), Constant.SP_WECHAT_LOGIN_FLAG, true); 
                listener.weChatPLoginSuccess(weChatProfile);
            }

            @Override
            public void onFail(int status, String message) {

            }
        });
    }

    public void destory(){
        handler = null;
    }

    public void setLoginListener(WeChatLoginListener listener){
        this.listener = listener;
    }

    public interface WeChatLoginListener{
        void weChatPLoginSuccess(WeChatProfile profile);
    }

}
```

###### 注：刷新或续期access_token使用、检验授权凭证（access_token）是否 这两种感觉对于登录功能用不到，猜测和登录后续的相关操作有关，如支付。


#### 3.混淆

```
-keep class com.tencent.mm.opensdk.** {
   *;
}
-keep class com.tencent.wxop.** {
   *;
}
-keep class com.tencent.mm.sdk.** {
   *;
}
```

[wechat-console]: https://open.weixin.qq.com
[wechat-signation]: https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419319167