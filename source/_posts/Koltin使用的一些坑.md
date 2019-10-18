---
title: Koltin使用的一些坑
date: 2019.09.30 12:04:43
tags: Android
categories: Android
---


##### 使用 Kotlin 时出现‘...has private access in...’异常, 无法获取其他类的实体或属性（非静态）
**实体类**：
      由于 Kotlin 中所有类和方法默认都是 final 的，不能直接继承或重写，需要继承的类或类中要重写的方法都应当在定义时添加 open 关键字。
**属性**：
Kotlin 生成 .java 文件时属性默认为 private，给属性添加 @JvmField 注解声明可以转成 public。需要注意，该属性不可为 null

```
    @JvmField var isBackgroundOpen: Boolean = false //是否可以播放背景音乐
    @JvmField var isSoundEffectOpen: Boolean = false //是否可以播放音效
```
