---
layout: post
title: Android Studio-Gradle更新
date: 2017-2-17
tags: Gradle
categories: post
lead: adb命令失效
---

##### Android Studio-Gradle更新

今天遇到了Android studio 下的gradle 更新问题：

`Could not find com.android.tools.build:gradle`

所以查找了gradle在Android studio中的更新方式

```scala
project
  -app
  -gradle
  --wrapper
  ---gradle-wrapper.jar
  ---gradle-wrapper.properties
  -build.gradle
```

在主工程下的build.gradle中

```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.0-beta3'
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}

allprojects {
    repositories {
        jcenter()
    }
}
```

修改gradle版本为想要的版本号，[查找gradle的版本](http://services.gradle.org/distributions)

其中gradle-wrapper.jar ，是使用gradle-wrapper的一个辅助库

然后在**gradle-wrapper.properties**中修改

```groovy
distributionUrl=https\://services.gradle.org/distributions/gradle-3.3-all.zip
```

为对应的版本号

