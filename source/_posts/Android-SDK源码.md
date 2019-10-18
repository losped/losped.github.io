---
title: Android-SDK源码
date: 2019.10.14 11:02:39
tags: Android
categories: Android
---


##### 背景
在查看Android API源码时,Android.jar内部有大量@hide注解的代码。Android隐藏API是因为不能保证这些API还存在新系统版本,所以尽量少用隐藏API！因此，Android.jar并不完整，要么实现为空要么类缺失，不利于源码的研究。谷歌百度之后，寻觅了三种方法以获取源码。

##### 关于@hide注解
简单来说编程工具（AndroidStudio等）是引用SDK中的android.jar，这个包里没有hide和internal相关的类、属性和方法的，它是个删减版。当app开发完成装到手机上或虚拟里运行时，引用的却是framework.jar，这个包是完整版。举个形象的例子，我们在开发app时，编译环境只给了我们一个碗让我们去盛饭但是没有筷子，只有当app运行时，运行环境才会给我们一双筷子，这时我们才能吃饭。
也就是说hide只作用于编译期，在运行期它是没有作用的，所以才能通过反谢去调用hide方法。

##### 获取源码
**方法一** 在SDK Manager中下载源码，一般存放于SDK根目录下 \sdk\sources\android-xx。但经对比, 这种方式获取的源码依然有大部分的缺失。

**方法二** 在万能的GitHub已有人去除Android.jar中@hide注解, 可反编译使用。
地址: [https://github.com/anggrayudi/android-hidden-api](https://link.jianshu.com/?t=https://github.com/anggrayudi/android-hidden-api)
eg:  可替换SDK/platforms/android-版本/Android.jar。直接使用隐藏API,不需要反射，Android.jar并不会打包到APK,所以去除@hide的Android.jar,只是欺骗IDE/编译器,方便程序员查看使用！

**方法三** 在线查阅完整源码
安卓社区 [https://www.androidos.net.cn/android/8.0.0_r4/xref](https://www.androidos.net.cn/android/8.0.0_r4/xref)
androidxref [http://androidxref.com/8.0.0_r4/xref/](http://androidxref.com/8.0.0_r4/xref/)

**方法四** 官方下载源码（十分庞大）
清华镜像 [https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)
官方 [https://source.android.google.cn/setup/develop](https://source.android.google.cn/setup/develop)

参考文章：
[https://www.jianshu.com/p/c5d061d16b30](https://www.jianshu.com/p/c5d061d16b30)
[https://blog.csdn.net/lxhpkm01/article/details/55506968](https://blog.csdn.net/lxhpkm01/article/details/55506968)
