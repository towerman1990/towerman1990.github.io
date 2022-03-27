---
title: 社区版IDEA中启动Tomcat
date: 2022-03-27 11:34:36
tags:
---

因为工作需要，最近在学习Java。在学习的过程中，由于什么软件都喜欢用新版（┑(￣Д ￣)┍）踩了不少的坑，分享一下社区版IDEA中启动Tomcat的相关配置与代码Demo。
本人使用的软件环境为Java17 + Tomcat10 + IDEA2021.3.3，代码参考了廖雪峰老师的博客[<sup>[1]</sup>](#refer-anchor-1)。

<!-- more -->

### <span id="new-module">新建mudule</span>

首先在IDEA中新建一个名为`web-servlet-embedded`的module，修改`pom.xml`配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.towerman1990.web</groupId>
    <artifactId>web-servlet-embedded</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <java.version>17</java.version>
        <tomcat.version>10.1.0-M12</tomcat.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-core</artifactId>
            <version>${tomcat.version}</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
            <version>${tomcat.version}</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
</project>
```

### <span id="new-servlet">编写Servlet</span>

新建一个名为`HelloServlet`的类用来输出服务器端响应：

```java
package cn.towerman1990.web;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;
import java.io.PrintWriter;

@WebServlet(urlPatterns = "/")
public class HelloServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html");
        String name = req.getParameter("name");
        if (name == null) {
            name = "world";
        }
        PrintWriter pw = resp.getWriter();
        pw.write("<h1>Hello, " + name + "!</h1>");
        pw.flush();
    }
}
```

### <span id="new-tomcat">编写Tomcat启动类</span>

新建一个名为`Main`的类并编写`main()`方法，用来启动Tomcat：

```java
package cn.towerman1990.tomcat;

import org.apache.catalina.Context;
import org.apache.catalina.WebResourceRoot;
import org.apache.catalina.startup.Tomcat;
import org.apache.catalina.webresources.DirResourceSet;
import org.apache.catalina.webresources.StandardRoot;

import java.io.File;

public class Main {
    public static void main(String[] args) throws Exception {
        // 启动Tomcat:
        Tomcat tomcat = new Tomcat();
        tomcat.setPort(Integer.getInteger("port", 8080));
        tomcat.getConnector();
        // 创建webapp:
        Context ctx = tomcat.addWebapp("", new File("web-servlet-embedded/src/main/webapp").getAbsolutePath());
        WebResourceRoot resources = new StandardRoot(ctx);
        resources.addPreResources(
                new DirResourceSet(resources, "/WEB-INF/classes", new File("web-servlet-embedded/target/classes").getAbsolutePath(), "/"));
        ctx.setResources(resources);
        tomcat.start();
        tomcat.getServer().await();
    }
}
```

- 注：需要在IDEA->Run->Debug Configurations->Application->Main->右侧modify options勾选`Add dependencies with "Provided" scope`选项。


### <span id="debug">调试</span>

调用`main()`方法启动Tomcat服务器，然后访问浏览器，输入URL：`http://localhost:8080/?name=tom`并回车，可见浏览器页面显示`Hello, tom!`字样。

<div id="refer-anchor-1"></div>

- [1] [Servlet开发](https://www.liaoxuefeng.com/wiki/1252599548343744/1266264743830016)