---
layout: post
title: Nexus5x刷机到Android6.0.1
date: 2018-5-15 08:08:21
tags: Android
categories: post
lead: 
---

最近想要把自己的谷歌亲儿子从 Android8.1 降级到 Android6.0.1。

首先到 [刷机精灵去下载](http://www.shuame.com/faq/restore-tutorial/15311-google-nexus-5x--60-.html) Android6.0.1的固件。

然后需要将 Nexus5x 的设备进行 **解锁bootloader**

1. 首先肯定是要安装好adb和fastboot驱动。[platform-tools](http://www.androiddevtools.cn/#row)

2. 开启开发者选项，然后勾选开启usb调试。您的手机或者平板还会弹出类似是否允许usb调试的窗口，选择yes。

3. 开发者选项里开启“oem解锁”

4. 通过数据线链接设备和pc，然后在命令提示符输入`adb reboot bootloader`

5. 设备进入bootloader模式后，在命令提示符输入`fastboot oem unlock`,如果`fastboot oem unlock`没用的话，请使用`fastboot flashing unlock`

6. 这时会弹出相应的界面（英文的），问您是否解锁，用音量键加来选择yes，即可开始解锁。

   ![](http://www.inexus.co/data/attachment/portal/201501/02/191451qkjrjwe1jbqtr0x9.jpg)

7. 您的设备将解锁，过程会很短，然后当界面显示有start的时候，选择start，即可重启进入系统，完成解锁

8. 接下来重新进入 bootloader，然后在pc上双击`双击运行一键刷机`

   ​

   ​

