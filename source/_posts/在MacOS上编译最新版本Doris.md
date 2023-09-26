---
title: 在 Mac OS 上编译最新版本 Doris
categories:
  - 实时数仓
tags:
  - Doris
date: 2023-09-23 15:33:31
id: doris-built-on-mac
---

Doris 是基于 MPP 架构的 OLAP 数据库，可以用做企业级的实时数仓。本文主要讲解了如何在 Mac OS 上编译 Doris 源码，一是用于本地的测试环境，二是方便自己了解和学习 Doris 的源码

<!-- more -->

>**系统环境：**
>**操作系统：**Mac OS 13.5.2
>**芯片：**M2
>**Java：**11，推荐用 11，Java 1.8 亲测编译时会出现堆内存溢出的问题
>**Python：**3.11

## 1. 下载工程

```bash
git clone https://github.com/apache/doris.git
cd doris
```

## 2. 安装依赖

- 使用 `brew` 安装依赖包

    ```bash
    brew install automake autoconf libtool pkg-config texinfo coreutils gnu-getopt cmake ninja ccache bison byacc gettext wget pcre maven llvm@16 npm
    ```

    如果没有事先安装 `Java` 和 `Python` 环境，还需要执行下面语句安装

    ```bash
    brew install python@3 openjdk@11
    
    # 设置 Java 11 的软链接
    sudo ln -sfn /opt/homebrew/opt/openjdk@11/libexec/openjdk.jdk ~/Library/Java/JavaVirtualMachines/openjdk-11.jdk
    # 在当前会话中启用 Java 11
    export JAVA_HOME="~/Library/Java/JavaVirtualMachines/openjdk-11.jdk/Contents/Home"
    ```

- 安装第三方依赖

    ```bash
    cd thirdparty
    rm -rf installed
    # 下载依赖
    curl -L https://github.com/apache/doris-thirdparty/releases/download/automation/doris-thirdparty-prebuilt-darwin-arm64.tar.xz -o - | tar -Jxf -
    # 保证protoc和thrift能够正常运行
    cd installed/bin
    ./protoc --version
    ./thrift --version
    # 回到 doris 根目录
    cd ../../../
    ```

## 3. 编译

编译时间比较长，需要耐心等待

```bash
bash build.sh
```

## 4. 启动服务

- 设置 `file descriptors`

    ```bash
    echo 'ulimit -n 65536' >> ~/.zshrc
    ```

    使配置生效，并检查输出是否为 `65536`

    ```bash
    source ~/.zshrc
    ulimit -n
    ```

- 启动 `BE`

    ```bash
    cd output/be
    bin/start_be.sh --daemon
    ```

    如果出现 `bin/start_be.sh: line 86: swapon: command not found` 的报错，可以不用管，`BE` 进程也会正常启动

- 启动 `FE`

    ```bash
    cd ../fe
    bin/start_fe.sh --daemon
    ```

- 检查 `BE`,  `FE` 进程是否正常启动

    ```bash
    jps
    ```

    如果输出包含 `DorisBE` 和 `DorisFE` 进程，说明启动成功

    ![image-20230926135420783](./image/在MacOS上编译最新版本Doris/image-20230926135420783.png)

- 将 `BE` 节点添加到集群，如果本地没有 `MySQL` 客户端，需要先安装 `brew install mysql`

    ```bash
    mysql -h127.0.0.1 -P9030 -uroot -p
    
    mysql> alter system add backend "127.0.0.1:9050";
    
    检查 be 和 fe 节点，如果两者的 Alive 字段均为 true，说明两个节点都能正常工作
    mysql> show frontends;
    mysql> show backends;
    ```

## 5. 参考

- 官方文档：https://doris.apache.org/zh-CN/docs/dev/install/source-install/compilation-mac
