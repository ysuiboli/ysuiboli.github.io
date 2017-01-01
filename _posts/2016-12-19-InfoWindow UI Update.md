---
layout: post
title: "InfoWindow UI Update"
date: 2016-12-19 20:59 +0800
categories: issue map android
---
## Issue
Google map上进行二次开发，点击地图上的marker显示InfoWindow，InfoWindow中需要根据经纬度请求地理位置信息展示出来，经纬度转地理信息（逆地理编码）可以根据Google提供的Api:
`http://maps.google.com/maps/api/geocode/json?latlng=31.0,121.0&language=zh-CN&sensor=false`获取到。

接下来问题来了，当获取到地理位置信息在回调中更新UI时发现怎么也更新不了。

## Answer
No.1尝试，首先考虑是否在主线程中，使用了runOnUiThread方法并且打印当前线程确保当前在UI线程中
{% highlight android %}
TextView infoWindowLocation;

getLocation(lat, lng, new CallBack() {
                @Override
                public void CallBack(int status, String message, final GoogleLocation googleLocation) {
activity.runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                        
Log.e(TAG, "thread:"+Thread.currentThread().toString());  
                          infoWindowLocation.setText("lalala");
                            
                        }
                    });
                    });
                }
            });
{% endhighlight %}                    

试了下UI没有刷，打印`infoWindowLocation.getText()`发现值是有更新的。

No.2尝试，接下来调用`infoWindowLocation.invalidate();`,`infoWindowLocation.requestLayout();`方法，均不起作用。

No.3尝试，查到`invalidate()`方法在4.x某个版本后默认开启硬件加速中不起作用，于是关闭掉硬件加速，UI依旧没有更新。

No.4尝试，在同事的提醒下，会不会和Google的刷新有关，于是换个方向在网上搜索，
`http://stackoverflow.com/questions/26383464/how-can-i-update-contents-into-infowindowadapter`

`http://stackoverflow.com/questions/26383464/how-can-i-update-contents-into-infowindowadapter`

找到与所遇问题相符的第一个链接，在第一个链接中找到了第二个链接并发现`marker.showInfoWindow();`方法，添加在上述代码片段中`infoWindowLocation.setText("lalala");`后面，UI更新了！