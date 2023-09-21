---
title: 在 Kubesphere 上安装 Superset 3.0.0
date: 2023-09-21 10:03:38
id: superset-3-kubesphere-install
tags: 
  - Superset
categories:
  - 可视化
---

Kubesphere 是企业级的开源容器平台，Superset 是轻量级的 BI 工具。使用 Kubesphere 可以很方便地部署 Superset 的线上环境，本文记录了 Superset 新发布的 3.0.0 版本在 Kubesphere 上的部署过程

<!-- more -->

## 1. 在 Kubesphere 上创建工作负载

### 1.1 填写名称

![image-20230830165920411](./image/在Kubesphere上安装Superset-3.0.0/image-20230830165920411.png)

### 1.2 设置镜像

镜像使用官方提供的 `apache/superset:3.0.0` 版本

![image-20230920153225486](./image/在Kubesphere上安装Superset-3.0.0/image-20230920153225486.png)

### 1.3 设置环境变量

- `SUPERSET_SECRET_KEY` 是必须提供的参数，可以使用 `openssl rand -base64 42` 来生成一个值
- `TALISMAN_ENABLED` 需要设置为 `False`，否则可能会出现无法登录的情况，关于这个问题的讨论可以看这个 [issue](https://github.com/apache/superset/issues/24579)

![image-20230920153425701](./image/在Kubesphere上安装Superset-3.0.0/image-20230920153425701.png)

### 1.4 点击创建

存储设置和高级设置不用管，直接创建

![image-20230920153733346](./image/在Kubesphere上安装Superset-3.0.0/image-20230920153733346.png)

点击刚创建好的容器实例

![image-20230920154145920](./image/在Kubesphere上安装Superset-3.0.0/image-20230920154145920.png)

可以看到节点的 IP 地址为 `172.16.33.114`，记住这个地址方便之后访问

![image-20230920154446936](./image/在Kubesphere上安装Superset-3.0.0/image-20230920154446936.png)

## 2. 在 Kubesphere 上创建服务

### 2.1 创建服务

选择指定已有的工作负载来创建服务

![image-20230830171535196](./image/在Kubesphere上安装Superset-3.0.0/image-20230830171535196.png)

### 2.2 服务名称

注意，服务名称不能为 `superset`, 否则会导致镜像无法启动

![image-20230830171652574](./image/在Kubesphere上安装Superset-3.0.0/image-20230830171652574.png)

### 2.3 指定工作负载

指定工作负载为刚才创建的 `superset-test`, 同时修改容器端口和服务端口为默认的 `8088`

![image-20230830172026142](./image/在Kubesphere上安装Superset-3.0.0/image-20230830172026142.png)

### 2.4 选择外部访问方式

设置为 `NodePort`，点击创建

![image-20230830172206791](./image/在Kubesphere上安装Superset-3.0.0/image-20230830172206791.png)

可以看到外部访问的服务端口为 `30648`

![image-20230830172335163](./image/在Kubesphere上安装Superset-3.0.0/image-20230830172335163.png)

## 3. 初始化 Superset 服务

和之前的版本不同，`3.0.0` 版本需要进入容器中对服务进行初始化
首先进入在第一步中创建好的容器组，点击容器的终端图标，进入命令行界面

![image-20230920155730762](./image/在Kubesphere上安装Superset-3.0.0/image-20230920155730762.png)

依次执行以下命令

```bash
# 创建 admin 用户
superset fab create-admin --username admin --firstname Superset --lastname Admin --email admin@superset.com --password admin
# 更新本地数据库
superset db upgrade
# 设置用户角色
superset init
```

最后把上面获得的 IP 和端口组合起来，就可以访问 Superset 服务：`http://172.16.33.114:30648/login/`
