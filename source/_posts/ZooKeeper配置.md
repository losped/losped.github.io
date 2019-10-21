---
title: ZooKeeper配置
date: 2019.09.30 12:04:22
tags: JW
categories: JW
---


**win** 下监听端口情况
```
netstar-ano
netstar-ano findstr "xxx"
```

## ZooKeeper安装与部署
1.  安装jdk
2.  安装Zookeeper. 在官网[https://zookeeper.apache.org/releases.html](https://zookeeper.apache.org/releases.html)
下载zookeeper.我下载的是zookeeper-3.4.6版本。

## 一.单机模式
##### 运行配置
上面提到，conf目录下提供了配置的样例zoo_sample.cfg，要将zk运行起来，需要将其名称修改为zoo.cfg。
打开zoo.cfg，可以看到默认的一些配置。
tickTime 
时长单位为毫秒，为zk使用的基本时间度量单位。例如，1 * tickTime是客户端与zk服务端的心跳时间，2 * tickTime是客户端会话的超时时间。 
tickTime的默认值为2000毫秒，更低的tickTime值可以更快地发现超时问题，但也会导致更高的网络流量（心跳消息）和更高的CPU使用率（会话的跟踪处理）。
clientPort 
zk服务进程监听的TCP端口，默认情况下，服务端会监听2181端口。
dataDir 
无默认配置，必须配置，用于配置存储快照文件的目录。一般在根目录新建data文件夹存放。

##### 启动
在Windows环境下，直接双击zkServer.cmd即可。在Linux环境下，进入bin目录，执行命令
```./zkServer.sh start```
这个命令使得zk服务进程在后台进行。如果想在前台中运行以便查看服务器进程的输出日志，可以通过以下命令运行：
```./zkServer.sh start-foreground```
关闭：
```[root@localhost bin]# ./[zkServer.sh](https://links.jianshu.com/go?to=http%3A%2F%2FzkServer.sh) stop```
查看状态：
```[root@localhost bin]# ./[zkServer.sh](https://links.jianshu.com/go?to=http%3A%2F%2FzkServer.sh) status```

#####本地连接
如果是连接同一台主机上的zk进程，那么直接运行bin/目录下的zkCli.cmd（Windows环境下）或者zkCli.sh（Linux环境下），即可连接上zk。 
直接执行zkCli.cmd或者zkCli.sh命令默认以主机号 127.0.0.1，端口号 2181 来连接zk，如果要连接不同机器上的zk，可以使用 -server 参数，例如：
```bin/zkCli.sh -server 192.168.0.1:2181```

##集群模式
#####运行配置
```
注意 
在集群模式下，建议至少部署3个zk进程，或者部署奇数个zk进程。如果只部署2个zk进程，当其中一个zk进程挂掉后，剩下的一个进程并不能构成一个[quorum](http://baike.baidu.com/link?url=pqWrzgH-_VhMLnscR1iRTpPjovfyhxG-8Qs9HxGutiGi5bhnA_lX_pmabLQ-3MiDeigcHRFMYSbFg90RAYVAta)的大多数。因此，部署2个进程甚至比单机模式更不可靠，因为2个进程其中一个不可用的可能性比一个进程不可用的可能性还大。
```

在集群模式下，所有的zk进程可以使用相同的配置文件（是指各个zk进程部署在不同的**机器**上面），例如如下配置：
```
tickTime=2000
dataDir=/home/myname/zookeeper
clientPort=2181
initLimit=5
syncLimit=2
server.1=192.168.229.160:2888:3888
server.2=192.168.229.161:2888:3888
server.3=192.168.229.162:2888:3888
```
若部署伪集群模式，**WIN**下可复制成三份zoo-1.cfg,zoo-2.cfg,zoo-3.cfg文件。只需修改dataDir为不同的目录，并在目录下创建文件myid内容输入各自的myid。接着只要复制成三份zkServer-1.cmd,zkServer-2.cmd,zkServer-3.cmd，每份新增SETZOOCFG，用于启动即可。
```
@echo off
REM Licensed to the Apache Software Foundation (ASF) under one or more
REM contributor license agreements.  See the NOTICE file distributed with
REM this work for additional information regarding copyright ownership.
REM The ASF licenses this file to You under the Apache License, Version 2.0
REM (the "License"); you may not use this file except in compliance with
REM the License.  You may obtain a copy of the License at
REM
REM     http://www.apache.org/licenses/LICENSE-2.0
REM
REM Unless required by applicable law or agreed to in writing, software
REM distributed under the License is distributed on an "AS IS" BASIS,
REM WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
REM See the License for the specific language governing permissions and
REM limitations under the License.

setlocal
call "%~dp0zkEnv.cmd"

set ZOOMAIN=org.apache.zookeeper.server.quorum.QuorumPeerMain
set ZOOCFG=..\conf\zoo1.cfg  \\新增
echo on
call %JAVA% "-Dzookeeper.log.dir=%ZOO_LOG_DIR%" "-Dzookeeper.root.logger=%ZOO_LOG4J_PROP%" -cp "%CLASSPATH%" %ZOOMAIN% "%ZOOCFG%" %*

endlocal
pause
```

**initLimit**
ZooKeeper集群模式下包含多个zk进程，其中一个进程为leader，余下的进程为follower。 
当follower最初与leader建立连接时，它们之间会传输相当多的数据，尤其是follower的数据落后leader很多。initLimit配置follower与leader之间建立连接后进行同步的最长时间。
**syncLimit**
配置follower和leader之间发送消息，请求和应答的最大时间长度。
**tickTime**
tickTime则是上述两个超时配置的基本单位，例如对于initLimit，其配置值为5，说明其超时时间为 2000ms * 5 = 10秒。
server.id=host:port1:port2 
其中id为一个数字，表示zk进程的id，这个id也是dataDir目录下myid文件的内容。 
host是该zk进程所在的IP地址，port1表示follower和leader交换消息所使用的端口，port2表示选举leader所使用的端口。
**dataDir**
注意，路径'\'可能需要转义而使用'\\'。其配置的含义跟单机模式下的含义类似，不同的是集群模式下还需要在dataDir目录下创建一个myid文件。myid文件的内容只有一行，且内容只能为1 - 255之间的数字，这个数字亦即上面介绍server.id中的id，表示zk进程的id。

##### 启动
同时启动多个zkServer即可。

##### 连接测试
可以使用以下命令来连接一个zk集群：
```
bin/zkCli.sh -server 192.168.229.160:2181,192.168.229.161:2181,192.168.229.162:2181
```
成功连接后，可以看到如下输出：
```

2016-06-28 19:29:18,074 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=192.168.229.160:2181,192.168.229.161:2181,192.168.229.162:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@770537e4
Welcome to ZooKeeper!
2016-06-28 19:29:18,146 [myid:] - INFO  [main-SendThread(192.168.229.162:2181):ClientCnxn$SendThread@975] - Opening socket connection to server 192.168.229.162/192.168.229.162:2181. Will not attempt to authenticate using SASL (unknown error)
JLine support is enabled
2016-06-28 19:29:18,161 [myid:] - INFO  [main-SendThread(192.168.229.162:2181):ClientCnxn$SendThread@852] - Socket connection established to 192.168.229.162/192.168.229.162:2181, initiating session
2016-06-28 19:29:18,199 [myid:] - INFO  [main-SendThread(192.168.229.162:2181):ClientCnxn$SendThread@1235] - Session establishment complete on server 192.168.229.162/192.168.229.162:2181, sessionid = 0x3557c39d2810029, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
```
