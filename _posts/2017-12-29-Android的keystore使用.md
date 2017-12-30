---
layout: post
title: Android的keystore使用
date: 2017-12-29 16:14:21
tags: Android
categories: post
lead: 
---

android 查看 keystore信息

在terminal中执行下面操作

```shell
keytool -list -v -keystore *****.jks
```

输入后再输入密码则

可以查看到相关信息

遇到 *cannot load keystore* 这个问题。可以使用这个查看**alias**. 