---
layout: mypost
title: LibreChat 环境变量全解析
date: 2024-11-19
published: true
categories: 自部署
tags:
 - aitool 
 - librechat
---

## 部署配置

### 服务器配置

#### Redis配置

如果使用 Redis，配置更改后需要刷新缓存

-   **USE_REDIS**
    -   描述：是否使用Redis
        
    -   值：false
-   **REDIS_URI**

    -   描述：Redis链接
        
    -   值：redis://\[:password\]@host:port/db-number
    
-   **USE_REDIS_CLUSTER**
    -   描述：启用 Redis 集群

    -   值：true
-   **REDIS_CA**

    -   描述：Redis 证书路径

    -   值：/path/to/file.crt
-   **REDIS_KEY_PREFIX**
-   描述：Redis 键前缀
    
-   值：librechat-staging:
-   **REDIS_MAX_LISTENERS**

    -   描述：Redis 最大监听器数

    -   值：数值类型，例如 50

#### Docker权限配置

非 Docker 部署不考虑

-   **UID**
    
    -   描述：Docker Compose 用户ID
        
    -   值：1000
    
-   **GID**
    
    -   描述：Docker Compose 用户组ID
        
    -   值：1000
        

### 网络配置

#### 端口配置

-   **HOST**

    -   描述：主机
        
    -   值：localhost
-   **PORT**

    -   描述：端口
    -   值：3080
-   **TRUST_PROXY**
    -   描述：信任的代理层数
    -   值：1

#### 域名配置

-   **DOMAIN_CLIENT**
    
    -   描述：客户端域名
        
    -   值：`http://localhost:3080`
    
-   **DOMAIN_SERVER**
    -   描述：服务端域名
        
    -   值：`http://localhost:3080`
        

### 数据库配置

-   **MONGO_URI**
    -   描述：MongoDB 链接
        
    -   值：
        
        -   本地数据库：`mongodb+srv://<username>:<password>@<host>/<database>?<options>`
            
        -   在线数据库：`mongodb+srv://<username>:<password>@<host>/?retryWrites=true&appName=<appname>`
            
        -   documentDb：`mongodb+srv://<username>:<password>@<host>/<database>?retryWrites=false&tls=true&tlsCAFile=/path-to-ca/bundle.pem`

### RAG 配置

**注意：** RAG 服务需要自部署：[仓库地址](https://github.com/danny-avila/rag_api/)

-   **RAG_OPENAI_API_KEY**
    -   描述：RAG 服务 API 密钥
    -   值：字符串（API密钥）

-   **RAG_PORT**
    -   描述：RAG 服务端口
    -   值：8000

-   **RAG_HOST**
    -   描述：RAG 服务地址
    -   值：0.0.0.0

-   **COLLECTION_NAME**
    -   描述：向量集名称
    -   值：testcollection

-   **RAG_USE_FULL_CONTEXT**
    -   描述：获取文件前4条消息以外的上下文
    -   值：false

-   **CHUNK_SIZE**
    -   描述：文本处理大小，默认1500
    -   值：1500

-   **CHUNK_OVERLAP**
    -   描述：文本处理重叠，默认100
    -   值：100

-   **EMBEDDINGS_PROVIDER**
    -   描述：嵌入服务提供商，选项包括 openai、azure、huggingface、huggingfacetei 以及 ollama
    -   值：openai

-   **EMBEDDINGS_MODEL**
    -   描述：嵌入服务模型
    -   值：text-embedding-3-small

-   **HF_TOKEN**
    -   描述：使用 huggingface 嵌入服务时，hugginface 令牌
    -   值：hf_xxxxxxxxxxxxxxxxxxxxxxx
-   **OLLAMA_BASE_URL**
    -   描述：使用本地嵌入服务时，Ollama 服务地址
    -   值：`http://host.docker.internal:11434`

### 文件与缓存配置

#### 自定义配置文件

-   **CONFIG_PATH**
    
    -   描述：自定义配置文件路径，支持绝对路径、相对路径以及URL
        
    -   值：`https://raw.githubusercontent.com/danny-avila/LibreChat/main/librechat.example.yaml`
        

#### **静态文件设置**

仅当 NODE_ENV 设置为 production 才生效

-   **STATIC_CACHE_MAX_AGE**
    -   描述：本地缓存时间，默认4周
        
    -   值：172800（以秒为单位，此处为2天）
    
-   **STATIC_CACHE_S_MAX_AGE**
    
    -   描述：共享缓存时间，默认1周
        
    -   值：86400（以秒为单位，此处为1天）
    
-   **DISABLE_COMPRESSION**
    
    -   描述：是否禁用静态文件的压缩
        
    -   值：false
        

#### 主页缓存配置

-   **INDEX_HTML_CACHE_CONTROL**
    
    -   描述：主页缓存控制
        
    -   值：
        
        -   no-cache：每次请求时浏览器需要与服务器确认是否可以使用缓存文件
            
        -   no-store：禁止在本地或代理服务器上存储任何缓存内容
            
        -   must-revalidate：要求在使用缓存文件之前，先验证其是否被更改
    
-   **INDEX_HTML_PRAGMA**
    
    -   描述：旧版主页缓存控制
        
    -   值：no-cache（提供向后兼容性）
    
-   **INDEX_HTML_EXPIRES**
    
    -   描述：主页缓存过期时间
        
    -   值：86400（单位为秒，此处为1天，0表示永不过期）
        

### 日志配置

-   **DEBUG_LOGGING**

    -   描述：启用调试日志，默认启用
        
    -   值：true

-   **DEBUG_CONSOLE**
    -   描述：启用控制台输出日志
        
    -   值：false

-   **CONSOLE_JSON**
    -   描述：启用控制台输出详细JSON格式日志，适用于云部署（如GCP/AWS）
        
    -   值：false
    
-   **CONSOLE_JSON_STRING_LENGTH**
    - 描述：控制台或标准输出日志截断长度
    - 值：255

### 页面配置

#### 搜索引擎收录设置

-   **NO_INDEX**
    
    -   描述：是否禁止搜索引擎收录，默认禁止
        
    -   值：true
        

#### **标题和页脚**

-   **APP_TITLE**
    
    -   描述：网页标题
        
    -   值：LibreChat
    
-   **CUSTOM_FOOTER**
    
    -   描述：网页页脚，留空移除页脚
        
    -   值：\[Link 1\](http://example1.com) | \[Link 2\](http://example2.com)
        

#### 帮助链接

-   **HELP_AND_FAQ_URL**
    
    -   描述：帮助页链接
        
    -   值：`https://librechat.ai`
        

#### 生日图标

-   **SHOW_BIRTHDAY_ICON**
    
    -   描述：开启后会在安装 LibreChat 的每周年显示生日帽
        
    -   值：true
        

## 通道配置

### **全局配置**

-   **ENDPOINTS**
    
    -   描述：启用的服务通道，用逗号分隔
        
    -   值：openAI,assistants,azureOpenAI,bedrock,google,anthropic,custom,agents
    
-   **PROXY**
    -   描述：全局代理地址
        
    -   值：`http://127.0.0.1:8080`
    
-   **TITLE_CONVO**
    
    -   描述：全局生成对话标题开关
        
    -   值：true
        

### 对话通道

当 API_KEY 配置为 user_provided 时，表示允许用户通过客户端自行配置 API_KEY

#### AWS Bedrock

- **BEDROCK_AWS_DEFAULT_REGION**
  - 描述：AWS 通道服务地区
  - 值：us-east-1
- **BEDROCK_AWS_ACCESS_KEY_ID**
  - 描述：AWS 通道服务令牌
  - 值：your_access_key_id
- **BEDROCK_AWS_SECRET_ACCESS_KEY**
  - 描述：AWS 通道服务密钥
  - 值：your_secret_access_key
- **BEDROCK_AWS_MODELS**
  - 描述：AWS 通道启用模型列表
  - 值：anthropic.claude-3-5-sonnet-20240620-v1:0,meta.llama3-1-8b-instruct-v1:0

#### **Anthropic**

-   **ANTHROPIC_API_KEY**
    -   描述：Anthropic 通道 API_Key
        
    -   值：sk-ant-api03-xxxx
    
-   **ANTHROPIC_MODELS**
    
    -   描述：Anthropic 通道启用模型列表
        
    -   值：claude-3-opus-latest,claude-3-5-haiku-latest,claude-3-5-sonnet-latest,claude-3-7-sonnet-latest
    
-   **ANTHROPIC_REVERSE_PROXY**
    
    -   描述：Anthropic 通道中转地址
        
    -   值：`https://proxy.xxx.com/v1`
    
-   **ANTHROPIC_TITLE_MODEL**
    
    -   描述：Anthropic 通道生成标题的模型，现已弃用，请在 `librechat.yaml` 中配置
        
    -   值：claude-3-haiku-20240307
        

#### **BingAI**

-   **BINGAI_TOKEN**
    
    -   描述：Bing 通道访问令牌
        
    -   值：xxx-xxx-xxx
    
-   **BINGAI_HOST**
    
    -   描述：Bing 通道服务器地址
        
    -   值：`https://cn.bing.com`
        

#### **Google**

-   **GOOGLE_KEY**
    
    -   描述：Google 通道 API Key
        
    -   值：xxx-xxx-xxx
-   **GOOGLE_REVERSE_PROXY**
    
    -   描述：Google 通道中转地址
        
    -   值：`https://proxy.com/v1`
-   **GOOGLE_MODELS**
    -   描述：Google 通道启用模型列表
        
    -   值：gemini-1.5-pro,gemini-2.0-flash-lite,gemini-2.0-flash,gemini-2.0-pro-exp-0205,gemini-2.0-flash-thinking-exp-01-21,learnlm-1.5-pro-experimental
-   **GOOGLE_TITLE_MODEL**
    
    -   描述：Google 通道生成标题的模型，现已弃用，请在 `librechat.yaml` 中配置
        
    -   值：gemini-pro
-   **GOOGLE_LOC**
    -   描述：Google 云服务区域位置
        
    -   值：us-central1

以下是 Google 安全审查项，各安全审查项的选项包括

| 选项            | 说明                     |
| --------------- | ------------------------ |
| BLOCK_ALL       | 完全阻止所有安全风险行为 |
| BLOCK_ONLY_HIGH | 仅阻止高风险安全行为     |
| WARN_ONLY       | 只发出安全警告，不阻止   |
| OFF             | 不进行安全阻止和警告     |

-   **GOOGLE_SAFETY_SEXUALLY_EXPLICIT**
    
    -   描述：性露骨内容安全审查
        
    -   值：BLOCK_ALL
    
-   **GOOGLE_SAFETY_HATE_SPEECH**
    
    -   描述：仇恨言论内容安全审查
        
    -   值：BLOCK_ALL
    
-   **GOOGLE_SAFETY_HARASSMENT**
    
    -   描述：骚扰内容安全审查
        
    -   值：BLOCK_ALL
    
-   **GOOGLE_SAFETY_DANGEROUS_CONTENT**
    
    -   描述：危险内容安全审查
        
    -   值：BLOCK_ALL
        

#### **OpenAI**

-   **OPENAI_API_KEY**
    
    -   描述：OpenAI 通道 API Key
        
    -   值：sk-xxx
    
-   **OPENAI_MODELS**
    
    -   描述：OpenAI 通道可用模型列表
        
    -   值：gpt-4o-mini.gpt-4o,o1-mini,o1,o1-preview,o3-mini,gpt-4.5-preview
    
-   **DEBUG_OPENAI**
    
    -   描述：OpenAI 通道调试模式开关
        
    -   值：false
    
-   **OPENAI_TITLE_MODEL**
    -   描述：OpenAI 通道用于生成标题的模型，现已弃用，请在 `librechat.yaml` 中配置
        
    -   值：gpt-3.5-turbo
    
-   **OPENAI_SUMMARIZE**
    
    -   描述：总结对话开关，默认关闭
        
    -   值：false
    
-   **OPENAI_SUMMARY_MODEL**
    
    -   描述：OpenAI 通道总结对话的模型
        
    -   值：gpt-3.5-turbo
    
-   **OPENAI_FORCE_PROMPT**
    
    -   描述：OpenAI 通道强制注入系统提示词
        
    -   值：xxxxxxxxx
    
-   **OPENAI_REVERSE_PROXY**
    
    -   描述：OpenAI 通道中转地址
        
    -   值：`https://proxy.xxx.com/v1`
    
-   **OPENAI_ORGANIZATION**
    
    -   描述：OpenAI 通道组织编号
        
    -   值：xxxxxxxxx
        

#### **Assistants**

-   **ASSISTANTS_API_KEY**
    
    -   描述：用于Assistants API的 API Key
        
    -   值：xxx-xxx-xxx
    
-   **ASSISTANTS_MODELS**
    
    -   描述：Assistant 通道可用模型列表
        
    -   值：gpt-3.5-turbo-0125,gpt-3.5-turbo-16k-0613,gpt-3.5-turbo-16k,gpt-3.5-turbo,gpt-4,gpt-4-0314,gpt-4-32k-0314,gpt-4-0613,gpt-3.5-turbo-0613,gpt-3.5-turbo-1106,gpt-4-0125-preview,gpt-4-turbo-preview,gpt-4-1106-preview
    
-   **ASSISTANTS_BASE_URL**
    
    -   描述：自定义 Assistants 通道的 BASE_URL
        
    -   值：`https://api.xxx.com/v1`
        

#### Others

**注意：** 须在 `librechat.yaml` 定义，相应的也可添加更多其他通道

-   **ANYSCALE_API_KEY**
    -   官网地址：[Anyscale](https://www.anyscale.com/)

-   **APIPIE_API_KEY**

    -   官网地址：[Apipie](https://apipie.ai/)

-   **COHERE_API_KEY**
    -   官网地址：[Cohere](https://cohere.com/)
    
-   **FIREWORKS_API_KEY**
    -   官网地址：[Fireworks](https://fireworks.ai/)

-   **GROQ_API_KEY**
    -   官网地址：[Groq](https://groq.com/)

-   **MISTRAL_API_KEY**
    -   官网地址：[Mistral](https://mistral.ai/)
    
-   **OPENROUTER_KEY**
    -   官网地址：[OpenRouter](https://openrouter.ai/)
    
-   **PERPLEXITY_API_KEY**

    -   官网地址：[Perplexity](https://www.perplexity.ai/)

-   **SHUTTLEAI_API_KEY**

    -   官网地址：[ShuttleAI](https://shuttleai.com/)

-   **TOGETHERAI_API_KEY**
    -   官网地址：[TogetherAI](https://www.together.ai/)

- **GITHUB_TOKEN**
  - 官网地址：[Github](https://github.com/settings/tokens)

-   **DEEPSEEK_API_KEY**
    - 官网地址：[Deepseek](https://platform.deepseek.com/api_keys)

- **SAMBANOVA_API_KEY**
  - 官网地址：[SambaNova](https://cloud.sambanova.ai/apis)

### 搜索通道

- **SERPER_API_KEY**
  - 官网地址：[SerperAPI](https://serper.dev/api-key)
- **FIRECRAWL_API_KEY**
  - 官网地址：[FireCrawl](https://docs.firecrawl.dev/introduction#api-key)
- **FIRECRAWL_API_URL**
  - 可选，自定义实例端点地址
- **JINA_API_KEY**
  - 官网地址：[Jina](https://jina.ai/api-dashboard/)
- **COHERE_API_KEY**
  - 官网地址：[Cohere](https://dashboard.cohere.com/welcome/login)

### 插件通道

#### 基本配置

-   **PLUGIN_MODELS**
    -   描述：Plugins 通道可用模型列表，首个模型将视作默认模型
        
    -   值：gpt-4,gpt-4-turbo,gpt-4-turbo-preview,gpt-4-0125-preview,gpt-4-1106-preview,gpt-4-0613,gpt-3.5-turbo,gpt-3.5-turbo-0125,gpt-3.5-turbo-1106,gpt-3.5-turbo-0613
    
-   **DEBUG_PLUGINS**
    
    -   描述：Plugins 通道调试模式开关
        
    -   值：false
    
-   **EXPERIMENTAL_AGENTS**
    -   描述：替代 Plugins 通道的新功能，原 Plugins 通道将被淘汰
        
    -   值：默认 false，设置为 true 并在 ENDPOINTS 添加方可打开
        

#### 插件配置

##### Azure AI Search

-   **AZURE_AI_SEARCH_SERVICE_ENDPOINT**
    
    -   描述：Azure AI Search 服务地址
        
    -   值：YOUR_AZURE_AI_SEARCH_SERVICE_ENDPOINT
    
-   **AZURE_AI_SEARCH_INDEX_NAME**
    
    -   描述：Azure AI Search 索引名称
        
    -   值：YOUR_AZURE_AI_SEARCH_INDEX_NAME
    
-   **AZURE_AI_SEARCH_API_KEY**
    
    -   描述：Azure AI Search API_KEY
        
    -   值：YOUR_AZURE_AI_SEARCH_API_KEY
    
-   **AZURE_AI_SEARCH_API_VERSION**
    
    -   描述：Azure AI Search API 版本号
        
    -   值：YOUR_AZURE_AI_SEARCH_API_VERSION
    
-   **AZURE_AI_SEARCH_SEARCH_OPTION_QUERY_TYPE**
    
    -   描述：Azure AI Search 搜索类型
        
    -   值：YOUR_QUERY_TYPE
    
-   **AZURE_AI_SEARCH_SEARCH_OPTION_TOP**
    
    -   描述：Azure AI Search 处理选项数
        
    -   值：YOUR_TOP
    
-   **AZURE_AI_SEARCH_SEARCH_OPTION_SELECT**
    
    -   描述：Azure AI Search 处理字段
        
    -   值：YOUR_SELECT_FIELDS
        

##### DALL-E

-   **DALLE_API_KEY**
    -   描述：DALL-E 2 和 DALL-E 3 共用的 OpenAI API_KEY
        
    -   值：YOUR_DALLE_API_KEY
    
-   **DALLE3_API_KEY**
    
    -   描述：DALL-E 3 专用 OpenAI API_KEY
        
    -   值：YOUR_DALLE3_API_KEY
    
-   **DALLE2_API_KEY**
    
    -   描述：DALL-E 2 专用 OpenAI API_KEY
        
    -   值：YOUR_DALLE2_API_KEY
    
-   **DALLE3_SYSTEM_PROMPT**
    
    -   描述：DALL-E 3 系统提示词
        
    -   值："Your DALL-E-3 System Prompt here"
    
-   **DALLE2_SYSTEM_PROMPT**
    
    -   描述：DALL-E 2 系统提示
        
    -   值："Your DALL-E-2 System Prompt here"
    
-   **DALLE_REVERSE_PROXY**
    
    -   描述：DALL-E 2 和 DALL-E 3 的中转地址
        
    -   值：YOUR_DALLE_REVERSE_PROXY_URL
    
-   **DALLE3_BASEURL**
    
    -   描述：DALL-E 3 专用 BASE_URL
        
    -   值：`https://api.xxx.com/v1`， Azure：`https://<AZURE_OPENAI_API_INSTANCE_NAME>.openai.azure.com/openai/deployments/<DALLE3_DEPLOYMENT_NAME>/`
    
-   **DALLE2_BASEURL**
    -   描述：DALL-E 2 专用 BASE_URL
        
    -   值：`https://api.xxx.com/v1`， Azure：`https://<AZURE_OPENAI_API_INSTANCE_NAME>.openai.azure.com/openai/deployments/<DALLE2_DEPLOYMENT_NAME>/`  注意 Azure 已于2025年1月27日停用该模型
    
-   **DALLE3_AZURE_API_VERSION**
    -   描述：Azure OpenAI 服务 DALL-E 3 版本号
        
    -   值：3.0
    
-   **DALLE2_AZURE_API_VERSION**
    -   描述：Azure OpenAI 服务 DALL-E 2 版本号，注意 Azure 已于2025年1月27日停用该模型
        
    -   值：2.0
        

##### OpenAI Image

- **IMAGE_GEN_OAI_API_KEY**
  - 描述：GPT-Image-1 的专用 OpenAI API_KEY
  - 值：YOUR_IMAGE_GEN_OAI_API_KEY

- **IMAGE_GEN_OAI_BASEURL**
  - 描述：GPT-Image-1的专用 BASE_URL
  - 值：YOUR_IMAGE_GEN_OAI_BASEURL
- **IMAGE_GEN_OAI_AZURE_API_VERSION**
  - 描述：Azure OpenAI 服务 GPT-Image-1 的版本号
  - 值：2025-04-15
- **IMAGE_GEN_OAI_DESCRIPTION_WITH_FILES**
  - 描述：图生图功能的文字提示	
  - 值：基于上传的图片和描述，生成新的创意图像
- **IMAGE_GEN_OAI_DESCRIPTION_NO_FILES**
  - 描述：文生图功能的文字提示
  - 值：请输入文字描述生成新的图像，无需上传图片
- **IMAGE_EDIT_OAI_DESCRIPTION**
  - 描述：图像编辑工具功能的文字提示
  - 值：使用文字描述编辑和修改已上传的图片
- **IMAGE_GEN_OAI_PROMPT_DESCRIPTION**
  - 描述：生成图片时，提示用户输入的文字提示
  - 值：请描述您想生成的图像内容，例如“蓝天白云下的山脉”
- **IMAGE_EDIT_OAI_PROMPT_DESCRIPTION**
  - 描述：编辑图片时，提示用户输入的文字提示
  - 值：请输入您想对图片进行的修改说明，例如“把颜色调成秋天的暖色调”

##### Google Search

-   **GOOGLE_SEARCH_API_KEY**
    -   描述：Google Search API_KEY
        
    -   值：YOUR_GOOGLE_SEARCH_API_KEY
    
-   **GOOGLE_CSE_ID**
    
    -   描述：Google 自定义搜索引擎ID
        
    -   值：YOUR_GOOGLE_CSE_ID
        

##### SerpAPI

-   **SERPAPI_API_KEY**
    
    -   描述：SerpAPI API_KEY
        
    -   值：YOUR_SERPAPI_API_KEY
        

##### Stable Diffusion

-   **SD_WEBUI_URL**
    -   描述：本地 Stable Diffusion Web UI 地址
    
    -   值：
    
        -   本地安装：`http://127.0.0.1:7860`
        
    -   docker 安装：`http://host.docker.internal:7860`

##### **Code Interpreter**

- **LIBRECHAT_CODE_API_KEY**
  - 描述：LibreChat 官方代码助手的 API_KEY
  - 值：YOUR_LIBRECHAT_CODE_API_KEY
- **LIBRECHAT_CODE_BASEURL**
  - 描述：企业用户的自定义端点地址
  - 值：`https://your-custom-domain.com`

##### OPENWEATHER

- **OPENWEATHER_API_KEY**
  - 官网地址：[OpenWeather](https://home.openweathermap.org/api_keys)

##### Flux

- **FLUX_API_KEY**
  - 官网地址：[BlackForest](https://bfl.ai/)

##### Wolfram

-   **WOLFRAM_APP_ID**
    -   官网地址： [Wolfram Alpha](https://products.wolframalpha.com/api/)
        

##### Tavily

-   **TAVILY_API_KEY**
    -   官网地址：[Tavily](https://tavily.com/#api)

##### Traversaal

-   **TRAVERSAAL_API_KEY**
    -   官网地址：[Traversaal](https://api.traversaal.ai/dashboard)

##### Zapier

-   **ZAPIER_NLA_API_KEY**
    -   官网地址： [Zapier](https://nla.zapier.com/credentials/)

## 审查配置

### **基本审查配置**

-   **OPENAI_MODERATION**
    
    -   描述：启用审查（OpenAI 和 Plugins 通道）
        
    -   值：false
    
-   **OPENAI_MODERATION_API_KEY**
    
    -   描述：用于审查的 OpenAI API_KEY
        
    -   值：YOUR_OPENAI_API_KEY
    
-   **OPENAI_MODERATION_REVERSE_PROXY**
    
    -   描述：自定义用于审查的 OpenAI API_BASE_URL
        
    -   值：`https://api.openai.com/v1`
        

### **封禁配置**

-   **BAN_VIOLATIONS**
    
    -   描述：启用自动封禁（违规行为仍会被记录）
        
    -   值：true
    
-   **BAN_DURATION**
    
    -   描述：用户及关联IP封禁时间（单位：毫秒）
        
    -   值：1000\*60\*60\*2（即2小时）
    
-   **BAN_INTERVAL**
    
    -   描述：用户评分达到或超过该阈值将被封禁
        
    -   值：20
        

### **违规评分**

-   **LOGIN_VIOLATION_SCORE**
    
    -   描述：登录违规评分
        
    -   值：1
    
-   **REGISTRATION_VIOLATION_SCORE**
    
    -   描述：注册违规评分
        
    -   值：1
    
-   **CONCURRENT_VIOLATION_SCORE**
    
    -   描述：并发违规评分
        
    -   值：1
    
-   **MESSAGE_VIOLATION_SCORE**
    
    -   描述：消息违规评分
        
    -   值：1
    
-   **NON_BROWSER_VIOLATION_SCORE**
    
    -   描述：非浏览器违规评分
        
    -   值：20
    
-   **ILLEGAL_MODEL_REQ_SCORE**
    
    -   描述：非法模型请求评分
        
    -   值：5
        

### **速率限制（每用户和IP）**

-   **LIMIT_CONCURRENT_MESSAGES**
    
    -   描述：是否限制用户单次可发送消息
        
    -   值：true
    
-   **CONCURRENT_MESSAGE_MAX**
    
    -   描述：用户单次可发送最大消息数
        
    -   值：2
        

### **消息限制（每用户或IP）**

-   **IP限制**
    
    -   **LIMIT_MESSAGE_IP**
        
        -   描述：是否启用IP限制
            
        -   值：true
        
    -   **MESSAGE_IP_MAX**
        
        -   描述：每个时间窗口内单IP可发送的最大消息数
            
        -   值：40
        
    -   **MESSAGE_IP_WINDOW**
        -   描述：单IP发送消息的时间窗口，单位：分钟
            
        -   值：1
    
-   **用户限制**
    
    -   **LIMIT_MESSAGE_USER**
        
        -   描述：是否启用用户限制
            
        -   值：false
        
    -   **MESSAGE_USER_MAX**
        
        -   描述：每个时间窗口内单用户可发送的最大消息数
            
        -   值：40
        
    -   **MESSAGE_USER_WINDOW**
        
        -   描述：单用户发送消息的时间窗口，单位：分钟
            
        -   值：1
            

### **额度管理**

1000 credits = $0.001

-   **CHECK_BALANCE**
    
    -   描述：启用额度管理（OpenAI 和 Plugins 通道）
        
    -   值：false
    
-   **START_BALANCE**
    
    -   描述：用户初始额度
        
    -   值：20000
        

## 登录配置

### 基本配置

-   **ALLOW_EMAIL_LOGIN**
    
    -   描述：开启邮箱登录
        
    -   值：true
    
-   **ALLOW_REGISTRATION**
    
    -   描述：开启邮箱注册
        
    -   值：true
    
-   **ALLOW_SOCIAL_LOGIN**
    -   描述：开启第三方认证登录
        
    -   值：false
    
-   **ALLOW_SOCIAL_REGISTRATION**
    -   描述：开启第三方认证注册
        
    -   值：false
    
-   **ALLOW_PASSWORD_RESET**
    
    -   描述：允许用户重置密码
        
    -   值：false
    
-   **ALLOW_ACCOUNT_DELETION**
    
    -   描述：允许用户注销账户，默认允许
        
    -   值：true
    
-   **ALLOW_UNVERIFIED_EMAIL_LOGIN**
    
    -   描述：开启邮箱验证
        
    -   值：false
        

### 安全凭据配置

-   **SESSION_EXPIRY**
    
    -   描述：会话过期时间（免登陆间隔）
        
    -   值：1000\*60\*15（即15分钟）
    
-   **REFRESH_TOKEN_EXPIRY**
    
    -   描述：刷新令牌过期时间
        
    -   值：(1000\*60\*60\*24)\*7（即7天）
    
-   **CREDS_KEY**
    
    -   描述：用于安全存储凭证的32字节（以16进制表示为64字符）的密钥（必填）
        
    -   值：f34be427ebb29de8d88c107a71546019685ed8b241d8f2ed00c3df97ad2566f0
    
-   **CREDS_IV**
    
    -   描述：用于安全存储凭证的16字节（以16进制表示为32字符）的初始化向量（必填）
        
    -   值：e2341419ec3dd3d19b13a
        

-   **JWT_SECRET**
    
    -   描述：JWT 密钥，可通过 [**JWT Keys**](https://www.librechat.ai/toolkit/creds_generator) 快速生成
        
    -   值：16f8c0ef4a5d391b26034086c628469d3f9f497f08163ab9b40137092f2909ef
    
-   **JWT_REFRESH_SECRET**
    
    -   描述：JWT 刷新密钥，可通过 [**JWT Keys**](https://www.librechat.ai/toolkit/creds_generator) 快速生成
        
    -   值：eaa5191f2914e30b9387fd84e254e4ba6fc51b4654968
        

### 第三方登录配置

#### Apple 登录

- **APPLE_CLIENT_ID**
  - 描述：Apple 认证的客户端ID
  - 值：com.yourdomain.librechat.services
- **APPLE_TEAM_ID**
  - 描述：Apple 开发者团队ID
  - 值：YOUR_TEAM_ID
- **APPLE_KEY_ID**
  - 描述：密钥的 Apple Key ID
  - 值：YOUR_KEY_ID
- **APPLE_PRIVATE_KEY_PATH**
  - 描述： .p8 文件的绝对路径
  - 值：/path/to/AuthKey.p8
- **APPLE_CALLBACK_URL**
  - 描述：Apple 认证的回调 URL
  - 值：/oauth/apple/callback

#### **Discord 登录**

-   **DISCORD_CLIENT_ID**
    
    -   描述：Discord 认证的客户端ID
        
    -   值：YOUR_DISCORD_CLIENT_ID
    
-   **DISCORD_CLIENT_SECRET**
    
    -   描述：Discord 认证的客户端密钥
        
    -   值：YOUR_DISCORD_CLIENT_SECRET
    
-   **DISCORD_CALLBACK_URL**
    
    -   描述：Discord 认证回调 URL
        
    -   值：/oauth/discord/callback
        

#### **Facebook 登录**

-   **FACEBOOK_CLIENT_ID**
    
    -   描述：Facebook 认证的客户端ID
        
    -   值：YOUR_FACEBOOK_CLIENT_ID
    
-   **FACEBOOK_CLIENT_SECRET**
    
    -   描述：Facebook 认证的客户端密钥
        
    -   值：YOUR_FACEBOOK_CLIENT_SECRET
    
-   **FACEBOOK_CALLBACK_URL**
    
    -   描述：Facebook 回调地址
        
    -   值：/oauth/facebook/callback
        

#### **GitHub 登录**

-   **GITHUB_CLIENT_ID**
    
    -   描述：GitHub 认证的客户端ID
        
    -   值：YOUR_GITHUB_CLIENT_ID
    
-   **GITHUB_CLIENT_SECRET**
    
    -   描述：GitHub 认证的客户端密钥
        
    -   值：YOUR_GITHUB_CLIENT_SECRET
    
-   **GITHUB_CALLBACK_URL**
    
    -   描述：GitHub 回调地址
        
    -   值：/oauth/github/callback

-   **GITHUB_ENTERPRISE_BASE_URL**
    -   描述：GitHub 企业服务地址
    -   值：`https://github.mycompany.com`
-   **GITHUB_ENTERPRISE_USER_AGENT**
    -   描述：GitHub 企业 User-Agent 信息
    -   值：MyAppName/1.0

#### **Google 登录**

-   **GOOGLE_CLIENT_ID**

    -   描述： Google 认证的客户端ID
        
    -   值：YOUR_GOOGLE_CLIENT_ID
-   **GOOGLE_CLIENT_SECRET**

    -   描述： Google 认证的客户端密钥
        
    -   值：YOUR_GOOGLE_CLIENT_SECRET
-   **GOOGLE_CALLBACK_URL**

    -   描述：Google 回调地址
        
    -   值：/oauth/google/callback
        

#### **OpenID Connect**

-   **OPENID_CLIENT_ID**
    
    -   描述： OpenID 认证的客户端ID
        
    -   值：YOUR_OPENID_CLIENT_ID
    
-   **OPENID_CLIENT_SECRET**
    
    -   描述： OpenID 认证的客户端密钥
        
    -   值：YOUR_OPENID_CLIENT_SECRET
    
-   **OPENID_ISSUER**
    
    -   描述：OpenID 发布者地址
        
    -   值：YOUR_OPENID_ISSUER_URL
    
-   **OPENID_SESSION_SECRET**
    
    -   描述：OpenID 会话密钥
        
    -   值：YOUR_OPENID_SESSION_SECRET
    
-   **OPENID_SCOPE**
    
    -   描述：OpenID 授权范围
        
    -   值：openid profile email offline_access
    
-   **OPENID_CALLBACK_URL**
    
    -   描述：OpenID 回调地址
        
    -   值：/oauth/openid/callback
    
-   **OPENID_REQUIRED_ROLE**
    
    -   描述：验证身份
        
    -   值：YOUR_REQUIRED_ROLE
    
-   **OPENID_REQUIRED_ROLE_TOKEN_KIND**
    
    -   描述：验证身份令牌类型
        
    -   值：YOUR_ROLE_TOKEN_KIND
    
-   **OPENID_REQUIRED_ROLE_PARAMETER_PATH**
    
    -   描述：验证身份的参数路径
        
    -   值：YOUR_REQUIRED_ROLE_PARAMETER_PATH
    
-   **OPENID_BUTTON_LABEL**
    
    -   描述：OpenID 登录按钮标签
        
    -   值：YOUR_BUTTON_LABEL
    
-   **OPENID_IMAGE_URL**
    
    -   描述：OpenID 登录按钮图片地址
        
    -   值：YOUR_IMAGE_URL

- **OPENID_USE_END_SESSION_ENDPOINT**
  - 描述：注销自动跳转
  - 值：true
- **OPENID_AUTO_REDIRECT**
  - 描述：登录自动跳转
  - 值：true

-   **OPENID_REUSE_TOKENS**

    -   描述：启用令牌复用，需要 offline_access 授权

    -   值：true
-   **OPENID_JWKS_URL_CACHE_ENABLED**

    -   描述：启用 JWKS 缓存令牌

    -   值：true
-   **OPENID_JWKS_URL_CACHE_TIME**

    -   描述：JWKS 缓存令牌过期时间，单位为毫秒

    -   值：600000
-   **OPENID_ON_BEHALF_FLOW_FOR_USERINFRO_REQUIRED**

    -   描述：启用 On-Behalf-Of

    -   值：true
-   **OPENID_ON_BEHALF_FLOW_USERINFRO_SCOPE**

    -   描述：On-Behalf-Of 授权范围

    -   值：user.read

#### LDAP/AD

LDAP配置参考 [**LDAP/AD Authentication**](https://www.librechat.ai/docs/configuration/authentication/ldap)

-   **LDAP_URL**
    
    -   描述：LDAP 服务器地址
        
    -   值：ldap://localhost:389
    
-   **LDAP_BIND_DN**
    
    -   描述：LDAP 用户名
        
    -   值：cn=root
    
-   **LDAP_BIND_CREDENTIALS**
    
    -   描述：LDAP 用户密码
        
    -   值：password
    
-   **LDAP_USER_SEARCH_BASE**
    
    -   描述：LDAP 搜索条目
        
    -   值：o=users,o=example.com
    
-   **LDAP_SEARCH_FILTER**
    
    -   描述：LDAP 搜索条件
        
    -   值：mail={{username}}
    
-   **LDAP_CA_CERT_PATH**
    
    -   描述：LDAP 证书路径
        
    -   值：/path/to/root_ca_cert.crt
    
-   **LDAP_TLS_REJECT_UNAUTHORIZED**
    
    -   描述：LDAP TLS 验证开关
        
    -   值：true

- **LDAP_STARTTLS**
  - 描述：开启 StartTLS
  - 值：true

### 邮箱配置

若启用公共邮箱，请勿配置个人邮箱参数

#### 基本配置

-   **EMAIL_USERNAME**
    
    -   描述：邮箱登录用户名（必填）
        
    -   值：`your-email@example.com`
    
-   **EMAIL_PASSWORD**
    
    -   描述：邮箱登录密码（必填）
        
    -   值：your-email-password
    
-   **EMAIL_FROM_NAME**
    
    -   描述：发件人名称
        
    -   值：LibreChat Support
    
-   **EMAIL_FROM**
    
    -   描述：发件人地址（必填）
        
    -   值：noreply@librechat.ai
        

#### 公共邮箱

-   **EMAIL_SERVICE**
    
    -   描述：邮件服务供应商（必填，例如：Gmail, Outlook）
        
    -   值：gmail
        

#### 个人邮箱

-   **EMAIL_HOST**
    
    -   描述：邮件服务器地址（必填）
        
    -   值：smtp.example.com
    
-   **EMAIL_PORT**
    
    -   描述：邮件服务器端口（必填）
        
    -   值：25
    
-   **EMAIL_ENCRYPTION**
    
    -   描述：邮件加密方法（如starttls, tls等）
        
    -   值：starttls
        

-   **EMAIL_ENCRYPTION_HOSTNAME**
    
    -   描述：加密服务器地址
        
    -   值：your.encryption.hostname
    
-   **EMAIL_ALLOW_SELFSIGNED**
    
    -   描述：允许自签名证书
        
    -   值：false
        

## 其他配置

### **Artifacts**

默认使用 CodeSandbox 公共 CDN，自部署配置参考：[self-host the bundler](https://sandpack.codesandbox.io/docs/guides/hosting-the-bundler)

- **SANDPACK_BUNDLER_URL**
  - 描述：自部署 sandpack bundler 地址
  - 值：your-bundler-url

### Firebase CDN

Firebase CDN 配置参考 [**Firebase CDN Configuration**](https://www.librechat.ai/docs/configuration/firebase)

注意：若启用 firebase 文件存储策略，务必修改 librechat.yaml 中的环境变量 file_strategy 为 firebase

-   **FIREBASE_AUTH_DOMAIN**
    
    -   描述：Firebase 服务的 Auth_DOMAIN
        
    -   值：your-project-auth-domain
    
-   **FIREBASE_PROJECT_ID**
    
    -   描述：Firebase 服务的项目ID
        
    -   值：your-project-id
    
-   **FIREBASE_APP_ID**
    
    -   描述 : Firebase 服务的 App ID
        
    -   值 : your-app-id
    
-   **FIREBASE_API_KEY**
    
    -   描述：Firebase 服务的 API_KEY
        
    -   值：your-firebase-api-key
    
-   **FIREBASE_STORAGE_BUCKET**
    
    -   描述：Firebase 服务的存储桶
        
    -   值：your-storage-bucket
    
-   **FIREBASE_MESSAGING_SENDER_ID**
    
    -   描述：Firebase Cloud 消息发送者ID
        
    -   值：your-messaging-sender-id
        

### **Meilisearch**

-   **SEARCH**

    -   描述：启用搜索功能
        
    -   值：true
        

    注意：非 Docker 部署需要单独安装 Meilisearch

-   **MEILI_NO_ANALYTICS**

    -   描述：禁用MeiliSearch的匿名遥测分析以保护隐私
        
    -   值：true

-   **MEILI_HOST**

    -   描述：Meilisearch 搜索服务器地址，如果使用 docker compose 部署 MeiliSearch，将 0.0.0.0 替换为 meilisearch
        
    -   值：`http://0.0.0.0:7700`

-   **MEILI_MASTER_KEY**

    -   描述：MeiliSearch 主密钥，必须至少为16字节且包含有效的UTF-8字符，docker compose 部署 MeiliSearch 会自动生成主密钥
        
    -   值：DrhYf7zENyR6AlUCKmnz0eYASOQdl6zxH7s7MKFSfFCt
        

-   **MEILI_NO_SYNC**

    -   描述：禁用 MeiliSearch 索引同步
    -   值：true


### Google Analytics

Google Analytics 配置参考 [在跟踪代码管理器中设置 Google Analytics](https://support.google.com/tagmanager/answer/9442095?sjid=10155093630524971297-EU)

-   **ANALYTICS_GTM_ID**
    
    -   描述：启用 Google Tag Manager 分析
        
    -   值：GTM-XXXXXXX
        

## 参考链接

-   [LibreChat Env](https://www.librechat.ai/docs/configuration/dotenv)