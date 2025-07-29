---
layout: mypost
title: TypingMind 部署教程
date: 2025-05-27
published: true
categories: 自部署
tags: 
 - typingmind
 - linux
---

## TypingMind

**官方仓库：** https://github.com/TypingMind/typingmind

**官方文档：** https://docs.typingmind.com

需要注意，这是一款付费软件，可以付费支持，也可充电获取 Premium免激活版：[我的电铺](https://afdian.com/a/ireno)

### Vercel 部署

1. [点击这里](https://github.com/TypingMind/typingmind/fork) Fork 官方仓库
2. [点击这里](https://vercel.com/signup) 注册 Vercel 账户
3. [点击这里](https://vercel.com/new) 然后点击 **Continue with GitHub** 绑定 GitHub账户
4. 点击 `typingmind` 后的 **Import**，展开 **Building and Output Settings**，修改 **Output Directory** 为 `src` 后 **Deploy**

### Huggingface 部署

1. [点击这里](https://huggingface.co/join) 注册 Huggingface 账户
2. [点击这里](https://github.com/TypingMind/typingmind/archive/refs/heads/main.zip) 下载源码并解压进入 `src` 目录
3. [点击这里](https://huggingface.co/new-space?sdk=static) 创建空间，**space name** 填写 `TypingMind` 后下滑页面点击 **Create Space**
4. 点击 **Files** → **+ Contribute** → **Upload files**，`Ctrl +A` 全选 `src` 目录下的所有内容后拖动到 **Drag files/folders here or click to browse from your computer.** 注意这里必须拖动上传，点击上传无法上传目录下的文件夹
5. 点击 **Commit changes to `main`** 并保留页面稍待，因为文件较多，如果网络不好可能需要10~20分钟，耐心等待即可，部署成功后页面会自动跳转到 **Files**

## MCP

### 安装步骤

- [点击这里](https://huggingface.co/new-space?sdk=docker) 创建空间，**space name** 填写 `TM-MCP` 后下滑页面点击 **Create Space**
- 下滑找到并点击 **create the Dockerfile file**，粘贴以下内容后点击 **Commit changes to `main`** 

```dockerfile
FROM python:3.11-slim

RUN useradd -ms /bin/bash mcp

WORKDIR /app

RUN apt-get update && \
    apt-get install -y curl unzip && \
    curl -fsSL https://deb.nodesource.com/setup_18.x | bash - && \
    apt-get install -y nodejs && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

USER mcp
RUN curl -fsSL https://bun.sh/install | bash

ENV BUN_INSTALL="/home/mcp/.bun"
ENV PATH="${BUN_INSTALL}/bin:${PATH}"

USER root
RUN pip install --no-cache-dir mcpo uv

RUN mkdir -p /app/.cache/uv && chown -R mcp:mcp /app

ENV XDG_CACHE_HOME="/app/.cache"

USER mcp

ENV PORT=7860

CMD npx @typingmind/mcp $TOKEN
```

- 点击 **Settings** → **New secret** 添加一个环境变量 **TOKEN**，值设为＞10位的字符串

### 使用方法

- 点击 **设置** - **Model Context Protocol** - **Setup MCP Connector**

- 点击 **Remote Server** → **Next**，URL 输入 `https://用户名-tm-mcp.hf.space`，Token 与空间环境变量一致，点击 **Connect**，连接成功后，点击 **Get started** → **完成**

- 点击 **Edit Server**，粘贴 MCP 配置，示例如下，完成后点击 **保存更改**

      {
        "mcpServers": {
          "fetch": {
            "command": "uvx",
            "args": [
              "mcp-server-fetch"
            ]
          },
          "mcp-server-time": {
            "command": "uvx",
            "args": [
              "mcp-server-time",
              "--local-timezone=Asia/Shanghai"
            ],
            "alwaysAllow": [
              "get_current_time",
              "convert_time"
            ]
          },
          "filesystem": {
            "command": "npx",
            "args": [
              "-y",
              "@modelcontextprotocol/server-filesystem",
              "/home/mcp"
            ]
          },
          "puppeteer": {
            "command": "npx",
            "args": [
              "-y",
              "@modelcontextprotocol/server-puppeteer"
            ]
          },
          "edgeone-pages-mcp-server": {
            "command": "npx",
            "args": ["edgeone-pages-mcp"]
          }
        }
      }
  
  

## Plugin Server

### 安装步骤

- [点击这里](https://huggingface.co/new-space?sdk=docker) 创建空间，**space name** 填写 `TM-PS` 后下滑点击 **Create Space**
- 下滑找到并点击 **create the Dockerfile file**，粘贴以下内容后点击 **Commit changes to `main`** 

    ```dockerfile
    FROM node:current-slim
    
    WORKDIR /usr/src/app
    
    RUN apt-get update && apt-get install -y git && rm -rf /var/lib/apt/lists/*
    
    RUN git clone https://github.com/TypingMind/plugins-server/ .
    
    RUN npm ci
    
    RUN npm run build
    
    RUN chown node:node /usr
    
    EXPOSE 7860
    
    CMD PORT=7860 npm run start
    ```

​	注意添加环境变量 **RENDER_EXTERNAL_URL** 值为 `https://用户名-tm-ps.hf.space`

### 使用方法

- 点击 **插件**，点击插件旁的 ➕ 安装插件，比如 **Word Generator**、**Web Page Reader**

- 点击左侧安装好的插件，然后进入 **Settintgs** 标签，**Plugin Server** 填写 `https://用户名-tm-ps.hf.space`，下滑点击 **保存**

- 点击 **聊天**，输入框上方会多出一个工具选项，可以管理插件的启用状态

## Custon

### 添加插件

进入 `https://xxx.typingcloud.com/admin/plugins` 点击 **Browse Plugins** 添加需要的插件

点击 **Shared OAuth Connections** → **Add New Connection** 添加 Oauth2 认证

