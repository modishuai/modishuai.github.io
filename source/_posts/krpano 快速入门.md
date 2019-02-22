---
title: krpano 快速入门（一）
date: 2019-2-20 21:24:08
categories:
- krpano
tags:
- krpano
---

# krpano 全景查看器（入门篇）

> [官方链接](https://krpano.com/)
> 
> [工具下载](https://krpano.com/download/)
> 
> [入门教程](https://krpano.com/download/)

## 安装目录

![krpano 快速入门\install_directory](krpano 快速入门\install_directory.png)

该目录主要几部分如下：

- docu（官方文档）

- templates（配置文件、默认插件、默认皮肤、默认图片）

- viewer（官方示例）

- *.bat （批处理文件，将图片拖拽到此文件上执行批量处理）

### 批处理文件

- MAKE PANO (FLAT) droplet.bat  （生成平面平铺的多分辨率图像。）

  - 用于平面/ 2D平面/图像，平铺多分辨率，无尺寸/分辨率限制，平面全景/图像的皮肤/控制，仅限Flash

- MAKE PANO (MULTIRES) droplet.bat （生成平铺的多分辨率全景图）

  - 用于平铺多分辨率全景，无大小/分辨率限制，快速和动态加载，带按钮的默认皮肤，支持 HTML5 iPhone / iPad

- MAKE PANO (NORMAL) droplet.bat （生成正常（单分辨率）全景图）

  - 对于典型的全屏全景，完整的全景将立即加载，最大。 cubeize 默认限制为2200像素（可以在'templates / normal.config'文件中更改），没有皮肤 - 只需全息，支持 HTML5 iPhone / iPad

- MAKE VTOUR (MULTIRES) droplet.bat （生成平铺的多分辨率全景图并将它们连接到虚拟游览。）

  - 用于自动构建虚拟游览，将所有全景图像放在此 droplet 上，通过自动缩略图生成来移动皮肤，支持HTML5 iPhone / iPad

- MAKE VTOUR (NORMAL) droplet.bat （生成普通（单分辨率）全景图并将它们连接到虚拟游览。）

  - 同上

- MAKE VTOUR (VR-OPT) droplet.bat

  - 自动生成可漫游的全景，将所有全景图同时投放到 droplet 上，自动生成缩略图皮肤，支持 HTML5 iPhone/ iPad

关于这几个批处理文件详细信息及区别参考[krpano Droplets](https://krpano.com/tools/droplets/#makevtournormal)

### 生成全景目录

根据需要选择合适的 MAKE PANO droplets ，我这里使用 MAKE VTOUR (NORMAL) droplet.bat 来生成全景目录。

注意：全景图片文件名，不要以中文命名。

![生成全景图目录](krpano 快速入门\directory.png)

- panos （存放图片）

- plugins （存放插件）

- skin（皮肤目录，包含图片和 xml 配置文件）

- tour.xml （主要的配置文件）

注意：访问 tour.html 需要启动 tour_testingserver.exe 文件，或者部署到本地服务器进行访问。

# 小结

按照官方的快速入门教程指导如何快速生成全景应用程序，就几个关键步骤：

（1）安装工具并找到安装目录。

（2）按需选择*.bat 程序并将图片拖入此程序。

（3）输出全程应用程序（默认在图片存放的根路径下）

根据特定的需求选择合适的 makevtour 程序，想要在PC  和 Mobile 中都有更好的性能和稳定型，就选择`MAKE VTOUR (MULTIRES) droplet.bat`

需要注意的问题：1. 不能以中文命名图片文件名称； 2. 全景应用程序基于服务器不说才能访问。 3. 若改动默认的目录结构，就需要改变配置文件中定义的路径，包括 xml 和皮肤（不建议改动）。
