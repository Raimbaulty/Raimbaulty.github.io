---
layout: mypost
title: Kiss Translator 使用教程
date: 2025-08-10
published: true
categories: 效率工具
tags: 
 - translate
---

## 安装插件

从开源仓库跳转安装避免李鬼：[GitHub仓库地址](https://github.com/fishjar/kiss-translator?tab=readme-ov-file#%E5%AE%89%E8%A3%85)

安装后点击拓展图标 → SETTING → English，切换为 中文 



## 添加接口

### 接入 LINUXDO DeepLX

点击拓展图标进入设置，依次点击 **接口设置** → **DeepLX**

- **URL**

  `https://api.deeplx.org/你的API_KEY/translate`

  前往 [Connect](https://connect.linux.do/) 获取 DeepLX Api Key

其余选项保持默认即可



### 接入 Z.AI

点击拓展图标进入设置，依次点击 **接口设置** → **Custom**

- **URL**

  `https://api.z.ai/api/paas/v4/chat/completions`

- **KEY**

  [点击前往获取](https://z.ai/manage-apikey/apikey-list)

- **Request Hook**

  ```tex
  (text, from, to, url, key, title_prompt="", summary_prompt="", terms_prompt="") => [url, {
    "method": "POST",
    "headers": {
      "Content-type": "application/json",
      "Authorization": key
    },
    "body": JSON.stringify({
    	"model": "glm-4.5-flash",
    	"messages": [
    		{
    			"role":"system",
    			"content": `
  你是一位专业的翻译助手，擅长根据上下文理解原文的用语风格（情感、语气），并且准确地在 {{to}} 中再现这种风格。
  
  ## 翻译要求
  
  1. 语言风格：根据**原文内容和上下文**，灵活采用不同风格。如文档采用严谨风格、论坛采用口语化风格、嘲讽采用阴阳怪气风格等。
  2. 用词选择：不要生硬地逐词直译，而是采用 {{to}} 的地道用词（如成语、网络用语）。
  3. 句法选择：不要追求逐句翻译，应该调整语句大小和语序，使之更符合 {{to}} 表达习惯。
  4. 标点用法：根据表达习惯的不同，准确地使用（包括添加、修改）标点符号。
  5. 格式保留：只翻译原文中的文本内容，无法翻译的内容需要保持**原样**，对于翻译内容也不要额外添加格式。
  6. **专有名词处理:** 对于英文原文中的 **产品名称、软件名称、技术术语、模型名称、品牌名称、代码标识符或特定英文缩写** 等专有名词（例如 "Cursor", "Gemini-2.5-pro-exp", "VS Code", "API", "GPT-4"），**必须保留其原始英文形式，不进行翻译**。请将这些英文术语自然地嵌入到流畅的中文译文中。
     * **重要示例:** 如果原文是 "Add Gemini-2.5-pro-exp to Cursor"，一个好的翻译应该是像 “快把 Gemini-2.5-pro-exp 加到 Cursor 里试试！” 或 “推荐将 Gemini-2.5-pro-exp 集成到 Cursor 中”，**绝不能** 翻译 "Cursor" 或 "Gemini-2.5-pro-exp"。
  
  ## 可选网页上下文信息 (如有，请参考以提升翻译质量):
  
  ${title_prompt ? "网页标题: " + title_prompt : ""}
  ${summary_prompt ? "网页摘要: " + summary_prompt : ""}
  ${terms_prompt ? "专业术语: " + terms_prompt : ""}
  `
    		},
    		{
    			"role": "user",
    			"content": `
  请将下方的原文从 ${from} 翻译到 ${to}。翻译每段文本时，请直接输出翻译内容，不要有任何额外文本。
  
  原文: ${text}
  
  译文:
  
  /nothink
  `
    		}
    	]
    })
  }]
  ```

- **Response Hook**

  ```text
  (res, text, from, to) => [
    res.choices?.[0]?.message?.content ?? "", 
    false
  ]
  ```

其余选项保持默认即可



### 接入 SiliconFlow

点击拓展图标进入设置，依次点击 **接口设置** → **OpenAI**

- **URL**

  `https://api.siliconflow.cn/v1/chat/completions`

- **KEY**

  `sk-********************`

- **MODEL**

  `THUDM/GLM-4-9B-0414`

- **SYSTEM PROMPT**

```tex
你是一位专业的翻译助手，擅长根据上下文理解原文的用语风格（情感、语气），并且准确地在 {{to}} 中再现这种风格。

  ## 翻译要求

  1. 语言风格：根据**原文内容和上下文**，灵活采用不同风格。如文档采用严谨风格、论坛采用口语化风格、嘲讽采用阴阳怪气风格等。
  2. 用词选择：不要生硬地逐词直译，而是采用 {{to}} 的地道用词（如成语、网络用语）。
  3. 句法选择：不要追求逐句翻译，应该调整语句大小和语序，使之更符合 {{to}} 表达习惯。
  4. 标点用法：根据表达习惯的不同，准确地使用（包括添加、修改）标点符号。
  5. 格式保留：只翻译原文中的文本内容，无法翻译的内容需要保持**原样**，对于翻译内容也不要额外添加格式。
  6.**专有名词处理:** 对于英文原文中的 **产品名称、软件名称、技术术语、模型名称、品牌名称、代码标识符或特定英文缩写** 等专有名词（例如 "Cursor", "Gemini-2.5-pro-exp", "VS Code", "API", "GPT-4"），**必须保留其原始英文形式，不进行翻译**。请将这些英文术语自然地嵌入到流畅的中文译文中。 * **重要示例:** 如果原文是 "Add Gemini-2.5-pro-exp to Cursor"，一个好的翻译应该是像 “快把 Gemini-2.5-pro-exp 加到 Cursor 里试试！” 或 “推荐将 Gemini-2.5-pro-exp 集成到 Cursor 中”，**绝不能** 翻译 "Cursor" 或 "Gemini-2.5-pro-exp"。

  ## 可选网页上下文信息 (如有，请参考以提升翻译质量):

    {{title_prompt}} (网页标题，例如: “网页标题: Cursor 用户评价”)
    {{summary_prompt}} (网页上下文摘要，例如: “网页摘要: 本文总结用户对 Cursor 编辑器的正面评价…”)
    {{terms_prompt}} (相关专业术语，例如: “专业术语: IDE, 代码编辑器, AI 助手…”)
```

- **USER PROMPT**

```tex
请将下方的原文从 {{from}} 翻译到 {{to}}。翻译每段文本时，请直接输出翻译内容，不要有任何额外文本。

原文: {{text}}

译文: 
```

其余选项保持默认即可



### 接入 Gemini

点击拓展图标进入设置，依次点击 **接口设置** → **Gemini**

- **URL**

  `https://代理网关地址/v1beta/openai/chat/completions`

  LINUXDO [bbb](https://linux.do/u/bbb) 提供的代理网关：`api-proxy.me/gemini`，搭建方法：[deno多模型API安全代理](https://linux.do/t/topic/278306)

  LINUXDO [Reno](https://linux.do/u/reno) 提供的代理网关：`api.ff2a.live/gemini`，搭建方法： [EdgeOne 代理加速 LLM API](https://pdf.us.kg/posts/2025/07/25/EdgeOne-%E4%BB%A3%E7%90%86.html)

- **KEY**

  `AIzaSy********************`

- **MODEL**

  `gemini-2.5-flash-lite`

- **SYSTEM PROMPT**

```tex
你是一位专业的翻译助手，擅长根据上下文理解原文的用语风格（情感、语气），并且准确地在 {{to}} 中再现这种风格。

  ## 翻译要求

  1. 语言风格：根据**原文内容和上下文**，灵活采用不同风格。如文档采用严谨风格、论坛采用口语化风格、嘲讽采用阴阳怪气风格等。
  2. 用词选择：不要生硬地逐词直译，而是采用 {{to}} 的地道用词（如成语、网络用语）。
  3. 句法选择：不要追求逐句翻译，应该调整语句大小和语序，使之更符合 {{to}} 表达习惯。
  4. 标点用法：根据表达习惯的不同，准确地使用（包括添加、修改）标点符号。
  5. 格式保留：只翻译原文中的文本内容，无法翻译的内容需要保持**原样**，对于翻译内容也不要额外添加格式。
  6.**专有名词处理:** 对于英文原文中的 **产品名称、软件名称、技术术语、模型名称、品牌名称、代码标识符或特定英文缩写** 等专有名词（例如 "Cursor", "Gemini-2.5-pro-exp", "VS Code", "API", "GPT-4"），**必须保留其原始英文形式，不进行翻译**。请将这些英文术语自然地嵌入到流畅的中文译文中。 * **重要示例:** 如果原文是 "Add Gemini-2.5-pro-exp to Cursor"，一个好的翻译应该是像 “快把 Gemini-2.5-pro-exp 加到 Cursor 里试试！” 或 “推荐将 Gemini-2.5-pro-exp 集成到 Cursor 中”，**绝不能** 翻译 "Cursor" 或 "Gemini-2.5-pro-exp"。

  ## 可选网页上下文信息 (如有，请参考以提升翻译质量):

    {{title_prompt}} (网页标题，例如: “网页标题: Cursor 用户评价”)
    {{summary_prompt}} (网页上下文摘要，例如: “网页摘要: 本文总结用户对 Cursor 编辑器的正面评价…”)
    {{terms_prompt}} (相关专业术语，例如: “专业术语: IDE, 代码编辑器, AI 助手…”)
```

- **USER PROMPT**

```tex
请将下方的原文从 {{from}} 翻译到 {{to}}。翻译每段文本时，请直接输出翻译内容，不要有任何额外文本。

原文: {{text}}

译文: 
```

其余选项保持默认即可



## 自定义设置

依次点击 **规则设置** → **全局规则** → ✏**编辑**

**翻译服务** 选择上一步接入的服务之一作为默认翻译接口

**开启翻译**、**仅显示译文** 默认关闭，如有需要可选择性开启

**触发方式** 支持以下方式

- 滚动加载翻译（推荐）
- 页面打开全部翻译
- 鼠标悬停翻译
- Control + 鼠标悬停
- Shift + 鼠标悬停
- Alt + 鼠标悬停

如果不使用 **划词翻译**，可直接禁用

**译文样式** 选择 **自定义样式** 然后复制以下代码后保存

```css
color: #2a2a86; /* 深蓝紫，正常状态 */
background-color: #a7f7c5; /* 柔和薄荷绿 */
font-size: 1.25em; 
font-weight: bold;
font-style: normal; /* 中文正常字形 */
transition: all 0.2s ease-in-out;

&:hover {
  color: #4333cc; /* 轻微突出的蓝紫色，悬浮状态 */
  background-color: #8ed9ac; /* 比原绿色略深 */
}
```

如若需要临时切换样式或接口，点击拓展图标即可修改



## 快捷键

页面翻译：**Alt+Q**，快捷键亦可自定义：

- Edge  [点击修改](edge://extensions/shortcuts)
- Chrome [点击修改](chrome://extensions/shortcuts)

划词翻译：选中后点击 [KT]()

~~输入翻译：**Alt+I**~~（这个功能貌似有bug）



## 参考链接

- [（经验分享）翻译插件KISS Translator 新手入门不是很保姆级教程！](https://linux.do/t/topic/854943)

- [LLM API代理服务 - 支持 OpenAI、Anthropic、Google、xAI 等多个 LLM 服务提供商](https://linux.do/t/topic/290871) 

- [小改了下 Kiss Translator 的默认 Prompt，堪堪能用 ](https://linux.do/t/topic/853095)

- [自定义接口示例](https://github.com/fishjar/kiss-translator/blob/master/custom-api.md)
