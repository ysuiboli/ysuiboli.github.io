---
layout: post
title: Android获取本机WiFi MAC Address之坑
date: 2017-05-05 14:59
categories: android wifi mac
---

#### Issue

360的手机（android 6.0）测试发现获取自己的mac地址是02:00:00:00:00:00

#### Answer

Android M版本之后，通过wifiInfo.getMacAddress()获取的MAC地址是一个固定的假地址，值为02:00:00:00:00:00。


##### 1.WifiInfo 获取Mac地址方式

```java
public static void queryMyDevice(Context context) {
        WifiManager wm = (WifiManager) context.getSystemService(Context.WIFI_SERVICE);
        //检查Wifi状态
        if (!wm.isWifiEnabled()) return null;
        WifiInfo wi = wm.getConnectionInfo();
        //获取32位整型IP地址
        int ipAdd = wi.getIpAddress();
        //把整型地址转换成“*.*.*.*”地址
        String ip = intToIp(ipAdd);
        String mac = wi.getMacAddress();
        
        //结合下述第二种获取方式
        //if(TextUtils.equals(mac, "02:00:00:00:00:00")){
        //    String temp = getMacAddress();
        //    if(!TextUtils.isEmpty(temp)){
        //        mac = temp;
        //    }
        }
        
    }

    private static String intToIp(int i) {
        return (i & 0xFF ) + "." +
                ((i >> 8 ) & 0xFF) + "." +
                ((i >> 16 ) & 0xFF) + "." +
                ( i >> 24 & 0xFF) ;
    }

```

##### 2.网络设备信息获取Mac地址方式

```java
private static String getMacAddress(){
        String macAddress = "";
        try {
            Enumeration<NetworkInterface> interfaces = NetworkInterface.getNetworkInterfaces();
            while (interfaces.hasMoreElements()) {
                NetworkInterface iF = interfaces.nextElement();

                byte[] addr = iF.getHardwareAddress();
                if (addr == null || addr.length == 0) {
                    continue;
                }

                StringBuilder buf = new StringBuilder();
                for (byte b : addr) {
                    buf.append(String.format("%02X:", b));
                }
                if (buf.length() > 0) {
                    buf.deleteCharAt(buf.length() - 1);
                }
                String mac = buf.toString();
                Log.d("mac", "interfaceName="+iF.getName()+", mac="+mac);

                if(TextUtils.equals(iF.getName(), "wlan0")){
                    return mac;
                }
            }
        } catch (SocketException e) {
            e.printStackTrace();
            return macAddress;
        }

        return macAddress;
    }
```

###### 小结：先通过WifiInfo方式获取，判断当获取到的MAC地址值为02:00:00:00:00:00时，再通过第二种方式获取（见第一段代码注释处）。

[参考内容](http://www.tuicool.com/articles/ameQJfN)