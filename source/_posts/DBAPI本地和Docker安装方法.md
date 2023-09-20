---
title: DBApi 本地和 Docker 安装方法
id: dbapi-docker-install
date: 2023-07-28 16:33:09
tags:
  - DBApi
categories:
  - 后端开发
---

本人是数据端开发，最近在工作中遇到需要向前端后端提供数据服务的问题，各端之间交互最好是提供数据接口，但我们人手有限，无力再维护一套 `Java` 服务，同时数据端吐出的数据一般业务逻辑都不复杂，用 `SQL` 就可以搞定，所以更没有开发 `Java` 服务的必要

所以我们调研了一些可以通过 `SQL` 直接生成接口的开源项目，最后觉得`DBApi`是一个比较理想的选择，其优点在于：
1. 零代码：直接在页面上写 `SQL` 就可以直接生成接口，不需要开发代码，虽然页面用起来有一些不太合理的地方，但是功能比较齐全，足够应付 `SQL` 开发
2. 目前仍在维护：目前出到 `4.0.0`，文档还算比较全，看文档安装、本地调试都没有太大问题，同时源码有注释，大部分都能看懂


<!-- more -->
<br>

>**本机环境：**
>操作系统：Mac OS 13.4.1
>MySQL：8.0.33


## 1. 本地单机版安装

- 下载安装包
    ```bash
    wget https://github.com/freakchick/DBApi/releases/download/v4.0.0/dbapi-4.0.0-bin.tar.gz
    ```
- 解压
    ```bash
    tar zvxf dbapi-4.0.0-bin.tar.gz
    ```
- 确认是否安装 `Java 8+` 环境
    ```bash
    echo $JAVA_HOME
    ```
- 修改配置文件
    ```bash
    cd dbapi-4.0.0
    vim conf/application.properties
    ```
    按如下配置项修改，选择 `MySQL` 存储元数据和日志信息
    ```bash
    #################################### please configure properties below #####################################
    # api context
    dbapi.api.context=api
    
    # metadata database address
    # spring.datasource.dynamic.datasource.meta-db.driver-class-name=org.sqlite.JDBC
    # spring.datasource.dynamic.datasource.meta-db.url=jdbc:sqlite::resource:sqlite.db
    # spring.datasource.dynamic.datasource.meta-db.username=
    # spring.datasource.dynamic.datasource.meta-db.password=
    
    # metadata database address
    spring.datasource.dynamic.datasource.meta-db.driver-class-name=com.mysql.cj.jdbc.Driver
    spring.datasource.dynamic.datasource.meta-db.url=jdbc:mysql://localhost:3306/dbapi?useSSL=false&characterEncoding=UTF-8&serverTimezone=GMT%2B8
    spring.datasource.dynamic.datasource.meta-db.username=root
    spring.datasource.dynamic.datasource.meta-db.password=123456
    
    
    # the writer to write access log to database, value can be null/db/kafka
    # "db" means dbapi writes access log to database directly
    # "kafka" means dbapi writes access log to kafka, you need to collect log from kafka to database yourself
    # "null" means dbapi only writes access log to disk file(logs/dbapi-access.log), you need to collect log from disk to database yourself
    access.log.writer=db
    
    # access log database(recommend clickhouse) address
    spring.datasource.dynamic.datasource.access-log-db.driver-class-name=com.mysql.cj.jdbc.Driver
    spring.datasource.dynamic.datasource.access-log-db.url=jdbc:mysql://localhost:3306/dbapi_log?useSSL=false&characterEncoding=UTF-8&serverTimezone=GMT%2B8
    spring.datasource.dynamic.datasource.access-log-db.username=root
    spring.datasource.dynamic.datasource.access-log-db.password=123456
    spring.datasource.dynamic.datasource.access-log-db.druid.break-after-acquire-failure=true
    
    # kafka address, needed if access.log.writer=kafka
    access.log.kafka.topic=dbapi_access_log
    spring.kafka.bootstrap-servers=127.0.0.1:9092
    ```
- 进入 `MySQL`，创建对应数据库
    ```
    mysql -hlocalhost -uroot -p
    ```
    创建数据库
    ```bash
    mysql> create database dbapi;
    mysql> create database dbapi_log;
    ```
    初始化数据库
    ```bash
    mysql> use dbapi;
    mysql> source sql/ddl_mysql.sql;
    mysql> use dbapi_log;
    mysql> source sql/access_log_mysql.sql;
    mysql> exit
    ```
- 启动服务
    ```bash
    sh bin/dbapi-daemon.sh start standalone
    ```
- 进入页面：http://localhost:8520 ，初始用户名/密码为：`admin` / `admin`
- 停止服务
    ```bash
    sh bin/dbapi-daemon.sh stop standalone
    ```




## 2. Docker 单机版安装

- 元数据库、日志数据库都选择本地的 `MySQL`
- 预先执行 `sql` 文件，创建对应的表，文件可以在容器中找到，注意需要预先在数据库中建好对应的库
    ```bash
    mysql -hlocalhost -P3306 -uroot -p123456 -Ddbapi < ddl_mysql.sql
    mysql -hlocalhost -P3306 -uroot -p123456 -Ddbapi_log < access_log_mysql.sql
    ```
- 运行容器
    ```bash
    docker run -it \
    -p 8520:8520 \
    -e DB_URL="jdbc:mysql://host.docker.internal:3306/dbapi?allowPublicKeyRetrieval=true&useSSL=false&characterEncoding=UTF-8&serverTimezone=GMT%2B8" \
    -e DB_USERNAME="root" \
    -e DB_PASSWORD="123456" \
    -e DB_DRIVER="com.mysql.cj.jdbc.Driver" \
    -e ACCESS_LOG_WRITER="db" \
    -e LOG_DB_URL="jdbc:mysql://host.docker.internal:3306/dbapi_log?allowPublicKeyRetrieval=true&useSSL=false&characterEncoding=UTF-8&serverTimezone=GMT%2B8" \
    -e LOG_DB_USERNAME="root" \
    -e LOG_DB_PASSWORD="123456" \
    -e LOG_DB_DRIVER="com.mysql.cj.jdbc.Driver" \
    --name dbapi \
    --add-host=host.docker.internal:host-gateway \
    freakchicken/db-api:4.0.0 standalone
    ```




## 3. 参考

- https://www.51dbapi.com/v4.0.0/zh/install/
