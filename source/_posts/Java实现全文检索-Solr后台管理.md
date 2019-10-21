---
title: Java实现全文检索-Solr后台管理
date: 2019.02.12 16:16:43
tags: JW
categories: JW
---


## 1.1. Solr后台管理

### 1.1.1. 管理界面

![Solrmanager](/images/Java实现全文检索-Solr后台管理/Solrmanager.jpg)


### 1.1.2. Dashboard

仪表盘，显示了该Solr实例开始启动运行的时间、版本、系统资源、jvm等信息。

### 1.1.3. Logging

Solr运行日志信息

### 1.1.4. Cloud

Cloud即SolrCloud，即Solr云（集群），当使用Solr Cloud模式运行时会显示此菜单，如下图是Solr Cloud的管理界面：

![Solrmanager1](/images/Java实现全文检索-Solr后台管理/Solrmanager1.jpg)


### 1.1.5. Core Admin

Solr Core的管理界面。Solr Core 是Solr的一个独立运行实例单位，它可以对外提供索引和搜索服务，一个Solr工程可以运行多个SolrCore（Solr实例），一个Core对应一个索引目录。

添加solrcore：

第一步：复制collection1改名为collection2

第二步：修改core.properties。name=collection2

第三步：重启tomcat

### 1.1.6. java properties

Solr在JVM 运行环境中的属性信息，包括类路径、文件编码、jvm内存设置等信息。

### 1.1.7. Tread Dump

显示Solr Server中当前活跃线程信息，同时也可以跟踪线程运行栈信息。

### 1.1.8. Core selector

选择一个SolrCore进行详细操作，如下：

![Solrmanager2](/images/Java实现全文检索-Solr后台管理/Solrmanager2.jpg)

### 1.1.9. Analysis

![Solrmanager3](/images/Java实现全文检索-Solr后台管理/Solrmanager3.jpg)

通过此界面可以测试索引分析器和搜索分析器的执行情况。

### 1.1.10. Dataimport

可以定义数据导入处理器，从关系数据库将数据导入 到Solr索引库中。

### 1.1.11. Document

通过此菜单可以创建索引、更新索引、删除索引等操作，界面如下：

![Solrmanager4](/images/Java实现全文检索-Solr后台管理/Solrmanager4.jpg)

/update表示更新索引，solr默认根据id（唯一约束）域来更新Document的内容，如果根据id值搜索不到id域则会执行添加操作，如果找到则更新。

### 1.1.12. Query

![Solrmanager5](/images/Java实现全文检索-Solr后台管理/Solrmanager5.jpg)

通过/select执行搜索索引，必须指定“q”查询条件方可搜索。


## 2.1. Solr管理索引库

## 2.1. 添加/更新文档

添加或更新单个文档

![Solrmanager6](/images/Java实现全文检索-Solr后台管理/Solrmanager6.jpg)


## 2.2. 批量导入数据

使用dataimport插件批量导入数据。

第一步：把dataimport插件依赖的jar包添加到solrcore（collection1\lib）中

![Solrmanager7](/images/Java实现全文检索-Solr后台管理/Solrmanager7.jpg)

还需要mysql的数据库驱动。

第二步：配置solrconfig.xml文件，添加一个requestHandler。

```
 <requestHandler name="/dataimport"

class="org.apache.solr.handler.dataimport.DataImportHandler">

 <lst name="defaults">

 <str name="config">data-config.xml</str>

 </lst>

 </requestHandler>
```

第三步：创建一个data-config.xml，保存到collection1\conf\目录下

```
<?xml version="1.0" encoding="UTF-8" ?> 

<dataConfig>  

<dataSource type="JdbcDataSource"  

 driver="com.mysql.jdbc.Driver"

 url="jdbc:mysql://localhost:3306/lucene"

 user="root"

 password="root"/>

<document>  

 <entity name="product" query="SELECT pid,name,catalog_name,price,description,picture FROM products ">

 <field column="pid" name="id"/>

 <field column="name" name="product_name"/>

 <field column="catalog_name" name="product_catalog_name"/>

 <field column="price" name="product_price"/>

 <field column="description" name="product_description"/>

 <field column="picture" name="product_picture"/>

 </entity>

</document>  

</dataConfig>
```

第四步：重启tomcat

![Solrmanager8](/images/Java实现全文检索-Solr后台管理/Solrmanager8.jpg)


第五步：点击“execute”按钮导入数据

**到入数据前会先清空索引库，然后再导入。**

## 2.3. 删除文档

删除索引格式如下：

1） 删除制定ID的索引

```
<delete>

 <id>8</id>

</delete>

<commit/>
```

2） 删除查询到的索引数据

```
<delete>

 <query>product_catalog_name:幽默杂货</query>

</delete>
```

3） 删除所有索引数据

```
 <delete>

 <query>*:*</query>

</delete>
```

## a) 查询索引

通过/select搜索索引，Solr制定一些参数完成不同需求的搜索：

1. q - 查询字符串，必须的，如果查询所有使用*:*。

![Solrmanager9](/images/Java实现全文检索-Solr后台管理/Solrmanager9.jpg)


2. fq - （filter query）过虑查询，作用：在q查询符合结果中同时是fq查询符合的，例如：：

![Solrmanager10](/images/Java实现全文检索-Solr后台管理/Solrmanager10.jpg)

过滤查询价格从1到20的记录。

也可以在“q”查询条件中使用product_price:[1 TO 20]，如下：

![Solrmanager11](/images/Java实现全文检索-Solr后台管理/Solrmanager11.jpg)

也可以使用“*”表示无限，例如：

20以上：product_price:[20 TO *]

20以下：product_price:[* TO 20]

3. sort - 排序，格式：sort=<field name>+<desc|asc>[,<field name>+<desc|asc>]… 。示例：

![Solrmanager12](/images/Java实现全文检索-Solr后台管理/Solrmanager12.jpg)
 按价格降序

4. start - 分页显示使用，开始记录下标，从0开始

5. rows - 指定返回结果最多有多少条记录，配合start来实现分页。

![Solrmanager13](/images/Java实现全文检索-Solr后台管理/Solrmanager13.jpg)


显示前10条。

6. fl - 指定返回那些字段内容，用逗号或空格分隔多个。

![Solrmanager14](/images/Java实现全文检索-Solr后台管理/Solrmanager14.jpg)


显示商品图片、商品名称、商品价格

7. df-指定一个搜索Field

![Solrmanager15](/images/Java实现全文检索-Solr后台管理/Solrmanager15.jpg)

也可以在SolrCore目录 中conf/solrconfig.xml文件中指定默认搜索Field，指定后就可以直接在“q”查询条件中输入关键字。

![Solrmanager16](/images/Java实现全文检索-Solr后台管理/Solrmanager16.jpg)

8. wt - (writer type)指定输出格式，可以有 xml, json, php, phps, 后面 solr 1.3增加的，要用通知我们，因为默认没有打开。

9. hl 是否高亮 ,设置高亮Field，设置格式前缀和后缀。

![Solrmanager17](/images/Java实现全文检索-Solr后台管理/Solrmanager17.jpg)



e.g. 本文仅供个人笔记使用，借鉴部分网上资料。
