---
layout: mypost
title: 常用LLM-IDE-Cli 使用笔记
date: 2025-09-17
published: true
categories: 人工智能
tags: 
 - aitool
---

## Codex

使用官方服务需要订阅付费服务或开通API服务

官方仓库：https://github.com/openai/codex

官方文档：https://developers.openai.com/codex/quickstart

官方限额：https://help.openai.com/en/articles/11369540-using-codex-with-your-chatgpt-plan

首先查看 node 版本，要求版本 ≥ 20.0.0

```bash
 node -v
```

安装

```bash
npm install -g @openai/codex
```

运行

```bash
cd /path/to/the/project
codex
```

参考下面自定义配置，Windows系统配置文件位于 `%USERPROFILE%/.codex/config.toml`

使用 `codex --profile veloera` 指定供应商，未指定时使用 `veloera`

```toml
model_provider = "veloera"
model = "gpt-5"

[profiles.kilo]
model_provider = "kilo"
model = "openai/gpt-5"

[profiles.openrouter]
model_provider = "openrouter"
model = "openrouter/sonoma-sky-alpha"

[profiles.veloera]
model_provider = "veloera"
model = "gpt-5"

[profiles.linjinpeng]
model_provider = "linjinpeng"
model = "gpt-5"

[profiles.packycode]
model_provider = "packycode"
model = "gpt-5"
model_reasoning_effort = "high"
disable_response_storage = true


[model_providers.kilo]
name = "kilo"
base_url = "http://127.0.0.1:9000/v1"
wire_api = "chat"
model_reasoning_effort = "high"
http_headers = { "Authorization" = "Bearer your_api_key" }

[model_providers.openrouter]
name = "OpenRouter"
base_url = "https://openrouter.ai/api/v1"
wire_api = "chat"
http_headers = { "Authorization" = "Bearer your_api_key" }

[model_providers.veloera]
name = "veloera"
base_url = "https://zone.veloera.org/v1"
wire_api = "chat"
http_headers = { "Authorization" = "Bearer your_api_key" }

[model_providers.linjinpeng]
name = "linjinpeng"
base_url = "https://linjinpeng-veloera.hf.space/v1"
wire_api = "chat"
http_headers = { "Authorization" = "Bearer your_api_key" }

[model_providers.packycode]
name = "packycode"
base_url = "https://oai-api.fkclaude.com/v1"
wire_api = "responses"
env_key = "packycode"
```



## Google-Cli

官方仓库：https://github.com/google-gemini/gemini-cli

首先查看 node 版本，要求版本 ≥ 20.0.0

```bash
node -v
```

临时安装

```bash
npx https://github.com/google-gemini/gemini-cli
```

永久安装

```bash
npm install -g @google/gemini-cli
```

有三种认证方式

- 使用Google账户

  在打开的浏览器页面登录认证授权

- 使用Gemini-Key

    Windows系统： `%USERPROFILE%/.gemini/.env`

    ```ini
    GEMINI_API_KEY="AIzaSy********************"
    GOOGLE_GEMINI_BASE_URL="http://api.xxx.xxx"
    ```

- 使用Vertex

  Windows系统： `%USERPROFILE%/.gemini/.env`
  
  ```ini
  GOOGLE_CLOUD_PROJECT="xxx-xxx-xxx"
  GOOGLE_CLOUD_LOCATION="us-central1"
  GOOGLE_VERTEX_BASE_URL="http://api.xxx.xxx"
  ```
  

运行

```bash
cd /path/to/the/project
gemini
```



## Claude Code

官方仓库：https://github.com/anthropics/claude-code

官方文档：https://docs.claude.com/en/docs/claude-code/quickstart

首先查看 node 版本，要求版本 ≥ 18.0.0

```bash
node -v
```

安装

```bash
npm install -g @anthropic-ai/claude-code
```

运行

```bash
cd /path/to/the/project
claude
```



## 参考链接

- [codex配置文件](https://linux.do/t/topic/943034) 