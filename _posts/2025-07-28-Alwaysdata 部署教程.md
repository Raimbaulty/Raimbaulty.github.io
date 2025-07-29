---
layout: mypost
title: Alwaysdata 部署教程
date: 2025-07-28
published: true
categories: 免费容器
tags: 
 - alwaysdata
 - linux
---

# Alwaysdata 部署教程

## 开始之前

[点击 ](https://www.alwaysdata.com/en/register/)注册 Alwaysdata

## GPT-Load

[点击](https://admin.alwaysdata.com/ssh/add/) 添加ssh用户，然后使用ssh连接

```
mkdir gpt-load && cd $_
wget wget https://github.com/tbphp/gpt-load/releases/latest/download/gpt-load
chmod +x gpt-load
```
[点击](https://admin.alwaysdata.com/site/add/) 添加站点

- **Commande（命令）**：`./gpt-load`

- **Type（类型）**: Programme utilisateur（用户程序）
- **Répertoire racine（根目录）**: `/home/username/gpt-load`

添加环境变量

```ini
PORT=8100
AUTH_KEY=sk-1234567 # 注意修改
```

提交后按照感叹号的提示添加A记录和AAAA记录，注意关闭小黄云，会自动生成证书



## TinyFileManager

### 安装

在需要管理文件的目录下载程式

```bash
cd dir
wget -O username.php https://raw.githubusercontent.com/prasathmani/tinyfilemanager/master/tinyfilemanager.php
```

### 修改密码

[点击](https://tinyfilemanager.github.io/docs/pwd.html) 进入官方站点输入要修改的密码生成hash，然后替换 `username.php` 中的 `$auth_users` 字段

```
$auth_users = array(
    'username' => 'REPLACE YOUR GENERATED PASSWORD HERE'
);
```

### 创建站点

- **Type（类型）**: PHP

- **Répertoire racine（根目录）**: `/home/username/dir`



## Filebrowser（失败）

使用ssh连接，下载符合版本的二进制文件

```bash
mkdir filebrowser && cd $_
wget https://github.com/filebrowser/filebrowser/releases/latest/download/linux-amd64-filebrowser.tar.gz
tar -xzvf linux-amd64-filebrowser.tar.gz
find . ! -name 'filebrowser' -type f -delete
chmod +x filebrowser
```

然后创建站点

- **Commande（命令）**：`./filebrowser -p 8100`

- **Type（类型）**: Programme utilisateur（用户程序）
- **Répertoire racine（根目录）**: `/home/username/filebrowser`



## Jekyll

### 安装

Alwaysdata 默认内置了 Ruby，直接安装 Jekyll

```bash
gem install jekyll bundler 
```

### 初始化

- 使用minima默认主题

```bash
jekyll new blog && cd $_
```

- 使用其他主题

​	在 [主题市场](https://www.bestjekyllthemes.com/) 找到自己喜欢的主题克隆到本地，这里以 Indigo 为例

```
git clone --depth=1 https://github.com/sergiokopplin/indigo.git blog && cd $_
```

### 创建文章

修改 `Gemfile`

```bash
sed -i '/group :jekyll_plugins do/a\
  gem "jekyll-compose"\
  gem "jekyll-optional-front-matter"\
  gem "jekyll-titles-from-headings"
' Gemfile
```

安装插件

```bash
bundle install
```

修改 ` _config.yml`

```bash
sed -i '/^plugins:/a\
  - jekyll-optional-front-matter\
  - jekyll-titles-from-headings
' _config.yml
```

新建文章

```bash
bundle exec jekyll post "你的文章标题"
```

编辑完成后重新构建

```bash
bundle exec jekyll build
```

### 构建

```bash
bundle install && bundle exec jekyll build
```

### 创建站点

将整个目录上传到 alwaysdata 后，修改 username.alwaysdata.net 站点

- **Type（类型）**: Fichiers statiques（静态文件）
- **Répertoire racine（根目录）**: `/home/username/blog/_site`