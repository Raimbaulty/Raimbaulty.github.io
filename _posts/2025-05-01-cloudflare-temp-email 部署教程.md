---
layout: mypost
title: cloudflare-temp-email 部署教程
date: 2025-05-01
published: true
categories: 自部署
tags: 
 - cloudflare
 - mail
---

# cloudflare-temp-email 部署教程

## 项目介绍

https://github.com/dreamhunter2333/cloudflare_temp_email

请先 Fork 上面的仓库，然后继续

## 创建数据库

登录 Cloudflare，依次点击左侧菜单 **存储和数据库** → **D1 SQL 数据库** → **创建**，名称输入 `cloudflare_temp_email` 后点击创建

接着切换到 **控制台** 标签，右键打开 [schema.sql](https://raw.githubusercontent.com/dreamhunter2333/cloudflare_temp_email/main/db/schema.sql) 后复制所有内容粘贴到输入框，最后点击 **执行** 完成创建

## 下载压缩包

[点击这里](https://temp-mail-docs.awsl.uk/zh/guide/ui/pages.html) 打开页面找到第3项，在输入框输入后端地址（示例：https://mail.example.com，注意不带 `/`），点击 **生成** 然后 点击 **下载**，如若输入有误，需刷新页面后重试

## 创建 Pages

依次点击左侧菜单 **计算（Workers）**→  **Workers and Pages** → **创建**，切换到 **Pages** 标签并点击 **上传资产**

项目名称输入 `cloudflare-temp-email` 后点击 **创建项目**，接着点击上传上一步下载的 `frontend.zip` 后点击 **部署站点** 完成部署

## 创建 Action

进入 Fork 的仓库，切换到 **Settings** 标签，点击左侧 **Secrets and variables** - **Actions** → **New repository secret**，依次添加以下变量

| 变量名                | 示例值                | 描述                           |
| --------------------- | --------------------- | ------------------------------ |
| DEBUG_MODE            | true                  | 是否开启调试模式               |
| CLOUDFLARE_ACCOUNT_ID | abcdefg               | Cloudflare 账户的唯一标识      |
| CLOUDFLARE_API_TOKEN  | hijklmn               | 用于访问 Cloudflare API 的令牌 |
| FRONTEND_NAME         | cloudflare-temp-email | 前端 Pages 名称                |
| FRONTEND_ENV          | 见下文                | 前端环境变量                   |
| BACKEND_TOML          | 见下文                | 后端配置文件的路径或内容       |
- FRONTEND_ENV

  ```
  VITE_API_BASE=https://api.example.com
  VITE_CF_WEB_ANALY_TOKEN=
  VITE_IS_TELEGRAM=false
  ```

- BACKEND_TOML

  ```toml
  name = "cloudflare_temp_email"
  main = "src/worker.ts"
  compatibility_date = "2024-09-23"
  compatibility_flags = [ "nodejs_compat" ]
  
  routes = [
   { pattern = "https://api.example.com", custom_domain = true },
  ]
  
  [vars]
  PREFIX = "tmp"
  DOMAINS = ["temp1.com" , "temp2.com"]
  JWT_SECRET = "xxxxxxxxxxxxxxxxxxxx"
  ADMIN_PASSWORDS = ["password1", "password2"]
  
  ENABLE_USER_CREATE_EMAIL = true
  ENABLE_USER_DELETE_EMAIL = true
  
  [[d1_databases]]
  binding = "DB"
  database_name = "cloudflare_temp_email" # D1 数据库名称
  database_id = "xxxxxxxxx" # D1 数据库 ID
  
  # kv config 用于用户注册发送邮件验证码，如果不启用用户注册或不启用注册验证，可以不配置
  # [[kv_namespaces]]
  # binding = "KV"
  # id = "xxxx"
  
  # 如果你想要部署带有前端资源的 worker, 你需要添加 assets 配置
  # [assets]
  # directory = "../frontend/dist/"
  # binding = "ASSETS"
  # run_worker_first = true
  
  # 如果你想要使用定时任务清理邮件，取消下面的注释，并修改 cron 表达式
  # [triggers]
  # crons = [ "0 0 * * *" ]
  
  # 通过 Cloudflare 发送邮件
  # send_email = [
  #    { name = "SEND_MAIL" },
  # ]
  
  # 新建地址限流配置 /api/new_address
  # [[unsafe.bindings]]
  # name = "RATE_LIMITER"
  # type = "ratelimit"
  # namespace_id = "1001"
  # # 10 requests per minute
  # simple = { limit = 10, period = 60 }
  
  # 绑定其他 worker 处理邮件，例如通过 auth-inbox ai 能力解析验证码或激活链接
  # [[services]]
  # binding = "AUTH_INBOX"
  # service = "auth-inbox"
  ```

  然后切换到 **Actions** 标签，点击左侧 **Deploy Backend**，接着点击 **Run workflow** → **Run workflow** 部署后端，刷新页面就可以看到任务了，稍等片刻，等待任务执行，成功会显示✅，点击左侧 **Deploy Frontend**重复相同步骤即可
