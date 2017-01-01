---
layout: post
title: "Unknow Host Exception"
data: 2017-01-01 13:46 +0800
categories: issue net android
---
## Issue
Wifi开关打开着同时数据网络开启着，没有Wi-Fi可以使用，使用数据网络请求Wi-Fi密码，在连上Wi-Fi后忘记密码重新使用数据网络请求便会失败。一直使用6.0的机器测试，jni报异常：Unable to resolve host "xxx.yyy.com": No address associated with hostname,由于jni的调试不方便，翻译成java后找问题捕获异常：Unknow Host Exception。

## Answer
No.1 网上查找需要添加网络权限：	

`<uses-permission android:name="android.permission.INTERNET" />`	

检查AndroidManifest.xml中的网络相关的权限是有添加的

```
<!-- 网络相关权限 -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_MULTICAST_STATE"/>
    <!-- 地理位置（6.0以上wifi必须） -->
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```

No.2 也曾考虑过是否和数据网络的代理有关，但是log打出来代理是关着的。

No.3 找到一个帖子里遇到的问题几乎一致(<http://bbs.csdn.net/topics/390687083>)，贴中内容说是因为服务器端防火墙的原因，将防火墙关闭就好了。咨询了公司服务器端的同事，同事也并很清楚这里说的防火墙指的是啥，如果是指端口那么常用的端口都是开着的。

No.4 无意中换了一个5.x系统的机器测试到捕获到的异常不一样了，socket failed: errno 64 (Machine is not on the network)，根据关键字找到(<http://stackoverflow.com/questions/38572938/android-network-not-reconnecting-when-wifi-drops>)，指代码中设置了通过指定网络作网络请求。

`connectivityManager.setProcessDefaultNetwork(net);`	

全局搜索发现代码中在监听网络状态的广播中确实有作这样的设置：
{% highlight java %}
private ConnectivityStatus getConnectivityStatus(final Context context) {
        final String service = Context.CONNECTIVITY_SERVICE;
        final ConnectivityManager manager = (ConnectivityManager) context.getSystemService(service);
        final NetworkInfo networkInfo = manager.getActiveNetworkInfo();

        if (networkInfo == null || !networkInfo.isAvailable()) {
            return ConnectivityStatus.OFFLINE;
        }

        if (networkInfo.getType() == ConnectivityManager.TYPE_WIFI) {

//            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
//                for (Network net : manager.getAllNetworks()) {
//                    // 可以通过下面代码将app接下来的请求都绑定到这个网络下请求
//                    if (Build.VERSION.SDK_INT >= 23) {
////                        manager.bindProcessToNetwork(net);
//                    } else if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
//                        // 23后这个方法舍弃了
////                        ConnectivityManager.setProcessDefaultNetwork(net);
//                    }
//                }
//            }
            return ConnectivityStatus.WIFI_CONNECTED;
        } else if (networkInfo.getType() == ConnectivityManager.TYPE_MOBILE) {
            return ConnectivityStatus.MOBILE_CONNECTED;
        }

        return ConnectivityStatus.OFFLINE;
    }
{% endhighlight %}  

这也解释得通为啥第一次使用数据网络请求是正常的，连接过一次Wi-Fi后就无法使用数据网络了。这段代码的添加已经不记得，查找提交记录发现是自己几个月前添加到，估计当时考虑到节省数据流量只通过网络访问。最后，将其中设定的只允许通过Wi-Fi访问的限制注释掉就OK了。