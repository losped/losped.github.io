---
title: Android lint 自动检测并删除无用资源
date: 2019.06.13 11:48:21
tags: Android
categories: Android
---


检测过程（注意：使用反射获取的资源还是会出现在清单中）
1. build文件配置
```
lintOptions {
    //build release 版本 时 开启lint 检测
    checkReleaseBuilds true
    //lint 遇到 error 时继续 构建
    abortOnError false

}
```

2. 第三方资源文件过滤
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:wheel="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    tools:ignore="all"  <!-- 关键属性。或可用 tools:ignore="UnUsedResource"-->
    android:id="@+id/loading"
    android:layout_width="@dimen/alert_width"
    android:layout_height="wrap_content"
    android:theme="@style/alert_dialog">
</LinearLayout>
```

3. 在 Android Studio 终端选项下 执行 命令
```
gradle lint
```
在 yoru_project_dirctory/build/outputs/ 会生成 两个文件 lint-result.xml, lint-result.html 和文件夹 lint-result-files. 最重要的是 lint-result.xml 文件，里面包含了我们要解析的信息，包含项目中不再使用的资源文件信息。

4. 执行 命令
```
android-resource-remover --xml lint-result.xml
```
执行完这个命令,项目中除第三方资源外不再使用的资源文件，包含 string ，color ,value等，全都被删除掉
