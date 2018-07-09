---

## title: Hexo 写作

# Hexo 如何写作？

[Hexo 中文文档](https://hexo.io/zh-cn/docs/writing.html)

```
hexo new [layout] <title>
```

您可以在命令中指定文章的布局（layout），默认为 post，可以通过修改 _config.yml 中的 default_layout 参数来指定默认布局。

## 草稿

```
hexo publish [layout] <title>
```

## Front-matter

Front-matter 是文件最上方以 --- 分隔的区域，用于指定个别文件的变量，举例来说：

```
title: Hello World
date: 2013/7/13 20:46:25
---
```

## 分类和标签

```
categories:
- Diary
tags:
- PS3
- Games
```

## 资源文件

资源（Asset）代表 source 文件夹中除了文章以外的所有文件，例如图片、CSS、JS 文件等。比方说，如果你的Hexo项目中只有少量图片，那最简单的方法就是将它们放在 source/images 文件夹中。然后通过类似于```![](/images/image.jpg)``` 的方法访问它们。

## 文章资源文件夹

对于那些想要更有规律地提供图片和其他资源以及想要将他们的资源分布在各个文章上的人来说，Hexo也提供了更组织化的方式来管理资源。这个稍微有些复杂但是管理资源非常方便的功能可以通过将 config.yml 文件中的 post_asset_folder 选项设为 true 来打开。

```
_config.yml
post_asset_folder: true
```

当资源文件管理功能打开后，Hexo将会在你每一次通过 ```hexo new [layout] <title>``` 命令创建新文章时自动创建一个文件夹。这个资源文件夹将会有与这个 markdown 文件一样的名字。将所有与你的文章有关的资源放在这个关联文件夹中之后，你可以通过相对路径来引用它们，这样你就得到了一个更简单而且方便得多的工作流。

## 相对路径引用的标签插件

通过常规的 markdown 语法和相对路径来引用图片和其它资源可能会导致它们在存档页或者主页上显示不正确。在Hexo 2时代，社区创建了很多插件来解决这个问题。但是，随着Hexo 3 的发布，许多新的标签插件被加入到了核心代码中。这使得你可以更简单地在文章中引用你的资源。

```
{% asset_path slug %}
{% asset_img slug [title] %}
{% asset_link slug [title] %}
```

## 服务器

默认端口：4000 ```hexo server ``` 

指定端口:5000 ```hexo server -p 5000```

## 静态模式

在静态模式下，服务器只处理 public 文件夹内的文件，而不会处理文件变动，在执行时，您应该先自行执行 hexo generate，此模式通常用于生产环境（production mode）下。
```hexo server -s```

## 生成文件

生成静态文件
``` hexo generate ```

## 监听文件变动

```hexo generate --watch```

## 完成部署

```hexo gererate --deploy```
```hexo deploy --generate```

```hexo g -d``` 

```hexo d -g ```
