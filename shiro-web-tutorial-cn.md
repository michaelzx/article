原文地址：[http://shiro.apache.org/webapp-tutorial.html](http://shiro.apache.org/webapp-tutorial.html)

本文档是一个用Apache Shiro保护web应用的分步引导教程，架设你已经了解了shiro的入门知识，并且至少熟悉一下两个入门文档：

[Application Security with Apache Shiro](http://www.infoq.com/articles/apache-shiro)  
[Apache Shiro 10 Minute Tutorial](http://shiro.apache.org/10-minute-tutorial.html)

这个分布教程将花你大概45-60分钟的时间去完成，当你结束以后，你将会对shiro在web应用中是如何工作非常有概念.

#内容列表
[概述](#概述)  
[项目安装（初始化？）](#项目安装)  
Step 1: [启用Shiro](#step1启用shiro)  
Step 2: [连接用户信息](#step2连接用户信息)  
Step 3: [启用登陆（Login）和注销（Logout）](#step3)  
Step 4: [修改用户的特定界面](#step4)  
Step 5: [只允许经过认证的用户访问](#step5)  
Step 6: [基于角色的访问控制](#step6)  
Step 7: [基于权限的访问控制](#step7)  

#<span id="overview">概述</span>
Apache Shiro的核心设计目标允许它被用于保护任何基于JVM的应用程序，比如命令行应用程序、服务器守护进程、web应用程序等。这篇指南将重点放在最常用的用例：保护一个运行在servlet容器中的web应用，如Tomcat或Jetty。

##先决条件
下列工具要求必须已经安装到您本地开发机上，以便顺利进行本教程
* Git (tested w/ 1.7)  
* Java SDK 7  
* Maven 3  
* 你喜欢的开发工具, 比如 IntelliJ IDEA 、 Eclipse, 或者可以查看、编辑文件的简单文本编辑器

##教程形式
这篇一篇分布教程，教程及所有的步骤以一个git仓库的形式存在。当你克隆这个git仓库，它的master分支就是的开始的点。教程中的每个步骤都是一个单独的分支。你可以简单的依照你正在阅读教程步骤迁出相应git分支。

##应用程序
我们将构建的web应用是一个超级web应用，它可以被用作你自己应用的起点。他将展示用户登录、注销、用户特定的欢迎消息、对web应用的某一部分进行访问控制、集成可嵌入的安全数据存储。

我们通过配置项目开始，包括构建工具、声明依赖，当我们配置要servlet的web.xml文件，就可以启动web应用和shiro环境。

当我们完成安装设置，然后我们将功能分分割成独立的层，包括整合安全数据存储，以便实现用户的登陆、注销和访问控制

#项目安装
你不用手动设置目录结构和初始化基本的文件，我们已经在git仓库中为你做好了这些。

##1.Fork教程工程
在GitHub，访问[tutorial project](https://github.com/lhazlewood/apache-shiro-tutorial-webapp)并点击右上角的Fork按钮。
##2.克隆你自己教程工程
现在你已经fork了仓库到你自己的GitHub账号，把它克隆到你的本地：  
`$ git clone git@github.com:$YOUR_GITHUB_USERNAME/apache-shiro-tutorial-webapp.git`  
($YOUR_GITHUB_USERNAME应该是就是你自己的GitHub用户名)  
你开切换到已经克隆下来的文件夹，看看工程的结构：   
`$ cd apache-shiro-tutorial-webapp`

##3.回顾项目结构
在你克隆仓库后，你现在的master分支将会有以下结构： 
```
apache-shiro-tutorial-webapp/
  |-- src/
  |  |-- main/
  |    |-- resources/
  |      |-- logback.xml
  |    |-- webapp/
  |      |-- WEB-INF/
  |        |-- web.xml
  |      |-- home.jsp
  |      |-- include.jsp
  |      |-- index.jsp
  |-- .gitignore
  |-- .travis.yml
  |-- LICENSE
  |-- README.md
  |-- pom.xml
```
下面是每一个文件的含义：
* pom.xml: Maven工程的构建文件. 它有一个配置好的Jetty，所以你可以通过运行`mvn jetty:run`来测试的web应用。
* README.md: 一个简单项目说明文件
* LICENSE: 项目的Apache 2.0授权文件
* .travis.yml: A Travis CI config file in case you want to run continuous integration on your project to ensure it always builds.
* .gitignore: git的忽略文件，包含一些不需要进入版本控制的后缀、文件夹等。
* src/main/resources/logback.xml: 一个简单的后端日志配置文件.在本教程中，我们选择SLF4J作为我们的日志API和后端日志的实现. 这样可以是Log4J或者JUL.
* src/main/webapp/WEB-INF/web.xml: 我们的初始化web.xml文件，我们将会马上对它进行配置来启用shiro。
* src/main/webapp/include.jsp: 一个包含常规引用、声明、包含其他JSP页面的JSP页面。这允许我们在同一个地方合并引用和声明。
* src/main/webapp/home.jsp: 我们web应用默认主页，包含include.jsp (as will others, as we will soon see).

##4.运行web应用程序
项目你已经克隆好工程，你可以通过下面的命令行来运行这个web应用程序：
```
$ mvn jetty:run 
```
下一步，打开的你的浏览器访问localhost:8080，你会看见一个显示`Hello, World!`欢迎词的主页。

通过 ctrl+C (在mac上cmd+c) 可以关闭这个web应用。

#Step1:启用Shiro
我们最初仓库中的master分支仅仅是一个简单的通用web应用程序，它可以作为任何应用程序的模板。接下来让我们给这个web应用程序启用最小化的shiro。

执行下面git迁出目录来载入step1的分支：
```
$ git checkout step1
```
迁出这个分支后，你会发现2个变化：

1. 添加了一个新的`src/main/webapp/WEB-INF/shiro.ini file`被添加  
2. `src/main/webapp/WEB-INF/web.xml`被修改了  

##1a: 添加一个shiro.ini文件
在web应用程序中可以通过很多方式对Shiro进行配置，在你使用的web和/或MVC框架都可以。举个例子，可以通过Spring，Guice，Tapestry，许多许多。

暂时为了让事情简单些，我们通过shiro默认基于ini的配置启动一个shiro环境。

如果你迁出了step1分支，你可以看到`src/main/webapp/WEB-INF/shiro.ini`这个新文件的内容()
If you checked out the step1 branch, you’ll see the contents of this new src/main/webapp/WEB-INF/shiro.ini file (为了简洁删除了头部注释):
```
[main]
# Let's use some in-memory caching to reduce the number of runtime lookups against Stormpath.
# A real application might want to use a more robust caching solution (e.g. ehcache or a
# distributed cache).  When using such caches, be aware of your cache TTL settings: too high
# a TTL and the cache won't reflect any potential changes in Stormpath fast enough.  Too low
# and the cache could evict too often, reducing performance.
cacheManager = org.apache.shiro.cache.MemoryConstrainedCacheManager
securityManager.cacheManager = $cacheManager
```

This .ini contains simply a [main] section with some minimal configuration:

* 它定义了一个新的缓存管理实例. 缓存在Shiro的构架体系中是一个非常重要的部分 - 它减少了和数据存贮之间持续往返的通讯。这个例子是使用了在单个JVM上比较好使的MemoryConstrainedCacheManager。如果对你的应用是部署在多个服务器（比如服务器集群）的话，你将会想使用一个集群缓存管理器的实现来替代。
* 它将这个新的缓存管理器实例配置到在安全管理器上。shiro安全管理器实例永远是存在的，所以它不需要进行显示声明。

##1b:在web.xml中启用Shiro
当我们有一个shiro.ini配置，我们需要真正的加载它并启动启动一个新的Shiro环境，并且使环境在Web应用中可用。

我们通过添加一些配置到已存在的`src/main/webapp/WEB-INF/web.xml`文件中来做到这些。
```
<listener>
    <listener-class>org.apache.shiro.web.env.EnvironmentLoaderListener</listener-class>
</listener>

<filter>
    <filter-name>ShiroFilter</filter-name>
    <filter-class>org.apache.shiro.web.servlet.ShiroFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>ShiroFilter</filter-name>
    <url-pattern>/*</url-pattern>
    <dispatcher>REQUEST</dispatcher>
    <dispatcher>FORWARD</dispatcher>
    <dispatcher>INCLUDE</dispatcher>
    <dispatcher>ERROR</dispatcher>
</filter-mapping>
```

* `<listener>` 节点定义了一个ServletContextListener，在web应用程序启动的生时候启动Shiro环境（包括shiro的SecurityManager）默认情况下, 这个listener会自动去找我们的`WEB-INF/shiro.ini`文件中Shiro的配置.

* `<filter>` 节点定义了主要的ShiroFilter.这个filter被要求去过滤所有进入web应用程序的请求，因此shiro可以在一个请求到达应用程序之前进行必要的身份验证和访问控制。

* `<filter-mapping>` 节点确保所有请求类型通过被ShiroFilterare提出（filed）filter-mapping节点一般是不指定dispatcher元素的，但是shiro需要它们都被定义，以便它能够过滤所有可能被web应用执行的不同请求类型。

##1c: Run the webapp
在签出step1分支后，继续运行web应用程序：
```
$ mvn jetty:run
```
这次你讲看到类似以下日志输出，表明shiro确实已经运行在你的web应用程序中：
```
16:04:19.807 [main] INFO  o.a.shiro.web.env.EnvironmentLoader - Starting Shiro environment initialization.
16:04:19.904 [main] INFO  o.a.shiro.web.env.EnvironmentLoader - Shiro environment initialized in 95 ms.
```
按下ctrl+C (在mac上cmd+C)来关闭web应用程序

#Step2:连接用户信息

123123
