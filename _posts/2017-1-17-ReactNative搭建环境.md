---
layout: post
title: ReactNative搭建环境
date: 2017-1-17
tags: ReactNative
categories: post
lead: 搭建ReactNative开发环境。
---

windows中环境配置android 版本的react-native开发环境。请在执行下面的操作前配置好Android的开发环境.

1.`cmd中使用管理员权限来运行powershell脚本。`

```shell
powershell
```

###### powershell策略不允许执行脚本->

```powershell
get-executionpolicy
```

```powershell
set-executionpolicy remotesigned
```

然后选择确认，执行策略修改。

2.在powershell中执行下面的语句,[安装Chocolatey]()https://chocolatey.org/install

```sh
Set-ExecutionPolicy RemoteSigned
```

3.安装一个包管理工具

```powershell
iwr https://chocolatey.org/install.ps1 -UseBasicParsing | iex
```

4.安装好这个包管理工具。那么就可以安装Node.js了。(使用administrator权限启动命令行工具)

```sh
choco install nodejs.install
choco install python2
```

重新启动命令行工具就可以执行npm工具命令行了

```sh
npm install -g react-native-cli
```

然后需要设置npm环境。`C:\Users\admin\AppData\Roaming\npm`，在环境设置中path设置。

5.在powershell中执行下面的reactnative命令

```power
react-native init ProjectName
```

```powershell
cd ProjectName
```

6.运行Android版本的react-native

```powershell
react-native run-android
```

7.windows上面使用sublime text来进行开发

package control:Install Packages

安装插件:babel,react-native-snippets

#### 问题：

##### 1.[sublime text 出现了react代码反斜杠匹配的问题](http://stackoverflow.com/questions/35097267/what-ide-is-recommended-for-react-native/41712616#41712616)

view，syntax-> open all with current extension as -> babel -> javascript(Babel)

