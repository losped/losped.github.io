---
title: JW-log4j
date: 2019.04.19 17:20:09
tags: JW
categories: JW
---


##### log4j 日志记录插件

引入jar包
```
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
		</dependency>
```

log4j.properties
```
#打印INFO,A3,STDOUT信息
log4j.rootLogger=INFO,A3,STDOUT

#不写入文件
log4j.appender.STDOUT=org.apache.log4j.ConsoleAppender
log4j.appender.STDOUT.layout=org.apache.log4j.PatternLayout
log4j.appender.STDOUT.layout.ConversionPattern=[%p] [%l] %10.10c - %m%n

#写入文件
log4j.appender.A3=org.apache.log4j.RollingFileAppender
log4j.appender.A3.file=logs/server.log
log4j.appender.A3.MaxFileSize=1024KB
log4j.appender.A3.MaxBackupIndex=10
log4j.appender.A3.layout=org.apache.log4j.PatternLayout
log4j.appender.A3.layout.ConversionPattern=\n\n[%-5p] %d{yyyy-MM-dd HH\:mm\:ss,SSS} method\:%l%n%m%n
```

GlobalExceptionResolver.java
```
/**
 * 全局异常处理器
 * <p>Title: GlobalExceptionResolver</p>
 * <p>Description: </p>
 * <p>Company: www.itcast.cn</p>
 * @version 1.0
 */
public class GlobalExceptionResolver implements HandlerExceptionResolver {

	private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionResolver.class);

	@Override
	public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler,
			Exception ex) {
		//打印控制台
		ex.printStackTrace();
		//写日志
		logger.debug("测试输出的日志。。。。。。。");
		logger.info("系统发生异常了。。。。。。。");
		logger.error("系统发生异常", ex);
		//发邮件、发短信
		//使用jmail工具包。发短信使用第三方的Webservice。
		//显示错误页面
		ModelAndView modelAndView = new ModelAndView();
		modelAndView.setViewName("error/exception");
		return modelAndView;
	}

}
```

springmvc.xml
```
	<!-- 全局异常处理器 -->
	<bean class="cn.e3mall.search.exception.GlobalExceptionResolver"/>
```
