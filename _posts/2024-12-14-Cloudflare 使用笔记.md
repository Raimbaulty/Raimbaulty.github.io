---
layout: mypost
title: Cloudflare 使用笔记
date: 2024-12-24
published: true
categories: 笔记
tags: 
 - cloudflare
---

## AI

### API

#### 模型来源

```bash
curl --request GET \
  --url https://api.cloudflare.com/client/v4/accounts/<ACCOUNT_ID>/ai/authors/search \
  --header 'Authorization: Bearer <AI_TOKEN>' \
  --header 'Content-Type: application/json'
```

#### 模型列表

```bash
curl --request GET \
  --url https://api.cloudflare.com/client/v4/accounts/<ACCOUNT_ID>/ai/models/search \
  --header 'Authorization: Bearer <AI_TOKEN>' \
  --header 'Content-Type: application/json'
```

#### 请求格式

```bash
curl --request GET \
  --url https://api.cloudflare.com/client/v4/accounts/<ACCOUNT_ID>/ai/models/schema?model=<MODEL_NAME> \
  --header 'Authorization: Bearer <AI_TOKEN>' \
  --header 'Content-Type: application/json'
```

#### 请求示例

-   对话补全
    
    ```bash
    curl --request POST \
      --url https://api.cloudflare.com/client/v4/accounts/<ACCOUNT_ID>/ai/run/@cf/google/gemma-7b-it-lora \
      --header 'Authorization: Bearer <AI_TOKEN>' \
      --header 'Content-Type: application/json' \
      --data '{
      "text": "Hello world!"
    }'
    ```
    
-   声音识别
    
    ```bash
    curl --request POST \
      --url https://api.cloudflare.com/client/v4/accounts/<ACCOUNT_ID>/ai/run/@cf/openai/whisper \
      --header "Authorization: Bearer <AI_TOKEN>" \
      --header "Content-Type: application/octet-stream" \
      --data-binary "@C:\Users\Administrator\audio.mp3"
    ```
    
-   翻译文本
    
    ```bash
    curl --request POST \
      --url https://api.cloudflare.com/client/v4/accounts/<ACCOUNT_ID>/ai/run/@cf/meta/m2m100-1.2b\
      --header 'Authorization: Bearer <AI_TOKEN>' \
      --header 'Content-Type: application/json' \
      --data '{
      "source_lang": "en",
      "target_lang": "tr",
      "text": "I love you my baby."
    }'
    ```
    

#### 任务列表

```bash
curl --request GET \
  --url https://api.cloudflare.com/client/v4/accounts/<ACCOUNT_ID>/ai/tasks/search \
  --header 'Authorization: Bearer <AI_TOKEN>' \
  --header 'Content-Type: application/json'
```

## Worker

### Playground

直接在浏览器中测试预览，[点击立即创建](https://workers.cloudflare.com/playground)

### CLI （wrangler）

#### 安装 wrangler

安装之前请确保已安装 Node.js 和 npm

```bash
npm install wrangler --save-dev
```

#### 查看版本

```bash
npx wrangler -v
```

#### 更新

```bash
npm install wrangler@latest
```

#### 登录账户

-   临时登录
    
    ```bash
    export CLOUDFLARE_API_TOKEN=**********
    wrangler whoami
    ```
    
-   永久登录
    
    ```bash
    wrangler login
    ```
    

#### 创建项目

-   目录结构
    
        📂 hello-world/
        │
        ├── 📂 src/
        │   └── 📜 index.js
        ├── 📜 wrangler.toml
        └── 📜 README.md

-   wrangler.toml
    
    ```toml
    name = "hello-world"
    main = "src/index.js"
    compatibility_date = "2024-12-24"
    compatibility_flags = ["nodejs_compat"]
    
    [vars]
    API_BASEURL='aaaa'
    API_KEY = 'aaaa'
    
    kv_namespaces = [
      { binding = "<MY_NAMESPACE>", id = "<KV_ID>" }
    ]
    ```
    

#### 项目预览

```bash
wrangler dev
```

#### 项目发布

```bash
wrangler deploy
```

## Tunnel

### 安装cloudflared

直接从 Github 仓库 下载最新发行版本

```bash
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 -o /usr/bin/cloudflared
sudo chmod +x /usr/bin/cloudflared
```

或参考 [**Cloudflare packages**](https://pkg.cloudflare.com/index.html) 进行安装，下面是 **Ubuntu 22.04** 的安装指令

```bash
sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null

echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared jammy main' | sudo tee /etc/apt/sources.list.d/cloudflared.list

sudo apt-get update && sudo apt-get install cloudflared
```

### 创建隧道

-   访问 [**Tunnel 页**](https://one.dash.cloudflare.com/?to=/:account/networks/tunnels) ,点击 **\[Create a tunnel\]-\[Connect Cloudflared\]**，输入 **Tunnel Name**（隧道名称），点击 \[**save tunnel\]** 保存
    
-   复制 `Run the following command:` 下的代码块粘贴保存到记事本，接着下滑点击 **Next**
    
-   选择 **Domain**（域名），**Subdomain**（子域名）可以留空，**Type** 选择 `HTTP`，**URL** 按照开放端口填写 ，然后点击 Save the tunnel
    
    **注意**：同一隧道可接入多个端口，\[···\] - \[配置\] - \[公共主机名\] - \[添加公共主机名\] 然后添加即可
   
### 创建系统服务

```bash
sudo vi /etc/systemd/system/cloudflared-tunnel.service
```

写入以下内容，注意修改路径并替换 **token** 为创建隧道获得代码块中 `eyJ` 开头的部分

```ini
[Unit]
Description=cloudflared-tunnel
After=network.target

[Service]
TimeoutStartSec=0
Type=notify
ExecStart=/usr/bin/cloudflared --no-autoupdate --edge-ip-version auto tunnel run --token eyJ**************************************************************
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

初次运行前需先登录

```bash
cloudflared tunnel login
```

启动服务

```bash
sudo systemctl daemon-reload
sudo systemctl start cloudflared-tunnel.service
sudo systemctl enable cloudflared-tunnel.service
sudo systemctl status cloudflared-tunnel.service
```

### 删除系统服务

```bash
sudo systemctl stop cloudflared-tunnel.service
sudo systemctl disable cloudflared-tunnel.service
sudo rm /etc/systemd/system/cloudflared-tunnel.service
sudo systemctl daemon-reload
sudo rm /usr/bin/cloudflared
sudo rm -rf /etc/cloudflared/
```

## SSL

进入需要申请证书的域，左侧菜单栏点击 \[SSL/TLS\] - \[源服务器\]，然后点击 \[创建证书\]，默认证书有效期是15年，支持的有效期：

-   7天
    
-   30天
    
-   90天
    
-   1年
    
-   2年
    
-   3年
    
-   15年
    

选择合适的有效期后点击 \[创建\]，上下两个输出框内的内容分别为`certificate.crt`(证书)、`private.key` (密钥)，nginx 配置示例如下

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate     /etc/nginx/ssl/example.com/certificate.crt;
    ssl_certificate_key /etc/nginx/ssl/example.com/private.key;

    # 可选的其他 SSL 配置...
}
```

**注意**：离开证书创建页面后将无法再次查看私钥，请务必妥善保存

Cloudflare SSL默认配置为灵活，为杜绝中间人攻击，可在左侧菜单栏点击 \[SSL/TLS\] - \[概述\]，然后点击 \[配置\]，勾选 **完全(严格)**

## 参考链接

-   [**Welcome to Cloudflare**](https://developers.cloudflare.com/)