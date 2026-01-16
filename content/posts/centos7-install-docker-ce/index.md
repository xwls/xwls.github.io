---
title: CentOS7 安装 Docker CE
date: 2022-05-06 13:35:28+08:00
categories:
- 云原生
tags:
- 运维
- centos
- linux
- docker
summary: Docker 有两个分支版本：Docker CE 和 Docker EE，即社区版和企业版。本教程基于 CentOS 7 安装 Docker CE。
---

1. 执行如下命令，安装 Docker 的依赖库。

    ```bash
    yum install -y yum-utils device-mapper-persistent-data lvm2
    ```

2. 执行如下命令，添加 Docker CE 的软件源信息。

    ```bash
    yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    ```

3. 执行如下命令，安装 Docker CE。

    ```bash
    yum makecache fast
    yum -y install docker-ce
    ```

4. 执行如下命令，启动 Docker 服务。

    ```bash
    systemctl start docker
    ```

> Docker 的默认官方远程仓库是 hub.docker.com，由于网络原因，下载一个 Docker 官方镜像可能会需要很长的时间，甚至下载失败。为此，阿里云容器镜像服务 ACR 提供了官方的镜像站点，从而加速官方镜像的下载。下面介绍如何使用阿里云镜像仓库。

1. 复制容器镜像服务控制台地址，在 FireFox 浏览器打开新页签，粘贴并访问云容器镜像服务控制台。

    ```text
    https://cr.console.aliyun.com/
    ```

2. 在容器镜像服务控制台左侧导航栏中，选择镜像工具>镜像加速器。

    ![202210111337652](https://xwls-oss.netlify.app/images/202210111337652.webp)

3. 在镜像加速器页面的加速器区域，单击复制。

    ![202210111338648](https://xwls-oss.netlify.app/images/202210111338648.webp)

4. 切换至终端页面。执行如下命令配置 Docker 的自定义镜像仓库地址。请将下面命令中的镜像仓库地址`https://kqh8****.mirror.aliyuncs.com` 替换为上一步阿里云为您提供的专属镜像加速地址。

    ```bash
    tee /etc/docker/daemon.json <<-'EOF'
    {
    "registry-mirrors": ["https://kqh8****.mirror.aliyuncs.com"]
    }
    EOF
    ```

5. 重新加载服务配置文件。

    ```bash
    systemctl daemon-reload
    ```

6. 重启 Docker 服务。

    ```bash
    systemctl restart docker