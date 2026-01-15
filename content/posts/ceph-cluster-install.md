---
title: Ubuntu 22.04 安装 ceph 集群
date: 2023-02-16 10:43:34+08:00
categories:
- 云原生
tags:
- k8s
summary: 在 Ubuntu 22.04 使用 cephadm 安装 ceph 集群，并配置为 k8s 的存储类。
---

## 准备

准备了 3 台服务器用于安装 ceph 集群，服务器信息如下：

| 主机名     | IP            | 配置          | 磁盘                     |
| ---------- | ------------- | ------------- | ------------------------ |
| ceph-node1 | 192.168.56.31 | 2 vCPU；2 GiB | 系统盘 40G + 数据盘 6G * 2 |
| ceph-node2 | 192.168.56.32 | 2 vCPU；2 GiB | 系统盘 40G + 数据盘 6G * 2 |
| Ceph-node3 | 192.168.56.33 | 2 vCPU；2 GiB | 系统盘 40G + 数据盘 6G * 2 |

## 修改 hosts 文件

修改所有服务器的 hosts 文件

```shell
vim /etc/hosts
192.168.56.31 ceph-node1
192.168.56.32 ceph-node2
192.168.56.33 ceph-node3
```

## 时间同步

所有服务器全部执行：

```shell
# 启用时间同步
timedatectl set-ntp true
# 设置时区 Asia/Shanghai
timedatectl set-timezone Asia/Shanghai
# 查看状态
timedatectl status
```

## 安装 Docker

* 使用 apt 安装 docker-ce，所有服务器全部安装

```shell
apt update
apt install docker-ce -y
```

* 配置 docker 国内镜像地址

```shell
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://y037t07z.mirror.aliyuncs.com"]
}
EOF
systemctl restart docker
```

## SSH 免密登录

开启 ssh 免密登录，所有服务器都需要执行

```shell
# 生成 ssh 密钥
ssh-keygen -t rsa
# 拷贝密钥
ssh-copy-id ceph-node1 
ssh-copy-id ceph-node2
ssh-copy-id ceph-node3
# 测试
```

## 使用 cephadm 安装 ceph 集群

```shell
# 安装 cephadm
apt install cephadm -y
# 启用集群
cephadm bootstrap --mon-ip 192.168.56.31
# 安装 ceph-cli
apt install ceph-common -y
# 集群状态信息
ceph -s
# 查看节点信息
ceph orch host ls
# 拷贝公钥
ssh-copy-id -f -i /etc/ceph/ceph.pub ceph-node2
ssh-copy-id -f -i /etc/ceph/ceph.pub ceph-node3
# 添加节点
ceph orch host add ceph-node2
ceph orch host add ceph-node3
# 查看节点信息
ceph orch host ls
# 添加所有可用磁盘到集群
ceph orch apply osd --all-available-devices
# 查看 osd 状态
ceph osd tree
```

## ceph dashboard

[https://192.168.56.31:8443](https://192.168.56.31:8443)

![dashboard](https://xwls-oss.netlify.app/images/202302161143406.webp)

![hosts](https://xwls-oss.netlify.app/images/202302161144624.webp)

![osds](https://xwls-oss.netlify.app/images/202302161333204.webp)

```
创建一个用户
ceph auth get-or-create client.kubernetes mon 'profile rbd' osd 'profile rbd pool=kube' mgr 'profile rbd pool=kube'

[client.kubernetes]
  key = AQDpuN1jIlPBABAAaGhYtIUSonIiHirrM9SFkQ==

ceph mon dump

epoch 3
fsid 4e38b228-a375-11ed-bb7f-1f327548c6fd
last_changed 2023-02-03T03:56:20.328724+0000
created 2023-02-03T03:47:49.394295+0000
min_mon_release 17 (quincy)
election_strategy: 1
0: [v2:192.168.56.31:3300/0,v1:192.168.56.31:6789/0] mon.ceph-node1
1: [v2:192.168.56.32:3300/0,v1:192.168.56.32:6789/0] mon.ceph-node2
2: [v2:192.168.56.33:3300/0,v1:192.168.56.33:6789/0] mon.ceph-node3

csi-config-map.yaml

apiVersion: v1
kind: ConfigMap
data:
  config.json: |-
    [
      {
        "clusterID": "4e38b228-a375-11ed-bb7f-1f327548c6fd",
        "monitors": [
          "192.168.56.31:6789",
          "192.168.56.32:6789",
          "192.168.56.33:6789"
        ]
      }
    ]
metadata:
  name: ceph-csi-config

kubectl apply -f csi-config-map.yaml

csi-kms-config-map.yaml

apiVersion: v1
kind: ConfigMap
data:
  config.json: |-
    {}
metadata:
  name: ceph-csi-encryption-kms-config

kubectl apply -f csi-kms-config-map.yaml

ceph-config-map.yaml

apiVersion: v1
kind: ConfigMap
data:
  ceph.conf: |
    [global]
    auth_cluster_required = cephx
    auth_service_required = cephx
    auth_client_required = cephx

# keyring is a required key and its value should be empty

  keyring: |
metadata:
  name: ceph-config

kubectl apply -f ceph-config-map.yaml

csi-rbd-secret.yaml

apiVersion: v1
kind: Secret
metadata:
  name: csi-rbd-secret
  namespace: default
stringData:
  userID: kubernetes
  userKey: AQDpuN1jIlPBABAAaGhYtIUSonIiHirrM9SFkQ==

kubectl apply -f csi-rbd-secret.yaml

$ kubectl apply -f <https://raw.githubusercontent.com/ceph/ceph-csi/master/deploy/rbd/kubernetes/csi-provisioner-rbac.yaml>
$ kubectl apply -f <https://raw.githubusercontent.com/ceph/ceph-csi/master/deploy/rbd/kubernetes/csi-nodeplugin-rbac.yaml>

$ wget <https://raw.githubusercontent.com/ceph/ceph-csi/master/deploy/rbd/kubernetes/csi-rbdplugin-provisioner.yaml>
$ kubectl apply -f csi-rbdplugin-provisioner.yaml
$ wget <https://raw.githubusercontent.com/ceph/ceph-csi/master/deploy/rbd/kubernetes/csi-rbdplugin.yaml>
$ kubectl apply -f csi-rbdplugin.yaml

csi-rbd-sc.yaml

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: csi-rbd-sc
provisioner: rbd.csi.ceph.com
parameters:
   clusterID: b9127830-b0cc-4e34-aa47-9d1a2e9949a8
   pool: kubernetes
   imageFeatures: layering
   csi.storage.k8s.io/provisioner-secret-name: csi-rbd-secret
   csi.storage.k8s.io/provisioner-secret-namespace: default
   csi.storage.k8s.io/controller-expand-secret-name: csi-rbd-secret
   csi.storage.k8s.io/controller-expand-secret-namespace: default
   csi.storage.k8s.io/node-stage-secret-name: csi-rbd-secret
   csi.storage.k8s.io/node-stage-secret-namespace: default
reclaimPolicy: Delete
allowVolumeExpansion: true
mountOptions:

* discard
