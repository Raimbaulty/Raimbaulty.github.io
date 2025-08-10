---
layout: mypost
title: Cloudflare 免费 CDN
date: 2025-07-16
published: true
categories: 网站搭建
tags: 
 - cloudflare
 - cdn
---

## 开始之前

准备两个域名并托管到 Cloudflare

- **回退域**，建议使用价格低廉的6位数字xyz域名，下面以 `123456.xyz`为例
- **工作域**，建议使用接受度高的com、net、org域名，下面以 `example.com` 为例

  

## 配置回退域DNS

**注意**：替换 `123456.xyz` 为你的回退域

打开 `https://dash.cloudflare.com/?to=/:account/123456.xyz/dns/records` 添加两条记录

### 添加A记录

打开小黄云

- 名称： `origin`
- 内容：服务器IP地址

### 添加CNAME记录

关闭小黄云

- 名称： `cdn`

- 内容：优选域名，[点击获取](https://www.wetest.vip/page/cloudflare/cname.html) 获取

  

## 配置自定义域名

**注意：**替换 `123456.xyz` 为你的回退域

打开`https://dash.cloudflare.com/?to=/:account/123456.xyz/ssl-tls/custom-hostnames`

如果没有支付方式会要求绑定支付方式，选择Free计划，然后点击 **添加回退源**，填写 `origin.1234567.xyz`

接着点击 **添加自定义主机名**，填写 `example.com`，点击 ▼，按照提示添加下一步的TXT记录



## 配置工作域DNS

**注意：**替换`example.com` 为你的工作域，`123456.xyz` 为你的回退域

在新标签页打开 `https://dash.cloudflare.com/?to=/:account/example.com/dns/records` 添加条记录

### 添加 TXT记录

按照上一步的提示添加

### 添加CNAME记录

关闭小黄云

- 名称： `@`
- 内容： `origin.123456.xyz`

### 修改CNAME记录

回到第二步打开的页面，点击刷新等待验证生效后，将工作域添加的CNAME记录值由 `origin.123456.xyz` 改为 `cdn.123456.xyz`



## 验证优选结果

[点击这里](https://www.itdog.cn/) 输入域名后，点击单次查询查看连接情况