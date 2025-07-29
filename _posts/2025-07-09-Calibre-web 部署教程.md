---
layout: mypost
title: Calibre-Web 部署教程
date: 2025-07-09
published: true
categories: 自部署
tags: 
 - linux
---

## 一、安装 Calibre-Web

### 1. 切换至 root 用户

```bash
sudo -i
```

---

### 2. 安装依赖

```bash
apt install git imagemagick python3-pip python3-venv
```

---

### 3. 下载源码

```bash
cd /opt
git clone https://github.com/janeczku/calibre-web
cd calibre-web
```

---

### 4. 创建虚拟环境并安装依赖

```bash
python3 -m venv calibre
./calibre/bin/python3 -m pip install -r requirements.txt
```

---

### 5. 测试启动服务

```bash
./calibre/bin/python3 cps.py
```
> 在浏览器访问： [http://localhost:8083](http://localhost:8083)

---

### 6. 创建专用用户并修改权限

```bash
sudo useradd --system --no-create-home --shell /bin/false calibre-web
sudo chown -R calibre-web:calibre-web /opt/calibre-web
```

---

### 7. 配置 Systemd 服务

编辑 `/etc/systemd/system/calibre-web.service` 文件并填写：

```ini
[Unit]
Description=Calibre-Web
After=network.target

[Service]
Type=simple
User=calibre-web
Group=calibre-web
WorkingDirectory=/opt/calibre-web
ExecStart=/opt/calibre-web/calibre/bin/python3 /opt/calibre-web/cps.py
Restart=always
RestartSec=10
Environment=PYTHONPATH=/opt/calibre-web

NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/opt/calibre-web

[Install]
WantedBy=multi-user.target
```

---

### 8. 管理 Calibre-Web 服务

```bash
sudo systemctl daemon-reload
sudo systemctl enable calibre-web
sudo systemctl start calibre-web
```
常用命令：
```bash
sudo systemctl status calibre-web     # 查看状态
sudo systemctl stop calibre-web       # 停止服务
sudo systemctl restart calibre-web    # 重启服务
sudo systemctl disable calibre-web    # 取消开机自启
sudo journalctl -u calibre-web -f     # 实时日志
```

---

## 二、安装和配置 Caddy

### 1. 安装 Caddy

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

---

### 2. 配置反向代理

编辑 `/etc/caddy/Caddyfile`，写入：

```caddyfile
calibre-web.com {
    reverse_proxy localhost:8083
}
```

---

### 3. 格式化配置并检测应用 Caddy 配置

```bash
sudo caddy fmt --overwrite /etc/caddy/Caddyfile
sudo mkdir -p /var/log/caddy
sudo chown caddy:caddy /var/log/caddy
sudo caddy validate --config /etc/caddy/Caddyfile
sudo systemctl reload caddy
sudo systemctl status caddy
```

查看访问日志：
```bash
sudo journalctl -u caddy -f
sudo tail -f /var/log/caddy/calibre-web.log
```

---

## 三、验证部署结果

### 1. 端口监听状况

```bash
sudo netstat -tlnp | grep :443
sudo netstat -tlnp | grep :80
```

### 2. 域名解析与 HTTPS 测试

```bash
nslookup calibre-web.com
curl -I https://calibre-web.com
```

---

> 🎉 现在你可以通过 `https://calibre-web.com` 直接访问和使用你的 Calibre-Web 服务！

---
