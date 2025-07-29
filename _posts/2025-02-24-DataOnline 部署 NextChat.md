---
layout: mypost
title: DataOnline 部署 NextChat
date: 2025-02-24
published: true
categories: 自部署
tags: 
 - linux
---

# DataOnline 部署 NextChat

## 创建域名

[点击这里](https://sv66.dataonline.vn:2222/evo/user/domains) 创建域名，然后添加一条A记录指向`103.137.185.66`

## 部署服务

- SSH 登入

- 克隆仓库

  ```bash
  git clone https://github.com/ChatGPTNextWeb/ChatGPT-Next-Web NextChat && cd $_
  wget https://pan.660707.xyz/f/qZYh1/nextchat_linux.zip
  unzip nextchat_linux.zip && rm -fr nextchat_linux.zip
  mkdir -p .next/standalone/app/mcp
  vi .next/standalone/app/mcp/mcp_config.json
  ```

## 创建应用

- [点击这里](https://sv66.dataonline.vn:2222/evo/user/plugins/nodejs_selector) 然后点击 **CREATE APPLICATION**

- **Node.js version** 选择 `18.20.4`
- **Application mode** 选择 `Production`
- **Application root** 填入 `domains/example.com/NextChat/.next/standalone`
- **Application startup** file 填入 `server.js`
- 点击 **ADD VARIABLE**，**Name** 填写 `ENABLE_MCP`，**Value** 填写 `true`，点击 **Done** 完成添加
- 继续添加 **PORT**，**Value**填写20000+
- 点击 **CREATE** 创建应用，接着点击 **RUN JS script**，选择 start，点击 **RUN JS SCRIPT** 启动服务

## 参考链接

- [Webhostmost 部署 node-ws 项目](https://www.kejiland.com/post/273275c7.html)