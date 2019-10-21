---
title: 探索SSM（一）-MyBatis
date: 2019.02.13 11:45:08
tags: JW
categories: JW
---


**SSM框架在我看来可总结为**
Spring：MVC代替Servlet
Mybatis：Dao中数据库操作，简化数据库操作，专注于xml中编辑sql语句
Spring：所有对象转为Bean，统一加载。依赖注入（DI）和控制反转（IOC），AOP面向切面编程。AOP 可以进行权限校验 ,日志记录 ,性能监控 ,事务控制 .

## MyBatis（整合Spring）

**5.3.3.1. SqlMapConfig.xml**
配置文件是SqlMapConfig.xml，如下：
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<!-- 设置别名 -->
	<typeAliases>
		<!-- 2. 指定扫描包，会把包内所有的类都设置别名，别名的名称就是类名，大小写不敏感 -->
		<package name="cn.itcast.mybatis.pojo" />
	</typeAliases>

</configuration>
```

**5.3.3.2. applicationContext.xml**
SqlSessionFactoryBean属于mybatis-spring这个jar包
对于spring来说，mybatis是另外一个架构，需要整合jar包。

applicationContext.xml，配置内容如下
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd">

   <!-- 加载配置文件 -->
   <context:property-placeholder location="classpath:db.properties" />

	<!-- 数据库连接池 -->
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
		destroy-method="close">
		<property name="driverClassName" value="${jdbc.driver}" />
		<property name="url" value="${jdbc.url}" />
		<property name="username" value="${jdbc.username}" />
		<property name="password" value="${jdbc.password}" />
		<property name="maxActive" value="10" />
		<property name="maxIdle" value="5" />
	</bean>

	<!-- 配置SqlSessionFactory -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<!-- 配置mybatis核心配置文件 -->
		<property name="configLocation" value="classpath:SqlMapConfig.xml" />
		<!-- 配置数据源 -->
		<property name="dataSource" ref="dataSource" />
	</bean>
</beans>
```

5.3.3.3. db.properties
```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8
jdbc.username=root
jdbc.password=root
```

5.3.3.4. log4j.properties
```
# Global logging configuration
log4j.rootLogger=DEBUG, stdout
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

**5.4.3. Mapper代理形式开发dao**
5.4.3.1. 实现Mapper.xml
编写UserMapper.xml配置文件，如下：
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.itcast.mybatis.mapper.UserMapper">
	<!-- 根据用户id查询 -->
	<select id="queryUserById" parameterType="int" resultType="user">
		select * from user where id = #{id}
	</select>

	<!-- 根据用户名模糊查询用户 -->
	<select id="queryUserByUsername" parameterType="string"
		resultType="user">
		select * from user where username like '%${value}%'
	</select>

	<!-- 添加用户 -->
	<insert id="saveUser" parameterType="user">
		<selectKey keyProperty="id" keyColumn="id" order="AFTER"
			resultType="int">
			select last_insert_id()
		</selectKey>
		insert into user
		(username,birthday,sex,address) values
		(#{username},#{birthday},#{sex},#{address})
	</insert>
</mapper>
```

5.4.3.2. 实现UserMapper接口
```
public interface UserMapper {
	/**
	 * 根据用户id查询
	 *
	 * @param id
	 * @return
	 */
	User queryUserById(int id);

	/**
	 * 根据用户名模糊查询用户
	 *
	 * @param username
	 * @return
	 */
	List<User> queryUserByUsername(String username);

	/**
	 * 添加用户
	 *
	 * @param user
	 */
	void saveUser(User user);
}
```

创建POJO对象
```
public class User {
	private int id;
	private String username;// 用户姓名
	private String sex;// 性别
	private Date birthday;// 生日
	private String address;// 地址

get/set。。。
}
```


5.4.3.3. 方式一：配置mapper代理
在applicationContext.xml添加配置
MapperFactoryBean也是属于mybatis-spring整合包
```
<!-- Mapper代理的方式开发方式一，配置Mapper代理对象 -->
<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
	<!-- 配置Mapper接口 -->
	<property name="mapperInterface" value="cn.itcast.mybatis.mapper.UserMapper" />
	<!-- 配置sqlSessionFactory -->
	<property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>

```

5.4.3.5. 方式二：扫描包形式配置mapper（常用）
```
<!-- Mapper代理的方式开发方式二，扫描包方式配置代理 -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
	<!-- 配置Mapper接口 -->
	<property name="basePackage" value="cn.itcast.mybatis.mapper" />
</bean>
```

**4.3.2. Mapper.xml解析**
4.2.1. 方法一：使用resultType
在UserMapper.xml添加sql，如下
```
<!-- 查询订单，同时包含用户数据 -->
<select id="queryOrderUser" resultType="orderUser">
	SELECT
	o.id,
	o.user_id
	userId,
	o.number,
	o.createtime,
	o.note,
	u.username,
	u.address
	FROM
	`order` o
	LEFT JOIN `user` u ON o.user_id = u.id
</select>
```
![mybatis](/images/探索SSM（一）-MyBatis/mybatis.jpg)


4.2.2. 方法二：使用resultMap
①一对多查询
```
<resultMap type="user" id="userOrderResultMap">
	<id property="id" column="id" />
	<result property="username" column="username" />
	<result property="birthday" column="birthday" />
	<result property="sex" column="sex" />
	<result property="address" column="address" />

	<!-- 配置一对多的关系 -->
	<collection property="orders" javaType="list" ofType="order">
		<!-- 配置主键，是关联Order的唯一标识 -->
		<id property="id" column="oid" />
		<result property="number" column="number" />
		<result property="createtime" column="createtime" />
		<result property="note" column="note" />
	</collection>
</resultMap>

<!-- 一对多关联，查询订单同时查询该用户下的订单 -->
<select id="queryUserOrder" resultMap="userOrderResultMap">
	SELECT
	u.id,
	u.username,
	u.birthday,
	u.sex,
	u.address,
	o.id oid,
	o.number,
	o.createtime,
	o.note
	FROM
	`user` u
	LEFT JOIN `order` o ON u.id = o.user_id
</select>
```

对应POJO
![mybatis1](/images/探索SSM（一）-MyBatis/mybatis1.jpg)



②一对一查询
```
<resultMap type="order" id="orderUserResultMap">
	<id property="id" column="id" />
	<result property="userId" column="user_id" />
	<result property="number" column="number" />
	<result property="createtime" column="createtime" />
	<result property="note" column="note" />

	<!-- association ：配置一对一属性 -->
	<!-- property:order里面的User属性名 -->
	<!-- javaType:属性类型 -->
	<association property="user" javaType="user">
		<!-- id:声明主键，表示user_id是关联查询对象的唯一标识-->
		<id property="id" column="user_id" />
		<result property="username" column="username" />
		<result property="address" column="address" />
	</association>

</resultMap>

<!-- 一对一关联，查询订单，订单内部包含用户属性 -->
<select id="queryOrderUserResultMap" resultMap="orderUserResultMap">
	SELECT
	o.id,
	o.user_id,
	o.number,
	o.createtime,
	o.note,
	u.username,
	u.address
	FROM
	`order` o
	LEFT JOIN `user` u ON o.user_id = u.id
</select>

```

对应POJO
![mybatis2](/images/探索SSM（一）-MyBatis/mybatis2.jpg)

**4.2.3. 动态sql-标签使用**

3.1. If标签
```
<!-- 根据条件查询用户 -->
<select id="queryUserByWhere" parameterType="user" resultType="user">
	SELECT id, username, birthday, sex, address FROM `user`
	WHERE 1=1
	<if test="sex != null and sex != ''">
		AND sex = #{sex}
	</if>
	<if test="username != null and username != ''">
		AND username LIKE
		'%${username}%'
	</if>
</select>
```

3.2. Where标签
```
<!-- 根据条件查询用户 -->
<select id="queryUserByWhere" parameterType="user" resultType="user">
	SELECT id, username, birthday, sex, address FROM `user`
<!-- where标签可以自动添加where，同时处理sql语句中第一个and关键字 -->
	<where>
		<if test="sex != null">
			AND sex = #{sex}
		</if>
		<if test="username != null and username != ''">
			AND username LIKE
			'%${username}%'
		</if>
	</where>
</select>

```

3.3. Sql片段
```
<!-- 根据条件查询用户 -->
<select id="queryUserByWhere" parameterType="user" resultType="user">
	<!-- SELECT id, username, birthday, sex, address FROM `user` -->
	<!-- 使用include标签加载sql片段；refid是sql片段id -->
	SELECT <include refid="userFields" /> FROM `user`
	<!-- where标签可以自动添加where关键字，同时处理sql语句中第一个and关键字 -->
	<where>
		<if test="sex != null">
			AND sex = #{sex}
		</if>
		<if test="username != null and username != ''">
			AND username LIKE
			'%${username}%'
		</if>
	</where>
</select>

<!-- 声明sql片段 -->
<sql id="userFields">
	id, username, birthday, sex, address
</sql>

```

3.4. foreach标签
```
<!-- 根据ids查询用户 -->
<select id="queryUserByIds" parameterType="queryVo" resultType="user">
	SELECT * FROM `user`
	<where>
		<!-- foreach标签，进行遍历 -->
		<!-- collection：遍历的集合，这里是QueryVo的ids属性 -->
		<!-- item：遍历的项目，可以随便写，，但是和后面的#{}里面要一致 -->
		<!-- open：在前面添加的sql片段 -->
		<!-- close：在结尾处添加的sql片段 -->
		<!-- separator：指定遍历的元素之间使用的分隔符 -->
		<foreach collection="ids" item="item" open="id IN (" close=")"
			separator=",">
			#{item}
		</foreach>
	</where>
</select>

```

**逆向工程**

mybatis-generator-core-1.3.2配置文件参数
在generatorConfig.xml中配置Mapper生成的详细信息：
注意修改以下几点:
1.	修改要生成的数据库表
2.	pojo文件所在包路径
3.	Mapper所在的包路径

配置文件如下:
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
	<context id="testTables" targetRuntime="MyBatis3">
		<commentGenerator>
			<!-- 是否去除自动生成的注释 true：是 ： false:否 -->
			<property name="suppressAllComments" value="true" />
		</commentGenerator>
		<!--数据库连接的信息：驱动类、连接地址、用户名、密码 -->
		<jdbcConnection driverClass="com.mysql.jdbc.Driver"
			connectionURL="jdbc:mysql://localhost:3306/mybatis" userId="root" password="root">
		</jdbcConnection>
		<!-- <jdbcConnection driverClass="oracle.jdbc.OracleDriver" connectionURL="jdbc:oracle:thin:@127.0.0.1:1521:yycg"
			userId="yycg" password="yycg"> </jdbcConnection> -->

		<!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，为 true时把JDBC DECIMAL
			和 NUMERIC 类型解析为java.math.BigDecimal -->
		<javaTypeResolver>
			<property name="forceBigDecimals" value="false" />
		</javaTypeResolver>

		<!-- targetProject:生成PO类的位置 -->
		<javaModelGenerator targetPackage="cn.itcast.ssm.po"
			targetProject=".\src">
			<!-- enableSubPackages:是否让schema作为包的后缀 -->
			<property name="enableSubPackages" value="false" />
			<!-- 从数据库返回的值被清理前后的空格 -->
			<property name="trimStrings" value="true" />
		</javaModelGenerator>
		<!-- targetProject:mapper映射文件生成的位置 -->
		<sqlMapGenerator targetPackage="cn.itcast.ssm.mapper"
			targetProject=".\src">
			<!-- enableSubPackages:是否让schema作为包的后缀 -->
			<property name="enableSubPackages" value="false" />
		</sqlMapGenerator>
		<!-- targetPackage：mapper接口生成的位置 -->
		<javaClientGenerator type="XMLMAPPER"
			targetPackage="cn.itcast.ssm.mapper" targetProject=".\src">
			<!-- enableSubPackages:是否让schema作为包的后缀 -->
			<property name="enableSubPackages" value="false" />
		</javaClientGenerator>
		<!-- 指定数据库表 -->
		<table schema="" tableName="user"></table>
		<table schema="" tableName="order"></table>
	</context>
</generatorConfiguration>
```

生成代码组成
![mybatis3](/images/探索SSM（一）-MyBatis/mybatis3.jpg)

Example ：条件对象，可根据需求自行编辑
insertByExample：表示根据条件插入
insertSelective：表示选择性插入
deleteByPrimaryKey(Integer id)：根据主键删除
updateByExampleSelective(@Param("record")) Orders record)：可输入多参数

方法调用
```
public class UserMapperTest {
	private ApplicationContext context;

	@Before
	public void setUp() throws Exception {
		this.context = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
	}

	@Test
	public void testInsert() {
		// 获取Mapper
		UserMapper userMapper = this.context.getBean(UserMapper.class);

		User user = new User();
		user.setUsername("曹操");
		user.setSex("1");
		user.setBirthday(new Date());
		user.setAddress("三国");

		userMapper.insert(user);
	}

	@Test
	public void testSelectByExample() {
		// 获取Mapper
		UserMapper userMapper = this.context.getBean(UserMapper.class);

		// 创建User对象扩展类，用户设置查询条件
		UserExample example = new UserExample();
		example.createCriteria().andUsernameLike("%张%");

		// 查询数据
		List<User> list = userMapper.selectByExample(example);

		System.out.println(list.size());
	}

	@Test
	public void testSelectByPrimaryKey() {
		// 获取Mapper
		UserMapper userMapper = this.context.getBean(UserMapper.class);

		User user = userMapper.selectByPrimaryKey(1);
		System.out.println(user);
	}
}
```

注意：
1. 逆向工程生成的代码只能做单表查询
2. 不能在生成的代码上进行扩展，因为如果数据库变更，需要重新使用逆向工程生成代码，原来编写的代码就被覆盖了。
3. 一张表会生成4个文件



e.g. 本文仅供个人笔记使用，借鉴部分网上资料。
