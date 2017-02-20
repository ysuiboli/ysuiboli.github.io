---
layout: post
title: QQ Login SDK
date: 2017-02-17 15:59 +0800
categories: android sdk login qq
---

#### 1.注册开发者帐号
在[腾讯开发平台][tencent-open-platform]注册开发者帐号，并且创建项目获取APP_ID

#### 2.SDK接入
[SDK下载][qq-login-sdk]

[API文档][qq-login-sdk-api]

- libs下面放入jar包

```
mta-sdk-1.6.2.jar
open_sdk_r5781.jar
```

- module的build.gradle中添加

```
compile files('libs/mta-sdk-1.6.2.jar')
compile files('libs/open_sdk_r5781.jar')
```

- AndroidManifest中添加

```xml
	<activity
            android:name="com.tencent.tauth.AuthActivity"
            android:noHistory="true"
            android:launchMode="singleTask" >
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data android:scheme="tencent你的AppId" />
            </intent-filter>
        </activity>
        <activity android:name="com.tencent.connect.common.AssistActivity"
            android:theme="@android:style/Theme.Translucent.NoTitleBar"
            android:configChanges="orientation|keyboardHidden|screenSize"
            />
```

###### 注意：其中```<data android:scheme="tencent你的AppId" />```scheme是<mark>tencent拼接上APP_ID(创建项目的)</mark>

- 登录/注销、获取用户信息

```java
public class LoginActivity extends AppCompatActivity implements IUiListener 
	private Tencent mTencent;
    private static final String QQ_APP_ID = "110xxxx970";
    private static final String scope = "get_user_info"; //作用域 all:所有的权限 get_user_info:获取用户信息
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
        
        //创建实例
        mTencent = Tencent.createInstance(QQ_APP_ID, getApplicationContext());
    }
    
    //登录
    private void login(){
    	if (!mTencent.isSessionValid()){
       		mTencent.login(this, scope, this);
       	}
    }
    
    //注销
    private void loginOut(){
    	mTencent.logout(this);
    }
    
    //获取用户信息
    private void getUserInfo(){
        QQToken qqToken = mTencent.getQQToken();
        UserInfo userInfo = new UserInfo(getApplicationContext(), qqToken);
        userInfo.getUserInfo(new IUiListener() {
            @Override
            public void onComplete(Object o) {
                Log.e("yc", "qq:userInfo:"+o.toString());
                
            }

            @Override
            public void onError(UiError uiError) {}

            @Override
            public void onCancel() {}
        });

    }

	@Override
    public void onComplete(Object o) {
        Log.e("yc", "qq:"+o.toString());
        int ret = -1;
        try {
            JSONObject jsonObject = new JSONObject(o.toString());
            String accessToken = jsonObject.getString("access_token");
            String openId = jsonObject.getString("openid");
            String expire = ""+ jsonObject.getLong("expires_in");
            ret = jsonObject.getInt("ret");
            
            mTencent.setAccessToken(accessToken, expire);
            mTencent.setOpenId(openId);
        } catch (JSONException e) {
            e.printStackTrace();
            Log.e("yc", "qq:exception");
        }
        //登录成功
        if(ret == 0) {
            getUserInfo();
        }
    }

    @Override
    public void onError(UiError uiError) {}

    @Override
    public void onCancel() {}
    
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        
        Tencent.onActivityResultData(requestCode, resultCode, data, this);
    }
｝
```

- 响应码

响应码（ret） | 响应码含义
--- | ---
0 | 成功
100030 | 没有权限

[更多响应码说明][more-resp-code-define]

- 混淆

```
-keep class com.tencent.open.TDialog$*
-keep class com.tencent.open.TDialog$* {*;}
-keep class com.tencent.open.PKDialog
-keep class com.tencent.open.PKDialog {*;}
-keep class com.tencent.open.PKDialog$*
-keep class com.tencent.open.PKDialog$* {*;}
```


[tencent-open-platform]: http://op.open.qq.com
[qq-login-sdk]: http://wiki.open.qq.com/wiki/mobile/SDK下载
[qq-login-sdk-api]: http://wiki.open.qq.com/wiki/创建并配置工程
[more-resp-code-define]: http://wiki.open.qq.com/wiki/mobile/公共返回码说明