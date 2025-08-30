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

第一步：[官网注册账户](https://www.alwaysdata.com/en/register/)

第二步：[创建SSH用户](https://admin.alwaysdata.com/ssh/add/)

第三步：SSH 登入

```bash
ssh 用户名@ssh-账户名.alwaysdata.net
```



## GPT-Load

### 下载文件

```
mkdir gpt-load && cd $_
bash <(curl -sSL https://raw.githubusercontent.com/Raimbaulty/freebsd/main/gpt-load.sh)
```
### 添加站点

[点击添加](https://admin.alwaysdata.com/site/add/) 

- **Commande（命令）**：`./gpt-load`

- **Type（类型）**: Programme utilisateur（用户程序）
- **Répertoire racine（根目录）**: `gpt-load`

环境变量填写

```ini
PORT=8100
AUTH_KEY=sk-1234567 # 注意修改
```

提交后按照感叹号的提示添加A记录和AAAA记录，注意关闭小黄云，会自动生成证书



## TinyFileManager

### 下载文件

```bash
mkdir file && cd $_
wget -O file.php https://raw.githubusercontent.com/prasathmani/tinyfilemanager/master/tinyfilemanager.php
```

### 修改密码

[点击访问](https://tinyfilemanager.github.io/docs/pwd.html) 输入密码生成hash，然后替换 `file.php` 中的 `$auth_users` 字段

```
$auth_users = array(
    'username' => 'REPLACE YOUR GENERATED PASSWORD HERE'
);
```

### 创建站点

- **Type（类型）**: PHP

- **Répertoire racine（根目录）**: `file`



## Filebrowser

### 隧道服务

```bash
mkdir service && cd $_
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64
chmod +x cloudflared-linux-amd64
```

[添加隧道服务](https://admin.alwaysdata.com/service/)

- Command

  `./cloudflared-linux-amd64 -- tunnel --edge-ip-version auto --protocol http2 --heartbeat-interval 10s run --token 你的token`

- Working directory

  `service`

最后点击 Submit 完成添加

### 文件服务

下载二进制文件

```bash
mkdir filebrowser && cd $_
wget https://github.com/filebrowser/filebrowser/releases/latest/download/linux-amd64-filebrowser.tar.gz
tar -xzvf linux-amd64-filebrowser.tar.gz
find . ! -name 'filebrowser' -type f -delete
chmod +x filebrowser
filebrowser users add <用户名> <密码> --perm.admin
```

[添加文件服务](https://admin.alwaysdata.com/service/)

- Command

  `./filebrowser`

- Working directory

  `filebrowser`

访问站点地址，点击 Settings → User Management，取消默认admin用户的权限并修改使用用户的语言选项



## Jekyll

### 安装本体

#### 默认主题

默认使用minima主题

```bash
gem install jekyll bundler
jekyll new blog && cd blog
bundle install
bundle exec jekyll build
```

#### 第三方主题

可在 [主题市场](https://www.bestjekyllthemes.com/) 寻找其他主题，这里以 Indigo 为例

```bash
gem install jekyll bundler
git clone --depth=1 https://github.com/sergiokopplin/indigo.git blog && cd blog
bundle install
bundle exec jekyll build
```

### 安装插件

修改 `Gemfile`

```bash
sed -i '/group :jekyll_plugins do/a\
  gem "jekyll-compose"\
  gem "jekyll-optional-front-matter"\
  gem "jekyll-titles-from-headings"
' Gemfile
```

修改 `_config.yml`

```bash
sed -i '/^plugins:/a\
  - jekyll-optional-front-matter\
  - jekyll-titles-from-headings
' _config.yml
```

安装插件

```bash
bundle install
```

新建文章

```bash
bundle exec jekyll post "你的文章标题"
```

编辑完成后重新构建

```bash
bundle exec jekyll build
```

### 创建站点

将整个目录上传到 alwaysdata 后，修改 username.alwaysdata.net 站点

- **Type（类型）**: Fichiers statiques（静态文件）
- **Répertoire racine（根目录）**: `blog/_site`