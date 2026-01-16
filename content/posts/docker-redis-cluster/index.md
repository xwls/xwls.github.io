---
title: Docker 搭建 Redis 集群
date: 2022-10-18 16:15:27+08:00
categories:
- 环境搭建
tags:
- redis
- docker
- 集群
summary: 使用 docker 创建 redis 集群。
---

## 拉取 redis 镜像

```bash
docker pull redis
```

## 创建 redis 配置文件

```bash
for port in $(seq 6379 6384); 
do 
mkdir -p /home/redis/node-${port}/conf
touch /home/redis/node-${port}/conf/redis.conf
cat  << EOF > /home/redis/node-${port}/conf/redis.conf
port ${port}
requirepass 1234
bind 0.0.0.0
protected-mode no
daemonize no
appendonly yes
cluster-enabled yes 
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip  服务器就填公网 ip，或者内部对应容器的 ip
cluster-announce-port ${port}
cluster-announce-bus-port 1${port}
EOF
done
```

* 效果

```bash
[root@iZuf6brvjzmnf06szlrk7oZ ~]# cd /home/redis/
[root@iZuf6brvjzmnf06szlrk7oZ redis]# tree
.
├── node-6379
│   └── conf
│       └── redis.conf
├── node-6380
│   └── conf
│       └── redis.conf
├── node-6381
│   └── conf
│       └── redis.conf
├── node-6382
│   └── conf
│       └── redis.conf
├── node-6383
│   └── conf
│       └── redis.conf
└── node-6384
    └── conf
        └── redis.conf

12 directories, 6 files
```

## 启动 redis

```bash
for port in $(seq 6379 6384); \
do \
   docker run -it -d -p ${port}:${port} -p 1${port}:1${port} \
  --privileged=true -v /home/redis/node-${port}/conf/redis.conf:/usr/local/etc/redis/redis.conf \
  --privileged=true -v /home/redis/node-${port}/data:/data \
  --restart always --name redis-${port} --net myredis \
  --sysctl net.core.somaxconn=1024 redis redis-server /usr/local/etc/redis/redis.conf
done
```

* 效果

```bash
[root@iZuf6brvjzmnf06szlrk7oZ redis]# for port in $(seq 6379 6384); \
> do \
>    docker run -it -d -p ${port}:${port} -p 1${port}:1${port} \
>   --privileged=true -v /home/redis/node-${port}/conf/redis.conf:/usr/local/etc/redis/redis.conf \
>   --privileged=true -v /home/redis/node-${port}/data:/data \
>   --restart always --name redis-${port} --net myredis \
>   --sysctl net.core.somaxconn=1024 redis redis-server /usr/local/etc/redis/redis.conf
> done
c456167625319a7177036b09d516ff0813a31d61dbb24273ce1d1ff041f3f621
3fc3c14204e9c9b2e80211f13c68a8cc8e1ee65c26683078453ec2052f0b6b53
b04bb20f74e5e418e5588f1cc67e5aaf1a66e210f86f7ee1b433c489b157da0b
5a501f0f18a9891246aa70cfc0f770f6e311ad6888004cb46e59f24185f59991
13d7a9dcdc44d74592b25c12ea8e5d204bbd25200231f8fe9a55e5eac7803326
ad3e265183d8815e29d19d488bb357d6fb931949eda46dbfc95018b32e51cf85
```

## 创建集群

```bash
# 进入容器 bash
docker exec -it redis-6379 /bin/bash
# 创建集群
redis-cli  -a 1234 --cluster create 47.103.58.89:6379 47.103.58.89:6380 47.103.58.89:6381 47.103.58.89:6382 47.103.58.89:6383 47.103.58.89:6384 --cluster-replicas 1
```

* 效果

```bash
[root@iZuf6brvjzmnf06szlrk7oZ redis]# docker exec -it redis-6379 /bin/bash
root@c45616762531:/data# redis-cli  -a 1234 --cluster create 47.103.58.89:6379 47.103.58.89:6380 47.103.58.89:6381 47.103.58.89:6382 47.103.58.89:6383 47.103.58.89:6384 --cluster-replicas 1
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 47.103.58.89:6383 to 47.103.58.89:6379
Adding replica 47.103.58.89:6384 to 47.103.58.89:6380
Adding replica 47.103.58.89:6382 to 47.103.58.89:6381
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 3d1df32edcb46e9d2f0e9c6b4b0c68baf99ca32d 47.103.58.89:6379
   slots:[0-5460] (5461 slots) master
M: 8e833d82ff85892646552e4cfda9f5d024ce7dc9 47.103.58.89:6380
   slots:[5461-10922] (5462 slots) master
M: 535cd50270c353ca0e9196e09203d846788cf67a 47.103.58.89:6381
   slots:[10923-16383] (5461 slots) master
S: af331e16ecd61119e6b627fbd27590f3ffaae26a 47.103.58.89:6382
   replicates 3d1df32edcb46e9d2f0e9c6b4b0c68baf99ca32d
S: 34067385c2b8152575b83428d0cdcb50e1b837d5 47.103.58.89:6383
   replicates 8e833d82ff85892646552e4cfda9f5d024ce7dc9
S: fdca52748b918280deaa05bce48a214f4a365041 47.103.58.89:6384
   replicates 535cd50270c353ca0e9196e09203d846788cf67a
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
..
>>> Performing Cluster Check (using node 47.103.58.89:6379)
M: 3d1df32edcb46e9d2f0e9c6b4b0c68baf99ca32d 47.103.58.89:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: af331e16ecd61119e6b627fbd27590f3ffaae26a 47.103.58.89:6382
   slots: (0 slots) slave
   replicates 3d1df32edcb46e9d2f0e9c6b4b0c68baf99ca32d
S: fdca52748b918280deaa05bce48a214f4a365041 47.103.58.89:6384
   slots: (0 slots) slave
   replicates 535cd50270c353ca0e9196e09203d846788cf67a
S: 34067385c2b8152575b83428d0cdcb50e1b837d5 47.103.58.89:6383
   slots: (0 slots) slave
   replicates 8e833d82ff85892646552e4cfda9f5d024ce7dc9
M: 535cd50270c353ca0e9196e09203d846788cf67a 47.103.58.89:6381
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: 8e833d82ff85892646552e4cfda9f5d024ce7dc9 47.103.58.89:6380
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

## 查看集群信息

```bash
# 连接 redis
redis-cli -c -a 1234
# 查看集群信息
CLUSTER INFO
# 查看节点信息
CLUSTER NODES
```

* 效果

```bash
127.0.0.1:6379> CLUSTER INFO
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_ping_sent:81
cluster_stats_messages_pong_sent:90
cluster_stats_messages_sent:171
cluster_stats_messages_ping_received:85
cluster_stats_messages_pong_received:79
cluster_stats_messages_meet_received:5
cluster_stats_messages_received:169
total_cluster_links_buffer_limit_exceeded:0
127.0.0.1:6379> CLUSTER NODES
af331e16ecd61119e6b627fbd27590f3ffaae26a 47.103.58.89:6382@16382 slave 3d1df32edcb46e9d2f0e9c6b4b0c68baf99ca32d 0 1666080700700 1 connected
3d1df32edcb46e9d2f0e9c6b4b0c68baf99ca32d 47.103.58.89:6379@16379 myself,master - 0 1666080699000 1 connected 0-5460
fdca52748b918280deaa05bce48a214f4a365041 47.103.58.89:6384@16384 slave 535cd50270c353ca0e9196e09203d846788cf67a 0 1666080700000 3 connected
34067385c2b8152575b83428d0cdcb50e1b837d5 47.103.58.89:6383@16383 slave 8e833d82ff85892646552e4cfda9f5d024ce7dc9 0 1666080700820 2 connected
535cd50270c353ca0e9196e09203d846788cf67a 47.103.58.89:6381@16381 master - 0 1666080700644 3 connected 10923-16383
8e833d82ff85892646552e4cfda9f5d024ce7dc9 47.103.58.89:6380@16380 master - 0 1666080699697 2 connected 5461-10922