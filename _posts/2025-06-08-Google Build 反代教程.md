---
layout: mypost
title: Google Build 反代教程
date: 2025-06-08
published: true
categories: 逆向
tags: 
 - gemini
 - linux
---

# Google Build 反代教程

反代原作是类脑的 InterestingDarkness

反代后免费层级的API KEY就可以使用付费级的模型

部署这个反代需要两步

1. 登录 Google 账户 Build，使用浏览器代理，这里简称为前端
2. 本地安装 NodeJS 环境，搭建反代服务，这里简称为后端

## 前端

[点击这里](https://raimbaulty.serv00.net/?f23fcef61d44bd9d#B7By5kdzNK97cKVSFNs51GpwZDpASisNmLhNMr4yY3Ft) 获取前端代码，密码 `linux.do`

登录 Google 后访问控制台：[点击访问](https://aistudio.google.com/app/apps/bundled/blank?showPreview=true&showCode=true)

将复制到的前端代码粘贴到左侧输入框内后右上角保存，注意使用期间需要保持页面不能关闭

后续再使用，可以直接 [点击这里](https://aistudio.google.com/app/apps?source=user) 进入已保存的应用

## 后端

[点击这里](https://raimbaulty.serv00.net/?c092e5643f3e314a#5Xo8hZrWYQMdXwouMjK5dYZ1pVMx5f4FD3txgznDTqui) 获取后端代码，密码 `linux.do`

在本地创建项目目录 `dark-server`，将后端代码保存为 `dark-server.js`

安装依赖并启动服务，然后保持窗口运行

```bash
cd dark-server
npm install express ws
node dark-server.js
```

回到前端查看右侧输出框是否连接成功


## 酒馆
经实践，本地部署稳定性更好，推荐本地部署，不过需要注意的是官方仓库的版本没有更新Google最新的0605模型,如果需要使用需要自己在 index.html 添加，也可以直接使用我修改过的替换 index.html

https://note.ff2a.com/sillytavern

```bash
git clone https://github.com/SillyTavern/SillyTavern
cd SillyTavern
npm install
node server.js
```
稍待片刻会自动打开页面，配置API这里 **API** 选择聊天补全，**聊天补全来源** 选择Google AI Studio，展开**反向代理**，代理名称填写 **darkness**，**代理服务器 URL** 填写 `http://127.0.0.1:8889` 后点击保存，**Google AI Studio API 密钥** 填写Google Studio 的 KEY 即可，模型选择 `gemini-2.5-pro-preview-0605`

## 自启

快捷键 Win+R 输入 `shell:startup` 后回车，在打开的目录右键新建一个txt后重命名为 `sillytavern.bat` 并粘贴以下代码

```bash
@echo off
echo 正在启动应用程序...

REM 设置Node环境变量
set NODE_PATH=path\to\node
set PATH=%NODE_PATH%;%PATH%

REM 启动酒馆
start "SillyTavern" cmd /c "cd /d path\to\SillyTavern && node server.js"

REM 启动后端
start "KingFall" cmd /c "cd /d path\to\dark-server && node dark-server.js"

REM 打开Edge浏览器访问 Google Build
start msedge "https://aistudio.google.com/app/apps?source=user"

echo 所有应用程序已启动完成
```

## 参考链接
- [谷歌新王！kingfall-ab-test本地环境搭建手册和相关优化（酒馆适用）](https://linux.do/t/topic/704645)