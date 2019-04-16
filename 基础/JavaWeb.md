---
title:JavaWeb
---



# 第一章：Servlet

## 什么是Servlet
### 1. 定义
1. 定义一：Servlet运行于Servlet容器，可以被Servlet容器动态加载，扩展服务器的功能，提供特定服务。Servlet通过请求/响应的方式工作。
2. 定义二：Servlet是运行于Web服务器或应用服务器上的程序，它是作为来自Web浏览器或者Http客户端的请求和Http服务器上的数据库或者应用程序之间的中间层。
### 1.1 Servlet规范
1. servlet2规范：servlet2.5特性介绍：支持annotations。
    - 项目目录必须要有WEB-INF、web.xml等文件夹和文件。
    - 在web.xml中配置servlet、filter、listener，以web.xml为java web项目的统一入口
2. servlet3规范：
    - 项目中可以不需要WEB-INF、web.xml等文件夹和文件，在没有web.xml文件的情况下，通过注解实现servlet、filter、listener的声明，比如@WebServlet,@WebListener,@WebFilter，当使用注解时，容器自动进行扫描。
    - 关于一些异步请求的改动
    - 关于ServletContainerInitializer的使用

### 2. Servlet执行的主要任务
1. 读取：读取Http客户端传来的Http数据（比如网页表单数据，cookie信息、媒体类型等等）
2. 处理：处理数据并且生成结果。这个过程需要在服务器内部访问数据库或者其他服务，通过计算产生结果。
3. 响应：发送数据到客户端。文档（包含HTML 或 XML、二进制文件-GIF 图像、Excel 等），或者设置 cookies 和缓存参数，以及其他类似的任务。

### 3. Java Servlet
定义：是Java端实现了Servlet这个概念，并遵循Java Servlet规范的Servlet类。可以使用javax.servlet和javax.servlet.http包创建。

### 4. Servlet生命周期
```
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;
    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;
    void destroy();
}

```
Servlet的init、service、destroy方法就是Servlet的三个生命周期方法。  
web服务器启动时，并不会立刻创建Servlet实例，而是当第一次客户请求过来时，再由web服务器创建Servlet实例，调用init方法，同时执行service方法来执行具体业务。而之后访问，都只会调用service方法。所以针对一台服务器而言，Servlet是单例的。当web服务器重启时或者正常关闭时，会调用destroy方法，销毁Servlet实例。service方法为客户端每次访问时所用。每一次请求都会创建一个新的线程，并调用service方法。综上所述，我们知道Servlet就是单例多线程的。

### 5. 架构图
### 6. Servlet接口实现关系
![image-20190321192721369](https://ws2.sinaimg.cn/large/006tKfTcly1g1bj6fun5pj30p60fgjux.jpg)
> GenericServlet实现了Servlet，HttpServlet继承了GenericServlet并提供了doGet、doPost、doPut、doDelete方法。

## 共享数据域
1. ServletContext   
  当Web应用被加载入容器的时候创建，作用于整个Web应用。用于请求转发，读取资源文件。
2. Request域    
  在service方法调用前由web容器生成，传入service方法。作用于整个请求链（请求转发也存在）
3. Session域    
  针对一次会话，服务器会产生一个session，存于内存中。作用于一次会话，会话结束即销毁。
4. PageContext域    
  作用于JSP的页面