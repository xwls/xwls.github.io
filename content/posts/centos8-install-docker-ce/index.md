---
title: CentOS8 安装 Docker CE
date: 2022-10-15 09:56:28+08:00
categories:
- 云原生
tags:
- 运维
- centos
- linux
- docker
summary: 本步骤指导您如何在 ECS 实例上部署 Docker。
---

1. 执行如下命令，安装 dnf。dnf 是新一代的 rpm 软件包管理器。

    ```bash
    yum -y install dnf
    ```

2. 执行如下命令，安装 Docker 存储驱动的依赖包。

    ```bash
    dnf install -y device-mapper-persistent-data lvm2
    ```

3. 执行如下命令，添加稳定的 Docker 软件源。

    ```bash
    yum makecache fast
    yum -y install docker-ce
    ```

4. 执行如下命令，查看已添加的 Docker 软件源。

    ```bash
    dnf list docker-ce
    ```

    ![202210151002228](https://xwls-oss.netlify.app/images/202210151002228.webp)

5. 执行如下命令，安装 docker-ce。

    ```bash
    dnf install -y docker-ce --nobest
    ```

6. 执行如下命令，启动 Docker 服务。

    ```bash
    systemctl start docker
    ```

7. 执行如下命令，查看 Docker 服务的运行状态。

    ```bash
    systemctl status docker
    ```

    返回结果如下，表示 Docker 服务处于运行中的状态。按 q 键退出查看 Docker 服务的运行状态。

    ![202210151005814](https://xwls-oss.netlify.app/images/202210151005814.webp)

8. 执行如下命令，查看 Docker 的版本。

    ```bash
    docker -v
    ```

    返回结果如下，您可查看到 Docker 的版本。

    ![202210151008931](https://xwls-oss.netlify.app/images/202210151008931.webp)