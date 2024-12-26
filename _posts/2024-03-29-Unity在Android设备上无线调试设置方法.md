---
layout: post
title: "Unity在Android设备上无线调试设置方法"
---

以Unity 2022.3.22f1和Android 12设备设备为例，设置无线调试（Wireless debugging）。

### 环境

- Windows 10
- Unity 2022.3.22f1
- Android SDK Platform-Tools 35.0.1，建议[下载最新版](https://developer.android.com/tools/releases/platform-tools)
- 运行Android 12系统的手机

### 设置步骤

1. 手机和电脑连接相同WIFI。

2. 在手机上，选择开发者选项->无线调试->开启。

3. 在手机上，选择开发者选项->无线调试->使用配对码配对设备。将看到**用于验证配对的**WLAN配对码（6位数）和IP地址和端口。**暂时不要关闭当前界面**。

 > 注意区分：1）用于验证配对的IP地址和端口；2）用于调试的IP地址和端口。

4. 在电脑上，关闭（若有）已启动的adb服务器，命令为：`adb kill-server`

5. 在电脑上，启动（若没有）adb服务器，并查看已配对设备列表，命令为：`adb devices`

 ```
 adb devices
 * daemon not running; starting now at tcp:5037
 * daemon started successfully
 List of devices attached
 	// 列表为空
 ```

6. 在电脑上，配对adb服务器和Android手机，命令为：`adb pair IP:ADDR`，其中参数`IP:ADDR`是用于验证配对的IP地址和端口。然后，按要求输入WLAN配对码。

 ```
 Enter pairing code: 123456
 ```

7. 可以关闭在手机上使用配对码配对设备的界面。

8. 在电脑上，查看已配对的设备列表，记录下**用于调试的**IP地址和端口，命令为：`adb devices`

 ```
 adb devices
 List of devices attached
 192.168.1.110:45678    device
 ```

 最后一行显示手机的**用于调试通信的**IP地址和端口。当然，该地址也显示在手机->开发者选项->无线调试界面上。

9. 在Unity中，选择File->Build Settings->Platforms->Android->Run Device->下拉框Enter IP，输入用于调试的IP地址和端口。稍等片刻后，下拉框中将显示设备型号。

10. 在Unity中，选择File->Build Settings->Platforms->Android->Switch Platform。

11. 在Unity中，选择File->Build Settings->Build And Run。

### 参考文献

[1] Android Studio. Android 调试桥[Z/OL]. 2024-02-09. [https://developer.android.com/tools/adb](https://developer.android.com/tools/adb). 

[2] Unity Manual. Android Debug Bridge[Z/OL]. [https://docs.unity3d.com/Manual/android-debugging-on-an-android-device.html](https://docs.unity3d.com/Manual/android-debugging-on-an-android-device.html). 
