---
title: ROOT和刷写系统
date: 2019.12.19 15:19:59
tags: Android
categories: Android
---


Android底层是Linux内核，因此这里的ROOT就是获取Linux中的最高级别用户ROOT使用权限。了解Linux系统的都知道，$代表普通用户，#代表ROOT级别用户。
ROOT用户才能运行Linux的所有命令，功能完整的同时安全系数会相应降低。
Android系统一般有三个BOOT引导模式：System，bootloader和recovery。
一般ROOT包括以下几个流程：
1.OEM解锁
(1) 开发者选项里打开oem解锁
(2) ADB调试模式下执行 adb  reboot bootloader。进入bootloader模式。（或者关机状态下，按住音量下建不放，然后按住开关键不放进入bootloader页面）
(3) 解锁bootloader
```
fastboot flashing unlock
```
旧版本命令
```
fastboot oem unlock
\\或者
fastboot flashing unlock_critical
```
2.安装Rom
传闻需要安装开发版的Android系统才能开启ROOT权限，其实未必。主要还是看ROOT的方式来决定。
选择安装第三方自带ROOT的Rom的方式；TWRP+Magisk的方式不需硬性安装开发版系统但属于沙盒机制，不是真正意义上的ROOT权限
主要有官方镜像、OTA固件升级和GSI三种方式。
(1) [安装官方镜像](https://developers.google.com/android/images)
 a. adb reboot bootloader，进入bootloader模式
 b. 解压rom包
 c. 进入根目录，执行flash-all.bat/flash-all.sh
(2) [OTA升级](https://developers.google.com/android/ota)
固件升级只能低版本升到高版本，并且需要在recovery模式运行。
 a. adb reboot recovery，进入 recovery 模式（在bootloader中也可通过上下选择进入）
 b. adb sideload xxx/ota_file.zip
(3) [GSI](https://github.com/phhusson/treble_experimentations/wiki/Generic-System-Image-%28GSI%29-list)
GSI原本是用于测试和第三方定制。以源码为基础，加入自定义修改，最后编译成img。如比较纯净的AOSP，AOKP、Lineage等
以PixelExperience为例：
 *. 工具准备：adb、fastboot、开启USB调试
 a. 安装TWRP Recovery 或 PixelExperience Recovery，安装Recovery
 b. 双清、三清或四清
 c. TWRP-Install 或 adb sideload filename.zip，安装镜像包
其他命令：
fastboot flash boot_a boot.img：在a分区刷入boot系统底包
fastboot flash boot_b boot.img：在b分区刷入boot系统底包
fastboot flash system_a GSI.img：在a分区刷入镜像包
fastboot flash system_b GSI.img：在b分区刷入镜像包
fastboot --disable-verity --disable-verification flash vbmeta vbmeta.img：关闭vbmeta启动验证
官方链接：[https://developer.android.com/about/versions/11/get](https://developer.android.com/about/versions/11/get)

3.ROOT
如果安装的不是第三方自带ROOT的Rom，可通过TWRP+Magisk获取ROOT。目前的情况Android 10不能挂载/system和/，也许TWRP后续版本会修复此问题。
(1) [下载TWRP](https://twrp.me/Devices/)
TWRP其实就是一个第三方的Recovery。进去输入设备-选择版本, 下载版本对应的img文件和zip文件，一般下载最新的版本。例如：twrp-3.3.0-0-sailfish.img和twrp-pixel-installer-sailfish-3.3.0-0.zip。
(2) [下载Magisk](https://magiskmanager.com/)
Magisk是一个ROOT管家类框架，类似Xposed的模式提供了可嵌入模块。现在功能越来越强大。
(3) 安装TWRP和Magisk
 a. adb push twrp-pixel-installer-sailfish-3.3.0-0.zip xxx，放到手机xxx目录
 b. adb push Magisk-v20.1.zip xxx，放到手机xxx目录
 c. adb reboot bootloader，进入bootloader模式；fastboot boot xxx/twrp-3.3.0-0-sailfish.img，进入临时TWRP的recovery模式。
 d. install，选择twrp-pixel-installer-sailfish-3.3.0-0.zip安装；**返回键退回到主页，不需要重启**；install，选择Magisk-v20.1.zip安装
 e. 重启系统完成刷机，使用MagiskManager可管理ROOT权限。
