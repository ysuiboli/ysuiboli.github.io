---
layout: post
title: WeChat Share SDK
date: 2017-02-24 16:56 +0800
categories: android sdk share wechat
---

[WeChat Share SDK][WeChat-Share-SDK]

#### 1.注册开发者帐号

#### 2.创建应用
[创建应用][WeChat-Console] 申请APP_ID

下载[签名工具][WeChat-Sign-Tool] 应用信息资料里填些签名。

###### 签名工具是一个apk，下载后安装到手机，根据提示填写包名生成签名。

#### 3.接入SDK

- module下build.gradle添加：
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

（其中前者包含统计功能）

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

    public static final String APP_ID = "待应用审核通过填入";
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
                //分享成功
                Toast.makeText(App.getContext(), "分享成功", Toast.LENGTH_SHORT).show();
                finish();
                break;
            case BaseResp.ErrCode.ERR_USER_CANCEL:
                //分享取消
                Toast.makeText(App.getContext(), "取消了分享", Toast.LENGTH_SHORT).show();
                break;
            case BaseResp.ErrCode.ERR_AUTH_DENIED:
                //分享拒绝

                break;
        }
    }
}
```

业务处理类WeChatShareBiz.java

```java
public class WeChatShareBiz {

    private IWXAPI iwxapi;

    public WeChatShareBiz(Context context){
        iwxapi = WXAPIFactory.createWXAPI(context, WXEntryActivity.APP_ID);
        iwxapi.registerApp(WXEntryActivity.APP_ID);
    }

	//分享给朋友
    public void shareToFriend(){
        WXWebpageObject webpage = new WXWebpageObject();
        webpage.webpageUrl = "http://www.baidu.com";

        WXMediaMessage msg = new WXMediaMessage(webpage);
        msg.title = "Title";
        msg.description = "Content";
        BitmapDrawable bmpDraw = (BitmapDrawable) App.getContext().getResources().getDrawable(
                R.mipmap.ic_launcher);
        Bitmap thumb = bmpDraw.getBitmap();
        msg.setThumbImage(thumb);

        SendMessageToWX.Req req = new SendMessageToWX.Req();
        req.transaction = String.valueOf(System.currentTimeMillis());
        req.message = msg;
        req.scene = SendMessageToWX.Req.WXSceneSession;
        iwxapi.sendReq(req);
    }

	//分享到朋友圈
    public void shareToCircle(){
        WXWebpageObject webpage = new WXWebpageObject();
        webpage.webpageUrl = "http://www.baidu.com";

        WXMediaMessage msg = new WXMediaMessage(webpage);
        msg.title = "Title";
        msg.description = "Content";
        BitmapDrawable bmpDraw = (BitmapDrawable) App.getContext().getResources().getDrawable(
                R.mipmap.ic_launcher);
        Bitmap thumb = bmpDraw.getBitmap();
        msg.setThumbImage(thumb);

        SendMessageToWX.Req req = new SendMessageToWX.Req();
        req.transaction = String.valueOf(System.currentTimeMillis());
        req.message = msg;
        req.scene = SendMessageToWX.Req.WXSceneTimeline;
        iwxapi.sendReq(req);
    }

}
```

###### 注意：请求scene的区别，分享给朋友：SendMessageToWX.Req.WXSceneSession，分享到朋友圈：SendMessageToWX.Req.WXSceneTimeline

- 混淆

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




[WeChat-Share-SDK]: https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=1417751808
[WeChat-Console]: https://open.weixin.qq.com/cgi-bin/applist?t=manage/list&lang=zh_CN&token=2391700e28d06f19608b7bfc30e7d92bf0926cb1
[WeChat-Sign-Tool]: https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419319167