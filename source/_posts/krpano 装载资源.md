---
title: krpano 装载资源（二）
date: 2019-2-20 21:24:08
categories:
- krpano
tags:
- krpano
---

# 装载过程

下图用`MAKE VTOUR(NORMAL)droplet.bat` 为例展示 krpano 在生成全景应用程序过程中是有哪些设置和输出文件等。 

![krpano 装载资源\process](krpano 装载资源\process.png)

上面这副图展示了将图片拖入`MAKE VTOUR (NORMAL) droplet.bat`文件执行过程中所需要核心配置文件和载入的资源文件。

- vtour-normal.config

- basicettings.config

- vtourskin119.skin

- testing server

# 剖析 MAKE VTOUR (NORMAL) droplet.bat

`MAKE VTOUR (NORMAL) droplet.bat`

```shell
@echo off
echo MAKE VTOUR (NORMAL) droplet

IF "%~1" == "" GOTO ERROR
IF NOT EXIST "%~1" GOTO ERROR

set KRPANOTOOLSEXE=krpanotools64.exe
if "%PROCESSOR_ARCHITECTURE%" == "x86" set KRPANOTOOLSEXE=krpanotools32.exe

"%~dp0\%KRPANOTOOLSEXE%" makepano "%~dp0\templates\vtour-normal.config" %*
GOTO DONE

:ERROR
echo.
echo Drag and drop several panoramic images on this droplet to create 
echo automatically a virtual tour with normal single-res panos.

:DONE
echo.
pause
```

这段脚本中我们只关心有个`\templates\vtour-normal.config`外部配置文件。

# 剖析vtour-normal.config

此文件中在 3 处引入了外部内容

1. include basicsettings.config 

2. include vtourskin119.skin

3. include testing server

```bash
# Virtual Tour with Normal/Single-Resolution Panos
# Skin with Scrolling-Thumbnails, Bingmaps, Gyroscope, VR-Support
# Documentation: http://krpano.com/tools/kmakemultires/config?version=119
# krpano 1.19


# 基础设置
# basic settings
include basicsettings.config
panotype=autodetect
hfov=360
makescenes=true

# 设置 flash 和 html5 输出
# output
flash=true
html5=true

# 将全景图或球面图转换为立方图
# convert spherical/cylindrical to cubical
converttocube=true
converttocubelimit=360x45

# 多分辨率设置
# multiresolution settings
multires=false
maxsize=8000
maxcubesize=2048
stereosupport=true


# 输出立方体图片到目录
# output images path
tilepath=%INPUTPATH%/vtour/panos/%BASENAME%.tiles/pano[_c].jpg


# 预览图设置并输出到指定目录
# preview pano settings
preview=true
graypreview=false
previewsmooth=25
previewpath=%INPUTPATH%/vtour/panos/%BASENAME%.tiles/preview.jpg


# 生成比较小的手机图片（用来支持旧手机）
# generate smaller mobile images (optionally for supporting older mobile devices)
#customimage[mobile].size=1024
#customimage[mobile].stereosupport=true
#customimage[mobile].path=%INPUTPATH%/vtour/panos/%BASENAME%.tiles/mobile/pano_%s.jpg
#customimage[mobile].imagesettings=jpegquality=85 jpegsubsamp=444 jpegoptimize=true
#customimage[mobile].xml=<cube url="[PATH]" devices="mobile" />


# 设置并生成缩略图
# generate thumbnails
makethumb=true
thumbsize=240
thumbpath=%INPUTPATH%/vtour/panos/%BASENAME%.tiles/thumb.jpg


# 输出工程 xml
# xml output
xml=true
xmlpath=%INPUTPATH%/vtour/tour.xml


# 设置 html 输出
# html output/template
html=true
htmlpath=%INPUTPATH%/vtour/tour.html
htmltemplate_html5=auto

# 引入皮肤模板
# skin / xml template
include vtourskin119.skin


# include testing server
htmltemplate_additional_file=html/tour_testingserver.exe
htmltemplate_additional_file=html/tour_testingserver_macos+x
```

[krpanotools 配置文件](https://krpano.com/tools/kmakemultires/config/#top)

# 剖析 basicsettings.config

```bash
# Basic Settings
# Documentation: http://krpano.com/tools/kmakemultires/config?version=119
# krpano 1.19

# 图片滤镜和压缩设置
# image filtering and compression settings
filter=LANCZOS
jpegquality=82
jpegsubsamp=422
jpegoptimize=true

# 颜色描述文件处理
# color profile handling: skip, copy, convert or sRGB
profile=convert

# 从输入文件中加载 gps exif 信息 并转移到 xml文件
# load gps exif information from the input files and transfer them to the xml
parsegps=true

# 解析输入文件以获取方向/水平信息
# parse the input files for orientation / leveling information and either 
# 直接重新映射图像或在xml中添加预对齐设置以对其进行调整
# directly remap the images or add a prealign setting in the xml to level them
autolevel=remap


# 输出文件按字母排序
# sort the input files alphabetically
sortinput=true

# 保护设置（nomb= no Flash Player菜单栏，用于独立swf）
# protection settings (nomb = no Flashplayer menubar for standalone swf)
kprotectclparameters=-nomb
```

# 剖析vtourskin119.skin.conig

`vtourskin119.skin.config` 随着版本不断升级，此文件名也会随着升级但不会覆盖上一版本。

```bash
# Virtual Tour Skin - Design 1.19
# - Button bar
# - Scene/pano title
# - Previous/next scene buttons
# - Show/hide skin buttons
# - Hotspots style
# - Scrolling thumbnails
# - Bingmaps or Googlemaps plugin
# - Tooltips
# - Gyroscope plugin
# - MobileVR/WebVR support
# - Retina resolution images
# - Different mobile layout
# - Virtual Tour Editor compatible
# krpano 1.19-pr13

htmltemplate=html/embedpano.html

xmltemplate=xml/vtour.xml
xmltemplate_scene=xml/vtour_scenetemplate.xml
xmltemplate_view=xml/vtour_viewtemplate.xml
xmltemplate_additional_file=xml/skin/vtourskin.xml
xmltemplate_additional_file=xml/skin/vtourskin_design_black.xml
xmltemplate_additional_file=xml/skin/vtourskin_design_glass.xml
xmltemplate_additional_file=xml/skin/vtourskin_design_flat_light.xml
xmltemplate_additional_file=xml/skin/vtourskin_design_ultra_light.xml
xmltemplate_additional_file=xml/skin/vtourskin_design_117.xml
xmltemplate_additional_file=xml/skin/vtourskin_design_117round.xml
xmltemplate_additional_file=xml/skin/vtourskin.png
xmltemplate_additional_file=xml/skin/vtourskin_light.png
xmltemplate_additional_file=xml/skin/vtourskin_hotspot.png
xmltemplate_additional_file=xml/skin/vtourskin_mapspot.png
xmltemplate_additional_file=xml/skin/vtourskin_mapspotactive.png
xmltemplate_additional_file=xml/plugins/scrollarea.swf
xmltemplate_additional_file=xml/plugins/scrollarea.js
xmltemplate_additional_file=xml/plugins/combobox.xml
xmltemplate_additional_file=xml/plugins/bingmaps.swf
xmltemplate_additional_file=xml/plugins/bingmaps.js
xmltemplate_additional_file=xml/plugins/googlemaps.js
xmltemplate_additional_file=xml/plugins/videoplayer.swf
xmltemplate_additional_file=xml/plugins/videoplayer.js
xmltemplate_additional_file=xml/plugins/soundinterface.swf
xmltemplate_additional_file=xml/plugins/soundinterface.js
xmltemplate_additional_file=xml/plugins/gyro2.js
xmltemplate_additional_file=xml/plugins/webvr.js
xmltemplate_additional_file=xml/plugins/webvr.xml
xmltemplate_additional_file=xml/plugins/webvr_cursor_80x80_17f.png
xmltemplate_additional_file=xml/skin/rotate_device.png
```

该文件主要包含，主要配置（vtour.xml）、场景模板（vtour_scenetemplate.xml）、视图模板（vtour_viewtemplate.xml）、皮肤、插件。这些文件在生成全景应用程序的执行中会输出到其目录中。

# 文件路径说明

- ％FIRSTXML％ - 第一个加载的xml文件的路径。

- ％CURRENTXML％ - 当前加载的主xml文件的路径（不是包含的文件）。

- ％SWFPATH％ - krpano查看器文件的路径。

- ％HTMLPATH％ - html文件的路径。

- ％BASEDIR％ - 使用[basedir](https://krpano.com/docu/xml/#krpano.basedir)路径。

- ％$ VARIABLE％ - 使用给定'VARIABLE'的值 - 这可以是任何krpano变量，但必须在加载当前xml或场景之前定义，例如在嵌入期间已经在html文件中（通过[initvars](https://krpano.com/docu/html/#initvars)）或在loadpano（），loadscene（）调用之前。

- ％INPUTPATH％ - 这是当前输入文件的完整路径。它可用于在与输入文件相同的文件夹中生成输出文件

- ％BASENAME％ - 这是输入图像的“基本名称”。

# 小结

以上内容对 krpano 使用`MAKE VTOUR (NORMAL) droplet.bat`生成全景程序过程中需要装配资源进行了分析，有了初步认识我们可以按自己的需要来定制配置文件，使其满足个性化需求包括输入输出设定、皮肤、插件、图片等。

下一章节剖析 krpano 的 xml 配置文件
