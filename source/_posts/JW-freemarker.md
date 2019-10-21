---
title: JW-freemarker
date: 2019.04.22 17:28:58
tags: JW
categories: JW
---


freemarker是一种Java语言编写的模板引擎，一般用于表现层的实现，也可以转成JSP，XML或JAVA等。目前一般用于做静态页面或页面展示。

![as](/images/JW-freemarker/freemarker.jpg)

**添加依赖**
```
	<properties>
		<freemarker.version>2.3.23</freemarker.version>
	</properties>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.freemarker</groupId>
				<artifactId>freemarker</artifactId>
				<version>${freemarker.version}</version>
			</dependency>
		</dependencies>
	</dependencyManagement>
```

**模板Template**
```
<html>
<head>
	<title>student</title>
</head>
<body>
	学生信息：<br>
	学号：${student.id}&nbsp;&nbsp;&nbsp;&nbsp;
	姓名：${student.name}&nbsp;&nbsp;&nbsp;&nbsp;
	年龄：${student.age}&nbsp;&nbsp;&nbsp;&nbsp;
	家庭住址：${student.address}<br>
	学生列表：
	<table border="1">
		<tr>
			<th>序号</th>
			<th>学号</th>
			<th>姓名</th>
			<th>年龄</th>
			<th>家庭住址</th>
		</tr>
		<#list stuList as stu>
		<#if stu_index % 2 == 0>
		<tr bgcolor="red">
		<#else>
		<tr bgcolor="green">
		</#if>
			<td>${stu_index}</td> <!--> 循环下标 <-->
			<td>${stu.id}</td>
			<td>${stu.name}</td>
			<td>${stu.age}</td>
			<td>${stu.address}</td>
		</tr>
		</#list>
	</table>
	<br>
	<!-- 解析可以使用?date,?time,?datetime,?string(parten)-->
	当前日期：${date?string("yyyy/MM/dd HH:mm:ss")}<br>
	null值的处理：${val!"val的值为null"}<br>
	判断val的值是否为null：<br>
	<#if val??>
	val中有内容
	<#else>
	val的值为null
	</#if>
	引用模板测试：<br>
	<#include "hello.ftl">
</body>
</html>
```

**填充数据集生成静态页面**
```
public class FreeMarkerTest {

	@Test
	public void testFreeMarker() throws Exception {
		//1、创建一个模板文件
		//2、创建一个Configuration对象
		Configuration configuration = new Configuration(Configuration.getVersion());
		//3、设置模板文件保存的目录
		configuration.setDirectoryForTemplateLoading(new File("D:/workspaces-itcast/JavaEE32/e3-item-web/src/main/webapp/WEB-INF/ftl"));
		//4、模板文件的编码格式，一般就是utf-8
		configuration.setDefaultEncoding("utf-8");
		//5、加载一个模板文件，创建一个模板对象。
//		Template template = configuration.getTemplate("hello.ftl");
		Template template = configuration.getTemplate("student.ftl");
		//6、创建一个数据集。可以是pojo也可以是map。推荐使用map
		Map data = new HashMap<>();
		data.put("hello", "hello freemarker!");
		//创建一个pojo对象
		Student student = new Student(1, "小明", 18, "回龙观");
		data.put("student", student);
		//添加一个list
		List<Student> stuList = new ArrayList<>();
		stuList.add(new Student(1, "小明1", 18, "回龙观"));
		stuList.add(new Student(2, "小明2", 19, "回龙观"));
		stuList.add(new Student(3, "小明3", 20, "回龙观"));
		stuList.add(new Student(4, "小明4", 21, "回龙观"));
		stuList.add(new Student(5, "小明5", 22, "回龙观"));
		stuList.add(new Student(6, "小明6", 23, "回龙观"));
		stuList.add(new Student(7, "小明7", 24, "回龙观"));
		stuList.add(new Student(8, "小明8", 25, "回龙观"));
		stuList.add(new Student(9, "小明9", 26, "回龙观"));
		data.put("stuList", stuList);
		//添加日期类型
		data.put("date", new Date());
		//null值的测试
		data.put("val", "123");
		//7、创建一个Writer对象，指定输出文件的路径及文件名。
//		Writer out = new FileWriter(new File("D:/temp/JavaEE32/freemarker/hello.txt"));
		Writer out = new FileWriter(new File("D:/temp/JavaEE32/freemarker/student.html"));
		//8、生成静态页面
		template.process(data, out);
		//9、关闭流
		out.close();
	}
}
```

## 整合springmvc
pom.xml中需要引入兼容包
```
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context-support</artifactId>
		</dependency>
```

springmvc.xml配置
```
	<!-- 配置freemarker -->
	<bean id="freemarkerConfig"
		class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
		<property name="templateLoaderPath" value="/WEB-INF/ftl/" />
		<property name="defaultEncoding" value="UTF-8" />
	</bean>
```

Controller.java
```
@Controller
public class HtmlGenController {

	@Autowired
	private FreeMarkerConfigurer freeMarkerConfigurer;

	@RequestMapping("/genhtml")
	@ResponseBody
	public String genHtml() throws Exception {
		Configuration configuration = freeMarkerConfigurer.getConfiguration();
		//加载模板对象
		Template template = configuration.getTemplate("hello.ftl");
		//创建一个数据集
		Map data = new HashMap<>();
		data.put("hello", 123456);
		//指定文件输出的路径及文件名
		Writer out = new FileWriter(new File("D:/temp/JavaEE32/freemarker/hell2.html"));
		//输出文件
		template.process(data, out);
		//关闭流
		out.close();

		return "OK";
	}
}
```
