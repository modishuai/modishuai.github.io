---
title: IIS安装SSL证书
date: 2018-08-28 22:50:23
categories:
- web
tags:
- iis
- ssl

---

# IIS 安装SSL证书

## 获得SSL证书

| 序号  | 供应商                                         | 说明                                                                           |
| --- | ------------------------------------------- | ---------------------------------------------------------------------------- |
| 1   | [亚洲诚信](https://www.trustasia.com/trustasia) | SSL 证书代理商，腾讯云支持                                                              |
| 2   | [天威诚信](http://www.itrus.com.cn/)            | SSL 证书代理商，阿里云支持                                                              |
| 3   | [赛门铁克](https://www.symantec.com)            | Symantec是 SSL/TLS 证书的领先提供商，为全球一百多万台网络服务器提供安全防护。                              |
| 4   | [GeoTrust](https://www.geotrust.com/ssl/)   | GeoTrust是全球第二大数字证书颁发机构，已被Symantec收购。                                         |
| 5   | [GlobalSign](https://globalsign.cn/)        | GMO GlobalSign是全球最早的数字证书认证机构之一，一直致力于网络安全认证及数字证书服务，是一个备受信赖的 CA 和 SSL 数字证书提供商。 |
| 6   | [CFCA](http://www.cfca.com.cn/)             | 中国金融认证中心(CFCA)证书，由中国数字证书认证机构自主研发，纯国产证书。                                      |

IIS 下的证书文件`www.domain.com.pfx`

## 证书安装

1. 打开IIS服务管理器，点击服务器名称，双击IIS模块中的“服务器证书”。

   ![IIS安装SSL证书\1](IIS安装SSL证书\1.png)

2. 打开服务器证书后，单机右侧操作面板中的“导入”。

   ![IIS安装SSL证书\2](IIS安装SSL证书\2.png)

3. 打开导入证书窗口，选择“证书文件”，如果在申请证书的时候填写了私钥密码，这里也需要输入私钥密码，单击确定。

   ![IIS安装SSL证书\3](IIS安装SSL证书\3.png)

4. 选择你的站点名称，单击右侧操作面板中的“绑定”。

   ![IIS安装SSL证书\4](IIS安装SSL证书\4.png)

5. 打开网站绑定窗口，单击“添加”，添加网站绑定内容，指定类型为https，端口443，并指定SSL证书，单击确定。

   ![IIS安装SSL证书\5](IIS安装SSL证书\5.png)




