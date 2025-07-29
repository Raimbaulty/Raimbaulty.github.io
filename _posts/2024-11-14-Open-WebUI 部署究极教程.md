---
layout: mypost
title: Open-WebUI 部署究极教程
date: 2024-11-14
published: true
categories: 自部署
tags: 
 - open-webui
 - aitool
---

## 服务器配置

流畅运行：2核2G

勉强够用：1核1G

## Docker部署

**难度系数：★★★★★**

**推荐指数：★★★★★**

### 安装docker

生产环境请使用存储库安装，个人使用可选择使用脚本安装

-   **使用脚本安装**
    
    ```bash
     curl -fsSL https://get.docker.com -o get-docker.sh
     sudo sh get-docker.sh
    ```
    
-   **使用存储库安装**
    
    - **Ubuntu**
    
    ```bash
    sudo apt-get update
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```
    
    - **Debian**
    
    ```bash
    sudo apt-get update
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
     sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

其余系统请参考 [Docker Engine Install](https://docs.docker.com/engine/install)

### 配置文件

配置文件的目录结构如下，`Open-WebUI\_URL` 为您绑定到服务的域名

```
📂 open-webui/
│
├── 📜 nginx.conf
├── 📜 docker-compose.yaml
│
└── 📂 ssl/
    │
    └── 📂 Open-WebUI_URL/
        ├── 📜 certificate.crt
        ├── 📜 private.key
        └── 📜 ca_bundle.crt
```

1.  **新建 ssl 目录**

    前往 [**ZeroSSL**](https://app.zerossl.com/) 下载90天证书并将证书解压到 ssl 目录

2.  **新建 docker-compose.yaml**

    可直接修改下面的示例文件，这里推荐一个方便快捷的 [Open-WebUI 快速配置工具](https://reno.serv00.net/open-webui/)

    ```yaml
    services:
      redis:
        image: redis:latest
        container_name: redis
        restart: always
        networks:
          - backend
    
      postgres:
        image: postgres:latest
        container_name: postgres
        restart: always    
        environment:
          - POSTGRES_USER=myuser
          - POSTGRES_PASSWORD=mypassword
          - POSTGRES_DB=myname
        networks:
          - backend
        volumes:
          - pgdata:/var/lib/postgresql/data
    
      nginx:
        image: nginx:latest
        container_name: nginx
        restart: always
        ports:
          - "80:80"
          - "443:443"
        networks:
          - frontend
        volumes:
          - ./nginx.conf:/etc/nginx/nginx.conf:ro
          - ./ssl:/etc/nginx/ssl:ro
    
      open-webui:
        image: ghcr.io/open-webui/open-webui:main
        container_name: open-webui
        restart: always
        environment:
          # 网页设置
          - DEFAULT_LOCALE=zh-CN
          - WEBUI_NAME=WelcomeAboard
          - WEBUI_URL=http://open-webui:8080      
          - WEBUI_SECRET_KEY=1234567abcdefg
          - WEBUI_SESSION_COOKIE_SECURE=True
          # 安全设置
          - ENABLE_API_KEY=False
          - ENABLE_PERSISTENT_CONFIG=False
          - ENABLE_REALTIME_CHAT_SAVE=False   
          - ENABLE_ADMIN_CHAT_ACCESS=False
          - SHOW_ADMIN_DETAILS=False
          - BYPASS_MODEL_ACCESS_CONTROL=True
          # 缓存设置
          - REDIS_URL=redis://redis:6379
          # 数据库设置      
          - DATABASE_POOL_SIZE=10
          - DATABASE_POOL_MAX_OVERFLOW=20
          - DATABASE_URL=postgresql://myuser:mypassword@postgres:5432/mydb
          # 登录设置
          - WEBUI_AUTH=True
          - ENABLE_SIGNUP=False
          - ENABLE_LOGIN_FORM=True
          - DEFAULT_USER_ROLE=pending      
          # 对话设置
          - ENABLE_OPENAI_API=True
          - OPENAI_API_BASE_URL=************************************
          - OPENAI_API_KEY=*****************************************
          - DEFAULT_MODELS=*****************************************
          # 任务设置
          - TASK_MODEL_EXTERNAL=************************************
          # 文本嵌入设置
          - RAG_OPENAI_API_BASE_URL=********************************
          - RAG_OPENAI_API_KEY=*************************************            
          - RAG_EMBEDDING_MODEL=************************************
          - RAG_EMBEDDING_ENGINE=openai
          - RAG_FILE_MAX_SIZE=20
          - RAG_EMBEDDING_OPENAI_BATCH_SIZE=2
          - CHUNK_SIZE=8000
          - PDF_EXTRACT_IMAGES=False
          # 语音设置
          - AUDIO_TTS_OPENAI_API_BASE_URL=**************************
          - AUDIO_TTS_OPENAI_API_KEY=*******************************
          - AUDIO_TTS_ENGINE=openai
          - AUDIO_TTS_MODEL=tts-1
          - AUDIO_TTS_VOICE=alloy
          - AUDIO_STT_OPENAI_API_BASE_URL=**************************
          - AUDIO_STT_OPENAI_API_KEY=*******************************
          - AUDIO_STT_ENGINE=openai
          - AUDIO_STT_MODEL=whisper-1
          # 绘图设置
          - ENABLE_IMAGE_GENERATION=True
          - IMAGES_OPENAI_API_BASE_URL=*****************************
          - IMAGES_OPENAI_API_KEY=**********************************
          - IMAGE_GENERATION_ENGINE=openai
          - IMAGE_GENERATION_MODEL=*********************************
          - IMAGE_SIZE=1024x1024
          # 搜索设置
          - ENABLE_WEB_SEARCH=True
          - WEB_SEARCH_ENGINE=tavily
          - TAVILY_API_KEY=***********************************
          # OAuth2设置      
          - ENABLE_OAUTH_SIGNUP=False
          - GITHUB_CLIENT_ID=***************************************
          - GITHUB_CLIENT_SECRET=***********************************
          - GOOGLE_CLIENT_ID=***************************************
          - GOOGLE_CLIENT_SECRET=***********************************
          - MICROSOFT_CLIENT_ID=************************************
          - MICROSOFT_CLIENT_SECRET=********************************
          - MICROSOFT_CLIENT_TENANT_ID=*****************************
          # OIDC设置
          - OAUTH_PROVIDER_NAME=LINUX DO
          - OPENID_PROVIDER_URL=https://raw.githubusercontent.com/Raimbaulty/open-webui/main/openid-configuration
          - OAUTH_CLIENT_ID=****************************************
          - OAUTH_CLIENT_SECRET=************************************      
          # Ollama设置
          - ENABLE_OLLAMA_API=False
          - OLLAMA_BASE_URL=****************************************
          # 其他设置
          - ENABLE_MESSAGE_RATING=False
          - ENABLE_TAGS_GENERATION=False
          - ENABLE_COMMUNITY_SHARING=False
          - ENABLE_FOLLOW_UP_GENERATION=False
          - ENABLE_EVALUATION_ARENA_MODELS=False
          - ENABLE_AUTOCOMPLETE_GENERATION=False
        depends_on:
          - redis    	
          - postgres
        networks:
          - frontend
          - backend
        volumes:
          - ./data:/app/backend/data
          
    networks:
      frontend:
        name: frontend
      backend:
        name: backend
    
    volumes:
      pgdata:
        name: open-webui
    ```

3.  **新建** **nginx.conf**

    ```nginx
    user  nginx;
    worker_processes  auto;
    
    error_log  /var/log/nginx/error.log notice;
    pid        /var/run/nginx.pid;
    
    events {
        worker_connections  1024;
        use epoll;
    }
    
    http {
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;
    
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';
    
        access_log  /var/log/nginx/access.log  main;
    
        sendfile        on;
        keepalive_timeout  65;
    
        # gzip settings
        gzip  on;
        gzip_disable "MSIE [1-6]\.";
        gzip_proxied any;
        gzip_vary on;
        gzip_comp_level 3;
        gzip_min_length 1k;
        gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss;
    
        tcp_nopush     on;
        tcp_nodelay    on;
    
        client_max_body_size 100m;
    
        proxy_http_version 1.1;
    
        # SSL settings
        ssl_prefer_server_ciphers on;
        ssl_session_timeout 1d;
        ssl_session_cache shared:SSL:10m;
        ssl_session_tickets off;
    
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    
        ssl_certificate /etc/nginx/ssl/Open-WebUI_URL/certificate.crt;
        ssl_certificate_key /etc/nginx/ssl/Open-WebUI_URL/private.key;
        ssl_trusted_certificate /etc/nginx/ssl/Open-WebUI_URL/ca_bundle.crt;
    
        resolver 1.1.1.1 1.0.0.1 valid=300s;
        resolver_timeout 5s;
    
        # Server block
        server {
            listen 443 ssl;
            http2 on;
            server_name demo.com;
    
            # Hide headers
            proxy_hide_header Server;
            proxy_hide_header Date;
    
            client_max_body_size 20m;
            client_body_buffer_size 128k;
    
            # Timeouts
            keepalive_timeout 60s;
            proxy_connect_timeout 60s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
    
            # Response headers
            add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
            # Location for root
            location / {
                proxy_pass http://open-webui:8080;
    
                proxy_buffering on;
                proxy_buffer_size 8k;
                proxy_buffers 32 8k;
                proxy_busy_buffers_size 64k;
    
                gzip_comp_level 5;
                gzip on;
    
                # Request headers
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
            }
    
            # Location for WebSocket
            location /ws/socket.io {
                proxy_pass http://open-webui:8080;
    
                proxy_buffering off;
                gzip off;
    
                # Request headers
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
    
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_cache_bypass $http_upgrade;
    
                proxy_read_timeout 1800;
            }
        }
    }
    ```

### 启动open-webui

1.  **运行服务**
    
    ```bash
    docker compose up -d
    ```
    
2.  **查看日志**
    
    ```bash
    docker compose logs -f
    ```
    
3.  **停止服务**
    
    ```bash
    docker compose down
    ```
    

## Python部署

**难度系数：★**

**推荐指数：★★★**

### 配置conda

1.  **下载miniconda安装脚本**
    
    下载链接如果失效，请自行前往官网下载：[点击访问官网](https://docs.anaconda.com/miniconda/#miniconda-latest-installer-links)
    
    ```bash
    curl -fsSL  https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
    ```
    
2.  **安装miniconda**
    
    ```bash
    sudo sh Miniconda3-latest-Linux-x86_64.sh
    ```
    
3.  **配置python环境**
    
    截止本篇博客发布，python 版本要求 3.11 以上
    
    ```bash
    conda create -n open-webui python=3.11
    conda activate open-webui
    ```
    

### 配置open-webui

1.  **安装open-webui**
    
    ```bash
    pip install open-webui
    ```
    
2.  **测试运行**
    
    ```bash
    open-webui serve
    ```
    
    浏览器访问 http://ip:8080 开始注册使用
    

### 配置系统服务

1.  **创建open-webui.service**
    
    ```bash
    sudo bash -c 'cat << EOF > /etc/systemd/system/open-webui.service
    [Unit]
    Description=Open WebUI Service
    Documentation=https://docs.openwebui.com
    After=network.target
    Wants=network-online.target
    
    [Service]
    Type=simple
    User=root
    ExecStart=/openweb/miniconda3/envs/open-webui/bin/python /openweb/miniconda3/envs/open-webui/bin/open-webui serve
    ExecStop=/bin/kill -HUP \$MAINPID
    Restart=always
    StandardOutput=journal
    StandardError=inherit
    
    [Install]
    WantedBy=multi-user.target
    EOF'
    ```
    
2.  **运行服务**
    
    ```bash
    systemctl daemon-reload
    systemctl enable --now open-webui.service
    ```
    
3.  **查看日志**
    
    ```bash
    journalctl -n 20 -u open-webui
    ```
    

## 参考链接

-   [**Installing Miniconda**](https://docs.anaconda.com/miniconda/miniconda-install/)
    
-   [**Docker Engine Install**](https://docs.docker.com/engine/install)
    
-   [**How to Install Open WebUI**](https://docs.openwebui.com/getting-started/quick-start/)
    
-   [**openwebui标题生成提示词**](https://linux.do/t/topic/264260)
    
-   [**轻量级思维链Prompt实现**](https://linux.do/t/topic/262730)
    
-   [**有关OpenWebUI响应速度的优化-nginx配置**](https://linux.do/t/topic/214304)
    
-   [**openwebui 0.5.x nginx配置**](https://linux.do/t/topic/317621)