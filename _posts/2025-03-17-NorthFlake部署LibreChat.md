---
layout: mypost
title: NorthFlank 部署 LibreChat 教程
date: 2025-03-17
published: true
categories: 免费容器
tags: 
 - aitool
---

北方云（Northflank）的 **Developer Sandbox** 计划支持绑卡后免费创建最多1个项目，2个服务、2个任务以及1个数据库，注意月预估费用超过20刀后实例无法新建或使用，官方也不建议使用该计划部署生产服务，若有需要建议开通付费服务，支持 **Pay as you go** 

## 注册账号

点击 [这里](https://app.northflank.com/signup) 注册 Northflank 并参考 [官方文档](https://northflank.com/docs/v1/application/getting-started/create-a-project) 创建项目，注意选择 **Developer Sandbox** （免费计划）

- 价格可查看 [定价页](https://northflank.com/pricing) ，可使用 **Pricing Calculator** 方便计算，这里是估算的 [极限20刀](https://northflank.com/pricing?calculator=eyJtb2RlIjoid29ya2xvYWQiLCJ2YWx1ZXMiOnsicmVzb3VyY2VzIjpbeyJpZCI6MC41MTU1MDg2MTc4NTM4NDU3LCJ0eXBlIjoic2VydmljZSIsImlzQnlvYyI6ZmFsc2UsInBsYW4iOiJuZi1jb21wdXRlLTEwIiwiaW5zdGFuY2VzIjoxfSx7ImlkIjowLjAwNTMzNTY0Mjc4NDczMjg2NCwidHlwZSI6InNlcnZpY2UiLCJpc0J5b2MiOmZhbHNlLCJwbGFuIjoibmYtY29tcHV0ZS0yMCIsImluc3RhbmNlcyI6MX0seyJpZCI6MC45NTg0MzMwMDM0NjEyMzQxLCJ0eXBlIjoiYWRkb24iLCJpc0J5b2MiOmZhbHNlLCJraW5kIjoicG9zdGdyZXNxbCIsInBsYW4iOiJuZi1jb21wdXRlLTIwIiwicmVwbGljYXMiOjEsInN0b3JhZ2UiOjQwOTYsInN0b3JhZ2VDbGFzcyI6InNzZCJ9LHsiaWQiOjAuMzgzMTAwNDczNDY3MTI2NCwidHlwZSI6ImJ1aWxkIiwiaXNCeW9jIjpmYWxzZSwicGxhbiI6Im5mLWNvbXB1dGUtNDAwLTE2IiwiYnVpbGRNaW51dGVzIjozMH0seyJpZCI6MC41NzUzNDMyOTM3ODc2NDM5LCJ0eXBlIjoibmV0d29yayIsImdiIjoyMCwicmVxcyI6NDIwMDAwMCwiaXNCeW9jIjpmYWxzZX1dfX0=#calculator)

  | 费用项           | 价格                   |
  | ---------------- | ---------------------- |
  | **CPU 费用**     | $12.00/vCPU·月         |
  | **Memory 费用**  | $6.00/GB·月            |
  | **公网流出流量** | $0.15/GB               |
  | **请求费用**     | $0.50/100万次          |
  | **磁盘费用**     | $0.30/GB               |
  | **日志费用**     | $0.50/GB（前10GB免费） |

- 费用可查看 [账单页](https://app.northflank.com/s/account/billing)， 可以点击 `Add an email` 添加多个邮箱确保及时收信，计费说明可查看 [如何计费](https://northflank.com/docs/v1/application/billing/pricing-on-northflank)

-  在 [预警页](https://app.northflank.com/s/account/billing/create-billing-alert) 添加账单预警，预警值可设为 0 确保产生意外费用后立即发信

## 绑定域名

点击右上角 **+ Create new** → **Domain**

* 输入域名后点击 **Add domain**
* 按提示添加 TXT 记录
* 如果是子域名，还需要添加一条 CNAME 记录

## 开始搭建

下面是搭建框架的简单说明

- 向量数据库`VectorDB`
- 后端数据库`MongoDB`
- RAG向量化服务`RAG`
- 前端页面服务`LibreChat `

`````
+------------+        +------------+        +------------+
|            |        |            |        |            |
| LibreChat  | <----> |    RAG     | <----> |  Vector DB |
|  Service   |        |   Service  |        |            |
+------------+        +------------+        +------------+
        ^                    ^
        |                    | 
		|					 v
        |              +------------+
        |              |            |
        +------------> |  MongoDB   |
                       |            |
                       +------------+

`````

### 创建VectorDB

点击右上角 **+ Create new** → **Addon**

* **Addon type** 选择 `PostgreSQL`
* **Addon name** 填写 `vector`
* **Version** 选择 `ver.16` 
* **Networking** 勾选全部，一定要勾选 `publicly accessible`
* 最后点击 **Create addon**
* 左侧切换到 **Connection details** ，勾选 `External`，复制 `EXTERNAL_POSTGRES_URI` 的值

- 使用终端连接到数据库，注意替换 psql 后的 URI  为上一步复制的值

  ```bash
  psql postgres://myuser:mypassword@myhost:5432/mydatabase
  ```

- 添加 vector 拓展

  ```sql
  CREATE EXTENSION vector;
  SELECT * FROM pg_extension WHERE extname = 'vector';
  exit
  ```

添加拓展后在 **Settings** 取消勾选 `publicly accessible`

### 创建MongoDB

点击 [这里](https://account.mongodb.com/account/register) 注册 MongoDB 并创建项目，免费计划支持最高512M存储，具体使用限制可查看 [限制页](https://www.mongodb.com/zh-cn/docs/atlas/reference/free-shared-limitations/#std-label-atlas-free-tier)，如果需要开通付费服务可查看 [定价页](https://www.mongodb.com/pricing)

具体操作可参考 https://linux.do/t/topic/96509#p-657221-mongodb-5

### 创建 MongoExpress

点击右上角 **+ Create new** → **Service**

* **I want to...** 选择 `Deploy a Docker Image`

- **Service name** 填写 `express`

- **Deployment** 选择 `External image`

- **Image path** 填写 `library/mongo-express:latest`

- **Runtime variables** 切换到 `Env` 填写

    ```ini
    ME_CONFIG_MONGODB_URL=mongodb://**********@mongo-0.mongodb--*****.addon.code.run:27017/*****?replicaSet=rs0&authSource=*****&tls=true
    ME_CONFIG_MONGODB_ENABLE_ADMIN=false
    ME_CONFIG_MONGODB_SSL=true
    ME_CONFIG_BASICAUTH_USERNAME=aaaa
    ME_CONFIG_BASICAUTH_PASSWORD=bbbb
    ```

- **Protocol** 选择 `HTTP`，并勾选 `Publicly expose this port to the internet`

- **Custom domains & security rules (optional)** 展开在 `Add custom domain` 选中已添加的域名

- **Resources** 选择 `nf-compute-20` 机型

- 最后点击 **Create service**

### 创建RAG

点击右上角 **+ Create new** → **Service**

* **I want to...** 选择 `Deploy a Docker Image`

- **Service name** 填写 `rag`

- **Deployment** 选择 `External image`

- **Image path** 填写 `ghcr.io/danny-avila/librechat-rag-api-dev-lite:latest`

- **Runtime variables** 切换到 `Env` 填写

  ```ini
  DB_HOST="**********.run"
  DB_PORT="5432"
  DEBUG_RAG_API=true
  PDF_EXTRACT_IMAGES=false
  JWT_SECRET="aaaaaaaaaa"
  POSTGRES_DB="bbbbbbbbbb"
  POSTGRES_PASSWORD="cccccccccc"
  POSTGRES_USER="dddddddddd"
  VECTOR_DB_TYPE="pgvector"
  EMBEDDINGS_PROVIDER="openai"
  RAG_OPENAI_API_KEY="sk-123456"
  EMBEDDINGS_MODEL="text-embedding-3-large"
  RAG_OPENAI_BASEURL="https://api.openai.com/v1"
  ```

- **Protocol** 选择 `HTTP`，不要勾选 `Publicly expose this port to the internet`

- **Resources** 选择 `nf-compute-10` 机型

- 最后点击 **Create service**

### 创建LibreChat

点击右上角 **+ Create new** → **Service**

* **I want to...** 选择 `Deploy a Docker Image`

- **Service name** 填写 `web`

- **Deployment** 选择 `External image`

- **Image path** 填写 `ghcr.io/danny-avila/librechat-dev:latest`

- **Runtime variables** 切换到 `Env` 填写

  ```ini
  MONGO_URI="mongodb+srv://librechat:12345@abcde.net/?retryWrites=true&appName=librechat"
  CREDS_KEY="aaaaa"
  CREDS_IV="bbbbb"
  JWT_SECRET="ccccc"
  JWT_REFRESH_SECRET="ddddd"
  RAG_API_URL="http://rag:8000"
  RAG_OPENAI_API_KEY="sk-123456"
  RAG_OPENAI_BASEURL="https://api.openai.com/v1"
  ENDPOINTS="openAI,anthropic,google,agents,custom"
  OPENAI_API_KEY="**********"
  OPENAI_MODELS="gpt-4o-mini.gpt-4o,o1-mini,o1,o1-preview,o3-mini,gpt-4.5-preview"
  ANTHROPIC_API_KEY="**********"
  ANTHROPIC_MODELS="claude-3-opus-latest,claude-3-5-haiku-latest,claude-3-5-sonnet-latest,claude-3-7-sonnet-latest"
  GOOGLE_KEY="**********"
  GOOGLE_MODELS="gemini-1.5-pro,gemini-2.0-flash-lite,gemini-2.0-flash,gemini-2.0-pro-exp-0205,gemini-2.0-flash-thinking-exp-01-21,learnlm-1.5-pro-experimental"
  CONFIG_PATH="https://***********/librechat.yaml"
  ```

  **CONFIG_PATH** 指定路径或链接到 `librechat.yaml`，该文件格式可参考：https://www.librechat.ai/docs/configuration/librechat_yaml

- **Protocol** 选择 `HTTP`，并勾选 `Publicly expose this port to the internet`

- **Custom domains & security rules (optional)** 展开在 `Add custom domain` 选中已添加的域名

- **Resources** 选择 `nf-compute-20` 机型

- 最后点击 **Create service**

### 创建存储

点击右上角 **+ Create new** → **Volumes**

- **Volume name** 填写 `librechat`
- **Storage** 保持默认 5GBSSD
- 点击 **Add mount configuration** 填写 `/app/client/public/`
- **Resources** 选择 `web`
- 最后点击 **Create volume**

## 参考链接

- https://platform.openai.com/docs/models
- https://platform.openai.com/docs/pricing
- https://platform.openai.com/docs/guides/rate-limits
- https://docs.anthropic.com/en/docs/about-claude/models
- https://www.anthropic.com/pricing#anthropic-api
- https://docs.anthropic.com/en/api/rate-limits
- https://ai.google.dev/gemini-api/docs/models/gemini?hl=zh-cn
- https://ai.google.dev/gemini-api/docs/pricing?hl=zh-cn
- https://ai.google.dev/gemini-api/docs/rate-limits?hl=zh-cn
