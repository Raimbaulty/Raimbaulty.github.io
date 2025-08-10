---
layout: mypost
title: Cloudflare 部署 NextChat
date: 2025-08-05
published: true
categories: 自部署
tags: 
 - cloudflare
 - nextchat
---

## 部署

[点击复制仓库](https://github.com/ChatGPTNextWeb/NextChat/fork)

[点击创建页面](https://dash.cloudflare.com/?to=/:account/pages/new/provider/github) 

链接到 GitHub 仓库后选择上一步复制的仓库，类型选择 Next.js 并添加以下环境变量

| 变量                   | 值                                                  | 描述                                     |
| :--------------------- | :-------------------------------------------------- | :--------------------------------------- |
| NODE_VERSION           | 20.11                                               | 必填                                     |
| NEXT_TELEMETRY_DISABLE | 1                                                   | 必填                                     |
| YARN_VERSION           | 1.22.19                                             | 必填                                     |
| PHP_VERSION            | 7.4                                                 | 必填                                     |
| DEFAULT_MODEL          | gpt-40-mini                                         | 可选，默认模型                           |
| OPENAI_API_KEY         | sk-1234567                                          | 可选，你自己的OpenAI API Key             |
| BASE_URL               | `https://api.proxy.com/openai`                      | 可选，OpenAI反代地址或OneAPI地址         |
| GOOGLE_API_KEY         | AISy                                                | 可选，你自己的Google API Key             |
| GOOGLE_URL             | `https://api.proxy.com/gemini`                      | 可选，Gemini反代地址                     |
| DEEPSEEK_API_KEY       | sk-1234567                                          | 可选，你自己的DeepSeek API Key           |
| DEEPSEEK_URL           |                                                     | 可选，DeepSeek反代地址                   |
| CUSTOM_MODELS          | -all,gpt-5-chat-latest,gemini-2.5-pro,deepseek-chat | 可选，自定义模型                         |
| CODE                   |                                                     | 可选，站点密码，可以使用逗号隔开多个密码 |
| HIDE_USER_API_KEY      | 1                                                   | 可选，不让用户自行填入 API Key           |

点击继续处理项目，在设置里修改兼容性标志为 `nodejs_compat` 后重试部署

## 插件

访问站点，点击 **发现** → **插件** → **+ 新建**

- 硅基流动绘图

  - **授权方式**：Bearer
  - **位置：**Header
  - **Token：**sk-123456
  - **OpenAPI Schema：**`https://raw.githubusercontent.com/ChatGPTNextWeb/NextChat-Awesome-Plugins/refs/heads/main/plugins/KolorsSiliconFlow/openapi.json`

  从网页加载后确认

- Tavily联网搜索

  - **授权方式：**Bearer
  - **位置：**Header
  - **Token：**tvly-123456
  - **OpenAPI Schema：**`https://raw.githubusercontent.com/ChatGPTNextWeb/NextChat-Awesome-Plugins/refs/heads/main/plugins/tavilysearch/openapi.json`

  从网页加载后确认

- NPM 包检索

  - **授权方式：**不需要授权
  - **OpenAPI Schema：**`https://raw.githubusercontent.com/ChatGPTNextWeb/NextChat-Awesome-Plugins/refs/heads/main/plugins/npmsearch/openapi.json`

  从网页加载后确认