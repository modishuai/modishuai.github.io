---
title: krpano 配置文件（三）
date: 2019-2-22 21:24:08
categories:
- krpano
tags:
- krpano
---

# krpano XML

> krpano 使用xml 文件来存储 krpano 的设置。任何文本编辑器都能进行编辑，但需要遵守标准的 xml 语法规则。
> 
> [krpano XML 文档](https://krpano.com/docu/xml/#top)
> 
> 我将此文件称为“主配置文件”。因为它是整个全景程序运行的核心文件，充当着关键数据存储库，krpano 运行时将 xml 元素节点数取出存入运行时数据结构，此文件存放在根目录中。
> 
> 此文件默认名称`tour.xml`，文件名称取决于在`vtour-normal.config` 配置文件中的定义
> 
> ```powershell
> # xml output
> xml=true
> xmlpath=%INPUTPATH%/vtour/tour.xml
> ```

# krpano xml 文件结构

```powershell
<krpano>
    <include>
    <preview>
    <image>
    <view>
    <area>
    <display>
    <control>
    <cursors>
    <autorotate>
    <plugin>
    <layer>
    <hotspot>
    <style>
    <events>
    <action>
    <contextmenu>
    <network>
    <memory>
    <security>
    <textstyle>
    <lensflareset>
    <lensflare>
    <data>
    <scene>
    <set>
    <debug>
</krpano>
```



文件结构详细信息移步至[官方文档](官方文档)，这里着重介绍几点：

1. krpano 作为文件根节点，其他节点必须包含在内。

2. krpano 中的所有子节点是可选项，完全一样的节点定义多个会被后面的覆盖。

3. krpano 根节点中还可以包含krpano 节点。 

# 节点介绍

关于节点属性请点击表格中标签移步[官方文档](https://krpano.com/docu/xml/)浏览

| [include](https://krpano.com/docu/xml/#include)           | 包含其他 xml 文件，建议将所有包含外部节点放在根节点首行。  |
| --------------------------------------------------------- | -------------------------------- |
| [preview](https://krpano.com/docu/xml/#preview)           | 全景图像预览图，一般包含在<scene>节点中。         |
| [image](https://krpano.com/docu/xml/#image)               | 定义全景图像                           |
| [view](https://krpano.com/docu/xml/#view)                 | 视图，可配置视野位置、水平、垂直方向、缩放限制。         |
| [area](https://krpano.com/docu/xml/#area)                 | 定义应显示全景图像的区域/窗口                  |
| [display](https://krpano.com/docu/xml/#display)           | 配置显示模式和渲染质量以及性能                  |
| [control](https://krpano.com/docu/xml/#control)           | 鼠标键盘控制                           |
| [cursors](https://krpano.com/docu/xml/#cursors)           | 鼠标光标样式                           |
| [autorotate](https://krpano.com/docu/xml/#autorotate)     | 自动旋转，没有用户交互的情况下开启总动旋转。           |
| [plugin](https://krpano.com/docu/xml/#plugin)             | 引入外部插件                           |
| [layer](https://krpano.com/docu/xml/#layer)               | 用来定义图片、容器、文本                     |
| [hotspot](https://krpano.com/docu/xml/#hotspot)           | 热点，对鼠标悬停或点击做交互。可支持多边形            |
| [style](https://krpano.com/docu/xml/#style)               | 定义通用样式                           |
| [events](https://krpano.com/docu/xml/#events)             | 用于在特定事件发生时调用动作或函数。分全局事件和独立本地事件。  |
| [action](https://krpano.com/docu/xml/#action)             | 定义动作，                            |
| [contextmenu](https://krpano.com/docu/xml/#contextmenu)   | 自定义鼠标右键菜单                        |
| [network](https://krpano.com/docu/xml/#network)           | 网络设置，显示错误消息之前加载（服务器）错误的自动下载重试次数。 |
| [memory](https://krpano.com/docu/xml/#memory)             | 内存使用设置，HTML5：桌面150-400MB，手机：75MB |
| [security](https://krpano.com/docu/xml/#security)         | 跨域安全设置                           |
| [textstyle](https://krpano.com/docu/xml/#textstyle)       | 文本样式。不再推荐使用。                     |
| [lensflareset](https://krpano.com/docu/xml/#lensflareset) | 镜头光晕设置，仅限Flash                   |
| [data](https://krpano.com/docu/xml/#data)                 | 原始数据，可存储任何类型的信息或数据。krpano不会解析其内容 |
| [scene](https://krpano.com/docu/xml/#scene)               | 场景                               |
| [set](https://krpano.com/docu/xml/#set)                   | 在 xml 解析期间设置/定义变量                |
| [debug](https://krpano.com/docu/xml/#debug)               | 允许 xml 解析期间跟踪/记录某些内容。            |
| skin_settings                                             | 允许自定义设置，包括皮肤、地图、导航、文本等等。         |

## skin_settings 节点

官方网站没有介绍此节点，但是在生成后的“主配置文件”中包含此节点，并给出注解。

此节点的主要作用就是允许你自定义任何类型的配置节点。

```powershell
<!-- customize skin settings: maps, gyro, webvr, thumbnails, tooltips, layout, design, ... -->
	<skin_settings maps="false"
	               maps_type="google"
	               maps_bing_api_key=""
	               maps_google_api_key=""
	               maps_zoombuttons="false"
	               gyro="true"
	               webvr="true"
	               webvr_gyro_keeplookingdirection="false"
	               webvr_prev_next_hotspots="true"
	               littleplanetintro="true"
	               title="true"
	               thumbs="true"
	               thumbs_width="120" thumbs_height="80" thumbs_padding="10" thumbs_crop="0|40|240|160"
	               thumbs_opened="false"
	               thumbs_text="true"
	               thumbs_dragging="true"
	               thumbs_onhoverscrolling="false"
	               thumbs_scrollbuttons="false"
	               thumbs_scrollindicator="false"
	               thumbs_loop="false"
	               tooltips_buttons="false"
	               tooltips_thumbs="false"
	               tooltips_hotspots="false"
	               tooltips_mapspots="false"
	               deeplinking="false"
	               loadscene_flags="MERGE"
	               loadscene_blend="OPENBLEND(0.5, 0.0, 0.75, 0.05, linear)"
	               loadscene_blend_prev="SLIDEBLEND(0.5, 180, 0.75, linear)"
	               loadscene_blend_next="SLIDEBLEND(0.5,   0, 0.75, linear)"
	               loadingtext="加载中..."
	               layout_width="100%"
	               layout_maxwidth="814"
	               controlbar_width="-24"
	               controlbar_height="40"
				   scene_select_text="场景选择"

	               controlbar_offset="20"
	               controlbar_offset_closed="-40"
	               controlbar_overlap.no-fractionalscaling="10"
	               controlbar_overlap.fractionalscaling="0"
	               design_skin_images="vtourskin.png"
	               design_bgcolor="0x2D3E50"
	               design_bgalpha="0.8"
	               design_bgborder="0"
	               design_bgroundedge="1"
	               design_bgshadow="0 4 10 0x000000 0.3"
	               design_thumbborder_bgborder="3 0xFFFFFF 1.0"
	               design_thumbborder_padding="2"
	               design_thumbborder_bgroundedge="0"
	               design_text_css="color:#FFFFFF; font-family:Arial;"
	               design_text_shadow="1"
	               />
```

此节点中的自定义属性，可以在其他 xml 配置文件中获取。获取方式如下：`get:variable`

```powershell
<layer name="skin_btn_thumbs_text" type="text" align="center" x="50" y="10" html="get:skin_settings.controlbar_scene_select_text"  bg="false" />
```

# 小结

下一章节介绍下动作和脚本还有样式，回顾本章掌握 krpano xml 配置文件结构及其用途，关键节点上属性及作用官网文档有更好的说明就不再赘述。
