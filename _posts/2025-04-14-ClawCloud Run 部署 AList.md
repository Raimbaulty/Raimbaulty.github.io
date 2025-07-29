---
layout: mypost
title: ClawCloud Run 部署 AList
date: 2025-04-14
published: true
categories: 自部署
tags: 
 - claw
 - alist
---

## 介绍

Clawcloud 现在注册赠送5刀，当月有效。如果你绑定了GitHub 账号并且该账号注册超过180天 ，之后每月还能继续拿到5刀的赠额。免费计划不需要绑定信用卡，但功能和使用会受到一些限制。

![pricing.png](image.png)


**注意**：图中显示的额度是使用上限，实际使用按使用量计费（Pay as you go）

| 资源类型 | 价格（按月计费） | 价格（按小时计费）  | 价格（按其他单位）      |
| -------- | ---------------- | ------------------- | ----------------------- |
| CPU      | $4/vCPU/月       | $0.005556/vCPU/小时 | -                       |
| RAM      | $2/GB/月         | $0.002778/GB/小时   | -                       |
| Storage  | $0.12/GB/月      | $0.000352/GB/小时   | -                       |
| Network  | $0.05/GB         | -                   | $0.00005/MB 或 $0.05/GB |

## 准备
- 使用 Cloudflare 创建隧道，记录下隧道Token
- 使用注册时间超过 180 天的 github 账号登录 [clawcloud run](https://console.run.claw.cloud/)

## 开始
### 创建 AList 应用

点击 App Store，点选 AList，USERNAME 选择 admin，PASSWORD 输入你的密码，然后点击右上角 Deploy App 并 Confirm，等待创建完成后点击 Details 接着点击 Update，取消勾选 Enable Internet Access，并点击 /opt/alist/data 将容量增加到 10G，接着右上角点击 Update 然后点击 Yes 完成创建

**应用卸载**：App Store 点击 My Apps 标签进入需要卸载的项目点击右上角 Uninstall

### 创建 Cloudflared 隧道

- 点击 AppLauchpad - Create App
- Image Name 填写 cloudflard
- Image Name 填写 cloudflare/cloudflared:latest
- Usage 选择 0.1CPU 64M
- Network 勾选 Enable Internet Access
- 点击 custom domain，复制 CNAME 值并关闭小窗
- 在 cloudflare 添加一条名称 alist 的 CNAME 记录指向该值
- 重新点击 custom domain 填写 Custom Domain 为 `alist.example.com`
- Command 填写 `cloudflared tunnel --no-autoupdate run --token 你的token`
- 右上角点击 Deploy Application 然后点击 Yes 完成创建

最后，还需要配置 cloudflare 隧道的公共主机名，URL填写 alist 的内网地址，配置示例如下

![URL](clipboard.png)

## 参考链接

- [ClawCloud 免费容器 FREE ](https://linux.do/t/topic/555313)
- [clawcloud run 流量少的解决办法可能是？ ](https://linux.do/t/topic/556826)