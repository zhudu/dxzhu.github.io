---
layout: base
title: teamviewer
categories: database
description: teamviewer内网穿透
keywords: teamviewer
---

本文将介绍一种通过teamviewer的vpn功能实现外网访问内网的操作，简单来说，就是通过teamviewer的vpn实现外网访问内网的代理，再通过Proxifier辅助实现全局代理，即可实现外网访问内网的操作。这样的方法相对于proxychains4、端口转发等操作来说更方便，这里不做过多阐述。

首先我们要有一部内网中的电脑，并且安装好teamviewer。

# 一、安装teamviewer的vpn驱动

该操作是处于内外网的电脑都需要安装的。

找到teamviewer中