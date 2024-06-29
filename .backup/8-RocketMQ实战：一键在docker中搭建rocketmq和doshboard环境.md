[RocketMQ实战：一键在docker中搭建rocketmq和doshboard环境](https://github.com/denglei1024/denglei1024.github.io/issues/8)

在本篇博客中，我们将详细介绍如何在 Docker 环境中一键部署 RocketMQ 和其 Dashboard。这个过程基于一个预配置的 Docker Compose 文件，使得部署变得简单高效。

### 项目介绍

该项目提供了一套 Docker Compose 配置，用于快速部署 RocketMQ 及其 Dashboard。项目包含必要的配置文件和目录结构，以确保数据持久化和服务的正常运行。

### 快速开始指南

#### 第一步：克隆仓库

首先，克隆项目仓库到您的本地环境中：

```bash
git clone git@github.com:denglei1024/docker-rocketmq-dashboard.git
cd docker-rocketmq-dashboard
```

#### 第二步：修改配置文件

在启动容器之前，您可能需要根据实际需求修改 `conf/broker.conf` 配置文件。该文件包含 RocketMQ Broker 的相关配置。

#### 第三步：启动容器

使用以下命令启动 Docker 容器：

```bash
docker-compose -p rocketmql_project up -d
```

该命令将以分离模式启动所有容器。

#### 第四步：访问 Dashboard

容器启动完成后，您可以通过以下地址访问 RocketMQ Dashboard：

[http://localhost:8080/](http://localhost:8080)

### 项目结构

该项目的目录结构如下：

- `config/`: 包含 RocketMQ 的配置文件
  - `broker.conf`: Broker 配置文件
- `data/`: 数据持久化目录
  - `broker/`: Broker 数据和日志
  - `namesrv/`: Namesrv 数据和日志

![RocketMQ Dashboard](https://github.com/denglei1024/docker-rocketmq-dashboard/assets/16712298/6381ad11-2a18-4302-877a-f947bfa0fbb4)

### 结语

通过以上步骤，您可以快速部署并运行 RocketMQ 及其 Dashboard。该项目的 Docker Compose 配置简化了部署过程，使得开发和测试变得更加高效。
