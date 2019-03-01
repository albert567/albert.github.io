---
title: JS跨域请求
date: 2019-02-28 12:52:46
categories: IT技术
catalogue: Java web
tags: [JS,javascript,跨域]
---
首先js跨域请求是不安全的，可以通过前台调用后台服务器，通过后台httpclient请求url，返回结果。js跨域请求如下：
前端js：
```
	$.ajax({
		async : false,
		url : url,
		type : "GET",
		dataType : "jsonp", // 返回的数据类型，设置为JSONP方式
		jsonp : "jsonpCallback",
		success : function(response) {
			alert("请求成功");
		},
		error : function() {
			alert("请求失败");
		}
	});
```
<!--more-->
后台使用jfinal框架:
```
String key = getPara("key");
String jsonpCallback = getPara("jsonpCallback");
renderJson(jsonpCallback+"("+json+")");
```