---
layout: mypost
title: Serv00 部署 NextChat
date: 2025-04-17
published: false
categories: 自部署
tags: 
 - serv00
---

## PM2

```bash
mkdir -p ~/.npm-global && npm config set prefix "$HOME/.npm-global" && echo 'export PATH=$HOME/.npm-global/bin:$PATH' >> ~/.profile && source ~/.profile && npm install -g pm2 && pm2

cat > $HOME/ecosystem.config.js <<EOF
module.exports = {
    apps: [{
      name: "NextChat",
      cwd: "$HOME/NextChat/.next/standalone",
      script: "./server.js",
      watch: false,
      env: {
        NODE_ENV: "production",
		ENABLE_MCP: true,
        PORT: 6666,
        CODE: "password",
        DEFAULT_MODEL: "gpt-4.1-mini",
        OPENAI_API_KEY: "sk-***********",
        GOOGLE_API_KEY: "AIza**********",
      }
    }]
  };
EOF
```

## NewAPI

1. **配置网站**

    ```bash
    export NEWAPI_DOMAIN=api.example.com
    export NEWAPI_PORT=5555
    export SERVER_IP=207.180.248.6
    devil www add "$NEWAPI_DOMAIN" proxy localhost "$NEWAPI_PORT"
    devil ssl www add "$SERVER_IP" le le "$NEWAPI_DOMAIN"
    ```

2. **安装new-api**

    ```bash
    mkdir $HOME/new-api && cd $_
    curl -L -o new-api https://github.com/k0baya/new-api-freebsd/releases/latest/download/new-api
    chmod +x new-api
    ```

3. **配置文件**

   ```bash
   export NEWAPI_DIR=$HOME/new-api
   export REDIS_PORT=62611
   export REDIS_PASSWORD=$(openssl rand -hex 16)
   export SQL_DSN="root:password@tcp(localhost:3306)/new-api"
   
   fetch -o "$NEWAPI_DIR/redis.conf" https://raw.githubusercontent.com/redis/redis/7.4/redis.conf
   
   sed -i '' "s/^port .*/port $REDIS_PORT/" $NEWAPI_DIR/redis.conf; sed -i '' -E "s/^# requirepass .*/requirepass $REDIS_PASSWORD/" $NEWAPI_DIR/redis.conf; sed -i '' 's/^appendonly no$/appendonly yes/' $NEWAPI_DIR/redis.conf
   
   cat > $NEWAPI_DIR/start.sh << EOF
   #!/bin/sh
   export SQL_DSN="$SQL_DSN"
   export REDIS_CONN_STRING="redis://:$REDIS_PASSWORD@localhost:$REDIS_PORT"
   exec ./new-api --port 47416 --log-dir ./logs
   EOF
   chmod +x start.sh
   
   cat > $NEWAPI_DIR/ecosystem.config.js << EOF
   module.exports = {
     apps: [
       {
         name: 'Redis',
         cwd: '/home/welc/new-api',
         script: 'redis-server',
         args: 'redis.conf',
         autorestart: true,
         watch: false
       },
       {
         name: 'NewAPI',
         cwd: '/home/welc/new-api',
         script: './start.sh',
         autorestart: true,
         watch: false
       }
     ]
   };
   EOF
   
   pm2 start ecosystem.config.js
   ```

   

## Action

1. **New-API**

   ```yaml
   name: Build new-api
   on: workflow_dispatch
   jobs:
     test:
       runs-on: ubuntu-latest
       name: Build new-api
       steps:
         - uses: actions/checkout@v4
         - name: Set up bun
           uses: oven-sh/setup-bun@v1
           with:
             bun-version: latest
         - name: Build frontend
           run: |
             cd web
             bun install
             bun add sse
             REACT_APP_VERSION=$(cat ../VERSION) bun run build
             cd ..
         - name: Build backend for FreeBSD (cross-compile)
           env:
             GOOS: freebsd
             GOARCH: amd64
             CGO_ENABLED: "0"
           run: >
             go mod download
   
             go build -ldflags "-s -w -X 'one-api/common.Version=$(cat VERSION)'"
             -o new-api
         - name: Upload artifact
           uses: actions/upload-artifact@v4
           with:
             name: new-api-freebsd
             path: new-api
         - name: Generate release tag
           id: tag
           run: >
             echo "release_tag=$(wget -qO-
             https://api.github.com/repos/QuantumNous/new-api/tags \
               | gawk -F '["v]' '/name/{print "v"$5;exit}')" >> $GITHUB_OUTPUT
         - name: Create release
           uses: softprops/action-gh-release@v1
           env:
             GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
           with:
             tag_name: ${{ steps.tag.outputs.release_tag }}
             files: new-api
         - name: Delete workflow runs
           uses: Mattraks/delete-workflow-runs@v2
           with:
             token: ${{ github.token }}
             repository: ${{ github.repository }}
             retain_days: 1
             keep_minimum_runs: 8
   ```

   

2. **NextChat**

```yaml
name: Deploy
on:
  push:
    branches:
      - main
permissions:
  contents: write
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
jobs:
  build:
    name: Build artifact
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          lfs: true
      - name: Setup Node.js 18
        uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: yarn
      - name: Install yarn
        run: npm install -g yarn
      - name: Install dependencies and build
        run: |
          export ENABLE_MCP=true
          yarn install
          yarn build
      - name: Prepare standalone for deployment
        run: |
          cp -r public .next/standalone/
          mkdir -p .next/standalone/.next
          cp -r .next/static .next/standalone/.next/
      - name: Zip the standalone directory
        run: |
          zip -r release.zip .next/standalone
      - uses: actions/upload-artifact@v4
        with:
          name: release.zip
          path: release.zip
  deploy:
    name: Deploy artifact
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: release.zip
      - name: SCP Transfer
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USER }}
          key: ${{ secrets.KEY }}
          password: ${{ secrets.PASSWORD }}
          port: ${{ secrets.PORT }}
          source: release.zip
          target: /tmp/NextChat
      - name: Remote Deployment
        uses: appleboy/ssh-action@v0.1.7
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USER }}
          key: ${{ secrets.KEY }}
          password: ${{ secrets.PASSWORD }}
          port: ${{ secrets.PORT }}
          script: |
            set -ex
            source .profile
            pm2 delete all
            rm -fr NextChat
            git clone https://github.com/ChatGPTNextWeb/NextChat
            cd NextChat
            mv /tmp/NextChat/release.zip release.zip            
            unzip -qq -o release.zip
            rm -fr /tmp/NextChat release.zip
            cp ../ecosystem.config.js ecosystem.config.js
            pm2 start ecosystem.config.js
            pm2 save
            echo "Deployed successfully"
```

## MCP

1. **编译uv**

    ```yaml
    git clone https://github.com/astral-sh/uv.git
    cd uv
    cargo build --release -j 5
    ```

2. **添加uv**

   ```bash
   mkdir -p ~/.local/bin
   cp target/release/uv ~/.local/bin/
   chmod +x ~/.local/bin/uv
   
   cat >> ~/.profile << 'EOF'
   export PATH="$HOME/.local/bin:$PATH"
   EOF
   
   source ~/.profile
   uv --version
   ```

3. **安装工具**

    ```bash
    cd ~/NextChat
    export CARGO_BUILD_JOBS=5 
    uv tool install mcp-server-time 
    uv tool install mcp-server-fetch
    uv tool list
    # 打包备份
    tar -czvf uv-tools.tar.gz -C $HOME .local .cargo
    ```

4. **配置MCP**

    ```bash
    mkdir -p ~/NextChat/.next/standalone/app/mcp
    cat <<EOF > ~/NextChat/.next/standalone/app/mcp/mcp_config.json
    {
      "mcpServers": {
        "fetch": {
          "command": "uvx",
          "args": [
            "mcp-server-fetch"
          ]
        },
        "mcp-server-time": {
          "command": "uvx",
          "args": [
            "mcp-server-time",
            "--local-timezone=${TZ}"
          ],
          "alwaysAllow": [
            "get_current_time",
            "convert_time"
          ]
        },
        "grok2_images": {
          "command": "npx",
          "args": [
            "-y",
            "grok2-image-mcp-server@0.1.3"
          ],
          "env": {
            "XAIAPI_KEY": "${XAIAPI_KEY}",
            "XAIAPI_BASE_URL": "${XAIAPI_BASE_URL}"
          }
        },
        "tavily-mcp": {
          "command": "npx",
          "args": [
            "-y",
            "tavily-mcp@0.1.2"
          ],
          "env": {
            "TAVILY_API_KEY": "${TAVILY_API_KEY}"
          }
        }
      }
    }
    EOF
    ```

## FileServer

| 变量名             | 注释                       | 参考值             |
| :----------------- | :------------------------- | :----------------- |
| FILE_SERVER_IP     | serv00 的 IPv4 地址        | 207.180.248.6      |
| FILE_SERVER_DOMAIN | 自定义域名                 | `file.example.com` |
| FILE_SERVER_PORT   | 端口号                     | 22222              |
| FILE_SERVER_CODE   | 与NextChat的 **CODE** 一致 | CODE1,CODE2        |

```bash
export SERVER_IP=207.180.248.6
export FILE_SERVER_DOMAIN="file.example.com"
export FILE_SERVER_PORT=22222
export FILE_SERVER_CODE="CODE1,CODE2"
export FILE_SERVER_DIR="/home/$(whoami)/domains/$FILE_SERVER_DOMAIN/"

devil www add "$FILE_SERVER_DOMAIN" proxy localhost "$FILE_SERVER_PORT"

devil ssl www add "$SERVER_IP" le le "$FILE_SERVER_DOMAIN"

curl -sL "https://raw.githubusercontent.com/QAbot-zh/go-file-server/main/main.go" -o "$FILE_SERVER_DIR/main.go"

sed -i '' "s/:3456/:$FILE_SERVER_PORT/g" "$FILE_SERVER_DIR/main.go"

cat > "$FILE_SERVER_DIR/env.conf" <<EOF
accessCodes=${FILE_SERVER_CODE}
EOF

cat > "$FILE_SERVER_DIR/ecosystem.config.js" <<EOF
module.exports = {
  apps: [
    {
      name: "go-file-server",
      script: "go",
      args: "run main.go",
      cwd: "$FILE_SERVER_DIR"
    }
  ]
};
EOF

pm2 start "$FILE_SERVER_DIR/ecosystem.config.js" && pm2 save
```



## 参考链接

- [NextChat配置MCP服务器食用指南](https://zhuanlan.zhihu.com/p/32966458457)
