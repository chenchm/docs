# Servlet笔记

## 1.1什么是servlet

Servlet是sun公司制定的一种用来扩展web服务器功能的组件标准（服务器端的Java应用程序）。狭义的Servlet是指Java接口，广义的Servlet是指任何实现了这个Servlet接口的类，它运行在 Web 服务器或应用服务器上，作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP 服务器上的数据库或应用程序之间的中间层。

## 1.2Servlet与Servlet容器的关系

Servlet与Servlet容器的关系就像枪与子弹的关系，Servlet容器是为Servlet而设计的，使Servlet更加易用和强大；两者相互依存又独立发展，这一切都是为了适应工业化生产的结果。从技术角度来说是为了解耦，通过标准化接口来相互协作。Servlet容器的作用主要包括以下几点：

- 作为Web工程的入口，加载和启动WEB程序
- 在启动WEB项目过程中解析和加载web.xml中配置的Servlet对象
- 将客户端请求分配到指定的Servlet
- 管理Servlet的生命周期

 Servlet 容器作为一个独立发展的标准化产品，目前它的种类很多，但是它们都有自己的市场定位，很难说谁优谁劣，各有特点。下文将以大家比较熟悉的Servlet容器Tomcat为例说明Servlet容器如何对Servlet进行管理。

在Tomcat容器容器等级中，Servlet被封装在包装类**Wrapper**中由Context容器来直接管理，一个Context对应一个WEB工程因此Context容器的如何运行将直接影响Servlet的工作方式。

<div align="center">
    ![Tomcat 容器模型](images/tomcatmodel.png)Tomcat 容器模型
</div>
## 1.2Servlet容器的启动

本节以大家比较熟悉的Servlet容器Tomcat为例，介绍Tomcat的启动方式以及启动过程。

### 1.2.1Tomcat的启动方式

- 执行Tomcat脚本

	在本地安装好Tomcat以后，将WEB工程生成的war包放在Tomcat安装目录的`webapps`目录下，运行`bin`目录下的startup.bat或startup.sh脚本。Tomcat会根据`conf`目录下的xml配置文件（主要是server.xml）来启动web容器，解压`webapps`目下的tar包，并加载和管理相应的项目。

- 嵌入式

	Tomcat7开始支持嵌入式功能，增加了一个启动类`org.apache.catalina.startup.Tomcat`。创建并调用`Tomcat`对象的start方法就能容易地启动一个tomcat项目，还可以通过这个对象来增加和修改 Tomcat 的配置参数。下文将会详细地介绍此方式。

- maven插件

	可以在maven中使用Tomcat插件来启动Tomcat，配置如下：

	```xml
	<plugin>
	  <groupId>org.apache.tomcat.maven</groupId>
	  <artifactId>tomcat7-maven-plugin</artifactId>
	  <version>2.2</version>
	  <configuration>
	    <port>8080</port>
	    <path>/${project.artifactId}</path>
	    <uriEncoding>UTF-8</uriEncoding>
	  </configuration>
	</plugin>
	```

	接着在命令行执行`mvn tomcat7:run`启动Tomcat。

	---

	**说明**

	使用maven插件启动虽然比较方便，但是只支持较低版本的Tomcat。

### 1.2.2Tomcat的启动过程

### 参考

1. [https://www.cnblogs.com/whgk/p/6399262.html](https://www.cnblogs.com/whgk/p/6399262.html)
2. [https://www.ibm.com/developerworks/cn/java/j-lo-servlet/](https://www.ibm.com/developerworks/cn/java/j-lo-servlet/)
3. [https://www.cnblogs.com/zhouyuqin/p/5143121.html](https://www.cnblogs.com/zhouyuqin/p/5143121.html)

<==To be continue...

