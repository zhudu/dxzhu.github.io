---
layout: wiki
title: tomcat environment
categories: tomcat
description: tomcat环境配置
keywords: javaee, tomcat
---

关于tomcat的配置问题

## 下载链接

进入[tomcat官网](http://tomcat.apache.org/)后，选择目标版本进行下载即可。

## 配置安装

本人此处下载的为tomcat-8.5.51版本，下载文件为zip格式。

正常将压缩包解压后，打开`/bin`路径下的**startup.bat**若能正常运行并且浏览器打开[localhost:8080](localhost:8080)且正常显示如下：

![](images\wiki\tomcat.png)

即为tomcat环境配置成功。(tomcat需在jdk环境下运行)

若需关闭tomcat，运行`/bin`路径下的**shutdown.bat**即可关闭该进程。

## 遇到的问题

问题一：双击startup.bat时出现闪退情况。

解决办法：查看环境变量中是否有：JAVA_HOME与CATALINA_HOME

若没有便添加这两个相关数据：

JAVA_HOME指向jdk文件路径

CATALINA_HOME指向tomcat文件路径

添加完后重新打开startup.bat便可以正常运行了。

## tomcat配置到eclipse

打开工具栏的window->Preferences->server->Runtime Environmets

点击add进行添加tomcat路径，在新窗口中选择tomcat版本号->next，添加文件路径即可配置完成。