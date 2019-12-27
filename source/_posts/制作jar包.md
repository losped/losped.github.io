---
title: 制作jar包
date: 2019.12.26 10:30:05
tags: JW
categories: JW
---


一. 命令行打包jar
1. 使用javac编译java文件,得到class二进制字节码文件
```
javac -encoding utf-8 -d .\classes -classpath .\lib\x1.jar;.\lib\x2.jar; .\src\*.java
```
-d：.\out 指定编译后的class文件放置到out文件夹
.\src\*.java：src文件夹所有的java文件
-classpath：.\lib\x1.jar;x2.jar  指定依赖jar包的位置，有多个用分号（;）隔开（linux下用冒号（:）隔开）
2. 制作可执行jar包，需要有manifest清单文件。创建一个文件manifest，内容如下：
```
Manifest-Version: 1.0
Main-Class: mainClass
Class-Path: lib/internal_impl-24.2.1.jar #如果需要用到第三方jar包，需要在此注明，多个jar包之间用空格分隔，最后一个jar包后面加两个空格。AndroidV4的分包就做了依赖。

```
注意：冒号后加空格，每行内容加回车换行
3. 将class文件夹和manifest文件放入指定文件夹，如2jar。开始打包
```
jar -cvfm main.jar manifest -C 2jar .
```
注意：-m表示将清单文件合并到jar包中的 META-INF/MANIFEST.MF

二. 使用IDE打包，举例IDEA
1. Project Structure-Artifacts
![1](/images/制作jar包/jar1.png)
2. 选择需要打包的java文件
![2](/images/制作jar包/jar2.png)
3. 创建清单文件
![3](/images/制作jar包/jar3.png)
4. 生成jar文件
![4](/images/制作jar包/jar4.png)

部分参考自:https://www.cnblogs.com/jayworld/p/9765474.html
