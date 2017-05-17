---
layout: post
title: Android AsyncTask线程池之坑
date: 2017-05-17 13:16
categories: android AsyncTask 线程池
---

## Issue

AsyncTask的子类，执行了execute()方法后，doInBackground方法内部一直未走进。

## AsyncTask

主要有二个部分：

* 一个是与主线各的交互
* 另一个就是线程的管理调度。

虽然可能多个AsyncTask的子类的实例，但是AsyncTask的内部Handler和ThreadPoolExecutor都是进程范围内共享的，其都是static的，也即属于类的，类的属性的作用范围是CLASSPATH，因为一个进程一个VM，所以是AsyncTask控制着进程范围内所有的子类实例。

##### 1.与主线程交互

与主线程交互是通过Handler来进行的，因为本文主要探讨AsyncTask在任务调度方面的，所以对于这部分不做细致介绍，感兴趣的朋友可以去看AsyncTask的源码

##### 2.线程任务的调度

内部会创建一个进程作用域的线程池来管理要运行的任务，也就就是说当你调用了AsyncTask#execute()后，AsyncTask会把任务交给线程池，由线程池来管理创建Thread和运行Therad。对于内部的线程池不同版本的Android的实现方式是不一样的：

#### Android 2.3（API 10）以前的版本

局限性：

内部线程池限制5个，即只允许5个线程(Thread)同时运行，超过的线程职能等待。

解法：

以前的版本无法解决，如有大量的线程需执行，只能放弃使用AsyncTask，自己创建线程池管理Thread，或直接使用Thread不去管理也无妨。若使用AsyncTask，需错开使用AsyncTask的时间，尽力做到分时或保证数量不会大于5个。


#### Android 3.0 （API11）以后的版本

可能是Google意识到了AsyncTask的局限性了，从Android 3.0开始对AsyncTask的API做出了一些调整：

##### 1.#execute()提交的任务，按先后顺序每次只运行一个

即按提交的顺序同步执行，即相当于只有一个线程执行所提交的任务( Executors.newSingleThreadPool() )

##### 2.新增接口#executeOnExecutor()

该接口允许开发者提供自定义的线程池来运行和调度Thread，若想让所有的任务都能并发同时运行，可创建一个没有限制的线程池( Executors.newCachedThreadPool() )，提供给AsyncTask。这样这个AsyncTask实例就有了自己的线程池，而不必使用AsyncTask默认的。

##### 3.新增了2个预定义的线程池SERIAL_EXECUTOR和THREAD_POOL_EXECUTOR

其实THREAD_POOL_EXECUTOR并不是新增的，之前的就有，只不过之前(Android 2.3)它是AsyncTask私有的，未公开而已。THREAD_POOL_EXECUTOR是一个corePoolSize为5的线程池，也就是说最多只有5个线程同时运行，超过5个的就要等待。所以如果使用executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR)就跟2.3版本的AsyncTask.execute()效果是一样的。

而SERIAL_EXECUTOR是新增的，它的作用是保证任务执行的顺序，也就是它可以保证提交的任务确实是按照先后顺序执行的。它的内部有一个队列用来保存所提交的任务，保证当前只运行一个，这样就可以保证任务是完全按照顺序执行的，默认的execute()使用的就是这个，也就是executeOnExecutor(AsyncTask.SERIAL_EXECUTOR)与execute()是一样的。

## 使用心得：
继承AsyncTask的子类也会受到上述影响，除非不在乎时效性或者线程很少且均不会耗时很久的情况下可以使用execute()，项目中尽量使用executeOnExecutor()去提交任务执行，否则容易发生死等情况。

因为execute()提交的任务，是交给一个线程按先后顺序的，executeOnExecutor()可以使用系统提供的线程池AsyncTask.THREAD_POOL_EXECUTOR，也可以自定义。


[参考内容](https://zhidao.baidu.com/question/1671553161701081707.html)
