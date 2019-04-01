---
title: SpringBoot入门
date: 2019-03-29 12:52:46
categories: IT技术
catalogue: 服务框架
tags: [spring,boot]
---
<font color="red">本文转载自 https://www.cnblogs.com/wmyskxz/p/9010832.html<font>

### Spring Boot 概述

>
Build Anything with Spring Boot：Spring Boot is the starting point for building all Spring-based applications. 
Spring Boot is designed to get you up and running as quickly as possible, with minimal upfront configuration of Spring.

上面是引自官网的一段话，大概是说： Spring Boot 是所有基于 Spring 开发的项目的起点。Spring Boot 的设计是为了让你尽可能快的跑起来。Spring 应用程序并且尽可能减少你的配置文件。

#### 什么是 Spring Boot
+ 它使用 “习惯优于配置” （项目中存在大量的配置，此外还内置一个习惯性的配置，让你无须）的理念让你的项目快速运行起来。
+ 它并不是什么新的框架，而是默认配置了很多框架的使用方式，就像 Maven 整合了所有的 jar 包一样，Spring Boot 
整合了所有框架（引自：[springboot(一)：入门篇——纯洁的微笑](http://www.ityouknow.com/springboot/2016/01/06/springboot%28%E4%B8%80%29-%E5%85%A5%E9%97%A8%E7%AF%87.html)）

#### 使用 Spring Boot 有什么好处

回顾我们之前的 SSM 项目，搭建过程还是比较繁琐的，需要：
+ 1）配置 web.xml，加载 spring 和 spring mvc
+ 2）配置数据库连接、配置日志文件
+ 3）配置家在配置文件的读取，开启注解
+ 4）配置mapper文件
+ .....
<!--more-->

而使用 Spring Boot 来开发项目则只需要非常少的几个配置就可以搭建起来一个 Web 项目，并且利用 IDEA 可以自动生成生成，这简直是太爽了...
+ 划重点：简单、快速、方便地搭建项目；对主流开发框架的无配置集成；极大提高了开发、部署效率。

### Spring Boot 快速搭建

##### 第一步：新建项目
选择 Spring Initializr ，然后选择默认的 url 点击【Next】：
{% asset_img 20190329181901.png%}
然后修改一下项目的信息：
{% asset_img 20190329182044.png%}
勾选上 Web 模板：
{% asset_img 20190329182135.png%}
选择好项目的位置，点击【Finish】：
{% asset_img 20190329182237.png%}
如果是第一次配置 Spring Boot 的话可能需要等待一会儿 IDEA 下载相应的 依赖包，默认创建好的项目结构如下：
{% asset_img 20190329182350.png%}
项目结构还是看上去挺清爽的，少了很多配置文件，我们来了解一下默认生成的有什么：
+ SpringbootApplication： 一个带有 main() 方法的类，用于启动应用程序
+ SpringbootApplicationTests：一个空的 Junit 测试了，它加载了一个使用 Spring Boot 字典配置功能的 Spring 应用程序上下文
+ application.properties：一个空的 properties 文件，可以根据需要添加配置属性
+ pom.xml： Maven 构建说明文件

##### 第二步：HelloController
在【cn.altctrl.springboot】包下新建一个【HelloController】：
```
package cn.altctrl.springboot;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String hello() {
        return "Hello Spring Boot!";
    }
}
```
+ @RestController 注解： 该注解是 @Controller 和 @ResponseBody 注解的合体版

##### 第三步：利用 IDEA 启动 Spring Boot
我们回到 SpringbootApplication 这个类中，然后右键点击运行：
{% asset_img 20190329183119.png%}
+ 注意：我们之所以在上面的项目中没有手动的去配置 Tomcat 服务器，是因为 Spring Boot 内置了 Tomcat
等待一会儿就会看到下方的成功运行的提示信息：
{% asset_img 20190329183239.png%}
可以看到我们的 Tomcat 运行在 8080 端口，我们来访问 “/hello” 地址试一下：
{% asset_img 7896890-6111e1913c5bf6d6.png%}
可以看到页面成功显示出我们返回的信息。
