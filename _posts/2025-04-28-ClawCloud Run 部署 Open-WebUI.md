---
layout: mypost
title: ClawCloud Run 部署 Open-WebUI
date: 2025-04-28
published: true
categories: 自部署
tags: 
 - claw
 - open-webui
---

## Cloudflared
* Name: cloudflared
* Image: cloudflare/cloudflared:latest
* Usage: 0.1/64M, Cost: 0.01刀/天
* Port: 80
* Command: cloudflared tunnel --no-autoupdate run --token eyJ**********

## New-API

* Name: new-api
* Image: calciumion/new-api:latest
* Usage: 0.1/256M, Cost: 0.03刀/天
* Port: 3000
* Environment：
    	SQL_DSN=root:123456@tcp(localhost:3306)/oneapi

## Open-WebUI
* Name: open-webui
* Image: ghcr.io/open-webui/open-webui:main
* Usage: 0.2/1G, Cost: 0.12刀/天
* Port: 8080
* Environment:
    DEFAULT_LOCALE=zh
    WEBUI_URL=http://example.com
    WEBUI_SECRET_KEY=G5kW8Hq9rVzRY2faBsmP3TDXowxCjuLA
    WEBUI_SESSION_COOKIE_SECURE=True
    SHOW_ADMIN_DETAILS=False
    ENABLE_SIGNUP=True
    ENABLE_PERSISTENT_CONFIG=False
    BYPASS_MODEL_ACCESS_CONTROL=True
    OPENAI_API_BASE_URL=http://new-api.************.svc.cluster.local:3000/v1
    OPENAI_API_KEY=sk-************************************************
    DEFAULT_MODELS=gemini-2.0-flash
    TASK_MODEL_EXTERNAL=gemini-2.0-flash
    AUDIO_TTS_OPENAI_API_BASE_URL=https://api.openai.com/v1
    AUDIO_TTS_OPENAI_API_KEY=sk-************************************************
    AUDIO_TTS_ENGINE=openai
    AUDIO_STT_OPENAI_API_BASE_URL=https://api.openai.com/v1
    AUDIO_STT_OPENAI_API_KEY=sk-************************************************
    AUDIO_STT_ENGINE=openai
    ENABLE_IMAGE_GENERATION=True
    IMAGES_OPENAI_API_BASE_URL=https://api.openai.com/v1
    IMAGES_OPENAI_API_KEY=sk-************************************************
    IMAGE_GENERATION_MODEL=dall-e-3
    IMAGE_SIZE=1024x1024
    ENABLE_WEB_SEARCH=True
    WEB_SEARCH_ENGINE=exa
    EXA_API_KEY=************************************************
    ENABLE_OAUTH_SIGNUP=True
    GITHUB_CLIENT_ID=************************************************
    GITHUB_CLIENT_SECRET=************************************************
    MICROSOFT_CLIENT_ID=************************************************
    MICROSOFT_CLIENT_SECRET=************************************************
    MICROSOFT_CLIENT_TENANT_ID=************************************************
    ENABLE_OLLAMA_API=False
    ENABLE_MESSAGE_RATING=False
    ENABLE_TAGS_GENERATION=False
    ENABLE_COMMUNITY_SHARING=False
    ENABLE_EVALUATION_ARENA_MODELS=False
    ENABLE_AUTOCOMPLETE_GENERATION=False
* Storage: /app/backend/data, 5G