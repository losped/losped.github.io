---
title: Maven 远程热部署Tomcat
date: 2019.09.30 12:03:34
tags: JW
categories: JW
---


出处：
[http://www.cnblogs.com/letcafe/](http://www.cnblogs.com/letcafe/)
[https://www.cnblogs.com/xyb930826/p/5725340.html](https://www.cnblogs.com/xyb930826/p/5725340.html)


## 概述

Maven本身不提供任何插件将war包发布到远程站点，例如Tomcat这样类似的Servlet容器，但是，Apache Tomcat本身提供了一个Maven插件：

```
<dependency>
    <groupId>org.apache.tomcat.maven</groupId>
    <artifactId>tomcat7-maven-plugin</artifactId>
    <version>2.1</version>
</dependency>
```

tomcat7-maven-plugin是很久以前的插件版本，默认支持到Tomcat7，但是对于目前最新的Tomcat9同样可以使用该插件（虽然插件的ArtifactId的名字为tomcat7很奇怪）

插件介绍的官网文档为：

> [http://tomcat.apache.org/maven-plugin-2.1/index.html](http://tomcat.apache.org/maven-plugin-2.1/index.html)


## 一、Tomcat插件支持的目标

默认使用的Tomcat7插件支持多种目标，

调用格式如下：

```
mvn tomcat7:[goal]
```

例如，远程部署一个项目到Tomcat容器：

```
mvn tomcat7:deploy
```

文档如下：

| 目标 | 描述 |
| --- | --- |
| deploy | 部署war包到Tomcat中 |
| deploy-only | 不经过package阶段，直接将包部署到Tomcat中（传输现成的） |
| exec-war | 创建一个包含必要Apache Tomcat类库的自可执行jar包，这允许使
用类似'jar -jar mywebapp.jar'直接运行APP而不需要Tomcat实例 |
| exec-war-only | 同上exec-war，但是不使用package阶段 |
| help | 展示所有的帮助信息 |
| redeploy | 重新部署war包到Tomcat(等同于deploy目标加上update=true参数) |
| redeploy-only | 重新部署war包到Tomcat但是不经过package阶段(等同于deploy目
标加上update=true参数) |
| run | 将当前项目作为动态web程序(exploded)，使用嵌入的Tomcat服务器运行 |
| run-war | 将当前项目作为打包后的war(war)，使用嵌入的Tomcat服务器运行 |
| run-war-only | 同run-war，但是不使用package阶段 |
| shutdown | 关闭所有已经开始的嵌入式Tomcat服务器 |
| standalone-war | 创建嵌入Tomcat的可执行war，并且可以在别的地方部署 |
| standalone-war-only | 同standalone-war但是不使用package阶段 |
| undeploy | 从Tomcat服务器中，取消部署某一个项目 |

## 二、系统要求及插件引入

### 2.1 系统要求

要求如下：

| 组件 | Maven | JDK | 内存 | 硬盘 |
| --- | --- | --- | --- | --- |
| 要求 | 2.0+ | 1.5+ | 无要求 | 无要求 |

### 2.2 引入插件

引入方式：

```
<project>
  ...
  <build>
    <!-- 在POM中或父POM中使用这样的插件（IDEA会出现对应的插件栏） -->
    <plugins>
      <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <version>2.1</version>
      </plugin>
      ...
    </plugins>
  </build>
  ...
</project>
```

## 三、远程部署war到tomcat

命令格式：

```
mvn tomcat7:deploy
```

### 3.1 添加tomcat管理角色

修改tomcat的用户配置文件

%TOMCAT_HOME%/conf/tomcat-users.xml，添加一个用户：

```
<?xml version="1.0" encoding="UTF-8"?>
<tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">
    <role rolename="manager-gui" />
    <role rolename="manager-script"/>
    <user username="username" password="password" roles="manager-gui,manager-script"/>
</tomcat-users>
```

注意！可以给该用户添加多个角色，为了远程部署，需要调用script,至少需要这个角色：

*   manager-script

当然，也可以开启**manager-gui**用于可视化管理

### 3.15 添加manager.xml的配置
默认情况下，Tomcat的Manager和Host-Manager只接受本机的请求，而要让它接受远程的请求，需要添加manager.xml的配置

在tomcat服务器的conf/Catalina/localhost/目录下创建一个manager.xml文件，写入如下值:
```
<?xml version="1.0" encoding="UTF-8"?>
<Context privileged="true" antiResourceLocking="false"
         docBase="${catalina.home}/webapps/manager">
             <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.*$" />
</Context>
 ```
保存退出。

由于Tomcat的Manager可以执行项目的部署、卸载等敏感操作，如果你只想允许特定的IP地址访问Manager，可在上面的allow属性中设置规则。具体规则设置见下面的链接：
http://tomcat.apache.org/tomcat-7.0-doc/config/valve.html#Remote_Address_Filter
问题说明：http://tomcat.apache.org/tomcat-7.0-doc/manager-howto.html#Configuring_Manager_Application_Access

### 3.2 本地Maven设置Server

为了让本地发布的Maven可以找到对应的服务器并完成鉴权

需要修改settings.xml，并添加服务器，这里的账号、密码需要和部署的tomcat服务器配置的**一致**：

```
<servers>
    <server>
        <id>tomcatServer</id>
        <username>username</username>
        <password>password</password>
    </server>
</servers>
```

### 3.3 项目配置Tomcat插件

```
<build>
    <!-- 在POM中或父POM中使用这样的插件（IDEA会出现对应的插件栏） -->
    <plugins>
        <plugin>
            <groupId>org.apache.tomcat.maven</groupId>
            <artifactId>tomcat7-maven-plugin</artifactId>
            <version>2.2</version>
            <configuration>
                <url>http://{yourIP}:8080/manager/text</url>  <!--部署到哪台服务器, 使用的是tomcat的默认项目manager进行deploy, 因此/manager/text是manager固定的接口。用户名密码使用的是tomcat的tomcat-users.xml配置文件中具备manager-script的用户名密码, 另外需要在maven的setting.xml配置同样的用户名密码-->
                <server>tomcatServer</server>
                <update>true</update>
                <path>/${project.artifactId}</path>
            </configuration>
        </plugin>
      ...
    </plugins>
</build>
```

此处指定了插件所使用的<server style="margin: 0px; padding: 0px;">切记需要一致(setting.xml和pom.xml)</server>

### 3.4 插件参数说明

tomcat7-maven-plugin为每个目标提供了多个参数，每个目标可以有相关的配置，具体说明可参考官方文档：

> [http://tomcat.apache.org/maven-plugin-2.1/tomcat7-maven-plugin/plugin-info.html](http://tomcat.apache.org/maven-plugin-2.1/tomcat7-maven-plugin/plugin-info.html)

#### 3.4.1 必选参数

以下参数必选，但是可以在pom中空缺，空缺时将采用默认值

| 名称 | 描述 | 默认值 |
| --- | --- | --- |
| charset | 在与Tomcat Manager通信是的URL编
码字符集 | ISO-8859-1 |
| mode | 部署的模式，值可为：war,context,both | war |
| path | 应用程序运行的上下文路径，必须以'/'开始 | /${project.artifactId} |
| update | 当部署已存在的应用时，是否自动
undeploy该应用 | false |
| url | Tomcat Manager实例使用的全路径 | [http://localhost:8080](http://localhost:8080/)
/manager/text |
| warFile | 部署warFile的路径 | ${project.build.directory}
/${project.build.finalName}.war |

#### 3.4.2 可选参数

对于个性化的需求，tomcat7插件提供了可配置的参数

| 名称 | 描述 |
| --- | --- |
| contextFile | Tomcat的context的XML路径，对于mode=war不适用，默认为${project.build.directory}/${project.build.finalName}/META-INF/context.xml |
| ignorePackaging | 如果设置为true，忽略类型不是war的项目 |
| username | 部署到Tomcat的username |
| password | 部署到Tomcat的password |
| server | 指定Maven的setting.xml中配置的server id用于Tomcat认证 |
| tag | 应用程序在Tomcat中使用的标签的名字 |

### 3.5 运行结果

如果是第一次部署，运行mvn tomcat7:deploy进行自动部署(对于tomcat8,9，也是使用tomcat7命令)，如果是更新了代码后重新部署更新，运行mvn tomcat7:redeploy，如果第一次部署使用mvn tomcat7:redeploy，则只会执行上传war文件，服务器不会自动解压部署。**如果路径在tomcat服务器中已存在并且使用mvn tomcat7:deploy命令的话，上面的配置中一定要配置<update>true</update>，不然会报错。tomcat7:deploy前必须先maven:install从父项目到子项目部署到本地仓库**

调用：mvn tomcat7:deploy命令得到下图：

![result](/images/Maven 远程热部署Tomcat/result.jpg)

成功快速部署到tomcat中

![result1](/images/Maven 远程热部署Tomcat/result1.jpg)

## 四、远程undeploy

此外，如果快速卸载(undeploy)Tomcat服务器的项目，使用：

```
mvn tomcat7:undeploy
```

效果如下：

![undeploy](/images/Maven 远程热部署Tomcat/undeploy.jpg)

![undeploy1](/images/Maven 远程热部署Tomcat/undeploy1.jpg)

## 五、其他问题
**(1)自动部署显示成功，war包也上传成功，但是war不自动解压自动部署。**

如果你在tomcat的server.xml中通过设置<Context>标签来部署相同名称的项目的话，maven发布到该服务器的war不会被自动解压，部署，更新，需要去掉server.xml中该项目的<Context>标签。

**(2)内存泄漏**
使用上面的方法进行部署后会出现严重的内存泄漏现象。tomcat的manager提供了诊断在部署时是否产生内存泄漏的功能，在上面提到的http://serverip:port/manager/html这个页面底部有一个“Find leaks”的按钮，如下：

![other](/images/Maven 远程热部署Tomcat/other.jpg)

点击按钮，网页头部出现如下信息说明在部署的时候有内存泄漏：

![other1](/images/Maven 远程热部署Tomcat/other1.jpg)

上面的消息显示部署的test项目存在内存泄漏，如果同一项目多次重新部署，则一个项目名可能会出现多次。

部署时产生内存泄漏的原因是每次（重新）部署时，Tomcat会为项目新建一个类加载器，而旧的类加载器没有被GC回收。maven的库classloader-leak-prevention-servlet可以用来解决这个问题。具体方案为：

（1）添加maven依赖：

<pre style="margin: 0px; padding: 0px; white-space: pre-wrap; overflow-wrap: break-word; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;"><dependency>
    <groupId>se.jiderhamn.classloader-leak-prevention</groupId>
    <artifactId>classloader-leak-prevention-servlet</artifactId>
    <version>2.1.0</version>
</dependency></pre>

（2）在项目的web.xml中添加一个Listener（**必须让此Listener成为web.xml中的第一个Listener，否则不起作用**）

<pre style="margin: 0px; padding: 0px; white-space: pre-wrap; overflow-wrap: break-word; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;"><listener>
    <listener-class>se.jiderhamn.classloader.leak.prevention.ClassLoaderLeakPreventorListener</listener-class>
</listener></pre>

这样部署时的内存泄漏就解决了。
注意：
1） 添加这个Listener后，默认在tomcat关闭5s后jvm会进行内存回收的操作，具体时间设置可在下面的第三个参考链接中找到，所以，在关闭后的5s内，再次启动tomcat，可能会存在问题，导致启动无效（如果出现tomcat重启后日志显示正常但是服务器不工作的话考虑一下是不是这个问题）。
2）这个Listener只解决部署的内存泄漏，其他问题（如jdbc等）产生的内存泄漏还需要自己解决。
参考：
http://stackoverflow.com/questions/7788280/memory-leak-when-redeploying-application-in-tomcat#answer-36295683
http://java.jiderhamn.se/2011/12/11/classloader-leaks-i-how-to-find-classloader-leaks-with-eclipse-memory-analyser-mat/
https://github.com/mjiderhamn/classloader-leak-prevention
