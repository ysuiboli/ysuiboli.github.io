---
layout: post
title: FB Login SDK
date: 2017-02-16 15:00 +0800
categories: android sdk login
---


[FB Login SDK][FB-Login-SDK] (需翻墙)

#### 1.注册开发者帐号

#### 2.创建应用

#### 3.接入SDK

- minSdkVersion : 15

- module的build.gradle中添加

```
repositories {
	mavenCentral()
}
```

- module的dependecies中添加：
```compile 'com.facebook.android:facebook-android-sdk:[4,5)' ```

- string.xml中：
```<string name="facebook_app_id">fb_app_id</string>```

- AndroidManifest.xml
添加网络访问权限：
```<uses-permission android:name="android.permission.INTERNET"/>```

- 添加meta-data:

```xml
<application android:label="@string/app_name" ...>
    ...
    <meta-data android:name="com.facebook.sdk.ApplicationId" android:value="@string/facebook_app_id"/>
    ...
</application>
```

#### 4.代码调用
##### FB登录代码调用分为两种：

其一，使用FB提供的登录按钮

```xml
	<com.facebook.login.widget.LoginButton
        android:id="@+id/login_fb"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
```

```java
	LoginButton loginFb;
    CallbackManager callbackManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);

        callbackManager = CallbackManager.Factory.create();
        loginButton.setReadPermissions("email");
		// If using in a fragment
		// loginButton.setFragment(this);    
		// Other app specific specialization
        loginFb.registerCallback(callbackManager, new FacebookCallback<LoginResult>() {
            @Override
            public void onSuccess(LoginResult loginResult) {
                // App code
            }

            @Override
            public void onCancel() {
                // App code
            }

            @Override
            public void onError(FacebookException error) {
                // App code
            }
        });
    }
    
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        callbackManager.onActivityResult(requestCode, resultCode, data);
    }  
```

其二，使用自定义的登录按钮
与第一种方式的区别：
注册回调时通过LoginManager而非LoginButton
```LoginManager.getInstance().registerCallback（..., ...）```
登录调用

```java
List<String> permissions;

permissions = Arrays.asList("email", "user_likes", "user_status", "user_photos", "user_birthday", "public_profile", "user_friends");

LoginManager.getInstance().logInWithReadPermissions(activity, permissions);

```

##### 检查登录状态

您的应用一次只能登录一个用户，LoginManager 会为该用户设置当前的 AccessToken 和 Profile。FacebookSDK 会将该数据保存在共享首选项中，并在 SDK 初始化过程中进行设置。 您可以通过检查 AccessToken.getCurrentAccessToken() 和 Profile.getCurrentProfile() 来查看用户是否已登录。

##### 获取用户资料
仅需简单的个人资料可通过Profile.getCurrentProfile()获得的，Profile包含字段：

- id
- firstName
- middleName
- lastName
- name
- linkUri (用户主页链接)

若需获取更详细的信息可通过以下方式查询获取：

```java
	GraphRequest request = GraphRequest.newMeRequest(accessToken, new GraphRequest.GraphJSONObjectCallback() {
            @Override
            public void onCompleted(JSONObject object, GraphResponse response) {
                Log.e("yc", "object:"+object.toString());
                if (object != null) {
                    String id = object.optString("id");   
                    String name = object.optString("name");  
                    String gender = object.optString("gender"); 
                    String emali = object.optString("email"); 
                    JSONObject object_pic = object.optJSONObject("picture");
                    JSONObject object_data = object_pic.optJSONObject("data");
                    String photo = object_data.optString("url");
                    String locale = object.optString("locale");   

                    FbProfile selfInfo = GsonT.parseJson(object.toString(), FbProfile.class);

                    if (fbLoginListener != null) {
                        fbLoginListener.accessProfile(selfInfo);
                    }
                }
            }
        });

        Bundle parameters = new Bundle();
        parameters.putString("fields", "id,name,link,gender,birthday,email,picture,locale,updated_time,timezone,age_range,first_name,last_name");
        request.setParameters(parameters);
        request.executeAsync();
```


###### 附：获取Hash值命令

```
keytool -exportcert -alias YOUR_RELEASE_KEY_ALIAS(别名) -keystore YOUR_RELEASE_KEY_PATH（key文件） | openssl sha1 -binary | openssl base64
```

[FB-Login-SDK]: https://developers.facebook.com/docs/facebook-login/android