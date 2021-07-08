---
title: ButterKnife原理
date: 2021.04.16 11:16:29
tags: Android
categories: Android
---

##### 流程
一.注册注解（注解都是一个接口，这里通过反射调用value()等方法）
二.注册注解处理器
1.查找并处理注解信息，保存到Map<TypeElement, BindingClass>（TypeElement为父节点），查找并处理view注解和listener注解
2.利用 BindingClass 遍历集合
3.用 javapoet 生成java文件

Tip：在注解处理器中，我们扫描 java 源文件，源代码中的每一部分都是Element的一个特定类型。换句话说：Element代表程序中的元素，比如说 包，类，方法。每一个元素代表一个静态的，语言级别的结构.
BindingClass.addField(...)：添加新类的成员变量
BindingClass.addMethod(...)：添加新类的方法
...

参考文章：https://blog.csdn.net/ta893115871/article/details/52497297

