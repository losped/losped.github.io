---
title: 探索SSM（三）-Spring MVC
date: 2019.02.14 16:46:50
tags: JW
categories: JW
---


## springmvc.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

	<!-- 配置controller扫描包 -->
	<context:component-scan base-package="cn.itcast.springmvc.controller" />

</beans>
```

##web.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	id="WebApp_ID" version="2.5">
	<display-name>springmvc-first</display-name>
	<welcome-file-list>
		<welcome-file>index.html</welcome-file>
		<welcome-file>index.htm</welcome-file>
		<welcome-file>index.jsp</welcome-file>
		<welcome-file>default.html</welcome-file>
		<welcome-file>default.htm</welcome-file>
		<welcome-file>default.jsp</welcome-file>
	</welcome-file-list>

	<!-- 配置SpringMVC前端控制器 -->
	<servlet>
		<servlet-name>springmvc-first</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<!-- 指定SpringMVC配置文件 -->
		<!-- SpringMVC的配置文件的默认路径是/WEB-INF/${servlet-name}-servlet.xml -->
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc.xml</param-value>
		</init-param>
	</servlet>

	<servlet-mapping>
		<servlet-name>springmvc-first</servlet-name>
		<!-- 设置所有以action结尾的请求进入SpringMVC -->
		<url-pattern>*.action</url-pattern>
	</servlet-mapping>

	<!-- 解决post乱码问题 -->
	<filter>
		<filter-name>encoding</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<!-- 设置编码参是UTF8 -->
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>encoding</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
</web-app>
```

若get请求中文参数出现乱码解决方法
方法一：
```
<Connector URIEncoding="utf-8" connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>
```
方法二：
```
//ISO8859-1是tomcat默认编码，需要将tomcat编码后的内容按utf-8编码
String userName = new String(request.getParamter("userName").getBytes("ISO8859-1"),"utf-8")
```


##Spring mvc框架结构
![mvc](/images/探索SSM（三）-Spring MVC/mvc.jpg)

说明：在springmvc的各个组件中，处理器映射器、处理器适配器、视图解析器称为springmvc的三大组件。
需要用户开发的组件有handler、view

**3.5. 组件扫描器**
```
<!-- 配置controller扫描包，多个包之间用,分隔 -->
<context:component-scan base-package="cn.itcast.springmvc.controller" />
```
**3.6. 注解映射器和适配器**

3.6.1. 配置处理器映射器（弃用）
```
<!-- 配置处理器映射器 -->
<bean
	class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping" />
```
注解描述：
@RequestMapping：定义请求url到处理器功能方法的映射

3.6.2. 配置处理器适配器（弃用）
```
<!-- 配置处理器适配器 -->
<bean
	class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter" />
```

3.6.3. 注解驱动（常规）
直接配置处理器映射器和处理器适配器比较麻烦，可以使用注解驱动来加载。
```
<!-- 注解驱动 -->
<mvc:annotation-driven />
```

**3.7. 视图解析器**
```
	<!-- Example: prefix="/WEB-INF/jsp/", suffix=".jsp", viewname="test" ->
		"/WEB-INF/jsp/test.jsp" -->
	<!-- 配置视图解析器 -->
	<bean
		class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<!-- 配置逻辑视图的前缀 -->
		<property name="prefix" value="/WEB-INF/jsp/" />
		<!-- 配置逻辑视图的后缀 -->
		<property name="suffix" value=".jsp" />
	</bean>
```

modelAndView中指定视图名
```
// @RequestMapping：里面放的是请求的url，和用户请求的url进行匹配
// action可以写也可以不写
@RequestMapping("/itemList.action")
public ModelAndView queryItemList() {
	// 创建页面需要显示的商品数据
	List<Item> list = new ArrayList<>();
	list.add(new Item(1, "1华为 荣耀8", 2399, new Date(), "质量好！1"));
	list.add(new Item(2, "2华为 荣耀8", 2399, new Date(), "质量好！2"));

	// 创建ModelAndView，用来存放数据和视图
	ModelAndView modelAndView = new ModelAndView();
	// 设置数据到模型中
	modelAndView.addObject("itemList", list);
	// 设置视图jsp，需要设置视图的物理地址
	// modelAndView.setViewName("/WEB-INF/jsp/itemList.jsp");
	// 配置好视图解析器前缀和后缀，这里只需要设置逻辑视图就可以了。
	// 视图解析器根据前缀+逻辑视图名+后缀拼接出来物理路径
	modelAndView.setViewName("itemList");
	return modelAndView;
}
```

##参数绑定
**6.1.6. 默认支持的参数类型**
HttpServletRequest：通过request对象获取请求信息
HttpServletResponse：通过response处理响应信息
HttpSession：通过session对象得到session中存放的对象

**6.1.7. 返回类型**
①返回ModelAndView
②返回void
1、使用request,forward转发页面，如下：
```
request.getRequestDispatcher("页面路径").forward(request, response);
request.getRequestDispatcher("/WEB-INF/jsp/success.jsp").forward(request, response);
```
2、使用response，Redirect重定向页面：
```
response.sendRedirect("url")
response.sendRedirect("/springmvc-web2/itemEdit.action");
```
3、可以通过response指定响应结果，例如响应json数据如下：
```
response.getWriter().print("{\"abc\":123}");
```
③返回字符串
1、逻辑视图名
2、Redirect重定向
```
/**
 * 更新商品
 *
 * @param item
 * @return
 */
@RequestMapping("updateItem")
public String updateItemById(Item item) {
	// 更新商品
	this.itemService.updateItemById(item);

	// 修改商品成功后，重定向到商品编辑页面
	// 重定向后浏览器地址栏变更为重定向的地址，
	// 重定向相当于执行了新的request和response，所以之前的请求参数都会丢失
	// 如果要指定请求参数，需要在重定向的url后面添加 ?itemId=1 这样的请求参数
	return "redirect:/itemEdit.action?itemId=" + item.getId();
}
```
3、forward转发
```
/**
 * 更新商品
 *
 * @param item
 * @return
 */
@RequestMapping("updateItem")
public String updateItemById(Item item) {
	// 更新商品
	this.itemService.updateItemById(item);

	// 修改商品成功后，重定向到商品编辑页面
	// 重定向后浏览器地址栏变更为重定向的地址，
	// 重定向相当于执行了新的request和response，所以之前的请求参数都会丢失
	// 如果要指定请求参数，需要在重定向的url后面添加 ?itemId=1 这样的请求参数
	// return "redirect:/itemEdit.action?itemId=" + item.getId();

	// 修改商品成功后，继续执行另一个方法
	// 使用转发的方式实现。转发后浏览器地址栏还是原来的请求地址，
	// 转发并没有执行新的request和response，所以之前的请求参数都存在
	return "forward:/itemEdit.action";

}
//结果转发到editItem.action，request可以带过去
return "forward: /itemEdit.action";
```

Model
如果使用Model则可以不使用ModelAndView对象，Model对象可以向页面传递数据，View对象则可以使用String返回值替代。
```
/**
 * 根据id查询商品,使用Model
 *
 * @param request
 * @param model
 * @return
 */
@RequestMapping("/itemEdit")
public String queryItemById(HttpServletRequest request, Model model) {
	// 从request中获取请求参数
	String strId = request.getParameter("id");
	Integer id = Integer.valueOf(strId);

	// 根据id查询商品数据
	Item item = this.itemService.queryItemById(id);

	// 把结果传递给页面
	// ModelAndView modelAndView = new ModelAndView();
	// 把商品数据放在模型中
	// modelAndView.addObject("item", item);
	// 设置逻辑视图
	// modelAndView.setViewName("itemEdit");

	// 把商品数据放在模型中
	model.addAttribute("item", item);

	return "itemEdit";
}
```

ModelMap（弃用。跟Modle效果一样- -）
```
/**
 * 根据id查询商品,使用ModelMap
 *
 * @param request
 * @param model
 * @return
 */
@RequestMapping("/itemEdit")
public String queryItemById(HttpServletRequest request, ModelMap model) {
	// 从request中获取请求参数
	String strId = request.getParameter("id");
	Integer id = Integer.valueOf(strId);

	// 根据id查询商品数据
	Item item = this.itemService.queryItemById(id);

	// 把结果传递给页面
	// ModelAndView modelAndView = new ModelAndView();
	// 把商品数据放在模型中
	// modelAndView.addObject("item", item);
	// 设置逻辑视图
	// modelAndView.setViewName("itemEdit");

	// 把商品数据放在模型中
	model.addAttribute("item", item);

	return "itemEdit";
}
```

**6.1.7. 绑定简单类型**
当请求的参数名称和处理器形参**名称一致**时会将请求参数与形参进行绑定。如下id：
```
/**
 * 根据id查询商品,绑定简单数据类型
 *
 * @param id
 * @param model
 * @return
 */
@RequestMapping("/itemEdit")
public String queryItemById(int id, ModelMap model) {
	// 根据id查询商品数据
	Item item = this.itemService.queryItemById(id);

	// 把商品数据放在模型中
	model.addAttribute("item", item);

	return "itemEdit";
}
```

支持自动绑定的类型
参数类型**推荐使用包装数据类型**，因为基础数据类型不可以为null
整形：Integer、int
字符串：String
单精度：Float、float
双精度：Double、double
布尔型：Boolean、boolean  说明：对于布尔类型的参数，请求的参数值为true或false。或者1或0

请求url：
http://localhost:8080/xxx.action?id=2&status=false
处理器方法：
public String editItem(Model model,Integer id,Boolean status)

**6.2.2. @RequestParam**
简单类型绑定
value：参数名字，即入参的请求参数名字，如value=“itemId”表示请求的参数  区中的名字为itemId的参数的值将传入
required：是否必须，默认是true，表示请求中一定要有相应的参数，否则将报错  TTP Status 400 - Required Integer parameter 'XXXX' is not present
defaultValue：默认值，表示如果请求中没有同名参数时的默认值

代码演示
```
@RequestMapping("/itemEdit")
public String queryItemById(@RequestParam(value = "itemId", required = true, defaultValue = "1") Integer id,
		ModelMap modelMap) {
	// 根据id查询商品数据
	Item item = this.itemService.queryItemById(id);

	// 把商品数据放在模型中
	modelMap.addAttribute("item", item);

	return "itemEdit";
}
```

**6.3. 绑定POJO类型**
要求：pojo对象中的属性名和表单中input的name属性一致。

Pojo
```
public class Item{
    private Integer id;
    private String name;
    private Float price;
    private String detail;
}
```

Controller
```
/**
 * 更新商品,绑定pojo类型
 *
 * @param item
 * @param model
 * @return
 */
@RequestMapping("/updateItem")
public String updateItem(Item item) {
	// 调用服务更新商品
	this.itemService.updateItemById(item);

	// 返回逻辑视图
	return "success";
}
```

**包装Pojo**
Pojo
```
public class QueryVo {
	private Item item;
set/get。。。
}
```
Controller
```
	// 绑定包装数据类型
	@RequestMapping("/queryItem")
	public String queryItem(QueryVo queryVo) {
		System.out.println(queryVo.getItem().getId());
		System.out.println(queryVo.getItem().getName());

		return "success";
	}
```

**6.4. 绑定数组类型**
页面选中多个checkbox向controller方法传递
```
<form action="${pageContext.request.contextPath }/queryItem.action" method="post">
<td><input type="checkbox" name="ids" value="${item.id}"/></td>
</form>
```

Pojo
```
public classa QueryVo{
    private Item item;
    private Integer[] ids;
}
```

Controller方法中可以用String[]接收，或者pojo的String[]属性接收。
```
/**
 * 包装类型 绑定数组类型，可以使用两种方式，pojo的属性接收，和直接接收
 *
 * @param queryVo
 * @return
 */
@RequestMapping("queryItem")
public String queryItem(QueryVo queryVo, Integer[] ids) {

	System.out.println(queryVo.getItem().getId());
	System.out.println(queryVo.getItem().getName());

	System.out.println(queryVo.getIds().length);
	System.out.println(ids.length);

	return "success";
}
```

**6.5. 绑定列表类型**
页面
```
<c:forEach items="${itemList }" var="item" varStatus="s">
<tr>
	<td><input type="checkbox" name="ids" value="${item.id}"/></td>
	<td>
		<input type="hidden" name="itemList[${s.index}].id" value="${item.id }"/>
		<input type="text" name="itemList[${s.index}].name" value="${item.name }"/>
	</td>
	<td><input type="text" name="itemList[${s.index}].price" value="${item.price }"/></td>
	<td><input type="text" name="itemList[${s.index}].createtime" value="<fmt:formatDate value="${item.createtime}" pattern="yyyy-MM-dd HH:mm:ss"/>"/></td>
	<td><input type="text" name="itemList[${s.index}].detail" value="${item.detail }"/></td>

	<td><a href="${pageContext.request.contextPath }/itemEdit.action?id=${item.id}">修改</a></td>

</tr>
</c:forEach>
```

Pojo
```
public classa QueryVo{
    private Item item;
    private Integer[] ids;
    private List<Item> itemList;
}
```

Controller
```
@RequestMapping("queryItem")
public String queryItem(QueryVo queryVo) {
	System.out.println(queryVo.getItemList().length);
	return "success";
}
```

**6.6RequestMapping**
value
添加在类上面：限制此类下的所有方法请求url必须以请求前缀开头
添加在方法上面：value的值是数组，可以将多个url映射到同一个方法
method
请求方法限定
```
/**
 * 查询商品列表
 * @return
 */
@RequestMapping(value = { "itemList", "itemListAll" }, method = RequestMethod.POST)
public ModelAndView queryItemList() {
	// 查询商品数据
	List<Item> list = this.itemService.queryItemList();

	// 创建ModelAndView,设置逻辑视图名
	ModelAndView mv = new ModelAndView("itemList");

	// 把商品数据放到模型中
	mv.addObject("itemList", list);
	return mv;
}
```

**6.7. 自定义Converter**
Converter：用于转换参数格式
```
//Converter<S, T>
//S:source,需要转换的源的类型
//T:target,需要转换的目标类型
public class DateConverter implements Converter<String, Date> {

	@Override
	public Date convert(String source) {
		try {
			// 把字符串转换为日期类型
			SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyy-MM-dd HH:mm:ss");
			Date date = simpleDateFormat.parse(source);

			return date;
		} catch (ParseException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		// 如果转换异常则返回空
		return null;
	}
}
```

springmvc.xml添加配置（常规）
```
<!-- 配置注解驱动 -->
<!-- 如果配置此标签,可以不用配置注解驱动<mvc:annotation-driven/> -->
<mvc:annotation-driven conversion-service="conversionService" />

<!-- 转换器配置 -->
<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
	<property name="converters">
		<set>
			<bean class="cn.itcast.springmvc.converter.DateConverter" />
		</set>
	</property>
</bean>
```

##异常处理器
6.2. 自定义异常类
```
public class MyException extends Exception {
	// 异常信息
	private String message;

	public MyException() {
		super();
	}

	public MyException(String message) {
		super();
		this.message = message;
	}

	public String getMessage() {
		return message;
	}

	public void setMessage(String message) {
		this.message = message;
	}

}
```

6.3. 自定义异常处理器
```
public class CustomHandleException implements HandlerExceptionResolver {

	@Override
	public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler,
			Exception exception) {
		// 定义异常信息
		String msg;

		// 判断异常类型
		if (exception instanceof MyException) {
			// 如果是自定义异常，读取异常信息
			msg = exception.getMessage();
		} else {
			// 如果是运行时异常，则取错误堆栈，从堆栈中获取异常信息
			Writer out = new StringWriter();
			PrintWriter s = new PrintWriter(out);
			exception.printStackTrace(s);
			msg = out.toString();

		}

		// 把错误信息发给相关人员,邮件,短信等方式
		// TODO

		// 返回错误页面，给用户友好页面显示错误信息
		ModelAndView modelAndView = new ModelAndView();
		modelAndView.addObject("msg", msg);
		modelAndView.setViewName("error");

		return modelAndView;
	}
}
```

6.4. 异常处理器配置
```
<!-- 配置全局异常处理器 -->
<bean
id="customHandleException" 	class="cn.itcast.ssm.exception.CustomHandleException"/>
```

##拦截器

配置拦截器实现HandlerInterceptor接口
```
public class LoginHandlerInterceptor implements HandlerInterceptor {
	// controller执行后且视图返回后调用此方法
	// 这里可得到执行controller时的异常信息
	// 这里可记录操作日志
	@Override
	public void afterCompletion(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2, Exception arg3)
			throws Exception {
		System.out.println("HandlerInterceptor1....afterCompletion");
	}

	// controller执行后但未返回视图前调用此方法
	// 这里可在返回用户前对模型数据进行加工处理，比如这里加入公用信息以便页面显示
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object arg2) throws Exception {
	// 从request中获取session
	HttpSession session = request.getSession();
	// 从session中获取username
	Object username = session.getAttribute("username");
	// 判断username是否为null
	if (username != null) {
		// 如果不为空则放行
		return true;
	} else {
		// 如果为空则跳转到登录页面
		response.sendRedirect(request.getContextPath() + "/user/toLogin.action");
	}

	return false;
	}

	// Controller执行前调用此方法
	// 返回true表示继续执行，返回false中止执行
	// 这里可以加入登录校验、权限拦截等
	@Override
	public boolean preHandle(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2) throws Exception {
		System.out.println("HandlerInterceptor1....preHandle");
		// 设置为true，测试使用
		return true;
	}
}
```

Controller
```
@Controller
@RequestMapping("user")
public class UserController {

	/**
	 * 跳转到登录页面
	 *
	 * @return
	 */
	@RequestMapping("toLogin")
	public String toLogin() {
		return "login";
	}

	/**
	 * 用户登录
	 *
	 * @param username
	 * @param password
	 * @param session
	 * @return
	 */
	@RequestMapping("login")
	public String login(String username, String password, HttpSession session) {
		// 校验用户登录
		System.out.println(username);
		System.out.println(password);

		// 把用户名放到session中
		session.setAttribute("username", username);

		return "redirect:/item/itemList.action";
	}

}
```

springmvc.xml配置拦截器
```
<mvc:interceptor>
	<!-- 配置商品被拦截器拦截 -->
	<mvc:mapping path="/item/**" />
	<!-- 配置具体的拦截器 -->
	<bean class="cn.itcast.ssm.interceptor.LoginHandlerInterceptor" />
</mvc:interceptor>
```


##上传图片
在tomcat上配置图片虚拟目录，在tomcat下conf/server.xml中添加：
<Context docBase="D:\develop\upload\temp" path="/pic" reloadable="false"/>
访问http://localhost:8080/pic即可访问D:\develop\upload\temp下的图片。

需要jar包
![mvc1](/images/探索SSM（三）-Spring MVC/mvc1.jpg)

jsp页面
![mvc2](/images/探索SSM（三）-Spring MVC/mvc2.jpg)
设置表单可以进行文件上传，如下图：
![mvc3](/images/探索SSM（三）-Spring MVC/mvc3.jpg)

springmvc.xml配置上传解析器
```
<!-- 文件上传,id必须设置为multipartResolver -->
<bean id="multipartResolver"
	class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
	<!-- 设置文件上传大小 -->
	<property name="maxUploadSize" value="5000000" />
</bean>
```

Controller（使用MultipartFile上传文件）
```
/**
 * 更新商品
 *
 * @param item
 * @return
 * @throws Exception
 */
@RequestMapping("updateItem")
public String updateItemById(Item item, MultipartFile pictureFile) throws Exception {
	// 图片上传
	// 设置图片名称，不能重复，可以使用uuid
	String picName = UUID.randomUUID().toString();

	// 获取文件名
	String oriName = pictureFile.getOriginalFilename();
	// 获取图片后缀
	String extName = oriName.substring(oriName.lastIndexOf("."));

	// 开始上传
	pictureFile.transferTo(new File("C:/upload/image/" + picName + extName));

	// 设置图片名到商品中
	item.setPic(picName + extName);
	// ---------------------------------------------
	// 更新商品
	this.itemService.updateItemById(item);

	return "forward:/itemEdit.action";
}
```

##参数-Json数据交互

@RequestBody
将读到的**Json数据**转换为**java对象**并绑定到Controller方法的参数上。
@ResponseBody
将Controller的方法**返回的对象**转换为指定格式的数据如：**json,xml**，通过Response响应给客户端

Jar包
![mvc4](/images/探索SSM（三）-Spring MVC/mvc4.jpg)

Controller
```
/**
 * 测试json的交互
 * @param item
 * @return
 */
@RequestMapping("testJson")
// @ResponseBody
public @ResponseBody Item testJson(@RequestBody Item item) {
	return item;
}
```

springmvc.xml添加json转换器配置
```
<!--处理器适配器 -->
  <bean class=*"org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"*>
  <property name=*"messageConverters"*>
  <list>
  <bean class=*"org.springframework.http.converter.json.MappingJacksonHttpMessageConverter"*></bean>
  </list>
  </property>
  </bean>
```

##RESTful风格
即利用**url**获取数据
```
/**
 * 使用RESTful风格开发接口，实现根据id查询商品
 *
 * @param id
 * @return
 */
@RequestMapping("item/{id}")
@ResponseBody
public Item queryItemById(@PathVariable() Integer id) {
	Item item = this.itemService.queryItemById(id);
	return item;
}
```
如果@RequestMapping中表示为"item/{id}"，id和形参名称一致，**@PathVariable**不用指定名称。如果不一致，例如"item/{ItemId}"则需要指定名称**@PathVariable("itemId")**

注意： @PathVariable是获取**url上数据**的。@RequestParam获取**请求参数**的（包括post表单提交）



e.g. 本文仅供个人笔记使用，借鉴部分网上资料。
