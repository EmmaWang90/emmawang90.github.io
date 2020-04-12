---
title: web应用基础概念
date: 2019-02-10 11:50:13
tags:
   - web
   - servlet
   - war
categories:
   - web
---

# web应用基础概念

## web容器

也叫servlet容器，负责管理servlet的生命周期、将URL映射到特定的servlet，并确保URL请求者有正确的访问权限。web容器实现了Java EE体系结构的Web组件契约，此体系结构为其他web组建指定运行时环境，包括安全性、并发性、生命周期管理、事务、附属和其他服务。

其中比较常见的容器有tomcat、glassfish、jetty。



## Servlet

Servlet是web容器中管理的、用于为客户端请求提供服务的类。部署web服务时，servlet由web容器进行管理。

servlet的生命周期有三个阶段，`init()`，`service()`和`destroy()`。它们由每个servlet实现，并在特定时间由web容器调用。

- 在servlet [生命周期的](https://en.wikipedia.org/wiki/Object_lifetime)初始化阶段，Web容器通过调用[`init()`](http://docs.oracle.com/javaee/7/api/javax/servlet/Servlet.html#init)方法初始化servlet实例，并传递实现该[`javax.servlet.ServletConfig`](http://docs.oracle.com/javaee/7/api/javax/servlet/ServletConfig.html)接口的对象。此配置对象允许servlet 从Web应用程序访问[名称 - 值](https://en.wikipedia.org/wiki/Attribute%E2%80%93value_pair)初始化参数。
- 初始化之后，servlet实例可以为客户端请求提供服务。每个[请求](https://en.wikipedia.org/wiki/HTTP_request#Request_message)都在自己的独立线程中提供服务。Web容器`service()`为每个请求调用servlet 的方法。该`service()`方法确定正在进行的请求的类型，并将其分派给适当的方法来处理请求。servlet的开发人员必须为这些方法提供实现。如果对servlet未实现的方法发出请求，则调用父类的方法，通常会导致将错误返回给请求者。
- 最后，Web容器调用`destroy()`使servlet停止服务的方法。这个`destroy()`方法`init()`在servlet的生命周期中只调用一次。

以下是这些方法的典型用户场景。

1. 假设用户请求访问URL

   - 然后，浏览器为此URL生成HTTP请求。
   - 然后将此请求发送到适当的服务器。

2. HTTP请求由Web服务器接收并转发到servlet容器。

   - 容器将此请求映射到特定的servlet。
   - 动态检索servlet并将其加载到容器的地址空间中。

3. 容器调用init()

   servlet 的方法。

   - 仅当servlet首次加载到内存中时才会调用此方法。
   - 可以将初始化参数传递给servlet，以便它可以自行配置。

4. 容器调用service()方法。

   - 调用此方法来处理HTTP请求。
   - servlet可以读取HTTP请求中提供的数据。
   - servlet还可以为客户端制定HTTP响应。

5. servlet保留在容器的地址空间中，可用于处理从客户端收到的任何其他HTTP请求。

   - `service()`为每个HTTP请求调用该方法。

6. 在某些时候，容器可能决定从其内存中卸载servlet。

   - 做出此决定的算法特定于每个容器。

7. 容器调用servlet的`destroy()`方法来放弃任何资源，例如为servlet分配的文件句柄; 重要数据可以保存到持久性存储中。

8. 然后可以对为servlet及其对象分配的内存进行垃圾回收。

## WAR包

war包是Sun提出的一种web应用程序格式，与jar类似，是很多文件的压缩包。

war包中的文件按照一定目录结构来组织。通常其根目录下包含有html和jsp文件，或者包含有这两种文件的目录，另外还有WEB-INF目录。通常在WEB-INF目录下含有一个web.xml文件和一个classes目录，web.xml是这个应用的配置文件，而classes目录下则包含编译好的servlet类和jsp，或者servlet所依赖的其他类（如JavaBean）。通常这些所依赖的类也可以打包成jar包放在WEB-INF下的lib目录下。

以Tomcat来说，将war包放置在其\webapps\目录下，然后启动Tomcat，这个包就会自动解压，就相当于发布了。