---
title: JW-Mycat及服务器配置
date: 2019.04.28 13:34:14
tags: JW
categories: JW
---


## 数据库分片
指通过某种特定的条件，将我们存放在同一个数据库中的数据分散存放到多个数据库（主机）上面，以达到分散单台设备负载的效果。
两种切分模式
（1）一种是按照不同的表（或者Schema）来切分到不同的数据库（主机）之上，这种切可以称之为数据的垂直（纵向）切分
![burst](/images/JW-Mycat及服务器配置/burst.jpg)

（2）另外一种则是根据表中的数据的逻辑关系，将同一个表中的数据按照某种条件拆分到多台数据库（主机）上面，这种切分称之为数据的水平（横向）切分。
![burst1](/images/JW-Mycat及服务器配置/burst1.jpg)

## Mycat
数据库中间件产品支持mysql集群，或者mariadb cluster，提供高可用性及数据分片集群。你可以像使用mysql一样使用mycat。对于开发人员来说根本感觉不到mycat的存在。支持Oracle，Mysql，SQL SERVER，MongoDB等数据库。

![mycat](/images/JW-Mycat及服务器配置/mycat.jpg)
逻辑库(schema)：数据库中间件可以被看做是一个或多个数据库集群构成的逻辑
逻辑表（table）：写数据的表就是逻辑表，可以是数据切分后，分布在一个或多个分片库中，也可以不做数据切分，不分片，只有一个表构成。分片表：是指那些原有的很大数据的表，需要切分到多个数据库的表。非分片表：一个数据库中并不是所有的表都很大，某些表是可以不用进行切分的。
分片节点(dataNode)：数据切分后，一个大表被分到不同的分片数据库上面，每个表分片所在的数据库就是分片节点。
节点主机(dataHost) ：数据切分后，每个分片节点（dataNode）不一定都会独占一台机器，同一机器上面可以有多个分片数据库，这样一个或多个分片节点（dataNode）所在的机器就是节点主机（dataHost）,为了规避单节点主机并发数限制，尽量将读写压力高的分片节点（dataNode）均衡的放在不同的节点主机（dataHost）。

##### 安装
安装环境
1、jdk：要求jdk必须是1.7及以上版本
2、Mysql：推荐mysql是5.5以上版本
3、Mycat：
Mycat的官方网站：[http://www.mycat.org.cn/](http://www.mycat.org.cn/)
下载地址：[https://github.com/MyCATApache/Mycat-download](https://github.com/MyCATApache/Mycat-download)

##### 启动
启动：./mycat start
停止：./mycat stop
mycat 支持的命令{ console | start | stop | restart | status | dump }
Mycat的默认端口号为：8066

##### 配置schema.xml
schema配置文件管理着MyCat的逻辑库、表、分片规则、DataNode以及DataSource
schema 标签用于定义MyCat实例中的逻辑库
Table 标签定义了MyCat中的逻辑表
dataNode 标签定义了MyCat中的数据节点，也就是我们通常说所的数据分片。
dataHost标签在mycat逻辑库中也是作为最底层的标签存在，直接定义了具体的数据库实例、读写分离配置和心跳语句。
注意：若是LINUX版本的MYSQL，则需要设置为Mysql大小写不敏感，否则可能会发生表找不到的问题。
在MySQL的配置文件中/etc/my.cnf [mysqld] 中增加一行
　　lower_case_table_names=1

schema.xml
```
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://org.opencloudb/">

        <schema name="e3mall" checkSQLschema="false" sqlMaxLimit="100">
                <!-- auto sharding by id (long) -->
                <table name="tb_item" dataNode="dn1,dn2,dn3" rule="auto-sharding-long" />
        </schema>
        <dataNode name="dn1" dataHost="localhost1" database="db1" />
        <dataNode name="dn2" dataHost="localhost2" database="db2" />
        <dataNode name="dn3" dataHost="localhost1" database="db3" />
        <dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
                writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
                <heartbeat>select user()</heartbeat>
                <!-- can have multi write hosts -->
                <writeHost host="hostM1" url="192.168.25.134:3306" user="root"
                        password="root">
                        <!-- can have multi read hosts -->

                </writeHost>
        </dataHost>
        <dataHost name="localhost2" maxCon="1000" minCon="10" balance="0"
                writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
                <heartbeat>select user()</heartbeat>
                <!-- can have multi write hosts -->
                <writeHost host="hostM1" url="192.168.25.166:3306" user="root"
                        password="root">
                        <!-- can have multi read hosts -->

                </writeHost>
        </dataHost>
</mycat:schema>
```

##### 配置server.xml
server.xml几乎保存了所有mycat需要的系统配置信息。最常用的是在此配置用户名、密码及权限。
```
<user name="test">
    <property name="password">test</property>
    <property name="schemas">e3mall</property>
    <property name="readOnly">false</property>
</user>

```

##### 配置rule.xml
我们可以灵活的对表使用不同的分片算法，或者对表使用相同的算法但具体的参数不同。这个文件里面主要有tableRule和function这两个标签。

分片规则“auto-sharding-long”：
根据id进行分片
Datanode1：1~5000000
Datanode2：5000000~10000000
Datanode3：10000001~15000000
当15000000以上的id插入时报错：
[Err] 1064 - can't find any valid datanode :TB_ITEM -> ID -> 15000001
此时需要添加节点了。

##### 配置读写分离

![mycat1](/images/JW-Mycat及服务器配置/mycat1.jpg)
主从配置需要注意的地方
1、主DB server和从DB server数据库的版本一致
2、主DB server和从DB server数据库数据名称一致
3、主DB server开启二进制日志,主DB server和从DB server的server_id都必须唯一

**Mysql主服务器配置**
1.修改my.conf文件
```
binlog-do-db=db1
binlog-ignore-db=mysql
#启用二进制日志
log-bin=mysql-bin
#服务器唯一ID，一般取IP最后一段
server-id=134
```
2.重启mysql服务：service mysqld restart
3.建立帐户并授权slave
mysql>GRANT FILE ON *.* TO 'backup'@'%' IDENTIFIED BY '123456';
mysql>GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* to 'backup'@'%' identified by '123456';
**一般不用root帐号，“%”表示所有客户端都可能连，只要帐号，密码正确，此处可用具体客户端IP代替，如192.168.145.226，加强安全。**
刷新权限
mysql> FLUSH PRIVILEGES;
查看mysql现在有哪些用户
mysql>select user,host from mysql.user;
4.查询master的状态
```
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      120 | db1          | mysql            |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set
```

**Mysql从服务器配置**
1.修改my.conf文件
```
[mysqld]
server-id=166
```
2.配置从服务器
mysql>change master to master_host='192.168.25.134',master_port=3306,master_user='backup',master_password='123456',master_log_file='mysql-bin.000001',master_log_pos=120
注意语句中间不要断开，master_port为mysql服务器端口号(无引号)，master_user为执行同步操作的数据库账户，master_log_pos的值就是show master status 中看到的position的值，这里的mysql-bin.000001就是file对应的值。
3.启动从服务器复制功能
Mysql>start slave;
4.检查从服务器复制功能状态：
mysql> show slave status
……………………(省略部分)
Slave_IO_Running: Yes //此状态必须YES
Slave_SQL_Running: Yes //此状态必须YES
……………………(省略部分)
注：Slave_IO及Slave_SQL进程必须正常运行，即YES状态，否则都是错误的状态(如：其中一个NO均属错误)。
```
错误处理：
如果出现此错误：
Fatal error: The slave I/O thread stops because master and slave have equal MySQL server UUIDs; these UUIDs must be different for replication to work.
因为是mysql是克隆的系统所以mysql的uuid是一样的，所以需要修改。
解决方法：
删除/var/lib/mysql/auto.cnf文件，重新启动服务。

```

Mycat 1.4 支持MySQL主从复制状态绑定的读写分离机制，让读更加安全可靠
scheme.xml配置如下：
```
<dataNode name="dn1" dataHost="localhost1" database="db1" />
	<dataNode name="dn2" dataHost="localhost1" database="db2" />
	<dataNode name="dn3" dataHost="localhost1" database="db3" />
	<dataHost name="localhost1" maxCon="1000" minCon="10" balance="1"
		writeType="0" dbType="mysql" dbDriver="native" switchType="2"  slaveThreshold="100">
		<heartbeat>show slave status</heartbeat>
		<writeHost host="hostM" url="192.168.25.134:3306" user="root"
			password="root">
			<readHost host="hostS" url="192.168.25.166:3306" user="root"
			password="root" />
		</writeHost>
	</dataHost>
```
(1)	设置 balance="1"与writeType="0"
Balance参数设置：
1. balance=“0”, 所有读操作都发送到当前可用的writeHost上。
**2. balance=“1”，所有读操作都随机的发送到readHost。**
3. balance=“2”，所有读操作都随机的在writeHost、readhost上分发
WriteType参数设置：
**1. writeType=“0”, 所有写操作都发送到可用的writeHost上。**
2. writeType=“1”，所有写操作都随机的发送到readHost。
3. writeType=“2”，所有写操作都随机的在writeHost、readhost分上发。
 “readHost是从属于writeHost的，即意味着它从那个writeHost获取同步数据，因此，当它所属的writeHost宕机了，则它也不会再参与到读写分离中来，即“不工作了”，这是因为此时，它的数据已经“不可靠”了。基于这个考虑，目前mycat 1.3和1.4版本中，若想支持MySQL一主一从的标准配置，并且在主节点宕机的情况下，从节点还能读取数据，则需要在Mycat里配置为两个writeHost并设置banlance=1。”
(2)	设置 switchType="2" 与slaveThreshold="100"
switchType 目前有三种选择：
-1：表示不自动切换
1 ：默认值，自动切换
**2 ：基于MySQL主从同步的状态决定是否切换**
“Mycat心跳检查语句配置为 show slave status ，dataHost 上定义两个新属性： switchType="2" 与slaveThreshold="100"，此时意味着开启MySQL主从复制状态绑定的读写分离与切换机制。Mycat心跳机制通过检测 show slave status 中的 "Seconds_Behind_Master", "Slave_IO_Running", "Slave_SQL_Running" 三个字段来确定当前主从同步的状态以及Seconds_Behind_Master主从复制时延。
其他配置...


## 参考某项目服务器配置搭建
**服务器配置**
理想配置
![server](/images/JW-Mycat及服务器配置/server.jpg)
现实配置
![server1](/images/JW-Mycat及服务器配置/server1.jpg)

**tomcat热部署**
（一）Tomcat后台管理功能，实现工程热部署。
1.  修改tomcat的conf/tomcat-users.xml配置文件。添加用户名密码及权限。
```
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="tomcat" password="tomcat" roles="manager-gui,manager-script">
```
2.重新启动tomcat

（二）使用maven实现热部署，工程随tomcat启动而部署。
1.修改工程的pom.xml文件，引入tomcat插件。注意不同工程不同Ip需配置不同端口
```
<project>
	<!-- 配置tomcat插件 -->
	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.tomcat.maven</groupId>
				<artifactId>tomcat7-maven-plugin</artifactId>
				<configuration>
					<path>/</path>
					<port>8080</port>
					<url>http://192.168.25.135:8080/manger/test</url>  <!--部署到哪台服务器 用户名密码-->
					<username>test</username>
					<password>test</password>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
```
2.执行Maven热部署
![deploy](/images/JW-Mycat及服务器配置/deploy.jpg)
![deploy1](/images/JW-Mycat及服务器配置/deploy1.jpg)

e.g. 本文仅供个人笔记使用，借鉴部分网上资料。
