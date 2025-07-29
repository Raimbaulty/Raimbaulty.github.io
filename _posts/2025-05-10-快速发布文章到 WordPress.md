---
layout: mypost
title:  快速发布文章到 WordPress
date: 2025-05-10
published: true
category: 效率工具
tags: 
 - wordprss
---

Push Markdown最新版本发布地址：[点击前往](http://download.szx.life/push-markdown)，下载后双击运行安装

打开  Push Markdown，点击 **文件** → **设置**，填写站点信息，URL 填写示例：`https://example.com/xmlrpc.php`

Push Markdown 要求文章页面开头包含 **YAML Frontmatter**，用于定义页面的元数据，以本文的元数据为例：

```yaml
---
layout: mypost
title:  快速发布文章到 Wordpress
abstract: 解放双手，修改即发布
url: pushmarkdown

date: 2025-05-10


category: 
- 效率工具
tags: 
- wordprss
thumbnail: https://gitee.com/raimbalt/picgo/raw/master/img/20250510183143.jpg
---
```

打开需要发布的文章，快捷键 `Ctrl+P` ，保持默认设置直接点击 **发布** 即可发布到 WordPress 站点

## 参考链接

- [Typora+PicGo+Lsky+push-markdown实现md向WordPress一键上传](https://blog.csdn.net/Jeff_12138/article/details/126756696)