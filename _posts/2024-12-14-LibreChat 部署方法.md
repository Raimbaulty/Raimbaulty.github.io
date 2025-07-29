---
layout: mypost
title: LibreChat 部署方法
date: 2024-12-14
published: true
categories: 自部署
tags: 
 - librechat
---

## 部署服务

### 服务器部署

-   拉取仓库
    
    ```bash
    git clone https://github.com/danny-avila/LibreChat.git
    cd LibreChat
    ```
    
-   配置 `.env`
    
    **功能**：添加各种环境变量，涉及登录认证、通道配置、用户管理、Agents等
    
    ```bash
    cp .env.example .env
    ```
    
    添加以下行
    
    ```ini
    UID=0
    GID=0
    ```
    
    根据需要修改以下行
    
    ```ini
    PROXY=https://proxy.xxx.com
    
    ANTHROPIC_API_KEY=***********************
    ANTHROPIC_MODELS=claude-3-5-haiku-20241022,claude-3-5-sonnet-20241022,claude-3-5-sonnet-latest,claude-3-5-sonnet-20240620,claude-3-opus-20240229,claude-3-sonnet-20240229,claude-3-haiku-20240307,claude-2.1,claude-2,claude-1.2,claude-1,claude-1-100k,claude-instant-1,claude-instant-1-100k
    
    GOOGLE_KEY=***************************
    GOOGLE_MODELS=gemini-2.0-flash-exp,gemini-2.0-flash-thinking-exp-1219,gemini-exp-1121,gemini-exp-1114,gemini-1.5-flash-latest,gemini-1.0-pro,gemini-1.0-pro-001,gemini-1.0-pro-latest,gemini-1.0-pro-vision-latest,gemini-1.5-pro-latest,gemini-pro,gemini-pro-vision
      
    OPENAI_API_KEY=***********************
    OPENAI_MODELS=gpt-4o,chatgpt-4o-latest,gpt-4o-mini,gpt-3.5-turbo-0125,gpt-3.5-turbo-0301,gpt-3.5-turbo,gpt-4,gpt-4-0613,gpt-4-vision-preview,gpt-3.5-turbo-0613,gpt-3.5-turbo-16k-0613,gpt-4-0125-preview,gpt-4-turbo-preview,gpt-4-1106-preview,gpt-3.5-turbo-1106,gpt-3.5-turbo-instruct,gpt-3.5-turbo-instruct-0914,gpt-3.5-turbo-16k
    ```
    运行后，打开 http://ip:3080 注册后登录使用，注册完毕及时修改 .env `ALLOW_REGISTRATION=false`
    
-   修改 `librechat.yaml`
    
    **功能**：添加 OpenAI API 格式标准的通道（如 Mistral、Cohere 等）以及各种拓展功能（如 TTS）
    
    ```bash
    cp librechat.example.yaml librechat.yaml
    ```
    
    `:%d` 清空文件后，参考下面填写
    
    ```yaml
    version: 1.1.8
    
    cache: true
    
    #fileStrategy:"firebase"
    
    secureImageLinks: true
    #imageOutputType: "webp"
    
    # interface:
    #   privacyPolicy:
    #     externalUrl: 'https://librechat.ai/privacy-policy'
    #     openNewTab: true
    #   termsOfService:
    #     externalUrl: 'https://librechat.ai/tos'
    #     openNewTab: true
    
    registration:
      socialLogins: ["google", "github", "discord"]
      allowedDomains:
      - "gmail.com"
      - "outlook.com"
    
    speech:
      tts:
        openai:
          apiKey: 'sk-****************************************'
          model: 'tts-1'
          voices: ['alloy','echo','fable','onyx','nova','shimmer']
          url: "https://api.openai.com/v1/audio/speech"
    
      stt:
        openai:
          apiKey: 'sk-L***************************************'
          model: 'whisper-1'
          url: 'https://api.openai.com/v1/audio/transcriptions'
        
    endpoints:
      custom:
        # cohere
        # Model list: https://dashboard.cohere.com/playground/chat
        - name: "cohere"
          apiKey: "user-provided"
          baseURL: "https://api.cohere.ai/v1"
          models:
            default: [
              "c4ai-aya-23-35b",
              "c4ai-aya-23-8b",
              "command",
              "command-light",
              "command-light-nightly",
              "command-nightly",
              "command-r",
              "command-r-plus",
              ]
            fetch: false
          modelDisplayLabel: "cohere"
          titleModel: "command"
          dropParams: ["stop", "user", "frequency_penalty", "presence_penalty", "temperature", "top_p"]
    ```
    
-   修改 `docker-compose.override.yml`
    
    **功能**：可视化管理 MongoDB 数据库，对compose编排微调，避免直接修改 docker-compose.yml
    
    ```bash
    cp docker-compose.override.yml.example docker-compose.override.yml
    ```
    
    `:%d` 清空文件后，参考下面填写
    
    ```yaml
    version: '3.4'
    
    services:
      mongo-express:
        image: mongo-express
        container_name: mongo-express
        environment:
          ME_CONFIG_MONGODB_SERVER: mongodb
          ME_CONFIG_BASICAUTH_USERNAME: "YOUR_ADMIN_NAME"
          ME_CONFIG_BASICAUTH_PASSWORD: "YOUR_ADMIN_PASSWORD"
        ports:
          - '8081:8081'
        depends_on:
          - mongodb
        restart: always
    
      api:
        volumes:
          - ./librechat.yaml:/app/librechat.yaml
    ```
    
    运行后，打开 http://ip:8081 输入管理员用户名密码即可登录
    

### 抱脸空间部署

-   配置数据库
    
    访问 [**MongoDB 官网**](https://account.mongodb.com/account/register) 注册账号并建立数据库
    
-   配置空间
    
    复制 [**LibreChat 空间**](https://huggingface.co/spaces/LibreChat/LibreChat?duplicate=true) 修改环境变量后等待自动重构
    

熟悉后可使用 [**LibreChat 部署工具**](https://fun.of.cloudns.be/) 快速部署，注意 librechat.yaml 通过环境变量 **CONFIG\_PATH** 指定路径或链接

## Agent

原 Plugins 已被 Agents 所取代，使用方法：通道选择 Agents，展开右侧边栏点击 \[代理构建器\] 新建

-   Google搜索
    
    提示词示例
    
    ```
    Assess the need for a web search based on the current question and prior interactions, but lean towards suggesting a Google search query if uncertain. Generate a Google search query even when the answer might be straightforward, as additional information may enhance comprehension or provide updated data. If absolutely certain that no further information is required, return an empty string. Default to a search query if unsure or in doubt. Today's date is {{CURRENT_DATE}}.
    
    Today's date is {{CURRENT_DATE}}.
    
    Current Question:
    {{prompt:end:4000}}
    
    Interaction History:
    {{MESSAGES:END:6}}
    ```
    
-   DALLE绘图
    
    提示词示例
    
    ```
    Instruction:
    
    Evaluate the user's image generation prompt for its suitability for DALL-E.
    
    If the prompt is sufficient for high-quality DALL-E image generation, instruct DALL-E to generate 4 images based on it.
    
    Otherwise, enhance the prompt with diverse stylistic variations to improve its quality for DALL-E, then instruct DALL-E to generate 4 images based on the enhanced prompt.
    
    Output: 4 DALL-E generated images.
    
    User's Image Generation Prompt:
    {{prompt:end:4000}}
    ```
    

## 参考链接

-   [LibreChat-AI/librechat-config-yaml](https://github.com/LibreChat-AI/librechat-config-yaml)