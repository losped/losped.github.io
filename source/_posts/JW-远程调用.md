---
title: JW-远程调用
date: 2019.04.04 14:56:37
tags: JW
categories: JW
---


1.WebService：基于SOAP,远程调用技术。使用xml形式。

2.接口形式Resful：Http+Json。使用Json形式。

3.Spring Cloud和Dubbo等

##### Dubbo
Dubbo基于RPC协议，直接使用socket通信。用于分布式远程调用的服务管理。
![Dubbo](/images/JW-远程调用/Dubbo.jpg)

节点角色说明：
•	Provider: 暴露服务的服务提供方。
•	Consumer: 调用远程服务的服务消费方。
•	Registry: 服务注册与发现的注册中心。
•	Monitor: 统计服务的调用次调和调用时间的监控中心。
•	Container: 服务运行容器。
调用关系说明：
•	0. 服务容器负责启动，加载，运行服务提供者。
•	1. 服务提供者在启动时，向注册中心注册自己提供的服务。
•	2. 服务消费者在启动时，向注册中心订阅自己所需的服务。
•	3. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
•	4. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
•	5. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

**配置方法**
单一工程中spring的配置
```
<bean id="xxxService" class="com.xxx.XxxServiceImpl" />
<bean id="xxxAction" class="com.xxx.XxxAction">
<property name="xxxService" ref="xxxService" />
</bean>
```

远程服务：
在本地服务的基础上，只需做简单配置，即可完成远程化：

将上面的local.xml配置拆分成两份，将服务定义部分放在服务提供方remote-provider.xml，将服务引用部分放在服务消费方remote-consumer.xml。
并在提供方增加暴露服务配置<dubbo:service>，在消费方增加引用服务配置<dubbo:reference>。
发布服务：
```
<!-- 和本地服务一样实现远程服务 -->
<bean id="xxxService" class="com.xxx.XxxServiceImpl" />
<!-- 增加暴露远程服务配置 -->
<dubbo:service interface="com.xxx.XxxService" ref="xxxService" />
```

调用服务：
```
<!-- 增加引用远程服务配置 -->
<dubbo:reference id="xxxService" interface="com.xxx.XxxService" />
<!-- 和本地服务一样使用远程服务 -->
<bean id="xxxAction" class="com.xxx.XxxAction">
<property name="xxxService" ref="xxxService" />
</bean>
```
**监控中心**
需要安装tomcat，然后部署监控中心即可。部署在tomcat，启动tomcat即可。
1、部署监控中心：
[root@localhost ~]# cp dubbo-admin-2.5.4.war apache-tomcat-7.0.47/webapps/dubbo-admin.war
2、启动tomcat
3、访问http://192.168.25.167:8080/dubbo-admin/
用户名：root
密码：root
如果监控中心和注册中心在同一台服务器上，可以不需要任何配置。
如果不在同一台服务器，需要修改配置文件：
/root/apache-tomcat-7.0.47/webapps/dubbo-admin/WEB-INF/dubbo.properties
![Dubbo1](/images/JW-远程调用/Dubbo1.jpg)

##### Zookeeper
注册中心负责服务地址的注册与查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互，注册中心不转发请求，压力较小。一般可用Redis、Zookeeper等作为注册中心。
Zookeeper是Apacahe Hadoop的子项目，是一个树型的目录服务，支持变更推送，适合作为Dubbo服务的注册中心。

Zookeeper安装
第一步：安装jdk
第二步：把zookeeper的压缩包上传到linux系统。
第三步：解压缩压缩包
tar -zxvf zookeeper-3.4.6.tar.gz
第四步：进入zookeeper-3.4.6目录，创建data文件夹。
第五步：把zoo_sample.cfg改名为zoo.cfg
[root@localhost conf]# mv zoo_sample.cfg zoo.cfg
第六步：修改data属性：dataDir=/root/zookeeper-3.4.6/data
第七步：启动zookeeper
[root@localhost bin]# ./zkServer.sh start
关闭：[root@localhost bin]# ./zkServer.sh stop
查看状态：[root@localhost bin]# ./zkServer.sh status
注意：需要关闭防火墙。
service iptables stop
永久关闭修改配置开机不启动防火墙：
chkconfig iptables off
如果不能成功启动zookeeper，需要删除data目录下的zookeeper_server.pid文件。

#####配置举例
**拆分表现层和服务层**
1）将表现层工程独立出来：
e3-manager-web
2）将原来的e3-manager改为如下结构
e3-manager
 |--e3-manager-dao
 |--e3-manager-interface
 |--e3-manager-pojo
 |--e3-manager-service（打包方式改为war）

**服务层工程**
第一步：把e3-manager的pom文件中删除e3-manager-web模块。
第二步：把e3-manager-web文件夹移动到e3-manager同一级目录。
第三步：e3-manager-service的pom文件修改打包方式
<packaging>war</packaging>
第四步：在e3-manager-service工程中添加web.xml文件
第五步：把e3-manager-web的配置文件复制到e3-manager-service中。
删除springmvc.xml
第六步：web.xml 中只配置spring容器。删除前端控制器
第七步：发布服务
1、在e3-manager-Service工程中添加dubbo依赖的jar包。pom.xml
```
<!-- dubbo相关 -->
	<dependency>
		<groupId>com.alibaba</groupId>
		<artifactId>dubbo</artifactId>
		<exclusions>
			<exclusion>
				<groupId>org.springframework</groupId>
				<artifactId>spring</artifactId>
			</exclusion>
			<exclusion>
				<groupId>org.jboss.netty</groupId>
				<artifactId>netty</artifactId>
			</exclusion>
		</exclusions>
	</dependency>
	<dependency>
		<groupId>org.apache.zookeeper</groupId>
		<artifactId>zookeeper</artifactId>
	</dependency>
	<dependency>
		<groupId>com.github.sgroschupf</groupId>
		<artifactId>zkclient</artifactId>
	</dependency>

```
2、在spring的配置文件中添加dubbo的约束，然后使用dubbo:service发布服务。
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
xmlns:dubbo="http://code.alibabatech.com/schema/dubbo" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.2.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.2.xsd
http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd
http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.2.xsd">
<context:component-scan base-package="cn.e3mall.service"></context:component-scan>

<!-- 使用dubbo发布服务 -->
<!-- 提供方应用信息，用于计算依赖关系 -->
<dubbo:application name="e3-manager" />
<dubbo:registry protocol="zookeeper"
	address="192.168.25.154:2181,192.168.25.154:2182,192.168.25.154:2183" />
<!-- 用dubbo协议在20880端口暴露服务 -->
<dubbo:protocol name="dubbo" port="20880" />
<!-- 声明需要暴露的服务接口 -->
<dubbo:service interface="cn.e3mall.service.ItemService" ref="itemServiceImpl" />

</beans>
```

**表现层工程**
改造e3-manager-web工程。
第一步：删除mybatis、和spring的配置文件。只保留springmvc.xml
第二步：修改e3-manager-web的pom文件，
1、修改parent为e3-parent
2、添加spring和springmvc的jar包的依赖
3、删除e3-mangager-service的依赖
4、添加dubbo的依赖
```
<!-- dubbo相关 -->
	<dependency>
		<groupId>com.alibaba</groupId>
		<artifactId>dubbo</artifactId>
		<exclusions>
			<exclusion>
				<groupId>org.springframework</groupId>
				<artifactId>spring</artifactId>
			</exclusion>
			<exclusion>
				<groupId>org.jboss.netty</groupId>
				<artifactId>netty</artifactId>
			</exclusion>
		</exclusions>
	</dependency>
	<dependency>
		<groupId>org.apache.zookeeper</groupId>
		<artifactId>zookeeper</artifactId>
	</dependency>
	<dependency>
		<groupId>com.github.sgroschupf</groupId>
		<artifactId>zkclient</artifactId>
</dependency>
```
5、e3-mangager-web添加对e3-manager-Interface的依赖。
第三步：修改springmvc.xml，在springmvc的配置文件中添加服务的引用。
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
xmlns:context="http://www.springframework.org/schema/context"
xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
xmlns:mvc="http://www.springframework.org/schema/mvc"
xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
			http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd
			http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd
			http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd">

<context:component-scan base-package="cn.e3mall.controller" />
<mvc:annotation-driven />
<bean
	class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	<property name="prefix" value="/WEB-INF/jsp/" />
	<property name="suffix" value=".jsp" />
</bean>

<!-- 引用dubbo服务 -->
<dubbo:application name="e3-manager-web"/>
<dubbo:registry protocol="zookeeper" address="192.168.25.154:2181,192.168.25.154:2182,192.168.25.154:2183"/>
<dubbo:reference interface="cn.e3mall.service.ItemService" id="itemService" />

</beans>
```
第四步：在e3-manager-web工程中添加tomcat插件配置。
```
<build>
	<plugins>
		<!-- 配置Tomcat插件 -->
		<plugin>
			<groupId>org.apache.tomcat.maven</groupId>
			<artifactId>tomcat7-maven-plugin</artifactId>
			<configuration>
				<path>/</path>
				<port>8081</port>
			</configuration>
		</plugin>
	</plugins>
</build>
```
