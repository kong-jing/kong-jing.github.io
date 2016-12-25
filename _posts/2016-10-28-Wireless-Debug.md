---
layout: post
title: Wireless Debug
date: 2016-10-28 16:14:21
tags: Android
categories: post
lead: google Wireless Debug
---

adb is usually used over USB. However, it is also possible to use over Wi-Fi, as described here.[使用WiFi来调试APK](https://www.google.co.uk/url?sa=t&rct=j&q=&esrc=s&source=web&cd=4&cad=rja&uact=8&ved=0ahUKEwiMgdGLx_zPAhVEl5QKHQ15DYUQFgg6MAM&url=https%3A%2F%2Fdeveloper.android.com%2Fstudio%2Fcommand-line%2Fadb.html&usg=AFQjCNEPxnaeOPDPEziHgSJ4_8uMoRcfCQ&sig2=bubcKLUYeKkn3Df9l4ZGXg)
<!--more-->
1. Connect your Android device and adb host computer to a common Wi-Fi network accessible to both. We have found that not all access points are suitable; you may need to use an access point whose firewall is configured properly to support adb.

   **Note: **If you are attempting to connect to a Wear device, force it to connect to Wi-Fi by shutting off Bluetooth on the phone connected to it.

2. Connect the device to the host computer with a USB cable.

3. Set the target device to listen for a TCP/IP connection on port 5555.

   ```shell
   $ adb tcpip 5555
   ```

4. Disconnect the USB cable from the target device.

5. Find the IP address of the Android device. For example, on a Nexus device, you can find the IP address at **Settings** > **About tablet** (or **About phone**) >**Status** > **IP address**. Or, on an Android Wear device, you can find the IP address at **Settings** > **Wi-Fi Settings** > **Advanced** > **IP address**.

6. Connect to the device, identifying it by IP address.

   ```shell
   $ adb connect <device-ip-address>
   ```

7. Confirm that your host computer is connected to the target device:

   ```shell
   $ adb devices
   List of devices attached
   <device-ip-address>:5555 device
   ```

You're now good to go!

If the adb connection is ever lost:

1. Make sure that your host is still connected to the same Wi-Fi network your Android device is.

2. Reconnect by executing the "adb connect" step again.

3. Or if that doesn't work, reset your adb host:

   ```shell
   adb kill-server
   ```

   and then start over from the beginning.
