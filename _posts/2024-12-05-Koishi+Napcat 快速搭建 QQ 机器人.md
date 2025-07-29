---
layout: post
title: Koishi+Napcat 快速搭建 QQ 机器人
date: 2024-12-05
published: true
categories: 机器人
tags: 
 - bot
---

## 搭建框架

**框架：**`koishi`

**协议：**`napcat`

## 安装篇

### docker编排

```yaml
x-defaults: &common-settings
  restart: always
  environment:
    TZ: "Asia/Shanghai"

services:
  redis:
    <<: *common-settings
    image: redis/redis-stack:latest
    container_name: redis
    volumes:
      - ./redis:/data    
    networks:
      onebot:
        ipv4_address: 192.168.100.2

  napcat:
    <<: *common-settings
    image: mlikiowa/napcat-docker:latest
    container_name: napcat
    mac_address: 16:4A:D2:89:B7:5E
    ports:
      - 6099:6099
    environment:
      - WEBUI_TOKEN=网页登录token
      - NAPCAT_GID=1000
      - NAPCAT_UID=1000
    volumes:
      - "./napcat/QQ:/app/.config/QQ"
      - "./napcat/config:/app/napcat/config"
    networks: 
      onebot:
        ipv4_address: 192.168.100.3

  koishi:
    <<: *common-settings
    image: koishijs/koishi:latest
    container_name: koishi
    ports:
      - 5140:5140
    depends_on:
      - redis
      - napcat
    volumes:
      - ./koishi:/koishi
    networks:
      onebot:
        ipv4_address: 192.168.100.4

networks:
  onebot:
    name: onebot
    ipam:
      config:
        - subnet: 192.168.100.0/24
```

### 网络配置

网页配置和文件配置二选一，正向和反向ws也是二选一

1.  **网页配置**
    

**1.1 正向ws**

启动容器后，访问 **https://ip:6099/webui** 使用 **网页登录token** 登录 napcat面板，左侧点击 \[网络配置\]-\[新建\]- \[**Web Socket 服务器\]**，点击启用，名称自定义，Token 注意与后续 **onebot** 插件配置的 **token** 一致，其他配置保持默认即可

![](屏幕截图 2025-02-18 170746.png)

**1.2 反向ws**

启动容器后，访问 **https://ip:6099/webui** 使用 **网页登录token** 登录 napcat面板，左侧点击 \[网络配置\]-\[新建\]- \[**Web Socket 客户端\]**，点击启用，名称自定义，URL 填入 **ws://koishi:5140/onebot**，Token 注意与后续 **onebot** 插件配置的 **token** 一致，其他配置保持默认即可

![](屏幕截图 2025-02-18 170215.png)

2.  **文件配置**
    
    **v4.5.3** 后支持载入 ./config/onebot11.json 作为默认配置
    
    ```bash
    mkdir -p ./napcat/config
    vi onebot11.json
    ```
    

根据类型选择，复制修改代码

​	**2.1 正向ws**

```json
{
  "network": {
    "httpServers": [],
    "httpClients": [],
    // WS服务端组/正向WS  可以配置多个 这里演示为一个
    "websocketServers": [
      {
        "name": "WsServer",// 名字不能重复 唯一标识
        "enable": true,//启用状态
        "host": "0.0.0.0",// 监听主机
        "port": 3001,// 监听端口
        "messagePostFormat": "array",// 消息上报格式 string/array
        "reportSelfMessage": false,// 是否上报自身消息
        "token": "",// 鉴权密钥
        "enableForcePushEvent": true,// 暂时没有作用
        "debug": false,// raw数据上报
        "heartInterval": 30000,// 心跳周期
      }
    ],
    "websocketClients": []
  },
  "musicSignUrl": "",
  "enableLocalFile2Url": false,
  "parseMultMsg": false
}
```

​	**2.2 反向ws**

```json
{
  "network": {
    "httpServers": [],
    "httpClients": [],
    "websocketServers": [],
    // WS客户端组/反向WS 可以配置多个 这里演示为一个
    "websocketClients": [
      {
        "name": "WsClient",// 名字不能重复 唯一标识
        "enable": true,//启用状态
        "url": "ws://koishi:5140/onebot",// 上报地址
        "messagePostFormat": "array",// 消息上报格式 string/array
        "reportSelfMessage": false,// 是否上报自身消息
        "reconnectInterval": 5000,// 重连间隔
        "token": "",// 鉴权密钥
        "debug": false,// raw数据上报
        "heartInterval": 30000,// 心跳周期
      }
    ]
  },
  "musicSignUrl": "",
  "enableLocalFile2Url": false,
  "parseMultMsg": false
}
```

## 插件篇

访问 **https://ip:5140** 访问 koishi面板，首先检查更新插件依赖，点击左侧 **依赖管理**，点击 **刷新** 按钮，再点击 **小火箭**，最后点击 **对钩** 执行

安装插件的方法主要有三种

-   **方法一**
    

点击左侧 **插件市场**，在输入框输入或者点击左侧分类查找插件，点击添加即可安装，安装好等待重启后回到 **插件市场** 找到安装的插件点击 修改 进行配置并启动插件

-   **方法二**
    

点击左侧 **依赖管理**，点击 + 手动添加，输入插件仓库地址进行添加，然后点击 对钩 符号执行安装。安装好等待重启后回到 **依赖管理** 找到安装的插件点击 修改 进行配置并启动插件

-   **方法三**
    

进入 koishi 安装目录手动克隆仓库安装

安装好插件后，可在左侧 **沙盒** 进行调试，下面介绍几款系统默认安装插件的配置

### auth 插件

点击左侧 **插件配置**，在搜索框搜索 **auth**，启用管理员账号，修改管理员用户名和密码，然后先点击右上角保存键再点击播放键运行，运行成功后会刷新页面，点击左下角的登录入口进行登录即可

![](msedge_UoAnGNASlw.png)

### **market 插件**

xxxxxxxxxx5 1api.duplicate_space(2    from_id="username/my-cool-training-space",3    secrets=[{"key"="HF_TOKEN", "value"="hf_api_***"}, ...],4    variables=[{"key"="MODEL_REPO_ID", "value"="user/repo"}, ...],5)bash

![](msedge_OVQ7vXkMW7.png)

### qq 适配器

点击左侧 **插件配置**，找到 **adapter-qq** 点击进行配置，前往 [**QQ 开放平台**](https://q.qq.com/) 注册后登陆 [**机器人管理后台**](https://q.qq.com/#/app/bot) 创建机器人后，在「开发设置」界面获取机器人的 id 等参数，注意打开 **USERS\_MESSAGE** 和 **AUDIO\_ACTION**

**注意**：QQ 已于2024年年底取消支持 websocket，因此 **protocol** 现在只能选择 webhook，并还需在 [**webhook配置页**](https://q.qq.com/qqbot/#/developer/webhook-setting) 修改接收的消息类型并添加回调地址（服务器地址后跟 `/qq`(如 `http://1.1.1.1/qq` 或 `https://example.com/qq`)

![](%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202024-12-05%20141251.png)

## 接入篇

### 接入onebot

在插件市场搜索安装 **adapter-onebot** 并进入配置，**selfId** 填写QQ号，**token** 与 网络配置中填写的要一致

1.  **正向ws配置**
    

![](屏幕截图 2025-02-18 174707.png)

2.  **反向ws配置**
    
    ![](msedge_9QRZX9CY4n.png)
    

### **接入wecom**

1.  点击左侧 **插件配置**，找到 **adapter-wecome** 并进入配置
    
2.  前往 [**我的企业**](https://work.weixin.qq.com/wework_admin/frame#profile)，复制页面下方企业 ID，填入插件的 corpId。
    
3.  前往 [**应用管理**](https://work.weixin.qq.com/wework_admin/frame#apps) 页面下方点击创建应用，填写应用信息。
    
4.  复制 `AgentId` 填入插件的 agentId 字段，查看 Secret 填入插件的 `secret` 字段。
    
5.  在下方功能板块点击「设置 API 接收」，URL 填写服务器地址后跟 `/wecom`(如 `http://1.1.1.1/wecom` 或 `https://example.com/wecom`)，Token 和 EncodingAESKey 点击随机获取，分别填入插件的 `token` 和 `aesKey` 字段。先启用插件，再在「API 接收消息」页面点击保存.
    
6.  于页面左上角返回应用页面，在下方开发者接口板块点击「企业可信 IP」的「配置」，设置白名单 IP，确定后稍等几分钟即可使用插件。
    
7.  推荐在几分钟后停用并启用一次插件，以便加载出平台信息呈现在控制台内。
    

### 接入微信公众号

1.  点击左侧 **插件配置**，找到 **adapter-wechat-official** 并进入配置
    
2.  根据 [**注册流程指引**](https://kf.qq.com/product/weixinmp.html#hid=87) 注册公众平台。
    
3.  在微信公众平台登录后，页面左侧展开「设置与开发」，进入「账号设置」，翻至页面底部，复制 `原始 ID` 填入插件的 account 字段。
    
4.  页面左侧进入「设置与开发\]-\[安全中心\] 设置白名单 IP。
    
5.  页面左侧进入「设置与开发」- \[开发接口管理\] -「基本配置」，复制 `开发者ID` 填入插件的 appId 字段，点击重置获取开发者密码填入插件的 secret 字段。
    
6.  页面下方修改服务器配置，URL 填写服务器地址后跟 `/wechat-official` (如 `http://1.1.1.1/wechat-official` 或 `https://example.com/wechat-official`)；在插件配置和公众平台上填入相同的 Token；在公众平台上生成 EncodingAESKey 并填入插件的 aesKey 字段；三种消息加解密方式均可选择。然后修改插件的配置与微信公众号平台一致后启动插件，再点击保存方可成功保存，保存后启用服务器配置。
    
7.  如果公众号为企业主体，且通过了微信认证，可在插件配置中启用 customerService。客服接口提供了更宽松的消息回复能力。
    

## 开发篇

在开始开发之前，需要先搭建好 NodeJS 环境，要求版本不低于 **v18**，下载地址：[**点击前往**](https://nodejs.org/zh-cn/download/package-manager)

环境搭建好后，进入koishi的安装目录，对于本教程是 **koishi** 目录

```bash
cd koishi
```

先后输入 plugin name（插件名）和 description（描述）来创建新插件，这里假设插件名为 example，请根据实际名称修改，打开源文件

```bash
vi ./plugin/example/src/index.ts
```

现在，这个文件中默认给出了一些示例代码，你可以在这里开始编写你的插件

```ts
// 导入插件上下文和配置结构
import { Context, Schema } from 'koishi'

// 定义插件名称和配置项
export const name = 'example'

export interface Config {}

export const Config: Schema<Config> = Schema.object({})

export function apply(ctx: Context) {
  // write your plugin here
}
```

## 参考链接

-   [**NapNeko/NapCat-Docker**](https://github.com/NapNeko/NapCat-Docker)
    
-   [**NapCatQQ 基础配置**](https://napneko.pages.dev/config/basic)
    
-   [**QQ/QQ 频道 登录指南**](https://forum.koishi.xyz/t/topic/810)
    
-   [**自建QQ机器人**](https://blog.xzh.gs/archives/qq-bot)
    
-   [**快速部署一个发布memos的QQ机器人**](https://www.avnvu.com/posts/1669/)
    
-   [**Koishi 插件开发入门教程(一)**](https://forum.koishi.xyz/t/topic/425)