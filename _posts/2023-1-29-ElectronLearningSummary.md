---
title: Electron编写服务器学习总结
author: wittykyrie
date: 2023-1-29 09:00:00 +0800
categories: ['Electron']
tags: ['web','electron','server']
render_with_liquid: false
---

## 学习资源相关文档地址

[Electron官方文档](https://www.electronjs.org/zh/docs/latest/tutorial/tutorial-prerequisites)

[Web学习相关知识](https://developer.mozilla.org/zh-CN/docs/Learn)

[Node.Js相关文档](https://nodejs.dev/zh-cn/learn/)

## 需求清单

   1. 使用Electron框架编写服务器，通过http来进行访问
   2. 服务器可以自助打开或者关闭，当关闭时游戏客户端请求开启服务会返回False

## 关键概念

### Electron框架

Electron是一个使用 JavaScript、HTML 和 CSS 构建桌面应用程序的框架。嵌入 Chromium 和 Node.js 到 二进制的 Electron 允许您保持一个 JavaScript 代码代码库并创建 在Windows上运行的跨平台应用 macOS和Linux——不需要本地开发 经验。

### package.json

在初始化并且安装完 Electron 之后，您的 package.json 应该长下面这样。 文件夹中会出现一个 node_modules 文件夹，其中包含了 Electron 可执行文件；还有一个 package-lock.json 文件，指定了各个依赖的确切版本。

您在 package.json 中指定的脚本文件 main 是所有 Electron 应用的入口点。 这个文件控制 主程序 (main process)，它运行在 Node.js 环境里，负责控制您应用的生命周期、显示原生界面、执行特殊操作并管理渲染器进程 (renderer processes)

因为 Electron 的主进程是一个 Node.js 运行时，您可以使用 electron 命令执行任意 Node.js 代码（甚至将其用作 REPL）。 要执行这个脚本，在 package.json 的 scripts 字段中添加一个 start 命令，执行内容为 electron . 。 这个命令会告诉 Electron 在当前目录下寻找主脚本，并以开发模式运行它。

```json
{
  "name": "my-electron-app",
  "version": "1.0.0",
  "description": "Hello World!",
  "main": "main.js",
  "author": "Jane Doe",
  "license": "MIT",
  "scripts": {
    "start": "electron ."
  },
  "devDependencies": {
    "electron": "^19.0.0"
  }
}
```

### BrowserWindow

在 Electron 中，每个窗口展示一个页面，后者可以来自本地的 HTML，也可以来自远程 URL。

窗口的生命周期时通过监听app和BrowserWindow模块的事件来自行的自定义窗口。

### preLoad预加载脚本

Electron拥有两类进程，一类是主进程，一类是渲染进程，渲染进程默认在网页页面上。因此当我们希望两个进程之间变量可以互相通信的时候，就需要使用preload脚本来进行预加载。在BrowserWindow构造器中传入脚本的的路径来进行使用。

在preload脚本当中，我们在脚本内定义全局变量，新建renderer脚本，通过DOM接口来替换HTML元素的显示文本。具体内容可以看文章最上面的官网示例。

主进程和渲染进程之间有着清楚的分工，因此如果我们想要在进程直接互相访问接口则需要使用Electron的ipcMain模块和ipcRenderer模块来进行进程之间的通信。

为了从你的网页向主进程发送消息，你可以使用 ipcMain.handle 设置一个主进程处理程序（handler），然后在预处理脚本中暴露一个被称为 ipcRenderer.invoke 的函数来触发该处理程序（handler）。

因此我们可以在renderer脚本当中调用在preload脚本当中暴露的接口，其返回值和逻辑在主进程当中设置handle监听器来确定。


