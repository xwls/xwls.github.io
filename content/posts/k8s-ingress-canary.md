---
title: k8s 通过 Ingress 进行灰度发布
date: 2023-02-02 11:03:34+08:00
categories:
- 云原生
tags:
- k8s
- Ingress
- 灰度
summary: k8s 通过 Ingress 通过设置 权重/请求头 进行灰度发布。
---

## 部署 Deployment V1 应用

本步骤指导你如何部署 Deployment V1 应用，并使用 Ingress 实现 Deployment V1 应用的外部访问。

执行如下命令，创建名为 app-v1.yaml 的 YAML 文件。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-v1
  labels:
    app: my-app
spec:
  ports:
  - name: http
    port: 80
    targetPort: http
  selector:
    app: my-app
    version: v1.0.0
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v1
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
      version: v1.0.0
  template:
    metadata:
      labels:
        app: my-app
        version: v1.0.0
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9101"
    spec:
      containers:
      - name: my-app
        image: registry.cn-hangzhou.aliyuncs.com/containerdemo/containersol-k8s-deployment-strategies
        ports:
        - name: http
          containerPort: 8080
        - name: probe
          containerPort: 8086
        env:
        - name: VERSION
          value: v1.0.0
        livenessProbe:
          httpGet:
            path: /live
            port: probe
          initialDelaySeconds: 5
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /ready
            port: probe
          periodSeconds: 5
```

执行如下命令，部署 Deployement V1 应用。

```shell
kubectl apply -f app-v1.yaml
```

执行如下命令，创建名为 ingress-v1.yaml 的 Ingress YAML 文件。

将如下代码复制到文件中，并将 hots 参数中的`集群id`修改为`k8s集群id`。

> 说明：您可在云产品资源列表中查看 k8s 集群id。

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  rules:
  - host: my-app.集群id.cn-shanghai.alicontainer.com
    http:
      paths:
      - backend:
          service:
            name: my-app-v1
            port:
              number: 80
        path: /
        pathType: Prefix
```

执行如下命令，部署 Ingress 资源。

```shell
kubectl apply -f ingress-v1.yaml
```

执行如下命令，进行测试。

```shell
curl my-app.<集群id>.cn-shanghai.alicontainer.com
```

返回结果如下，表示您已成功访问到 Deployment V1 应用。

```text
Host: my-app-v1-6fc68db67d-xd2kf, Version: v1.0.0
```

## 部署 Deployment V2 应用

本步骤指导你如何部署 Deployment V2 应用。

执行如下命令，创建名为 app-v2.yaml 的 YAML 文件。将如下代码复制到文件中。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-v2
  labels:
    app: my-app
spec:
  ports:
  - name: http
    port: 80
    targetPort: http
  selector:
    app: my-app
    version: v2.0.0
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v2
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
      version: v2.0.0
  template:
    metadata:
      labels:
        app: my-app
        version: v2.0.0
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9101"
    spec:
      containers:
      - name: my-app
        image: registry.cn-hangzhou.aliyuncs.com/containerdemo/containersol-k8s-deployment-strategies
        ports:
        - name: http
          containerPort: 8080
        - name: probe
          containerPort: 8086
        env:
        - name: VERSION
          value: v2.0.0
        livenessProbe:
          httpGet:
            path: /live
            port: probe
          initialDelaySeconds: 5
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /ready
            port: probe
          periodSeconds: 5
```

执行如下命令，部署 Deployement V2 应用。

```shell
kubectl apply -f app-v2.yaml
```

## 按照权重策略灰度到 Deployment V2 应用

本步骤指导你如何按照权重策略灰度到 Deployment V2 应用。

执行如下命令，创建名为 ingress-v2-canary-weigth.yaml 的 YAML 文件。将如下代码复制到文件中，并将 hots 参数中的`集群id`修改为`k8s 集群id`。

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-canary
  labels:
    app: my-app
  annotations:
    # Enable canary and send 10% of traffic to version 2
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"
spec:
  rules:
  - host: my-app.集群id.cn-shanghai.alicontainer.com
    http:
      paths:
      - backend:
          service:
            name: my-app-v2
            port:
              number: 80
        path: /
        pathType: Prefix
```

执行如下命令，部署 Ingress 资源。

```shell
kubectl apply -f ingress-v2-canary-weigth.yaml
```

执行如下命令进行测试。

```shell
while sleep 0.1;do curl my-app.<集群id>.cn-shanghai.alicontainer.com; done
```

返回结果如下，有 10% 返回版本为 `v2.0.0`，表示已成功按照权重策略灰度到 Deployment V2 应用。按 CTRL+C 键退出。

```text
Host: my-app-v1-6fc68db67d-xd2kf, Version: v1.0.0
Host: my-app-v1-6fc68db67d-xd2kf, Version: v1.0.0
Host: my-app-v1-6fc68db67d-xd2kf, Version: v1.0.0
Host: my-app-v1-6fc68db67d-xd2kf, Version: v1.0.0
Host: my-app-v1-6fc68db67d-xd2kf, Version: v1.0.0
Host: my-app-v1-6fc68db67d-xd2kf, Version: v1.0.0
Host: my-app-v1-6fc68db67d-xd2kf, Version: v1.0.0
Host: my-app-v1-6fc68db67d-xd2kf, Version: v1.0.0
Host: my-app-v2-648947bdcb-2z4s5, Version: v2.0.0
Host: my-app-v1-6fc68db67d-xd2kf, Version: v1.0.0
Host: my-app-v2-648947bdcb-2z4s5, Version: v2.0.0
Host: my-app-v1-6fc68db67d-xd2kf, Version: v1.0.0
```

## 按照 Header 策略灰度到 Deployment V2 应用

本步骤指导你如何按照 Header 策略灰度到 Deployment V2 应用。

执行如下命令，创建名为 ingress-v2-canary-header.yaml 的 YAML 文件。将如下代码复制到文件中，并将 hots 参数中的集群id 修改为 k8s 集群id。

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-canary
  labels:
    app: my-app
  annotations:
    # Enable canary and send traffic with headder x-app-canary to version 2
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-by-header: "x-app-canary"
    nginx.ingress.kubernetes.io/canary-by-header-value: "true"
spec:
  rules:
  - host: my-app.集群id.cn-shanghai.alicontainer.com
    http:
      paths:
      - backend:
          service:
            name: my-app-v2
            port:
              number: 80
        path: /
        pathType: Prefix
```

执行如下命令，部署 Ingress 资源。

```shell
kubectl apply -f ingress-v2-canary-header.yaml
```

执行如下命令，并将命令中的集群id 修改为 k8s 集群id，进行测试。

```shell
curl my-app.<集群id>.cn-shanghai.alicontainer.com
```

返回结果如下，访问到 Deployment V1 应用。

```text
Host: my-app-v1-6fc68db67d-xd2kf, Version: v1.0.0
```

执行如下命令，并将命令中的集群id 修改为 k8s 集群id，设置新的 header。

```shell
curl my-app.<集群id>.cn-shanghai.alicontainer.com -H "x-app-canary: true"
```

返回结果如下，访问到 Deployment V2 应用，表示已成功按照 Header 策略灰度到 Deployment V2 应用。

```text
Host: my-app-v2-648947bdcb-2z4s5, Version: v2.0.0