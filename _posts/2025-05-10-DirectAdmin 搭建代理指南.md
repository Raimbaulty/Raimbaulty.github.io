---
layout: mypost
title: DirectAdmin 使用指南
date: 2025-05-10
published: false
category: 网站搭建
tags:
 - directadmin
---

目前已知的支持 DirectAdmin 面板的免费服务商：

- [DataOnline](https://dataonline.vn)
- [FreeCloud](https://web.freecloud.ltd/)
- [WebHostMost](https://client.webhostmost.com)

## 准备工作

登录 DirectAdmin 面板后，点击 **账号管理** → **域名设置**，勾选并删除原域名后输入新域名完成创建

然后点击 **SSL 证书** → **从 ACME Provider 自动获取证书** → **保存** 申请证书

## 配置文件

使用 ssh 登录并切换到域名下 `public_html` 目录创建以下两个文件

```bash
cd domains/example.com/public_html
```

### package.json

```json
{
  "name": "node-ws",
  "version": "1.0.0",
  "description": "Node.js Server",
  "main": "app.js",
  "author": "",
  "repository": "",
  "license": "MIT",
  "private": false,
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "ws": "^8.14.2",
    "axios": "^1.6.2"
  },
  "engines": {
    "node": ">=14"
  }
}
```

### app.js

建议自定义路径 `/sub` 避免被扫

- 简单版


```js
const http = require('http');
const { Buffer } = require('buffer');
const net = require('net');
const { WebSocket, createWebSocketStream } = require('ws');
const UUID = process.env.UUID || '9e2b1a67-3e64-4abc-a661-c3e3b7f3da23';
const uuid = UUID.replace(/-/g, "");
const DOMAIN = process.env.DOMAIN || 'example.com';  //项目域名或已反代的域名，不带前缀，建议填已反代的域名
const NAME = process.env.NAME || 'freeshare';
const PORT = process.env.PORT || 33333;

// 创建HTTP路由
const httpServer = http.createServer((req, res) => {
  if (req.url === '/') {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello, World\n');
  } else if (req.url === '/sub') {
    const vlessURL = `vless://${UUID}@skk.moe:443?encryption=none&security=tls&sni=${DOMAIN}&type=ws&host=${DOMAIN}&path=%2F#${NAME}`;
    
    const base64Content = Buffer.from(vlessURL).toString('base64');

    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end(base64Content + '\n');
  } else {
    res.writeHead(404, { 'Content-Type': 'text/plain' });
    res.end('Not Found\n');
  }
});

httpServer.listen(PORT, () => {
  console.log(`HTTP Server is running on port ${PORT}`);
});

// WebSocket 服务器
const wss = new WebSocket.Server({ server: httpServer });
wss.on('connection', ws => {
  console.log("WebSocket 连接成功");
  ws.on('message', msg => {
    if (msg.length < 18) {
      console.error("数据长度无效");
      return;
    }
    try {
      const [VERSION] = msg;
      const id = msg.slice(1, 17);
      if (!id.every((v, i) => v == parseInt(uuid.substr(i * 2, 2), 16))) {
        console.error("UUID 验证失败");
        return;
      }
      let i = msg.slice(17, 18).readUInt8() + 19;
      const port = msg.slice(i, i += 2).readUInt16BE(0);
      const ATYP = msg.slice(i, i += 1).readUInt8();
      const host = ATYP === 1 ? msg.slice(i, i += 4).join('.') :
        (ATYP === 2 ? new TextDecoder().decode(msg.slice(i + 1, i += 1 + msg.slice(i, i + 1).readUInt8())) :
          (ATYP === 3 ? msg.slice(i, i += 16).reduce((s, b, i, a) => (i % 2 ? s.concat(a.slice(i - 1, i + 1)) : s), []).map(b => b.readUInt16BE(0).toString(16)).join(':') : ''));
      console.log('连接到:', host, port);
      ws.send(new Uint8Array([VERSION, 0]));
      const duplex = createWebSocketStream(ws);
      net.connect({ host, port }, function () {
        this.write(msg.slice(i));
        duplex.on('error', err => console.error("E1:", err.message)).pipe(this).on('error', err => console.error("E2:", err.message)).pipe(duplex);
      }).on('error', err => console.error("连接错误:", err.message));
    } catch (err) {
      console.error("处理消息时出错:", err.message);
    }
  }).on('error', err => console.error("WebSocket 错误:", err.message));
});
```

- 探针版

```js
const os = require('os');
const http = require('http');
const { Buffer } = require('buffer');
const fs = require('fs');
const axios = require('axios');
const path = require('path');
const net = require('net');
const { exec, execSync } = require('child_process');
const { WebSocket, createWebSocketStream } = require('ws');
const logcb = (...args) => console.log.bind(this, ...args);
const errcb = (...args) => console.error.bind(this, ...args);
const UUID = process.env.UUID || '9e2b1a67-3e64-4abc-a661-c3e3b7f3da23';
const uuid = UUID.replace(/-/g, "");
const NEZHA_SERVER = process.env.NEZHA_SERVER || 'nezha.gvkoyeb.eu.org';
const NEZHA_PORT = process.env.NEZHA_PORT || '443';        // 端口为443时自动开启tls
const NEZHA_KEY = process.env.NEZHA_KEY || '';             // 哪吒三个变量不全不运行
const DOMAIN = process.env.DOMAIN || 'example.com';  //项目域名或已反代的域名，不带前缀，建议填已反代的域名
const NAME = process.env.NAME || 'freeshare';
const PORT = process.env.PORT || 33333;

// 创建HTTP路由
const httpServer = http.createServer((req, res) => {
  if (req.url === '/') {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello, World\n');
  } else if (req.url === '/sub') {
    const vlessURL = `vless://${UUID}@skk.moe:443?encryption=none&security=tls&sni=${DOMAIN}&type=ws&host=${DOMAIN}&path=%2F#${NAME}`;
    
    const base64Content = Buffer.from(vlessURL).toString('base64');

    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end(base64Content + '\n');
  } else {
    res.writeHead(404, { 'Content-Type': 'text/plain' });
    res.end('Not Found\n');
  }
});

httpServer.listen(PORT, () => {
  console.log(`HTTP Server is running on port ${PORT}`);
});

// 判断系统架构
function getSystemArchitecture() {
  const arch = os.arch();
  if (arch === 'arm' || arch === 'arm64') {
    return 'arm';
  } else {
    return 'amd';
  }
}

// 下载对应系统架构的ne-zha
function downloadFile(fileName, fileUrl, callback) {
  const filePath = path.join("./", fileName);
  const writer = fs.createWriteStream(filePath);
  axios({
    method: 'get',
    url: fileUrl,
    responseType: 'stream',
  })
    .then(response => {
      response.data.pipe(writer);
      writer.on('finish', function() {
        writer.close();
        callback(null, fileName);
      });
    })
    .catch(error => {
      callback(`Download ${fileName} failed: ${error.message}`);
    });
}

function downloadFiles() {
  const architecture = getSystemArchitecture();
  const filesToDownload = getFilesForArchitecture(architecture);

  if (filesToDownload.length === 0) {
    console.log(`Can't find a file for the current architecture`);
    return;
  }

  let downloadedCount = 0;

  filesToDownload.forEach(fileInfo => {
    downloadFile(fileInfo.fileName, fileInfo.fileUrl, (err, fileName) => {
      if (err) {
        console.log(`Download ${fileName} failed`);
      } else {
        console.log(`Download ${fileName} successfully`);

        downloadedCount++;

        if (downloadedCount === filesToDownload.length) {
          setTimeout(() => {
            authorizeFiles();
          }, 3000);
        }
      }
    });
  });
}

function getFilesForArchitecture(architecture) {
  if (architecture === 'arm') {
    return [
      { fileName: "npm", fileUrl: "https://github.com/eooce/test/releases/download/ARM/swith" },
    ];
  } else if (architecture === 'amd') {
    return [
      { fileName: "npm", fileUrl: "https://github.com/eooce/test/releases/download/bulid/swith" },
    ];
  }
  return [];
}

// 授权并运行ne-zha
function authorizeFiles() {
  const filePath = './npm';
  const newPermissions = 0o775;
  fs.chmod(filePath, newPermissions, (err) => {
    if (err) {
      console.error(`Empowerment failed:${err}`);
    } else {
      console.log(`Empowerment success:${newPermissions.toString(8)} (${newPermissions.toString(10)})`);

      // 运行ne-zha
      let NEZHA_TLS = '';
      if (NEZHA_SERVER && NEZHA_PORT && NEZHA_KEY) {
        if (NEZHA_PORT === '443') {
          NEZHA_TLS = '--tls';
        } else {
          NEZHA_TLS = '';
        }
        const command = `./npm -s ${NEZHA_SERVER}:${NEZHA_PORT} -p ${NEZHA_KEY} ${NEZHA_TLS} --skip-conn --disable-auto-update --skip-procs --report-delay 4 >/dev/null 2>&1 &`;
        try {
          exec(command);
          console.log('npm is running');
        } catch (error) {
          console.error(`npm running error: ${error}`);
        }
      } else {
        console.log('NEZHA variable is empty,skip running');
      }
    }
  });
}
downloadFiles();

// WebSocket 服务器
const wss = new WebSocket.Server({ server: httpServer });
wss.on('connection', ws => {
  console.log("WebSocket 连接成功");
  ws.on('message', msg => {
    if (msg.length < 18) {
      console.error("数据长度无效");
      return;
    }
    try {
      const [VERSION] = msg;
      const id = msg.slice(1, 17);
      if (!id.every((v, i) => v == parseInt(uuid.substr(i * 2, 2), 16))) {
        console.error("UUID 验证失败");
        return;
      }
      let i = msg.slice(17, 18).readUInt8() + 19;
      const port = msg.slice(i, i += 2).readUInt16BE(0);
      const ATYP = msg.slice(i, i += 1).readUInt8();
      const host = ATYP === 1 ? msg.slice(i, i += 4).join('.') :
        (ATYP === 2 ? new TextDecoder().decode(msg.slice(i + 1, i += 1 + msg.slice(i, i + 1).readUInt8())) :
          (ATYP === 3 ? msg.slice(i, i += 16).reduce((s, b, i, a) => (i % 2 ? s.concat(a.slice(i - 1, i + 1)) : s), []).map(b => b.readUInt16BE(0).toString(16)).join(':') : ''));
      console.log('连接到:', host, port);
      ws.send(new Uint8Array([VERSION, 0]));
      const duplex = createWebSocketStream(ws);
      net.connect({ host, port }, function () {
        this.write(msg.slice(i));
        duplex.on('error', err => console.error("E1:", err.message)).pipe(this).on('error', err => console.error("E2:", err.message)).pipe(duplex);
      }).on('error', err => console.error("连接错误:", err.message));
    } catch (err) {
      console.error("处理消息时出错:", err.message);
    }
  }).on('error', err => console.error("WebSocket 错误:", err.message));
});
```

## 创建应用

- 点击 **附加功能** → **Setup Node.js App** 然后点击 **CREATE APPLICATION**

- **Node.js version** 选择 `18.20.4`
- **Application mode** 选择 `Production`
- **Application root** 填入 `/home/username/domains/example.com/public_html`
- **Application startup** file 填入 `app.js`
- 点击 **ADD VARIABLE**，依次添加 `UUID`, `DOMAIN`, `NAME`, `PORT`
- 点击 **CREATE** 创建应用，然后点击 **Run NPM Install**，接着点击 **RUN JS script**，选择 start，点击 **RUN JS SCRIPT** 启动服务

对于没有 **Setup Node.js App** 的面板，可以直接运行
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install 18.20.4
cd domains/example.com/public_html
npm install
nohup node app.js &
```

## 开始使用

访问 `https://example.com/sub` 全选复制页面内容后粘贴到 V2rayN 使用即可

## 重置机器

如果遇到 quota exceed 错误，执行以下命令重置机器

```bash
cd
chmod -R 755 ~/.[!.]* ~/..?*
rm -rf ~/.[!.]* ~/*
```



