---
layout: mypost
title: HTTrack 带 cookies 爬站
date: 2025-05-27
published: true
category: 逆向
tags: 
 - httrack
---

以 Edge 浏览器为例，[点击安装](https://microsoftedge.microsoft.com/addons/detail/cookieeditor/neaplmfkghagebokkhpjpoebhdledlfi) Cookie-Editor 插件，前往目标站点登录后，点击插件图标 → **Export** → **Netscape** 复制 Cookie

打开记事本，将复制到的内容粘贴保存为 `cookies.txt`，其格式应该类似于

```
# HTTrack Website Copier Cookie File
# This file format is compatible with Netscape cookies
www.httrack.com	TRUE	/	FALSE	1999999999	foo	bar
www.example.com	TRUE	/folder	FALSE	1999999999	JSESSIONID	xxx1234
www.example.com	TRUE	/hello	FALSE	1999999999	JSESSIONID	yyy1234
```

[点击这里](https://download.httrack.com/httrack-3.49.2.exe) 下载 HTTrack，截止博客更新时，最新版本 v3.49.2，下载后双击运行安装

打开 HTTrack，点击 **Next**，输入 **Project name** 后点击 **Next**，在 **Web Addresses: (URL)** 输入目标站点地址，点击 **Set options** → **Spider**，确保勾选 **Accept cookies**，另外点击 **Limits** 可调节爬取深度，点击 **Scan Rules** 可调节爬取内容，修改后点击 **确定** 保存

这时先将创建的 `cookies.txt` 文件复制到 `C:\My Websites\Project name` 下，然后点击 **Next** → **Complete** 开始爬取站点

爬取好的站点位于 `C:\My Websites\Project name\URL`

