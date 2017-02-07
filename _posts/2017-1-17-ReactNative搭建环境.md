---
layout: post
title: ReactNative搭建环境
date: 2017-1-17
tags: ReactNative
categories: post
lead: 搭建ReactNative开发环境。
---

windows中环境配置

1.cmd中使用管理员权限来运行powershell脚本。

2.Set-ExecutionPolicy RemoteSigned

3.iwr https://chocolatey.org/install.ps1 -UseBasicParsing | iex

4.安装好这个包管理工具。

5.react-native init ProjectName

6.cd ProjectName

7.react-native run-android

8.windows上面使用sublime text来进行开发

问题：

1.sublime text 出现了react代码反斜杠匹配的问题，

view，syntax-> open all with current extension as -> babel -> javascript(Babel)

