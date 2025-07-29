---
layout: mypost
title: SuperAssistant 使用教程
date: 2025-06-08
published: true
categories: 效率工具
tags: 
 - mcp
 - windows
---

# MCP SuperAssistant 使用教程

MCP SuperAssistant 是一款浏览器插件，让你在网页对话时也可调用 MCP，支持的平台

- [ChatGPT](https://chatgpt.com/)
- [Google Gemini](https://gemini.google.com/)
- [Perplexity](https://perplexity.ai/)
- [Grok](https://grok.com/)
- [Google AI Studio](https://aistudio.google.com/)
- [OpenRouter Chat](https://openrouter.ai/chat)
- [DeepSeek](https://chat.deepseek.com/)
- [Kagi](https://kagi.com/assistant)
- [T3 Chat](https://t3.chat/)

使用前需要先部署 MCP SSE Server

## 本地部署

在工作目录下创建 `mcp_config.json`，格式形如

```json
{
  "mcpServers": {
    "edgeone-pages-mcp-server": {
      "command": "npx",
      "args": ["edgeone-pages-mcp"]
    }    
  }
}
```

然后执行

```bash
npx @srbhptl39/mcp-superassistant-proxy@latest --config ./mcp_config.json
```



## 抱脸部署

[点击这里](https://huggingface.co/new-space?sdk=docker) 创建空间

- Dockerfile

  ```dockerfile
  FROM node:20-alpine
  
  RUN apk add --no-cache \
      python3 \
      py3-pip \
      curl \
      bash \
      jq
  
  RUN npm install -g @srbhptl39/mcp-superassistant-proxy@latest
  
  USER node
  
  WORKDIR /home/node
  
  COPY --chown=node:node entrypoint.sh /home/node/entrypoint.sh
  RUN chmod +x /home/node/entrypoint.sh
  
  EXPOSE 7860
  
  CMD ["/home/node/entrypoint.sh"]
  ```

- entrypoint.sh

  ```bash
  #!/bin/bash
  
  cat <<EOF > /home/node/config.json
  {
    "mcpServers": {
      "edgeone-pages-mcp-server": {
        "command": "npx",
        "args": ["edgeone-pages-mcp"]
      }    
    }
  }
  EOF
  
  exec mcp-superassistant-proxy --config /home/node/config.json --host 0.0.0.0 --port 7860 --cors true
  ```

部署后，安装拓展：[前往安装](https://chromewebstore.google.com/detail/kngiafgkdnlkgmefdafaibkibegkcaef)，启用拓展，刷新网页对话的页面，点击新的 MCP 按钮，勾选所有选项

然后展开右侧状态栏，点击⚙️修改 MCP SSE Server 地址，本地部署无需修改，抱脸部署需要修改为 `https://用户名-空间名.hf.space/sse`，修改后点击 **Save & Reconnect** 重新连接

再次点击 MCP 按钮，点击 **Insert** 就会将 MCP 服务的信息以提示词的方式插入到输入框，直接在末尾输入问题即可通过提示词调用MCP



## 参考链接

- [MCP Servers using ChatGPT](https://medium.com/data-science-in-your-pocket/mcp-servers-using-chatgpt-cd8455e6cbe1)