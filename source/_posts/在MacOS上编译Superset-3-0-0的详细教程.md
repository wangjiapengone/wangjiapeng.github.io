---
title: 在 Mac OS 上编译 Superset 3.0.0 的详细教程
tags:
  - Superset
categories:
  - 可视化
date: 2023-09-22 15:21:41
id: superset-3-built-on-mac
---

Superset 近期发布了 3.0.0 版本，支持图表交互功能，以及 Excel 格式导出功能。本文记录了 Superset 3.0.0 版本在 Mac OS 上进行编译的详细步骤，以便于创建本地的测试环境

<!-- more -->


> **系统环境：**
> 操作系统：Mac OS 13.5.2
> Python：3.11

## 1. 安装环境依赖

- 先下载 `Superset` 源码，切换到 3.0 分支

    ```bash
    git clone https://github.com/apache/superset.git
    git checkout 3.0
    cd superset
    ```

- 如果没有安装 `XCode`，先进行安装

    ```bash
    xcode-select --install
    ```

    如果出现报错：`xcode-select: error: command line tools are already installed, use "Software Update" in System Settings to install updates`，说明已经安装过了

- 如果没有安装 `Homebrew`，先进行安装

    ```bash
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    ```

- 使用 `brew` 命令安装依赖包

    ```bash
    brew install readline pkg-config libffi openssl mysql postgresql@14 zlib
    ```

- （可选）安装 `Pyenv` 和 `virtualenv`

    ```bash
    # 安装 Pyenv
    brew install pyenv
    # 安装虚拟环境管理工具 virtualenv
    brew install pyenv-virtualenv
    # 配置环境
    echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
    echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
    echo 'eval "$(pyenv init -)"' >> ~/.zshrc
    echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc
    # 使配置生效
    source ~/.zshrc
    ```

- （可选）安装 `Python 3.11` 并创建虚拟环境

    ```bash
    # 安装 Python 3.11.5
    pyenv install 3.11.5
    # 创建虚拟环境
    pyenv virtualenv 3.11.5 superset
    # 使虚拟环境生效，注意，执行前先用 pwd 确认当前目录为刚下载的 superset 目录
    pyenv local superset
    ```

- 升级 `pip` 和 `setuptools`

    ```bash
    pip install --upgrade setuptools pip
    ```

- 设置环境变量

    ```bash
    export LDFLAGS="-L$(brew --prefix openssl)/lib"
    export CFLAGS="-I$(brew --prefix openssl)/include"
    ```

## 2. 安装并初始化 Superset

- 安装 `apache-superset` 包，版本为 3.0.0

    ```bash
    pip install apache-superset=='3.0.0'
    ```

- 创建配置文件目录：`mkdir pythonpath`

- 创建配置文件：`vim pythonpath/superset_config.py`

    复制以下配置信息到文件中

    ```bash
    # Superset specific config
    ROW_LIMIT = 5000
    
    # Flask App Builder configuration
    # Your App secret key will be used for securely signing the session cookie
    # and encrypting sensitive information on the database
    # Make sure you are changing this key for your deployment with a strong key.
    # Alternatively you can set it with `SUPERSET_SECRET_KEY` environment variable.
    # You MUST set this for production environments or the server will not refuse
    # to start and you will see an error in the logs accordingly.
    SECRET_KEY = 'vlYT3V/KSmJaCb/1rpMb5dqNuNuyQEkH9ae+sSVAaZLdzpxjrJyK0GGj'
    
    # The SQLAlchemy connection string to your database backend
    # This connection defines the path to the database that stores your
    # superset metadata (slices, connections, tables, dashboards, ...).
    # Note that the connection information to connect to the datasources
    # you want to explore are managed directly in the web UI
    SQLALCHEMY_DATABASE_URI = 'mysql://root:123456@localhost:3306/superset'
    
    # Flask-WTF flag for CSRF
    WTF_CSRF_ENABLED = True
    # Add endpoints that need to be exempt from CSRF protection
    WTF_CSRF_EXEMPT_LIST = []
    # A CSRF token that expires in 1 year
    WTF_CSRF_TIME_LIMIT = 60 * 60 * 24 * 365
    
    # Set this API key to enable Mapbox visualizations
    MAPBOX_API_KEY = ''
    ```
    
    **注意：**
    1. `SECRET_KEY` 是通过 `openssl rand -base64 42` 生成的
    1. `SQLALCHEMY_DATABASE_URI` 可以注释掉，那么默认会使用内置的 `sqlite` 数据库，这里是连接了本机已经启动的 `MySQL` 数据库，且已经创建了 `superset` 库

- 把刚才的配置文件添加到 `PYTHONPATH` 中

    ```bash
    export PYTHONPATH=$(pwd)/pythonpath:$PYTHONPATH
    ```

- 如果刚才使用了 `MySQL`，需要按安装 `mysqlclient` 包

    ```bash
    pip install mysqlclient
    ```

- 初始化数据库

    ```bash
    superset db upgrade
    ```

- 创建 `Admin` 用户

    ```bash
    export FLASK_APP=superset
    superset fab create-admin --username admin --firstname Superset --lastname Admin --email admin@superset.com --password admin
    ```

- 创建 `roles` 和相应的权限

    ```bash
    superset init
    ```

- 加载样例数据，这个要看网络情况，如果下载不了可以跳过

    ```bash
    superset load_examples
    ```

## 3. 前端编译

- **如果没有修改前端代码的话可以跳过这一步，直接启动**

- 安装 `Node.js`，`npm` 会一起安装

    ```bash
    brew install node
    ```

- 编译前端代码，这一步需要的时间比较久

    ```bash
    cd superset-frontend
    npm ci
    npm run build
    cd ..
    ```

## 4. 启动 Superset

执行以下命令，访问 `http://localhost:8088/` 即可

```bash
superset run -p 8088 --with-threads --reload --debugger
```

之后在每次启动时，都需要重新指定 `PYTHONPATH`，才可以启动

```bash
export PYTHONPATH=$(pwd)/pythonpath:$PYTHONPATH
superset run -p 8088 --with-threads --reload --debugger
```
