---
layout: post
title: Modbus协议设备以太网socket通信实践
date: 2017-08-8 15:50:00
tags: IoT
categories: post
---



最近使用到了Modbus的协议的485转接器，在使用过程中，需要对以太网的I/O设备进行do1状态的改变，使设备产生高低电的变化。

首先需要使用Android设备与设置好了IP地址的485转接器socket服务器建立socket连接。

```java
new Thread(new Runnable() {
      @Override public void run() {
        try {
          socket = new Socket("192.168.40.144", 502);
          Log.e(TAG, "建立485转接器首次连接：" + socket);
        } catch (UnknownHostException e) {
          e.printStackTrace();
        } catch (IOException e) {
          e.printStackTrace();
        }
      }
    }).start();
```

然后在建立成功的连接后，可以发送一个关闭do1的socket，然后再发送一个打开do1的socket。然后就能够产生一个开关的作用

```java
new Thread() {
      @Override public void run() {
        try {
          byte[] sendInfoClose = new byte[] {
              0x00, 0x06, 0x00, 0x00, 0x00, 0x06, (byte) 0xFF, 0x05, 0x00, 0x64, (byte) 0xFF, 0x00
          };//发送给Modbus 协议的设备的消息，用于关闭do1状态灯
          DataOutputStream writer = new DataOutputStream(socket.getOutputStream());
          writer.write(sendInfoClose);
          Log.e(TAG, "run: 发送关闭状态灯信息");
        } catch (IOException e) {
          e.printStackTrace();
        }
        try {
          byte[] sendInfoOpen = new byte[] {
              0x00, 0x06, 0x00, 0x00, 0x00, 0x06, (byte) 0xFF, 0x05, 0x00, 0x64, 0x00, 0x00
          };  //发送给Modbus Slave软件的消息 ,打开状态灯
          DataOutputStream writer = new DataOutputStream(socket.getOutputStream());
          writer.write(sendInfoOpen);
          Log.e(TAG, "run: 发送打开状态灯信息");
        } catch (IOException e) {
          e.printStackTrace();
        }
      }
    }.start();
```

