+++
title = 'MacOS 使用 minikube 搭建 Kubernetes 环境'
date = 2022-10-19T16:14:16+08:00
categories = ['云原生']
tags = ['k8s', 'docker', 'mac']
summary = 'MacOS 使用 minikube 搭建 Kubernetes 环境。'
+++

在 MacOS 下，使用 minikube 搭建 Kubernetes 环境，遇到了大量的问题，包括 [kubelet-check] Initial timeout of 40s passed.、拉取镜像失败、dashboard 启动失败，等等。

## 安装 minikube

* minikube 官网：[https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)

安装时最开始使用的是 brew 进行安装，brew 安装有个问题，会默认安装依赖项 kubernetes-cli，安装的版本比较新，会有奇奇怪怪的问题导致初始化时报错，后来删除 brew 安装的 minikube 和 kubernetes-cli，使用 binary 安装。

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

## 启动 minikube

* 启动失败解决：[https://www.jeeinn.com/2022/07/1715/](https://www.jeeinn.com/2022/07/1715/)

启动时可以使用`--image-mirror-country='cn'`指定镜像为中国镜像，这样拉取镜像会快很多，顺便使用`--kubernetes-version=v1.23.8`指定版本。

```bash
minikube start --image-mirror-country='cn' --kubernetes-version=v1.23.8
```

## 测试

```bash
minikube kubectl -- get po -A
```

## 解决镜像拉取失败的问题

* 解决方案：[https://blog.csdn.net/w757227129/article/details/123512692](https://blog.csdn.net/w757227129/article/details/123512692)

使用`minikube ssh`进入 minikube 的 docker 节点

执行 touch /etc/docker/daemon.json

将以下配置添加到 daemon.json 文件中

```json
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]
}
```

然后重启 minikube

再次运行 kubectl create 创建资源的时候，镜像会被成功拉取