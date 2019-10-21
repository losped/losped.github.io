---
title: JW-SolrCloud
date: 2019.04.16 11:44:15
tags: JW
categories: JW
---


SolrCloud是Solr提供的分布式搜索方案，用于大规模（高并发）、容错、分布式索引和检索能力时的环境。主要使用zookeepper作为集群的配置信息中心。有如下特点：①集中式的配置信息②自动容错③近实时搜索④查询时负载均衡。

![结构图](/images/JW-SolrCloud/结构图.jpg)
同一shard不同部分存储的内容相同，用于主备负载均衡。shard可拓展，用于海量存储。

**zookeeper集群配置**
创建data目录，该目录下创建myid，键入1(2/3)保存
①zoo.cfg配置datadir指向创建的data及port
②zoo.cfg添加如下
server.1=192.168.25.163:2881:3881
server.2=192.168.25.163:2882:3882
server.3=192.168.25.163:2883:3883

:3881~:3883是zookeeper节点间投票选举使用的端口
查看zookeeper集群选举状态。命令：
```
zookeeper-3.4.6/bin$  ./zkServer.sh status
```

**solr集群配置**
①修改各tomcat中web.xml的Solr/home地址
②修改各solrHome中solr.xml的host和port
![code](/images/JW-SolrCloud/code.jpg)

**关联zookeeper集群和solr集群**
修改各tomcat中bin/catalina.sh的JAVA-OPTS，关联zookeeper地址
```
# Uncomment the following line to make the umask available when using the
# org.apache.catalina.security.SecurityListener
#JAVA_OPTS="$JAVA_OPTS -Dorg.apache.catalina.security.SecurityListener.UMASK=`umask`"

# ----- Execute The Requested Command -----------------------------------------
JAVA_OPTS="-DzkHost=192.168.25.163:2181,192.168.25.163:2182,192.168.25.163:2183"
# Bugzilla 37848: only output this if we have a TTY
```

**zookeeper集中管理配置文件**
将solrHome中solrCore的conf配置文件上传到zookeeper服务器

进入solr/example/scripts/cloud-scripts/目录下，执行上传
```
./zkcli.sh -zkhost 192.168.25.163:2181,192.168.25.163:2182,192.168.25.163:2183 -cmd upconfig -confdir /usr/local/solr-cloud/solrhome01/collection1/conf -confname myconf
```

eg：连接指定zookeeper
进入zookeeper01/bin目录，
./zkcli.sh server 192.168.25.163:2182

配置完成，启动tomcat/solr即可进入solr后台。

**创建和删除collection索引库**
①官方后台上执行接口。
![添加collection](/images/JW-SolrCloud/添加collection.jpg)
![删除collection](/images/JW-SolrCloud/删除collection.jpg)

![solr后台](/images/JW-SolrCloud/solr后台.jpg)


②**solrj客户端管理集群**
![添加文档](/images/JW-SolrCloud/添加文档.jpg)

![查询文档](/images/JW-SolrCloud/查询文档.jpg)

修改spring配置文件applicationContext-solr.xml，以连接到集群。
![配置](/images/JW-SolrCloud/配置.jpg)

solr中对文档进行添删改查操作同时需要同步索引库，为减少耦合区分业务服务，可使用Activemq消息队列。

e.g. 本文仅供个人笔记使用，借鉴部分网上资料。
