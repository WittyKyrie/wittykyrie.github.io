---
title: 创建一个本地的Ftp服务器
author: wittykyrie
date: 2022-10-25 09:00:00 +0800
categories: [Ftp,Server]
tags: [Ftp,Server]
render_with_liquid: false
---

FTP是File Transfer Protocol(文件传输协议)，支持FTP协议的服务器就是FTP服务器。也可以理解为是在互联网上提供文件存储和访问服务的计算机。

## 创建一个FTP服务器

FTP服务器的搭建有各种各样的软件，且在不同的操作系统（windows，linux，mac）上都能进行部署，这里我是在Windows的操作系统上进行的部署，用到的软件是[FileZilla](https://www.filezilla.cn/)，是一款免费的Ftp服务器部署软件。可以点击链接去官网进行下载。

下载完成之后双击点击安装，基本上一路点击确定就行。
![FilleZilla服务器界面](/assets/2022-10-25-Assets/ftpServer.jpg){: .shadow width="1000" height="600" style="max-width: 90%" }

点击菜单栏的编辑按钮点击用户，在右边框框内点击添加，设置相应的账号密码,点击左侧的Shared folders，选择作为服务器根节点的文件夹，将文件的权限全部打开。设置完成之后应该是如下图一样。
![](/assets/2022-10-25-Assets/1.jpg){: .shadow width="1000" height="600" style="max-width: 90%" }

## 在Unity当中如何链接FTP服务器

