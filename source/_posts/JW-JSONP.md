---
title: JW-JSONP
date: 2019.04.23 17:41:14
tags: JW
categories: JW
---


去年学习React时碰到过JS跨域访问的问题，当时尝试的解决方案有Jsonp、服务器允许跨域、设置Content-Security-Policy等。一般Web上普遍使用JSONP解决主流浏览器的跨域数据访问的问题。
```
<!-- 设置Content-Security-Policy -->
<meta http-equiv="Content-Security-Policy" content="default-src 'self' data: gap: https://ssl.gstatic.com 'unsafe-eval'; style-src 'self' 'unsafe-inline'; media-src *;connect-src *;">
```

JSONP是JSON的一种“使用模式”,可用于解决主流浏览器的跨域数据访问的问题。
JS能够跨域加载JS文件，如果请求时自定义一个Funtion放入callback参数中。Jsonnp识别callback参数，将返回数据放入callback参数中,相当于响应一个JS文件给浏览器。浏览器相当于调用JS文件中的Funtion。
![jsonp](/images/JW-JSONP/jsonp.jpg)

jQuery封装了定义Funtion的方法，因此不用自己创建方法。


**Ajax请求**
```
var E3MALL = {
	checkLogin : function(){
		var _ticket = $.cookie("TT_TOKEN");
		if(!_ticket){
			return ;
		}
		$.ajax({
			url : "http://localhost:8088/user/token/" + _ticket,
			dataType : "jsonp",
			type : "GET",
			success : function(data){
				if(data.status == 200){
					var username = data.data.username;
					var html = username + "，欢迎来到宜立方购物网！<a href=\"http://www.e3mall.cn/user/logout.html\" class=\"link-logout\">[退出]</a>";
					$("#loginbar").html(html);
				}
			}
		});
	}
}

$(function(){
	// 查看是否已经登录，如果已经登录查询登录信息
	E3MALL.checkLogin();
});
```


**Controller**
```
@Controller
public class TokenController {

	@Autowired
	private TokenService tokenService;

	@RequestMapping(value="/user/token/{token}",
			produces=MediaType.APPLICATION_JSON_UTF8_VALUE"application/json;charset=utf-8")
	@ResponseBody
	public String getUserByToken(@PathVariable String token, String callback) {
		E3Result result = tokenService.getUserByToken(token);
		//响应结果之前，判断是否为jsonp请求
		if (StringUtils.isNotBlank(callback)) {
			//把结果封装成一个js语句响应
			return callback + "(" + JsonUtils.objectToJson(result)  + ");";
		}
		return JsonUtils.objectToJson(result);
	}

	@RequestMapping(value="/user/token/{token}")
	@ResponseBody
	public Object getUserByToken(@PathVariable String token, String callback) {
		E3Result result = tokenService.getUserByToken(token);
		//响应结果之前，判断是否为jsonp请求
		if (StringUtils.isNotBlank(callback)) {
			//把结果封装成一个js语句响应
			//即封装成 callback + "(" + JsonUtils.objectToJson(result)" + ");";
			//若直接返回String，浏览器会解析为text/html。可修改@RequestMapping指定。
			MappingJacksonValue mappingJacksonValue = new MappingJacksonValue(result);
			mappingJacksonValue.setJsonpFunction(callback);
			return mappingJacksonValue;
		}
		return result;
	}
}
```

e.g. 本文仅供个人笔记使用，借鉴部分网上资料。
