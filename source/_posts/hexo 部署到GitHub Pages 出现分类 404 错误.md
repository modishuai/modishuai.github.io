---
title: hexo 部署到GitHub Pages 出现分类 404 错误
categories:
- hexo
tags:
- hexo
---
# hexo 部署到GitHub Pages 出现分类 404 错误

[参考解决方案](http://1mhz.me/2015/hexo-deploy-case-sensitive/)

1\. 修改 `.deploy_git\.git\config` 配置文件.

    将 ignorecase = true 修改成 ignorecase = false

2\. 删除`.deploy_git\`下所有文件

```shell
cd .deploy_git\ 
git rm -rf *
git commit -m 'clean all file'
git push
```

3\. 让 hexo 清除缓存和生成部署

```shell
cd ../
hexo clean
hexo deploy -generate
```
