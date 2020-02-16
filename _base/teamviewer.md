---
layout: post
title: teamviewer实现内网穿透
categories: Tools
description: teamviewer实现内网穿透
keywords: teamviewer,渗透
---

本文将介绍一种通过teamviewer的vpn功能实现外网访问内网的操作，简单来说，就是通过teamviewer的vpn实现外网访问内网的代理，再通过Proxifier辅助实现全局代理，即可实现外网访问内网的操作。这样的方法相对于proxychains4、端口转发等操作来说更方便，这里不做过多阐述。

首先我们要有一部内网中的电脑，并且安装好teamviewer。

# 一、安装teamviewer的vpn驱动

该操作是处于内外网的电脑都需要安装的。

找到teamviewer中`其他`->`选项`

![TIM图片20200204134847](/images/posts/tools/TIM图片20200204134847.jpg)

打开高级中的高级选项

![TIM图片20200204134943](/images/posts/tools/TIM图片20200204134943.jpg)

找到vpn该选项，进行安装

![TIM图片20200204134950](/images/posts/tools/TIM图片20200204134950.jpg)

安装完成后，我们可以在主界面找到vpn这个选项

![TIM图片20200204135241](/images/posts/tools/TIM图片20200204135241.jpg)

那么我们处于外网中的电脑就可以通过该vpn选项连接到内网那台电脑上，连接id与密码皆与原本相同。

连接成功后将会出现如下窗口：我们可以看到vpn连接后内外网电脑的ip地址，我们也可以尝试ping通来检查是否可以成功连接到内网，注意这里ping的ip地址是箭头所指的`伙伴的ip`

![TIM图片20200204135726](/images/posts/tools/TIM图片20200204135726.jpg)

# 二、更改虚拟网络属性

打开`“网络和internet”设置`->`更改适配器选项`，仅保留iPv4，其他都不选。

![TIM图片20200204140536](/images/posts/tools/TIM图片20200204140536.jpg)

# 三、关闭防火墙

设置中可以找到，不做过多阐述。

# 四、安装并配置Proxifier

安装包可在网上找到，双击即可安装，安装过程不再叙述。

安装完成后打开`配置文件`->`代理服务器`

![image-20200204141840207](/images/posts/tools/image-20200204141840207.png)

添加配置

![TIM图片20200204142006](/images/posts/tools/TIM图片20200204142006.png)

配置如图：地址即为teamviewer窗口中显示的`伙伴的ip`

![TIM图片20200204142147](/images/posts/tools/TIM图片20200204142147.png)

填写完成后，点击确定即可。这样我们就能连接上内网了，并且所有上网操作都是先访问到内网，再通过内网的网络进行操作。

个人在服务器上编程通常使用vs code，该方法亦可支持vs code的远程访问，对于代码修改，调参等方面都是较为方便的。而在往后要使用时，仅需链接teamviewer的vpn并且打开proxifier即可连接到内网。