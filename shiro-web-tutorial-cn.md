原文地址：[http://shiro.apache.org/webapp-tutorial.html](http://shiro.apache.org/webapp-tutorial.html)

本文档是一个用Apache Shiro保护web应用的分步引导教程，架设你已经了解了shiro的入门知识，并且至少熟悉一下两个入门文档：

[Application Security with Apache Shiro](http://www.infoq.com/articles/apache-shiro)  
[Apache Shiro 10 Minute Tutorial](http://shiro.apache.org/10-minute-tutorial.html)

这个分布教程将花你大概45-60分钟的时间去完成，当你结束以后，你将会对shiro在web应用中是如何工作非常有概念.

#内容列表
[概述](#概述)  
[项目安装（初始化？）](#项目安装)  
Step 1: [启用Shiro](#step1)  
Step 2: [链接用户数据存储](#step2)  
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

