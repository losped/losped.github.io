---
title: JW-Redis
date: 2019.04.19 16:58:44
tags: JW
categories: JW
---


Redis是用C语言开发的高性能键值对数据库。一个redis实例可以包括多个数据库，一个实例最多可提供16个数据库，下标0到15。
主要应用场景有：
缓存（数据查询，短连接，详情内容等）大多数
任务队列（12306）
应用排行榜
网站访问统计
数据过期管理
聊天室好友列表
分布式集群中的session分离

支持的键值数据类型有：
1. 字符串类型
2. 散列类型
3. 列表类型
4. 集合类型
5. 有序集合类型

## Redis的启动
**前端模式**
使用bin/redis-server启动，启动完成后不能再进行其他操作 - -。不推荐使用
**后端模式**
修改 /usr/local/redis/redis.conf配置文件，daemonize yes以后端模式启动。
![1.jpg](https://upload-images.jianshu.io/upload_images/14588055-a80a34bd62afe439.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
启动时指定配置文件，redis默认端口6379
```
cd /usr/local/redis/
./bin/redis-server ./redis.conf
```

## Redis停止
```
cd /usr/local/redis
./bin/redis-cli shutdown
```

## 连接客户端
![redis](/images/JW-Redis/redis.jpg)

## redis客户端命令

选择数据库
![redis1](/images/JW-Redis/redis1.jpg)

ping，如果正常回复会收到PONG
![redis2](/images/JW-Redis/redis2.jpg)

get/set
![redis3](/images/JW-Redis/redis3.jpg)

del
![redis4](/images/JW-Redis/redis4.jpg)

key*，获取当前库中所有key值

## 数据类型相关命令
**存储String**
字符串类型的Value中最多可容纳的数据长度为512M，二进制安全
赋值：set key value
取值：get key
先取值再赋值：getset key value

**存储Hash**
可看成具有String Key和String Value的Map容器， 每一个Hash可以存储4594967295个键值对
赋值：hset key fileld value
赋多个值：hmset key fileld value[fileld1 value1...]
取值：hget key fileld
取多个值：hmget key fileld fileld1...
获取key中所有fileld value：hgetall key

删除一到多字段：hdel key field[field1...]
删除整个list：del key

**存储Hash**
按照插入顺序排序的字符串链表。List中可以包含的最大元素数量是4294967295。
![redis5](/images/JW-Redis/redis5.jpg)
![redis6](/images/JW-Redis/redis6.jpg)

头部弹出元素：lpop key
尾部弹出元素：rpop key
获取列表中元素个数：llen key

**存储Set**
无序。Set不允许出现重复元素。Set可以包含的最大元素数量是4294967295。
![redis7](/images/JW-Redis/redis7.jpg)
![redis8](/images/JW-Redis/redis8.jpg)

**存储Sorted-Set**
Sorted-Set和Set类型极为相似，差别是Sorted-Set每个成员都有一个分数（Score）与他关联。通过分数进行从小到大排列，Score是可以重复的。
![redis9](/images/JW-Redis/redis9.jpg)
![redis10](/images/JW-Redis/redis10.jpg)
![redis11](/images/JW-Redis/redis11.jpg)

## 消息订阅和发布
![redis12](/images/JW-Redis/redis12.jpg)
![redis13](/images/JW-Redis/redis13.jpg)

## Jedis
**单实例示例**
```
public void testJedisSingle(){
     //1.设置IP地址和端口
     Jedis jedis = new Jedis("192.168.137.128", 6379);
     //2.设置数据
     jedis.set("name", "fan");
     //3.获得数据
     jedis.get("name");
     //4.释放资源
     jedis.close();
}
```

连接超时处理：
![redis14](/images/JW-Redis/redis14.jpg)

**连接池连接**
```
public void testJedisPool(){
     //1.获得连接池配置对象
     JedisPoolConfig config = new JedisPoolConfig();
     //2.最大连接数
     config.setMaxTotal(30);
     //3.最大空闲连接数
     config.setMaxidle(10);
     //4.获得连接池
     JedisPool jedisPool = new JedisPool(config, "192.168.137.128", 6379");

     //5.获得核心对象
     Jedis jedis = null;
     try{
     jedis = pool.getResource();

     //2.设置数据
     jedis.set("name", "fan");
     //3.获得数据
     jedis.get("name");
     //4.释放资源
     jedis.close();
     }
     catch (Exception e){
     e.printException();
     }
     finally{
       if(jedis != null){
         jedis.close();
       }
       //虚拟机关闭时释放资源
       if(jedisPool != null){
           jedisPool.close();
       }
     }
}
```

使用jedis缓存示例
applicationContext-redis.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.2.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.2.xsd
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.2.xsd">

	<!-- 连接redis单机版 -->
	<bean id="jedisClientPool" class="cn.e3mall.common.jedis.JedisClientPool">
		<property name="jedisPool" ref="jedisPool"></property>
	</bean>
	<bean id="jedisPool" class="redis.clients.jedis.JedisPool">
		<constructor-arg name="host" value="192.168.25.162"/>
		<constructor-arg name="port" value="6379"/>
	</bean>

	<!-- 连接redis集群 -->
	<bean id="jedisClientCluster" class="cn.e3mall.common.jedis.JedisClientCluster">
		<property name="jedisCluster" ref="jedisCluster"/>
	</bean>
	<bean id="jedisCluster" class="redis.clients.jedis.JedisCluster">
		<constructor-arg name="nodes">
			<set>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.25.162"></constructor-arg>
					<constructor-arg name="port" value="7001"></constructor-arg>
				</bean>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.25.162"></constructor-arg>
					<constructor-arg name="port" value="7002"></constructor-arg>
				</bean>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.25.162"></constructor-arg>
					<constructor-arg name="port" value="7003"></constructor-arg>
				</bean>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.25.162"></constructor-arg>
					<constructor-arg name="port" value="7004"></constructor-arg>
				</bean>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.25.162"></constructor-arg>
					<constructor-arg name="port" value="7005"></constructor-arg>
				</bean>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.25.162"></constructor-arg>
					<constructor-arg name="port" value="7006"></constructor-arg>
				</bean>
			</set>
		</constructor-arg>
	</bean>
</beans>
```

JedisClientCluster.java封装类
```
public class JedisClientCluster implements JedisClient {

	private JedisCluster jedisCluster;


	public JedisCluster getJedisCluster() {
		return jedisCluster;
	}

	public void setJedisCluster(JedisCluster jedisCluster) {
		this.jedisCluster = jedisCluster;
	}

	@Override
	public String set(String key, String value) {
		return jedisCluster.set(key, value);
	}

	@Override
	public String get(String key) {
		return jedisCluster.get(key);
	}

	@Override
	public Boolean exists(String key) {
		return jedisCluster.exists(key);
	}

	@Override
	public Long expire(String key, int seconds) {
		return jedisCluster.expire(key, seconds);
	}

	@Override
	public Long ttl(String key) {
		return jedisCluster.ttl(key);
	}

	@Override
	public Long incr(String key) {
		return jedisCluster.incr(key);
	}

	@Override
	public Long hset(String key, String field, String value) {
		return jedisCluster.hset(key, field, value);
	}

	@Override
	public String hget(String key, String field) {
		return jedisCluster.hget(key, field);
	}

	@Override
	public Long hdel(String key, String... field) {
		return jedisCluster.hdel(key, field);
	}

	@Override
	public Boolean hexists(String key, String field) {
		return jedisCluster.hexists(key, field);
	}

	@Override
	public List<String> hvals(String key) {
		return jedisCluster.hvals(key);
	}

	@Override
	public Long del(String key) {
		return jedisCluster.del(key);
	}

}
```

JedisClientPool.java封装类
```
public class JedisClientPool implements JedisClient {

	private JedisPool jedisPool;

	public JedisPool getJedisPool() {
		return jedisPool;
	}

	public void setJedisPool(JedisPool jedisPool) {
		this.jedisPool = jedisPool;
	}

	@Override
	public String set(String key, String value) {
		Jedis jedis = jedisPool.getResource();
		String result = jedis.set(key, value);
		jedis.close();
		return result;
	}

	@Override
	public String get(String key) {
		Jedis jedis = jedisPool.getResource();
		String result = jedis.get(key);
		jedis.close();
		return result;
	}

	@Override
	public Boolean exists(String key) {
		Jedis jedis = jedisPool.getResource();
		Boolean result = jedis.exists(key);
		jedis.close();
		return result;
	}

	@Override
	public Long expire(String key, int seconds) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.expire(key, seconds);
		jedis.close();
		return result;
	}

	@Override
	public Long ttl(String key) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.ttl(key);
		jedis.close();
		return result;
	}

	@Override
	public Long incr(String key) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.incr(key);
		jedis.close();
		return result;
	}

	@Override
	public Long hset(String key, String field, String value) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.hset(key, field, value);
		jedis.close();
		return result;
	}

	@Override
	public String hget(String key, String field) {
		Jedis jedis = jedisPool.getResource();
		String result = jedis.hget(key, field);
		jedis.close();
		return result;
	}

	@Override
	public Long hdel(String key, String... field) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.hdel(key, field);
		jedis.close();
		return result;
	}

	@Override
	public Boolean hexists(String key, String field) {
		Jedis jedis = jedisPool.getResource();
		Boolean result = jedis.hexists(key, field);
		jedis.close();
		return result;
	}

	@Override
	public List<String> hvals(String key) {
		Jedis jedis = jedisPool.getResource();
		List<String> result = jedis.hvals(key);
		jedis.close();
		return result;
	}

	@Override
	public Long del(String key) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.del(key);
		jedis.close();
		return result;
	}

}
```

JedisClientTest.java
```
	@Test
	public void testJedisClient() throws Exception {
		//初始化spring容器
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring/applicationContext-redis.xml");
		//从容器中获得JedisClient对象
		JedisClient jedisClient = applicationContext.getBean(JedisClient.class);
		jedisClient.set("mytest", "jedisClient");
		String string = jedisClient.get("mytest");
		System.out.println(string);


	}
```

e.g. 本文仅供个人笔记使用，借鉴部分网上资料。
