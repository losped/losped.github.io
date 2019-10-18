---
title: 解读Android.mk文件
date: 2019.10.14 11:01:44
tags: Android
categories: Android
---


## 一、介绍
本文章会介绍构建 Android.mk文件的构建过程；Android.mk文件会将我们的 C 和 C++ 文件描述为 Android NDK

## 二、概述
Android.mk文件是描述源文件在构建系统的作用，更具体来说：

1. 这个Android.mk是一个微小版的在构建过程中解析一次或多次的Makefile，最好尽量减少在这个文件中声明变量的数量，不要使用没有定义的变量
2. 它可以将你的源文件编译成一个模块，这个模块可以是如下之一：
  -.a 静态库
  -.so 动态库
  -一个独立的可执行文件
```
构建系统只会将动态库安装或者复制到应用程序包，也就是说这个动态库的大小不会改变；此外，静态库可以作为源文件编译生成动态库
 ```
你可以在每个 Android.mk文件中定义一个或多个模块，并且可以在多个模块中使用相同的源文件

## 三、例子解释
一般来说，我们编写的 Android.mk文件的基本内容如下：
```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE    := hello-jni
LOCAL_SRC_FILES := hello-jni.c
include $(BUILD_SHARED_LIBRARY)
```
现在我来解释一下这些代码的含义
 ```
LOCAL_PATH := $(call my-dir)
```
一个 Android.mk文件的开头必须从定义这个变量开始，这个变量用于在开发过程中定位 Android.mk文件在项目中的路径；my-dir是一个宏函数，由系统提供，调用这个函数会返回当前文件所在目录的路径
  ```
include $(CLEAR_VARS)
 ```
这个 CLEAR_VARS变量是系统提供的，它指向一个特殊的 GNU Makefile文件，它会清除很多 LOCAL_XXX的变量（如：LOCAL_MODULE、LOCAL_SRC_FILES、LOCAL_STATIC_LIBRARIES等）；为什么要清除？因为很多的控制文件都是在单个 GUN make中生成的，这些变量都是GUN make中的全局变量，如果不清除，上一次构建的数据信息会影响下一次的构建
 ```
LOCAL_MODULE := hello-jni
 ```
该命令用于为每一个模块定义其名称，这个名称必须是唯一且不包含空格；在构建过程中，系统会自动为模块加上前缀，如：hello-jni 生成的模块为 libhello-jni.so
PS：如果模块名为libhello-jni，那么生成的模块还是 libhello-jni.so
  ```
LOCAL_SRC_FILES := hello-jni.c
 ```
该命令用于列举编译模块需要的 C/C++源文件，同时不要列举头文件和包含文件，因为系统在构建的过程中自动计算这些文件的依赖，我们只需要列举出正确的C/C++源文件就行
  ```
include $(BUILD_SHARED_LIBRARY)
 ```
BUILD_SHARED_LIBRARY是系统的提供的一个变量，它指向 GUN Makefile的一个脚本，这个脚本会收集你定义的所有 LOCAL_XXX形式的变量，然后根据这些变量构建 .so动态库。其他的变量还有：BUILD_STATIC_LIBRARY（构建静态库）
  ```
$(call import-add-path,f:/transcode-1.1.7/)
$(call import-module,avilib)
 ```
import-add-path 表示在将后面的路径（也就是说磁盘 f:/transcode-1.1.7/路径）下的资源添加到NDK构建的路径中；
import-module 表示从NDK的构建路径中引入模块
这个两个参数一般都是配套使用的，用于引入指定的模块，引入成功后，如果要使用模块里面的功能，那么还要在你的so库构建模块中添加 （LOCAL_STATIC_LIBRARIES += libName），libName就是引入模块中的Android.mk文件中构建的模块名称

## 四、Android.mk的变量
我们可以在 Android.mk文件中使用自己的用法定义其他变量，但是 NDK的构建系统保留以下变量名，这些变量在会在 Android.mk文件执行之前被系统定义：

1. 以 LOCAL_XXX开头的变量名，这些变量名是系统定义的
2. 以 PRIVATE_、NDK_、APP_ 开头的变量
3. 小写字母的变量名，如：my-dir；

因为系统的一些变量是小写字母，我们在定义变量名的时候最好不能用小写，而全部使用大写

**现在我们来看看 NDK提供的变量**

BUILD_SHARED_LIBRARY
这个变量指定编译生成 .so静态库，但是在使用这个变量时一定要先定义 LOCAL_MODULE 和 LOCAL_SRC_FILES两个变量
 
BUILD_STATIC_LIBRARY
这个变量指定编译生成 .a静态库，使用这个变量前也需要先定义 LOCAL_MODULE 和 LOCAL_SRC_FILES两个变量
 
PREBUILT_SHARED_LIBRARY
用于指定预编译的so库，也就是在编译目的so库的时候需要用到源so库的东西时，那么就要在构建源so库的时候，使用这个变量指定它编译生成的库类型

————————————————
版权声明：本文为CSDN博主「ytempest」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/mytempest/article/details/81530928
