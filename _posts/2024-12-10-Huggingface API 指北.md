---
layout: mypost
title: Huggingface API 指北
date: 2024-12-10
published: true
categories: 免费容器
tags: 
 - huggingface
---

Huggingface 可通过 Hub Python Library 实现 API 调用  

## 入门篇

### 安装

```bash
pip install --upgrade huggingface_hub
```

### 令牌

访问 [**令牌页**](https://huggingface.co/settings/tokens) ，点击 Create new token，然后先后点击 **Read** 和 **Write** 创建两种不同权限的密钥，为方便管理命名可直接使用 **Rad** 和 **Write**

### 登录

执行下面命令，输入获取的令牌登录，token 保存在 `~/.cache/huggingface/token`

```bash
huggingface-cli login
```

查看当前登录用户的 **username**，这个参数后面将会用到

```bash
huggingface-cli whoami
```

### 创建模型仓库

若要创建仓库，需要使用 **Write** 权限的令牌登录

```bash
from huggingface_hub import HfApi
api = HfApi()
api.create_repo(repo_id="super-cool-model", private=True)
```

### 上传文件

注意修改 **username** 和上传路径，**path\_or\_fileobj** 对应源文件路径，**path\_in\_repo** 对应上传到仓库的路径

```bash
from huggingface_hub import HfApi
api = HfApi()
api.upload_file(
    path_or_fileobj="/path/to/local/file",
    path_in_repo="/path/to/repo/file",
    repo_id="username/super-cool-model",
)
```

上传成功会返回文件的访问链接，形如

    https://huggingface.co/username/super-cool-model/blob/main/path/to/repo/file

## 空间篇

### 创建空间

注意替换 **username**

```bash
from huggingface_hub import HfApi
api = HfApi()
api.create_repo(repo_id="username/my-cool-training-space", repo_type="space", space_sdk="gradio")
```

创建成功后同样会返回空间的链接

### 复制空间

```bash
from huggingface_hub import HfApi
api = HfApi()
api.duplicate_space("open-webui/open-webui")
```

### 上传项目

注意替换 **username，folder\_path** 为待上传项目的目录

```bash
from huggingface_hub import HfApi
api = HfApi()
api.upload_folder(repo_id="username/my-cool-training-space", repo_type="space", folder_path="src/")
```

### 暂停空间

```bash
from huggingface_hub import HfApi
api = HfApi()
api.pause_space(repo_id="username/my-cool-training-space")
```

### 重启空间

```bash
from huggingface_hub import HfApi
api = HfApi()
api.restart_space(repo_id="username/my-cool-training-space")
```

### 环境变量

-   添加环境变量
    
    ```bash
    api.add_space_secret(repo_id="username/my-cool-training-space", key="HF_TOKEN", value="hf_api_***")
    api.add_space_variable(repo_id="username/my-cool-training-space", key="MODEL_REPO_ID", value="user/repo")
    ```
    
-   删除环境变量
    
    ```bash
    api.delete_space_secret(repo_id="username/my-cool-training-space", key="HF_TOKEN")
    api.delete_space_variable(repo_id="username/my-cool-training-space", key="MODEL_REPO_ID")
    ```
    
-   在创建空间时添加
    
    ```bash
    api.create_repo(
        repo_id="username/my-cool-training-space",
        repo_type="space",
        space_sdk="gradio",
        space_secrets=[{"key"="HF_TOKEN", "value"="hf_api_***"}, ...],
        space_variables=[{"key"="MODEL_REPO_ID", "value"="user/repo"}, ...],
    )
    ```
    
-   在复制空间时添加（**貌似已失效**）
    
    ```bash
    api.duplicate_space(
        from_id="username/my-cool-training-space",
        secrets=[{"key"="HF_TOKEN", "value"="hf_api_***"}, ...],
        variables=[{"key"="MODEL_REPO_ID", "value"="user/repo"}, ...],
    )
    ```
    