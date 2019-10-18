---
title: IDEA
date: 2019.02.25 17:22:47
tags: IDE
categories: IDE
---


**Maven**
IDEA自带的maven：C:\Users\Administrator\.m2\repository
本地安装的maven： D:\Software\apache-maven-3.3.9\LocalWarehouse

IDEA 导入项目：
含pom.xml的项目按maven导入
含pom.xml的Ecliplse项目也用maven导入
项目导入完成后检查**web，spring，dependence**配置

将**不在中央仓库**完整的包导入**本地仓库**：进入根目录 → mvn clean install

**WAR打包**
web application exploded: 这个是以文件夹形式（War Exploded）发布项目，选择这个，发布项目时就会自动生成文件夹在指定的output directory
web application archive: 就是war包形式，每次都会重新打包全部的,将项目打成一个war包在指定位置
如果前面勾选了Build on make选项，可以在运行项目时生成war包。如果没有勾选，可以通过Build-->Build Artifacts来生成war包。
