---
layout: post
title: AndroidStudio中打包jar包
date: 2018-4-28 08:08:21
tags: Android
categories: post
lead: 
---

Android Studio中打包jar包的方法

Android studio中添加Moudle文件，Android类型，不是eclipse java类型。

build工程，那么就可以在下面的路径中找到jar包，然后

```
./mylibrary/build/intermediates/bundles/debug/classes.jar./mylibrary/build/intermediates/bundles/release/classes.jar
```

Moudle中的build.gradle加入task来实现

```groovy
task clearJar(type: Delete) {
    delete 'build/fcdev_v1.0.jar' //jar包的名字，随便命名
}
task makeJar(type:org.gradle.api.tasks.bundling.Jar) {
    //指定生成的jar名
    baseName 'fcdev_v1.0'
    //从哪里打包class文件
    from('build/intermediates/classes/release/')
    //打包到jar后的目录结构
    into('/')
    //去掉不需要打包的目录和文件
    exclude('test/', 'BuildConfig.class', 'R.class')
    //去掉R开头的文件
    exclude{it.name.startsWith('R');}
}
makeJar.dependsOn(clearJar, build)
```

右侧gradle 中打包 **makeJar** 功能双击,build中的 lib文件夹

