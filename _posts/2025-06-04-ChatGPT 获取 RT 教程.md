---
layout: mypost
title: ChatGPT 获取 RT 教程
date: 2025-06-04
published: true
categories: 逆向
tags: 
 - chatgpt
 - windows
---

**注意：** 本教程仅介绍 Windows 系统下的操作

## 开始之前

准备一台可以开启代理的电脑，一部 ios 系统的移动设备，并注意避免同一device token登录大量账号以防封号

## 第一步，开启局域网代理

1. **主机开启代理**

   首先确保手机和电脑连接到同一 WIFI，然后在电脑端的代理软件开启局域网代理

   - **V2rayN**

     设置 - 参数设置 - 允许来自局域网的连接

   - **Fclash**

     工具 - 基本配置 - 常规 - 局域网代理

2. **查看 ipv4 地址**

    ```bash
    ipconfig
    ```

​	找到 形如`IPv4 地址 . . . . . . . . . . . . : 192.168.0.104` 的行，`192.168.0.104` 就是主机内网地址

3. **WIFI 手动代理**

​	点击连接到的 WIFI，设置 代理 - 手动，地址填入主机内网地址，端口填写电脑的代理端口，比如 V2rayN 是 `10808`

4. **测试连接**

   测试手机是否正常连接到代理

## 第二步，安装 OpenAI 客户端

1. **下载软件**

​	下载安装 **12.6.5.3 版 iTunes**：[点击下载](https://secure-appldnld.apple.com/itunes12/091-87819-20180912-69177170-B085-11E8-B6AB-C1D03409AD2A6/iTunes64Setup.exe)，下载后双击安装，安装完成后运行并登录 AppleID

​	下载 **ios 旧版应用工具**：[点击下载](https://wwl.lanzn.com/i9yo41oi30mj)，下载后解压缩，解压后进入目录，双击运行 `iOS旧版应用下载v6.0.exe`

2. **下载安装包**

​	ios 旧版应用工具搜索 **OpenAI**，地区选择 **美国**，双击搜素结果的第一个，接着下滑找到最后一个版本 `1.2024.233` 双击开始拦截

​	iTunes 右上角搜索 **OpenAI**，点击下载，然后在右上角点击箭头查看详情，如果下载大小是 63M 说明拦截成功

​	如果拦截失败，需要在 iTunes 删除下载，然后在 ios 旧版应用工具  **停止拦截** 再 **开始拦截** 后再在 iTunes 重新下载	

3. **安装 APP**

​	下载好后在ios 旧版应用工具的 **安装管理** 标签检查安装包版本 `1.2024.233` ，选中安装包后右键菜单选择 **去APP更新提示** 修改安装包，然后将修改后的安装包通过 **爱思助手** 导入手机完成安装

## 第三步，获取 device_token

下载 winpython：[点击下载](https://github.com/winpython/winpython/releases/download/15.2.20250412/Winpython64-3.12.10.0dotb3.exe)，双击解压缩，完成解压后进入目录双击运行 `WinPython Command Prompt.exe`

1. **修改局域网代理**

   只需要将端口修改为 `8080` 即可

2. **监听服务**

    ```bash
    pip install mitmproxy
    mitmweb --mode upstream:http://127.0.0.1:10808
    ```
    请自行修改代理端口，运行后会自动打开监听页面，注意不要关闭它

3. **安装证书**

   Safari 地址栏输入 `mitm.it` 后回车，下载 ios 描述文件并安装，接着在 通用 - 关于本机 - 证书信任设置信任该证书

4. **登录抓包**

   使用上一步安装的 OpenAI 客户端登录，然后在监听页面的 Search 输入框输入：`https://ios.chat.openai.com/backend-api/preauth_devicecheck` 查询，保存 `device_token` 和 `device_id` 的值

## 第四步，获取 refreshToken

不要关闭上一步的终端，重新双击运行 `WinPython Command Prompt.exe` 开启一个新终端

1. **下载 chromium**

   [点击下载](https://download-chromium.appspot.com/dl/Win_x64?type=snapshots)，下载后解压并记录解压后的目录位置

2. **安装依赖**

    ```bash
    pip install requests DrissionPage click pyinstaller
    ```

3. **运行主程序**

    ```bash
    notepad
    ```

​	粘贴以下代码，注意自行修改 `proxy`、 `device_token` 和 `device_id` 

```python
import base64
import hashlib
import json
import os
import random
import string
import time
from urllib.parse import urlparse

import click
import requests
from DrissionPage import ChromiumOptions
from DrissionPage._pages.web_page import WebPage

def to_time(t: int = None):
    return time.strftime('%Y-%m-%d %H:%M:%S', time.localtime(t))


def to_timestamp(t: str = None):
    return time.strptime(t, '%Y-%m-%d %H:%M:%S') if t else time.time()


class TokenManager:
    def __init__(
            self,
            proxy='http://127.0.0.1:10808',
            chrome_path=None,
            refresh_token=None,
            device_token=None,
            refresh_interval=60,
            storage_path='./token.json'
    ):
        self.refresh_token = refresh_token
        self.device_token = device_token
        self.refresh_interval = refresh_interval
        self.access_token = None
        self.storage_path = storage_path
        self.co = ChromiumOptions()
        if proxy:
            self.co.set_proxy(proxy)
            self.proxy = {'all': proxy}
        else:
            self.proxy = None
        self.load_token()
        self.save_token()

        if chrome_path:
            self.co.set_browser_path(chrome_path)

    def get_refresh_token(self):
        self.ensure_refresh_token()
        return self.refresh_token

    def get_access_token(self):
        if self.is_expired():
            self.refresh()
        return self.access_token

    def get_sess_key(self):
        response = requests.post(
            'https://api.openai.com/dashboard/onboarding/login',
            headers={
                "Authorization": f"Bearer {self.get_access_token()}",
                "Content-Type": "application/json",
                "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36 OPR/105.0.0.0",
            },
            proxies=self.proxy
        )
        if response.ok:
            data = json.loads(response.text)
            return {
                'sess_key': data['user']['session']['sensitive_id'],
                'created': to_time(data['user']['session']['created']),
                'last_use': to_time(data['user']['session']['last_use']),
            }

    def is_expired(self):
        if not self.access_token:
            return True
        payload = self.access_token.split('.')[1]
        payload = payload + '=' * - (len(payload) % - 4)
        exp = json.loads(base64.b64decode(payload).decode()).get('exp')
        return exp - time.time() < 60

    def refresh(self):
        self.ensure_refresh_token()
        self.access_token = self.generate_access_token()

    def ensure_refresh_token(self):
        if self.refresh_token:
            return
        code_verifier = self.generate_code_verifier()
        code_challenge = self.generate_code_challenge(code_verifier)
        preauth_cookie = self.get_preauth_cookie()
        if not preauth_cookie:
            raise Exception('抓取preauth_cookie失败')
        url = f'https://auth0.openai.com/authorize' \
              f'?client_id=pdlLIX2Y72MIl2rhLhTE9VV9bN905kBh' \
              f'&audience=https%3A%2F%2Fapi.openai.com%2Fv1' \
              f'&redirect_uri=com.openai.chat%3A%2F%2Fauth0.openai.com%2Fios%2Fcom.openai.chat%2Fcallback' \
              f'&scope=openid%20email%20profile%20offline_access%20model.request%20model.read%20organization.read%20offline' \
              f'&response_type=code' \
              f'&code_challenge={code_challenge}' \
              f'&code_challenge_method=S256' \
              f'&preauth_cookie={preauth_cookie}'

        url += '&prompt=login'
        # print(url)
        # code = input('code: ')
        page = WebPage(chromium_options=self.co)
        page.get(url)
        page.listen.start('com.openai.chat://auth0.openai.com/ios/com.openai.chat/callback')
        res = page.listen.wait()
        query1 = {args.split('=')[0]: args.split('=')[1] for args in urlparse(res.url).query.split('&')}
        code = query1.get('code')
        if not code:
            raise Exception('preauth_cookie已过期')
        # state = query1['state']
        page.close()
        resp_json = requests.post('https://auth0.openai.com/oauth/token', json={
            'redirect_uri': 'com.openai.chat://auth0.openai.com/ios/com.openai.chat/callback',
            'grant_type': 'authorization_code',
            'client_id': 'pdlLIX2Y72MIl2rhLhTE9VV9bN905kBh',
            'code': code,
            'code_verifier': code_verifier
        }, proxies=self.proxy).json()
        # json.dump(resp_json, open('./app.json', 'w'))
        # print(json.dumps(resp_json, indent=2))
        self.refresh_token = resp_json.get('refresh_token')
        self.save_token()

    def revoke_refresh_token(self, refresh_token):
        resp = requests.post('https://auth0.openai.com/oauth/revoke', json={
            'client_id': 'pdlLIX2Y72MIl2rhLhTE9VV9bN905kBh',
            'token': refresh_token
        }, proxies=self.proxy)
        assert resp.status_code == 200
        self.refresh_token = None
        self.save_token()

    @staticmethod
    def generate_code_verifier():
        return base64.urlsafe_b64encode(os.urandom(32)).decode().rstrip('=')

    @staticmethod
    def generate_code_challenge(code_verifier):
        m = hashlib.sha256()
        m.update(code_verifier.encode())
        return base64.urlsafe_b64encode(m.digest()).decode().rstrip('=')

    def get_preauth_cookie(self):
        # fakeopen已挂
        # return requests.get('https://ai.fakeopen.com/auth/preauth').json().get('preauth_cookie')
        if self.device_token:
            rsp = requests.post(
                'https://ios.chat.openai.com/backend-api/preauth_devicecheck',
                json={
                    "bundle_id": "com.openai.chat",
                    "device_id": "62345678-042E-45C7-962F-AC725D0E7770",
                    "device_token": self.device_token,
                    "request_flag": True
                },
                proxies=self.proxy
            )
            if rsp.status_code == 200 and rsp.json().get('is_ok'):
                return rsp.cookies.get('_preauth_devicecheck')
        raise Exception('抓取preauth_cookie失败')

    def generate_access_token(self):
        self.ensure_refresh_token()
        resp = requests.post('https://token.oaifree.com/api/auth/refresh', data={
            'refresh_token': self.refresh_token
        })
        if resp.status_code == 200:
            access_token = resp.json().get('access_token')
            self.access_token = access_token
            self.save_token()
            return access_token
        else:
            return self.generate_access_token_old()

    def generate_share_token(self, unique_name='share_token'):
        # share_token的有效期还取决于access_token
        resp = requests.post('https://chat.oaifree.com/token/register', data={
            'unique_name': unique_name,
            'access_token': self.get_access_token(),
            'expires_in': 20,
            'site_limit': None,
            'gpt35_limit': -1,
            'gpt4_limit': -1,
            'show_conversations': True,
            'show_userinfo': False,
            'reset_limit': True,
        })
        return resp.json().get('token_key')

    def generate_access_token_old(self):
        resp = requests.post(
            'https://auth0.openai.com/oauth/token',
            json={
                'redirect_uri': 'com.openai.chat://auth0.openai.com/ios/com.openai.chat/callback',
                'grant_type': 'refresh_token',
                'client_id': 'pdlLIX2Y72MIl2rhLhTE9VV9bN905kBh',
                'refresh_token': self.refresh_token
            },
            headers={'Content-Type': 'application/json'},
            proxies=self.proxy)
        if resp.status_code == 200:
            access_token = resp.json().get('access_token')
            self.access_token = access_token
            self.save_token()
            return access_token

    def load_token(self):
        if os.path.exists(self.storage_path):
            with open(self.storage_path, 'r') as file:
                token_json = json.load(file)
                if not self.access_token:
                    self.access_token = token_json.get('access_token')
                if not self.refresh_token:
                    self.refresh_token = token_json.get('refresh_token')
                if not self.device_token:
                    self.device_token = token_json.get('device_token')

    def save_token(self):
        with open(self.storage_path, 'w') as file:
            json.dump({
                'device_token': self.device_token,
                'refresh_token': self.refresh_token,
                'access_token': self.access_token
            }, file, indent=2)


@click.command()
@click.option('--proxy', "-p", help='A http proxy str. (http://127.0.0.1:8080)', required=False)
@click.option('--chrome', "-c", help='Path to Chrome executable', required=False)
@click.option("--refresh_token", "-r", help='Get refresh token.', is_flag=True)
@click.option("--access_token", "-a", help='Get access token.', is_flag=True)
@click.option("--sess_key", "-s", help='Get sess key.', is_flag=True)
@click.option("--share_token", "-f", help='Get share key.', is_flag=True)
def cli(proxy, chrome, refresh_token, access_token, sess_key, share_token):
    if proxy and chrome:
        obj = TokenManager(proxy=proxy, chrome_path=chrome)
    elif proxy:
        obj = TokenManager(proxy=proxy)
    elif chrome:
        obj = TokenManager(chrome_path=chrome)
    else:
        obj = TokenManager()
    if refresh_token:
        print({"refresh_token": obj.get_refresh_token()})
    if access_token:
        _access_token = obj.get_access_token()
        payload = _access_token.split('.')[1]
        payload = payload + '=' * - (len(payload) % - 4)
        exp = json.loads(base64.b64decode(payload).decode()).get('exp')
        print({"access_token": obj.get_access_token(), 'expired': to_time(exp)})
    if sess_key:
        print(obj.get_sess_key())
    if share_token:
        unique_name = ''.join(random.sample(string.ascii_letters + string.digits, 16))
        print({"share_token": obj.generate_share_token(unique_name), "unique_name": unique_name})


if __name__ == '__main__':
    cli()
```

将文件命名为 `main.py` 保存到 winpython 解压后的 notebooks 目录下然后执行

```bash
python main.py -ra -c "/path/to/chrome.exe"
```

这会自动拉起一个 chrome 网页，登录你的 ChatGPT，程序会自动获取 `refreshToken` 和 `accessToken` 并保存到 notebooks目录下的 `token.json` 文件中

## 参考链接

- [自己搞 refresh token](https://linux.do/t/topic/42253)

- [【12/09验证可行】始皇停止服务后自己获取refresh token——如何绕过ssl pinning证书校验](https://linux.do/t/topic/226241) 

- [苹果设备获取device token教程 帮助非三级号（不包括俺 已经三级了grin）实现refresh token自由](https://linux.do/t/topic/58580) 