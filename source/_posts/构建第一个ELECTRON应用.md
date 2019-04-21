---
title: 构建一个 Electron 应用
date: 2019-04-14 17:10:00
categories:
- Electron
tags:
- JavaScript
- node.js
---
> 准备环境、开发、运行、打包、安装应用

# 准备环境

Electron 依赖 Node.js 和 Chromium；

下载并安装[ Node.js]([https://nodejs.org/en/](https://nodejs.org/en/) 

## 淘宝NPM镜像

使用淘宝定制的[cnpm](https://github.com/cnpm/cnpm)(gzip 压缩支持) 命令行工具代替默认的`npm`:

```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

# 构建 Electron 应用

## 初始化项目目录

一、新建项目文件目录

二、进入项目目录在命令行工具中打开，执行`cnpm init`

三、在项目目录中 npm 会自动帮你创建 package.json 文件。

```json
{
  "name": "hello",
  "version": "0.1.0",
  "description": "first electron app",
  "main": "main.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
    "start": "electron ."

  },
  "author": "mofei",
  "license": "ISC"
}
```

**注意：**main 字段表示应用的启动脚本，若 main 字段不存在 Electron 则加载 index.js 脚本

## 安装 Electron

在项目中安装（首选）

```bash
cnpm install --save-dev electron
```

全局安装

```bash
cnpm install electron -g
```

## Hello Electron

创建 main.js

```javascript
const { app, BrowserWindow } = require('electron')

// 保持对window对象的全局引用，如果不这么做的话，当JavaScript对象被
// 垃圾回收的时候，window对象将会自动的关闭
let win

function createWindow () {
  // 创建浏览器窗口。
  win = new BrowserWindow({ width: 800, height: 600 })

  // 然后加载应用的 index.html。
  win.loadFile('index.html')

  // 打开开发者工具
  win.webContents.openDevTools()

  // 当 window 被关闭，这个事件会被触发。
  win.on('closed', () => {
    // 取消引用 window 对象，如果你的应用支持多窗口的话，
    // 通常会把多个 window 对象存放在一个数组里面，
    // 与此同时，你应该删除相应的元素。
    win = null
  })
}

// Electron 会在初始化后并准备
// 创建浏览器窗口时，调用这个函数。
// 部分 API 在 ready 事件触发后才能使用。
app.on('ready', createWindow)

// 当全部窗口关闭时退出。
app.on('window-all-closed', () => {
  // 在 macOS 上，除非用户用 Cmd + Q 确定地退出，
  // 否则绝大部分应用及其菜单栏会保持激活。
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

app.on('activate', () => {
  // 在macOS上，当单击dock图标并且没有其他窗口打开时，
  // 通常在应用程序中重新创建一个窗口。
  if (win === null) {
    createWindow()
  }
})

// 在这个文件中，你可以续写应用剩下主进程代码。
// 也可以拆分成几个文件，然后用 require 导入。
```

创建 index.html

```tsx
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Hello Electron!</title>
  </head>
  <body>
    <h1>Hello Electron!</h1>
  </body>
</html>
```

启动应用，在项目目录中执行

- cnpm start （首选）

- electron .

- electron ，将应用程序拖入窗口中。

![构建第一个ELECTRON应用\hello-electron](构建第一个ELECTRON应用\hello-electron.jpg)

# 应用打包

安装打包工具

全局安装

```bash
npm install electron-packager --save-dev
```

打包命令格式：参考 [usage.txt]([https://github.com/electron-userland/electron-packager/blob/master/usage.txt](https://github.com/electron-userland/electron-packager/blob/master/usage.txt)  详细说明

```bash
electron-packager <sourcedir> <appname> --platform=<platform> --arch=<arch> [optional flags...]
```

我的打包格式如下：

```bash
electron-packager . HelloElectron --platform=win32 --arch=ia32 HelloElectron

electron-packager ./ HelloElectron --platform=win32 --arch=ia32 --out ../Out/HelloElectron
```

打包所有平台：

```bash
electron-packager . --all
```

# 构建 Windows 安装程序

需要使用 Electron Installer Grunt 插件，[grunt]([https://gruntjs.com/getting-started](https://gruntjs.com/getting-started) 查看入门，尤其是 **[Installing Grunt and gruntplugins](https://gruntjs.com/getting-started#installing-grunt-and-gruntplugins)**  , 我在这里栽了个跟斗。

## 安装

```bash
cnpm install grunt --save-dev
cnpm install grunt-electron-installer --save-dev
```

运行 grunt 

提示如下错误：

```bash
grunt-cli: The grunt command line interface (v1.3.2)

Fatal error: Unable to find local grunt.

If you're seeing this message, grunt hasn't been installed locally to
your project. For more information about installing and configuring grunt,
please see the Getting Started guide:
```

解决，Unable to find local grunt 异常使用如下命令：

```bash
cnpm install grunt --save-dev
```

## 配置

在应用目录中新建 package.json（安装grunt 会自动创建） 和 Gruntfile.js

```json
{
  "name": "Hello",
  "version": "1.0.0",
  "devDependencies": {
    "grunt": "^1.0.4",
    "grunt-electron-installer": "^2.1.0"
  }
}
```

Gruntfile.js 中的 outputDirectory 字段值一定是要 installer 否则就没有安装程序，哪怕是grunt 运行成功。

```javascript
var grunt = require("grunt")

grunt.config.init({
    pkg: grunt.file.readJSON('package.json'),
    'create-windows-installer': {
        x64: {
            appDirectory: './hello-win32-x64',
            outputDirectory: './installer/64',

            authors: 'mofei',
            exe: 'hello.exe'
        },
        ia32: {
            appDirectory: './hello-win32-ia32',
            outputDirectory: './installer/32',

            authors: 'mofei',
            exe: 'hello.exe'
        }
    }
})
grunt.loadNpmTasks('grunt-electron-installer')
grunt.registerTask('default', ['create-windows-installer'])
```

## 运行

命令行工具输入 grunt 开始构建安装包

```bash
PS D:\Repository\Electron\electron-hello\build> grunt
Running "create-windows-installer:x64" (create-windows-installer) task

Running "create-windows-installer:ia32" (create-windows-installer) task

Done.
PS D:\Repository\Electron\electron-hello\build>
```

## 相关资料

[TAONPM:electron-package](https://npm.taobao.org/package/electron-packager#installation)

[NPM:electron-package](https://www.npmjs.com/package/electron-packager)

[GitHub:electron-package](https://github.com/electron-userland/electron-packager)

[github:grunt-electron-install](https://github.com/electron-archive/grunt-electron-installer)

[https://blog.csdn.net/qq_26626113/article/details/81022347](https://blog.csdn.net/qq_26626113/article/details/81022347)

[https://blog.csdn.net/w342916053/article/details/51701722](https://blog.csdn.net/w342916053/article/details/51701722)

[https://juejin.im/entry/5805e39ad20309006854e58f](https://juejin.im/entry/5805e39ad20309006854e58f)
