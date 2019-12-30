---
title: APK反编译
date: 2019.12.25 16:29:59
tags: Android
categories: Android
---

最近想抽出个OPPO的系统应用来试着反编译研究下代码，发现.apk里并没有代码可执行文件(.dex)，oat文件夹多了一个odex和vdex。
查了以下资料发现，如今许多Android手机的Rom包在生成过程进行了优化，将apk/jar文件的dex 文件删除，在同目录下生成oat/odex和vdex文件。
odex是优化后的dex文件，odex文件依赖系统中已经编译好的系统模块，一般是/system/framwork目录下的jar包，以提高Dalvik虚拟机的运行速度。

零.准备
将手机连接PC启动ADB, 点击系统应用界面。比如我点击应用管理-XX应用-通知管理。
接着查看输出日志，过滤tag为ActivityManager。
```
START u0 {act=com.coloros.notificationmanager.app.detail cmp=com.coloros.notificationmanager/.AppNotificationSettingsActivity (has extras)} from uid 1000 and from pid 14251
```
得到一个系统应用的Activity：com.coloros.notificationmanager/.AppNotificationSettingsActivity
接着凭推敲或者手机下载包名查看器, 查看该Activity的应用名.

一.smali方式
[下载地址](https://bitbucket.org/JesusFreke/smali/downloads)
1. 创建一个APK文件夹
2. 下载baksmali-x.x.x.jar、smali-x.x.x.jar、baksmali和smali四个文件。将四个文件放在我们的APK文件夹中。
3. adb查看系统应用
```
adb shell
cd /system/app/
ls
```
找到文件夹为NotificationCenter
4. 将应用文件pull出来，NotificationCenter中有一个NotificationCenter.apk和/oat/arm64/NotificationCenter.odex和/oat/arm64/NotificationCenter.vdex。
将NotificationCenter.odex和NotificationCenter.vdex放到我们的APK文件夹中。
```
adb pull /system/app/NotificationCenter D:/NotificationCenter
```
5. 将相关boot的依赖pull到我们的APK文件夹中。
```
adb pull /system/framework/arm64/* D:/APK
```
6. 反编译odex成smail
```
cd APK
java -jar baksmali-2.3.4.jar x NotificationCenter.odex
```
生成out目录，反编译出来的代码为smail格式
7. 编译smail成dex
```
java -jar smali-2.3.4.jar as out/ -o NotificationCenter.dex
```
生成dex文件
8. dex2jar
[源码地址](https://github.com/pxb1988/dex2jar)，需要自己动手打包jar
[脚本地址](https://sourceforge.net/projects/dex2jar/files/)
```
直接拖动dex到bat
# java -jar def2jar.jar NotificationCenter.dex #若清单有入口，可直接运行
# java def2jar.jar com.googlecode.dex2jar.tools.Dex2jarCmd NotificationCenter.dex #若清单无入口，可指定入口
```
查看代码

分析下d2j-dex2jar.bat中的批处理命令
d2j-dex2jar.bat
```
@echo off REM 不显示批处理命令(REM表示注释)

REM
REM dex2jar - Tools to work with android .dex and java .class files
REM Copyright (c) 2009-2013 Panxiaobo
REM
REM Licensed under the Apache License, Version 2.0 (the "License");
REM you may not use this file except in compliance with the License.
REM You may obtain a copy of the License at
REM
REM      http://www.apache.org/licenses/LICENSE-2.0
REM
REM Unless required by applicable law or agreed to in writing, software
REM distributed under the License is distributed on an "AS IS" BASIS,
REM WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
REM See the License for the specific language governing permissions and
REM limitations under the License.
REM

REM call d2j_invoke.bat to setup java environment
@"%~dp0d2j_invoke.bat" com.googlecode.dex2jar.tools.Dex2jarCmd %*


REM @表示命令行回显屏蔽符
REM %后接变量, %~dp0表示当前目录
REM %*表示输入的参数，这里是拉进来的文件路径:C:\Users\Administrator\Desktop\dex2jar-2.0\Notificat
ionCenter.dex
REM 实际运行语句为：
C:\Users\Administrator\Desktop\dex2jar-2.0>"C:\Users\Administrator\Desktop\dex2j
ar-2.0\d2j_invoke.bat" com.googlecode.dex2jar.tools.Dex2jarCmd C:\Users\Administ
rator\Desktop\dex2jar-2.0\NotificationCenter.dex
```
d2j_invoke.bat
```
REM @echo off
REM better invocation scripts for windows from lanchon, release in public domain. thanks!
REM https://code.google.com/p/dex2jar/issues/detail?id=192

setlocal enabledelayedexpansion
REM 启动变量延迟。并且变量要用一对叹号“!!”括起来，否则就没有变量延迟的效果。

set LIB=%~dp0lib

set CP=
for %%X in ("%LIB%"\*.jar) do (
    set CP=!CP!%%X;
)
REM 在cmd窗口中使用，变量名必须用单%引用（即：%variable）；在批处理脚本中使用，变量名必须用双%引用（即：%%variable）。
REM 此处CP相当于一个StringBuilder

java -Xms512m -Xmx1024m -cp "%CP%" %*
REM %*表示输入的参数，这里是指com.googleco
de.dex2jar.tools.Dex2jarCmd C:\Users\Administrator\Desktop\dex2jar-2.0\Notificat
ionCenter.dex
REM 实际运行语句为：
C:\Users\Administrator\Desktop\dex2jar-2.0>java -Xms512m -Xmx1024m -cp "C:\Users
\Administrator\Desktop\dex2jar-2.0\lib\antlr-runtime-3.5.jar;C:\Users\Administra
tor\Desktop\dex2jar-2.0\lib\asm-debug-all-4.1.jar;C:\Users\Administrator\Desktop
\dex2jar-2.0\lib\d2j-base-cmd-2.0.jar;C:\Users\Administrator\Desktop\dex2jar-2.0
\lib\d2j-jasmin-2.0.jar;C:\Users\Administrator\Desktop\dex2jar-2.0\lib\d2j-smali
-2.0.jar;C:\Users\Administrator\Desktop\dex2jar-2.0\lib\dex-ir-2.0.jar;C:\Users\
Administrator\Desktop\dex2jar-2.0\lib\dex-reader-2.0.jar;C:\Users\Administrator\
Desktop\dex2jar-2.0\lib\dex-reader-api-2.0.jar;C:\Users\Administrator\Desktop\de
x2jar-2.0\lib\dex-tools-2.0.jar;C:\Users\Administrator\Desktop\dex2jar-2.0\lib\d
ex-translator-2.0.jar;C:\Users\Administrator\Desktop\dex2jar-2.0\lib\dex-writer-
2.0.jar;C:\Users\Administrator\Desktop\dex2jar-2.0\lib\dx-1.7.jar;" com.googleco
de.dex2jar.tools.Dex2jarCmd C:\Users\Administrator\Desktop\dex2jar-2.0\Notificat
ionCenter.dex
```




[可使用jar包可视化工具](http://java-decompiler.github.io/)


*. 将dex编译成odex
dex编译成odex是在手机上进行的，所以生成了dex文件以后，我们把am.dex推到手机：
adb push am.dex /data/local/tmp
让手机连接到电脑，打开USB调试，在命令行执行adb shell，接着输入下面的命令进行编译：
export ANDROID_DATA=/data
export ANDROID_ROOT=/system

dex2oat --dex-file=/data/local/tmp/am.dex --oat-file=/data/local/tmp/am.odex  --instruction-set=arm64 --runtime-arg -Xms64m --runtime-arg -Xmx128m
--dex-file 指定要编译的dex文件
--oat-file 指定要输出的odex/oat文件
--instruction-set 指定cpu架构
--runtime-arg 指定dex2oat运行时的参数，如果编译时发生内存不足，可以把Xms和Xmx调大
编译成功后，会在/data/local/tmp目录生成odex/oat和vdex文件


二. 使用010Editor工具打开odex文件，删除dex.035之前的字段。(尚未尝试)


参考文章：
https://www.cnblogs.com/luoyesiqiu/p/11802947.html
https://blog.csdn.net/codebob/article/details/52848688
