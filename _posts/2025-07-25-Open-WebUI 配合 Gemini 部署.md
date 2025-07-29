---
layout: mypost
title: Open-WebUI 配合 Gemini 部署
date: 2025-07-25
published: true
categories: 自部署
tags: 
 - open-webui
 - linux
---

## 目录

```markdown
open-webui/
├── data/
├── api.yaml
└── docker-compose.yaml
```



### api.yaml

```yaml
api_keys: 
  - api: sk-***********************************
providers:
  - provider: gemini
    base_url: https://generativelanguage.googleapis.com/v1beta
    api:
      - AIzaS**********************************
      - AIzaS**********************************
      - AIzaS**********************************
    model:
      - gemini-2.5-pro
      - gemini-2.5-flash
      - gemini-2.5-flash-lite
      - gemini-2.5-pro: gemini-2.5-pro-search
      - gemini-2.5-flash: gemini-2.5-flash-search
      - gemini-2.5-flash-lite: gemini-2.5-flash-lite-search
      - gemini-2.5-pro: gemini-2.5-pro-think-0
      - gemini-2.5-flash: gemini-2.5-flash-think-0
      - gemini-2.5-flash-lite: gemini-2.5-flash-lite-think-0
      - gemini-2.5-pro: gemini-2.5-pro-think-0-search
      - gemini-2.5-flash: gemini-2.5-flash-think-0-search
      - gemini-2.5-flash-lite: gemini-2.5-flash-lite-think-0-search
    tools: true
    preferences:
      api_key_rate_limit:
        gemini-2.5-flash: 10/min,250/day
        gemini-2.5-pro: 5/min,100/day
        default: 10/min
      api_key_cooldown_period: 60
      api_key_schedule_algorithm: round_robin
      model_timeout:
        gemini-2.5-pro: 500
        gemini-2.5-pro-think-0: 500
        gemini-2.5-pro-think-0-search: 500
        default: 10
      keepalive_interval:
        gemini-2.5-pro: 50
        gemini-2.5-pro-think-0: 50
        gemini-2.5-pro-think-0-search: 50
      proxy: socks5://****:****@****:****
      post_body_parameter_overrides:
        gemini-2.5-pro-search:
          tools:
            - google_search: {}
            - url_context: {}
        gemini-2.5-flash-search:
          tools:
            - google_search: {}
            - url_context: {}
        gemini-2.5-flash-lite-search:
          tools:
            - google_search: {}
            - url_context: {}
        gemini-2.5-pro-think-0-search:
          tools:
            - google_search: {}
            - url_context: {}
        gemini-2.5-flash-think-0-search:
          tools:
            - google_search: {}
            - url_context: {}
        gemini-2.5-flash-lite-think-0-search:
          tools:
            - google_search: {}
            - url_context: {}
```



### docker-compose.yaml

```yaml
services:
  redis:
    image: redis:latest
    container_name: redis
    restart: always
    networks:
      - open-webui

  postgres:
    image: postgres:latest
    container_name: postgres
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=postgres
    networks:
      - open-webui
    volumes:
      - pgdata:/var/lib/postgresql/data

  uni-api:
    image: yym68686/uni-api:latest
    container_name: uni-api
    restart: always
    environment:
      - DISABLE_DATABASE=True
    networks:
      - open-webui
    volumes:
      - ./api.yaml:/home/api.yaml

  open-webui:
    image: yunxinc/study:custom-openwebui-0.6.18
    user: 1000:1000    
    container_name: open-webui
    restart: always
    ports:
      - "8080:8080"
    environment:
      - DEFAULT_LOCALE=zh-CN
      - WEBUI_URL=https://example.com
      - WEBUI_SECRET_KEY=*******************
      - WEBUI_SESSION_COOKIE_SECURE=True
      - REDIS_URL=redis://redis:6379
      - DATABASE_POOL_SIZE=10
      - DATABASE_POOL_MAX_OVERFLOW=20
      - DATABASE_URL=postgresql://postgres:postgres@postgres:5432/postgres
      - ENABLE_API_KEY=False
      - ENABLE_PERSISTENT_CONFIG=True
      - ENABLE_REALTIME_CHAT_SAVE=False
      - ENABLE_ADMIN_CHAT_ACCESS=False
      - SHOW_ADMIN_DETAILS=False
      - BYPASS_MODEL_ACCESS_CONTROL=True
      - ENABLE_SIGNUP=True
      - ENABLE_LOGIN_FORM=True
      - DEFAULT_USER_ROLE=pending
      - OPENAI_API_BASE_URL=http://uni-api:8000/v1
      - OPENAI_API_KEY=sk-*******************
      - DEFAULT_MODELS=gemini-2.5-flash
      - TASK_MODEL_EXTERNAL=gemini-2.5-flash-lite
      - AUDIO_TTS_OPENAI_API_BASE_URL=https://api.siliconflow.cn/v1
      - AUDIO_TTS_OPENAI_API_KEY=sk-*********
      - AUDIO_TTS_ENGINE=openai
      - AUDIO_TTS_MODEL=FunAudioLLM/CosyVoice2-0.5B
      - AUDIO_TTS_VOICE=FunAudioLLM/CosyVoice2-0.5B:claire
      - AUDIO_STT_OPENAI_API_BASE_URL=https://api.groq.com/openai/v1
      - AUDIO_STT_OPENAI_API_KEY=gsk_********
      - AUDIO_STT_ENGINE=openai
      - AUDIO_STT_MODEL=whisper-large-v3
      - ENABLE_IMAGE_GENERATION=True
      - IMAGES_OPENAI_API_BASE_URL=https://silicon.of.cloudns.be/v1
      - IMAGES_OPENAI_API_KEY=sk-************
      - IMAGE_GENERATION_MODEL=black-forest-labs/FLUX.1-schnell
      - IMAGE_SIZE=1024x1024 
      - ENABLE_OAUTH_SIGNUP=True
      - OAUTH_PROVIDER_NAME=LINUX DO
      - OPENID_PROVIDER_URL=https://raw.githubusercontent.com/Raimbaulty/open-webui/main/openid-configuration
      - OAUTH_CLIENT_ID=********************
      - OAUTH_CLIENT_SECRET=****************
      - ENABLE_OLLAMA_API=False
      - ENABLE_MESSAGE_RATING=False
      - ENABLE_TAGS_GENERATION=False
      - ENABLE_COMMUNITY_SHARING=False
      - ENABLE_FOLLOW_UP_GENERATION=False
      - ENABLE_EVALUATION_ARENA_MODELS=False
      - ENABLE_AUTOCOMPLETE_GENERATION=False
    depends_on:
      - redis
      - postgres
      - uni-api
    networks:
      - open-webui
    volumes:
      - ./data:/app/backend/data
      
networks:
  open-webui:
    name: open-webui

volumes:
  pgdata:
    name: open-webui
```



## 配置

### 反向代理

使用caddy快速实现

1. **安装caddy**

   ```bash
   sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
   curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
   curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
   chmod o+r /usr/share/keyrings/caddy-stable-archive-keyring.gpg
   chmod o+r /etc/apt/sources.list.d/caddy-stable.list
   sudo apt update
   sudo apt install caddy
   ```

   

2. **修改配置**

   ```bash
   cp /etc/caddy/Caddyfile /etc/caddy/Caddyfile.bak
   cat > /etc/caddy/Caddyfile <<EOF
   example.com {
       reverse_proxy http://localhost:8080
   }
   EOF
   ```

   

3. **重载配置**

   打开防火墙的80和443端口后

   ```bash
   sudo systemctl reload caddy
   ```

   

服务启动需要一会儿，等待30s左右再访问站点地址，初次使用需要创建管理员账户然后使用管理员账户登录



### 模型

管理员登录网站后，依次点击左下角头像 → 管理员面板 → 设置 → 模型，保留以下模型

- gemini-2.5-pro
- gemini-2.5-flash
- gemini-2.5-flash-lite

隐藏其他带后缀模型，点击 ··· → 隐藏模型

点击 外部连接，打开 缓存基础模型列表



### 函数

依次点击左下角头像 → 管理员面板 → 函数 → + → 从链接导入，注意修改函数ID

- Gemini 原生搜索

  `https://raw.githubusercontent.com/Raimbaulty/open-webui/main/functions/search`

- Gemini 关闭思考

  `https://raw.githubusercontent.com/Raimbaulty/open-webui/main/functions/think`
  
  

### 公告

依次点击左下角头像 → 管理员面板 → 设置 → 界面，点击 公告横幅 对应的 + 添加公告



### 图标

修改 docker-compose.yaml 为 open-webui 添加以下环境变量

```bash
STATIC_DIR=./data/static
```

然后重启服务

```bash
docker compose up -d
```

根据需要进入 `./data/static` 目录替换图标文件

- 常见 LLM Logo：https://icons.lobehub.com/

- 生成 favicon：https://realfavicongenerator.net/

- 生成 splash：https://onlinepngtools.com/convert-text-to-png