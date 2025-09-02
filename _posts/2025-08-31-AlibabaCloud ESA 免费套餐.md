---
layout: mypost
title: AlibabaCloud ESA 免费套餐
date: 2025-08-31
published: true
categories: 免费CDN
tags: 
 - cdn
---

## 开始之前

请自行准备

- 国外虚拟手机号，要求正常接码
- 国外虚拟银行卡，要求余额＞1刀

## 注册账户

[点击注册账户](https://account.alibabacloud.com/register/intl_register.htm)，按提示验证手机号

[点击添加银行卡](https://billing-cost.console.alibabacloud.com/fortune/payment/add)，有扣款验证，确保余额充足

## 添加站点

[点击添加站点](https://esa.console.aliyun.com/siteManage/add)，未备案域名区域只能选择 **全球(不包含中国内地)**，接入方式有 NS 和 CNAME 两种

选择 **Entrance**， **购买时长** 选择 1年，确认总费用0后点击完成并支付

在跳转页面点击 **支付**，支付完成后返回原页面点击 **我已完成**

接着按照提示完成站点验证即可，并且可以

## 续费套餐

[点击续费套餐](https://esa.console.aliyun.com/billManage/packageManagement)，点击 续费 → 设置续费规则进行续费，每次最长续费1年，可多次操作

## 加速站点

[访问站点列表](https://esa.console.aliyun.com/siteManage/list)，进入需要加速的站点，点击左侧 DNS → 记录 → 添加记录，支持添加 A/AAAA 和 CNAME 记录

默认开启了代理加速，注意只有 NS 接入才能关闭代理加速

在 DNS 记录的 **HTTPS 证书** 一列点击 **配置** 跳转到 SSL 配置点击 **确定**，等待几分钟即可

## 解锁内地

全球(不包含中国内地) 的代理加速效果差强人意，强烈建议解锁内地，但注意开通后还需使用备案域名

目前参加活动可免备案解锁内地加速，活动支持的社交平台

- Linux.do
- V2EX.com
- X.com
- LowEndTalk.com
- 您的个人博客
- 哔哩哔哩
- 即刻App
- 小红书
- facebook.com
- youtube.com

在以上任一社交平台发布一则推广信息，推广模板如下

```text
标题：【推广】推荐一款稳定好用的云加速工具 —— 阿里云 ESA

正文：
最近体验了一下阿里云 ESA（Edge Security Acceleration），整体感觉非常不错！
✅ 使用简单，上手很快
✅ 网络加速效果明显，跨境访问比之前顺畅很多
✅ 实测延迟低，稳定性佳

分享一个可以免费领取试用的官方链接：http://s.tb.cn/e6.0DENEf
如果你也想感受一下更快更稳的网络，可以去尝试一下～

（附上 ESA 控制台 / 测速截图）
```

[点击加入 Discord 群组](https://discord.gg/2XT2GB6KA6)，进入[中文频道](https://discord.com/channels/1200664824896569354/1385197590647279626)，私信管理员 `ESA Specialist`，私信模板如下

```text
管理员你好，我已经完成了 ESA 的推荐发帖，以下是帖子链接：
【在此粘贴你的帖子链接】
我的阿里云账号 ID 是：xxxxxxx
麻烦帮忙登记一下，申请解锁中国内地加速奖励，谢谢！
```

完成登记后即可解锁内地加速（全球地区）

1. **新增站点时**：在添加站点时选择**全球**区域加速，套餐选择Entrance版本接入

   ![image](p986094.png)
2. **已有站点时**：在控制台，选择[**站点管理**](https://esa.console.aliyun.com/siteManage/list)，在**站点**列单击目标站点。在**站点概况**右侧切换已有站点的加速区域为**全球**区域

   ![image](p986095.png)

## 参考链接

- [两种方式免费获得阿里云ESA套餐](https://www.alibabacloud.com/help/zh/edge-security-acceleration/esa/product-overview/how-to-get-esa-for-free?accounttraceid=bc073c2cdec14d42a54b693a04145496grar#8c1ca8383d3si)