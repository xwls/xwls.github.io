+++
title = 'MacOS 安装 Hadoop 3.3.4'
date = 2022-10-15T14:17:04+08:00
categories = ['大数据']
tags = ['环境搭建', '大数据', 'Hadoop', 'Mac']
summary = 'MacOS 12.6 安装 Hadoop 3.3.4 单机版。'
+++

## 下载

* Hadoop 官网：[https://hadoop.apache.org](https://hadoop.apache.org)
* 下载：[https://dlcdn.apache.org/hadoop/common/hadoop-3.3.4/](https://dlcdn.apache.org/hadoop/common/hadoop-3.3.4/)

![202210151420368](https://xwls-oss.netlify.app/images/202210151420368.webp)

## 配置

* 创建目录用来存放Hadoop

```bash
mkdir -p /Users/wangzengwei/Tools/hadoop-3.3.4
```

* 解压缩下载好的Hadoop文件

```bash
cd /Users/wangzengwei/Tools/hadoop-3.3.4
tar -zxvf hadoop-3.3.4.tar.gz
```

* 配置 hadoop-env.sh

```text
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_341.jdk/Contents/Home
export HADOOP_CONF_DIR=/Users/wangzengwei/Tools/hadoop-3.3.4/etc/hadoop
```

* 配置 core-site.xml

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <!--用来指定hadoop运行时产生文件的存放目录  自己创建-->
    <property>
        <name>hadoop.tmp.dir</name>
    <value>file:/Users/wangzengwei/Tools/hadoop-3.3.4/tmp</value>
    </property>
    <property>
        <name>fs.trash.interval</name>
        <value>1440</value>
    </property>
</configuration>
```

* 配置 hdfs-site.xml

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <!--不是root用户也可以写文件到hdfs-->
    <property>
        <name>dfs.permissions</name>
        <value>false</value>    <!--关闭防火墙-->
    </property>
    <!--把路径换成本地的name坐在位置-->
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/Users/wangzengwei/Tools/hadoop-3.3.4/namenodedir</value>
    </property>
    <!--在本地新建一个存放hadoop数据的文件夹，然后将路径在这里配置一下-->
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/Users/wangzengwei/Tools/hadoop-3.3.4/datanodedir</value>
    </property>
</configuration>
```

* 配置 mapred-site.xml

```xml
<configuration>
    <property>
        <!--指定mapreduce运行在yarn上-->
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

* 配置 yarn-site.xml

```xml
<configuration>
    <!-- Site specific YARN configuration properties -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>localhost:18040</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>localhost:18030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>localhost:18025</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>localhost:18141</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>localhost:18088</value>
    </property>
</configuration>
```

* Hadoop namenode 格式化

```bash
hdfs namenode -format
```

* 配置 ssh 免密登录

```bash
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

# 验证ssh 
ssh localhost
```

## 启动

```bash
cd /Users/wangzengwei/Tools/hadoop-3.3.4/sbin
./start-all.sh
```

## 验证

终端执行 jps 命令，在打印结果中会看到 5 个进程，分别是 namenode、 secondarynamenode、datanode、resourcemanager、nodemanager，说明启动成功

* 浏览器访问：[http://localhost:18088](http://localhost:18088)，查看Hadoop集群信息

![202210151435969](https://xwls-oss.netlify.app/images/202210151435969.webp)

* 浏览器访问：[http://localhost:9870](http://localhost:9870)，查看HDFS信息

![202210151436113](https://xwls-oss.netlify.app/images/202210151436113.webp)

## 停止

```bash
cd /Users/wangzengwei/Tools/hadoop-3.3.4/sbin
./stop-all.sh