---
layout: post
title: RxJava2与retrofit2使用
date: 2017-1-21
tags: Android
categories: post
lead: 开始学习RxJava2与retrofit2的使用
---

**Observable**：发射源，英文释义“可观察的”，在观察者模式中称为“被观察者”或“可观察对象”；

**Observer**：接收源，英文释义“观察者”，没错！就是观察者模式中的“观察者”，可接收Observable、Subject发射的数据；

**Subject**：Subject是一个比较特殊的对象，既可充当发射源，也可充当接收源，为避免初学者被混淆，本章将不对Subject做过多的解释和使用，重点放在Observable和Observer上，先把最基本方法的使用学会，后面再学其他的都不是什么问题；

**Subscriber**：“订阅者”，也是接收源，那它跟Observer有什么区别呢？Subscriber实现了Observer接口，比Observer多了一个最重要的方法`unsubscribe( )`，用来取消订阅，当你不再想接收数据了，可以调用`unsubscribe( )`方法停止接收，Observer 在 `subscribe()` 过程中,最终也会被转换成 Subscriber 对象，一般情况下，建议使用Subscriber作为接收源；

**Subscription **：Observable调用`subscribe( )`方法返回的对象，同样有`unsubscribe( )`方法，可以用来取消订阅事件；

**Action0**：RxJava中的一个接口，它只有一个无参call（）方法，且无返回值，同样还有Action1，Action2...Action9等，Action1封装了含有* 1 *个参的call（）方法，即call（T t），Action2封装了含有* 2 *个参数的call方法，即call（T1 t1，T2 t2），以此类推；

**Func0**：与Action0非常相似，也有call（）方法，但是它是有返回值的，同样也有Func0、Func1...Func9;

[Retrofit2的API](http://square.github.io/retrofit/)

**首先创建一个接口**

主要是用于数据接口创建，类似于下面的get请求。

``https://www.okcoin.cn/api/v1/ticker.do?symbol=ltc_cny``

```java
public interface APISevice {

  @GET("api/v1/ticker.do") Observable<BitDataBean> getBitData(@Query("symbol") String symbol);
}
```

**然后retrofit2的对象实例创建和使用**

```java
Retrofit retrofit = new Retrofit.Builder().client(new OkHttpClient())
        .baseUrl(Constant.url)
        .addConverterFactory(GsonConverterFactory.create())
        .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
        .build();
    APISevice service = retrofit.create(APISevice.class);
```

**然后进行rxjava2的数据请求**

```java
service.getBitData("ltc_cny")
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Observer<BitDataBean>() {
          @Override public void onSubscribe(@NonNull Disposable d) {
            Log.e(TAG, "onSubscribe: ");
          }

          @Override public void onNext(@NonNull BitDataBean bitDataBeen) {
            Log.e(TAG, "onNext: " + bitDataBeen.getTicker().getLast());
          }

          @Override public void onError(@NonNull Throwable e) {

          }

          @Override public void onComplete() {

          }
        });
```

