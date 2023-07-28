---
title: DBApi 如何进行本地调试
date: 2023-07-28 17:38:14
tags:
  - DBApi
categories:
  - 后端开发
---

在使用 `DBApi` 的过程中，可能会需要开发插件，以解决业务问题。
在开发插件前，首先要调通本地的开发环境，本篇会介绍如何编译 `DBApi` 源码，并在 `Intellij IDEA` 中启动服务


<!-- more -->

## 1. 编译

- 下载源码
    ```bash
    git clone https://github.com/freakchick/DBApi.git
    cd DBApi
    ```
- 确认安装环境是否OK
    ```bash
    # 是否安装Java
    echo $JAVA_HOME
    # 是否安装npm
    npm -v
    # 是否安装node
    node -v
    # npm设置国内源
    npm config set registry https://registry.npm.taobao.org
    ```
- 使用 `idea` 打开项目，进入 `dbapi-ui`，修改 `pom.xml`
    如下图所示，增加一行参数，否则在编译`dbapi-ui`时会报错
    ```bash
    <argument>--legacy-peer-deps</argument>
    ```
    ![截屏2023-07-26 17.36.29](./image/DBApi如何进行本地调试/截屏2023-07-26-17.36.29.png)
- 在项目根目录下用命令行编译
    ```bash
    mvn clean package -P release
    ```



## 2. 配置并启动服务

- 编译完成后，找到下图中的文件：`dbapi-service/src/main/resources/application.properties`，修改配置信息
    配置参数可以参考本篇文章：[DBApi 本地和 Docker 安装方法](https://wangjiapengone.github.io/DBAPI%E6%9C%AC%E5%9C%B0%E5%92%8CDocker%E5%AE%89%E8%A3%85%E6%96%B9%E6%B3%95/#1-%E6%9C%AC%E5%9C%B0%E5%8D%95%E6%9C%BA%E7%89%88%E5%AE%89%E8%A3%85)
    ![截屏2023-07-26 17.42.12](./image/DBApi如何进行本地调试/截屏2023-07-26-17.42.12.png)
- 再参照安装的步骤，创建数据库并初始化
    初始化文件在 `dbapi-assembly/sql` 目录下
- 准备工作完成，在 `idea` 下启动本地 `standalone` 服务
    启动类的位置：`dbapi-standalone/src/main/java/com/gitee/freakchicken/dbapi/DBApiStandalone.java`
    ![image-20230728175413171](./image/DBApi如何进行本地调试/image-20230728175413171.png)




## 3. 参考

- https://www.51dbapi.com/v4.0.0/zh/develop/
