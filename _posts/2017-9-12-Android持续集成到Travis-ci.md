---
layout: post
title: Android持续集成到Travis-ci
date: 2017-9-12
tags: Travis-ci
categories: post
lead: 持续集成
---

在github上的项目可以使用[Travis-ci](https://www.travis-ci.org/)进行持续集成。

在项目中部署一个文件，后缀名为``.travis.yml``

这里是官方的说明[**Building an Android Project**](https://docs.travis-ci.com/user/languages/android)

```yaml
android:
  components:
    - tools
    - platform-tools
    - tools

    # The BuildTools version used by your project
    - build-tools-25.0.0

    # The SDK version used to compile your project
    - android-25
```

在项目中添加了上面的文件后，只要点开运行，就可以看到了。

*问题*：

```shell
Travis-ci command "./gradlew build" exited with 1.
```

 我的 **script** 部分如下:

```
script:
- chmod +x ./gradlew
```

最终如下可以执行通过

```yaml
language: android
android:
components:
# Uncomment the lines below if you want to
# use the latest revision of Android SDK Tools
- platform-tools
- tools

# The BuildTools version used by your project
- build-tools-25.0.0

# The SDK version used to compile your project
- android-25


# Specify at least one system image,
# if you need to run emulator(s) during your tests
# - sys-img-armeabi-v7a-android-22
# - sys-img-armeabi-v7a-android-17

script:
- chmod +x ./gradlew

```

