+++
date = '2026-01-11T14:30:00+08:00'
draft = false
title = 'Docker 容器化技术入门与实践'
summary = '全面介绍 Docker 容器技术，从基础概念到生产实践'
tags = ['Docker', '容器化', 'DevOps', '微服务']
categories = ['技术教程']
+++

# Docker 容器化技术入门与实践

![Docker Logo](https://raw.githubusercontent.com/docker/.github/main/profile/docker-logo-blue.svg "Docker - Build, Ship, Run")

## 引言

> "在我机器上明明可以运行啊！" 🤷‍♂️

本文将带你深入了解 Docker 的核心概念和实践技巧。

---

## 什么是 Docker？

Docker 是一个开源的==容器化平台==，它允许开发者将应用程序及其依赖打包到一个轻量级、可移植的容器中。

### 容器 vs 虚拟机

| 特性 | 容器 (Container) | 虚拟机 (VM) |
|:----:|:----------------:|:-----------:|
| 启动时间 | 秒级 ⚡ | 分钟级 🐢 |
| 资源占用 | MB 级 | GB 级 |
| 性能损耗 | 接近原生 | 5-20% |
| 隔离级别 | 进程级 | 系统级 |
| 操作系统 | 共享宿主内核 | 完整 OS |

```
┌─────────────────────────────────────┐
│           Container Stack           │
├─────────┬─────────┬─────────────────┤
│  App A  │  App B  │     App C       │
├─────────┼─────────┼─────────────────┤
│ Bins/   │ Bins/   │    Bins/        │
│ Libs    │ Libs    │    Libs         │
├─────────┴─────────┴─────────────────┤
│           Docker Engine             │
├─────────────────────────────────────┤
│           Host OS (Linux)           │
├─────────────────────────────────────┤
│             Hardware                │
└─────────────────────────────────────┘
```

## 安装 Docker

### macOS

推荐使用 [Docker Desktop](https://www.docker.com/products/docker-desktop)：

```bash
# 使用 Homebrew
brew install --cask docker

# 启动 Docker Desktop
open /Applications/Docker.app
```

### Ubuntu

```bash
# 1. 卸载旧版本
sudo apt remove docker docker-engine docker.io containerd runc

# 2. 安装依赖
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release

# 3. 添加 Docker 官方 GPG 密钥
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 4. 设置仓库
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. 安装 Docker Engine
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 6. 验证安装
sudo docker run hello-world
```

### 配置国内镜像加速

编辑 `/etc/docker/daemon.json`：

```json
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "3"
  }
}
```

然后重启 Docker：

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 核心概念

Docker 的三大核心概念：

1. **镜像 (Image)** - 只读的应用模板
2. **容器 (Container)** - 镜像的运行实例
3. **仓库 (Registry)** - 镜像的存储中心

### 镜像层次结构

```
┌─────────────────────────┐
│    Application Layer    │  <- 你的应用
├─────────────────────────┤
│    Dependencies Layer   │  <- npm, pip 等依赖
├─────────────────────────┤
│     Runtime Layer       │  <- Node.js, Python 等
├─────────────────────────┤
│       Base Image        │  <- Ubuntu, Alpine 等
└─────────────────────────┘
```

## 常用命令

### 镜像管理

```bash
# 搜索镜像
docker search nginx

# 拉取镜像
docker pull nginx:latest
docker pull nginx:1.24-alpine  # 指定版本和变体

# 查看本地镜像
docker images
docker image ls

# 删除镜像
docker rmi nginx:latest
docker image prune  # 清理悬空镜像
docker image prune -a  # 清理所有未使用的镜像
```

### 容器管理

```bash
# 运行容器
docker run nginx                    # 前台运行
docker run -d nginx                 # 后台运行
docker run -d -p 8080:80 nginx      # 端口映射
docker run -d --name web nginx      # 指定名称
docker run -it ubuntu bash          # 交互式运行

# 常用参数组合
docker run -d \
    --name my-nginx \
    -p 80:80 \
    -v /host/path:/container/path \
    -e MY_VAR=value \
    --restart unless-stopped \
    nginx:alpine

# 查看容器
docker ps           # 运行中的容器
docker ps -a        # 所有容器

# 容器操作
docker start <container>
docker stop <container>
docker restart <container>
docker rm <container>
docker rm -f <container>  # 强制删除运行中的容器

# 进入容器
docker exec -it <container> bash
docker exec -it <container> sh  # Alpine 镜像

# 查看日志
docker logs <container>
docker logs -f <container>  # 实时跟踪
docker logs --tail 100 <container>  # 最后 100 行
```

### 数据卷管理

```bash
# 创建数据卷
docker volume create my-volume

# 查看数据卷
docker volume ls
docker volume inspect my-volume

# 删除数据卷
docker volume rm my-volume
docker volume prune  # 清理未使用的卷
```

## Dockerfile 详解

`Dockerfile` 是构建镜像的蓝图。

### 基本示例

```dockerfile
# 基础镜像
FROM node:18-alpine

# 元数据
LABEL maintainer="developer@example.com"
LABEL version="1.0"

# 设置工作目录
WORKDIR /app

# 复制依赖文件
COPY package*.json ./

# 安装依赖
RUN npm ci --only=production

# 复制源代码
COPY . .

# 构建应用
RUN npm run build

# 暴露端口
EXPOSE 3000

# 设置环境变量
ENV NODE_ENV=production

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

# 启动命令
CMD ["node", "dist/main.js"]
```

### 多阶段构建

多阶段构建可以显著减小镜像体积：

```dockerfile
# ============ 构建阶段 ============
FROM golang:1.21-alpine AS builder

WORKDIR /build

# 下载依赖
COPY go.mod go.sum ./
RUN go mod download

# 编译
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o app .

# ============ 运行阶段 ============
FROM alpine:3.18

# 安装 CA 证书（HTTPS 需要）
RUN apk --no-cache add ca-certificates

WORKDIR /app

# 从构建阶段复制二进制文件
COPY --from=builder /build/app .

# 使用非 root 用户
RUN adduser -D -g '' appuser
USER appuser

EXPOSE 8080

ENTRYPOINT ["./app"]
```

### Dockerfile 最佳实践

- [x] 使用 `.dockerignore` 排除不需要的文件
- [x] 合并 RUN 指令减少层数
- [x] 将变化频繁的指令放在后面
- [x] 使用多阶段构建
- [x] 不要以 root 用户运行
- [ ] 定期更新基础镜像

## Docker Compose

Docker Compose 用于定义和运行多容器应用。

### 示例配置

创建 `docker-compose.yml`：

```yaml
version: '3.8'

services:
  # Web 应用
  web:
    build: 
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://user:pass@db:5432/myapp
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped
    networks:
      - app-network

  # 数据库
  db:
    image: postgres:15-alpine
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: myapp
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d myapp"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  # Redis 缓存
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - app-network

  # Nginx 反向代理
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro
    depends_on:
      - web
    networks:
      - app-network

volumes:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

### 常用命令

```bash
# 启动服务
docker compose up -d

# 查看状态
docker compose ps

# 查看日志
docker compose logs -f

# 停止服务
docker compose down

# 停止并删除卷
docker compose down -v

# 重新构建
docker compose build --no-cache
docker compose up -d --build
```

## 网络配置

Docker 提供多种网络模式：

| 模式 | 说明 | 使用场景 |
|------|------|----------|
| `bridge` | 默认模式，容器间通过网桥通信 | 大多数场景 |
| `host` | 容器使用宿主机网络 | 性能敏感应用 |
| `none` | 禁用网络 | 安全隔离 |
| `overlay` | 跨主机网络 | Swarm 集群 |

```bash
# 创建自定义网络
docker network create --driver bridge my-network

# 连接容器到网络
docker network connect my-network my-container

# 查看网络详情
docker network inspect my-network
```

## 生产环境最佳实践

### 安全加固

1. **使用非 root 用户**
   ```dockerfile
   RUN addgroup -g 1001 -S appgroup && \
       adduser -u 1001 -S appuser -G appgroup
   USER appuser
   ```

2. **限制资源使用**
   ```bash
   docker run -d \
       --memory="512m" \
       --cpus="0.5" \
       --pids-limit=100 \
       my-app
   ```

3. **只读文件系统**
   ```bash
   docker run --read-only --tmpfs /tmp my-app
   ```

### 日志管理

```yaml
# docker-compose.yml
services:
  app:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

### 健康检查

```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD wget --quiet --tries=1 --spider http://localhost:8080/health || exit 1
```

## 常见问题排查

### 容器无法启动

```bash
# 查看容器日志
docker logs <container>

# 查看容器详情
docker inspect <container>

# 以交互模式调试
docker run -it --entrypoint sh <image>
```

### 磁盘空间不足

```bash
# 查看 Docker 磁盘使用
docker system df

# 清理所有未使用的资源
docker system prune -a --volumes
```

### 网络问题

```bash
# 测试容器间连通性
docker exec -it container1 ping container2

# 查看容器 IP
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container>
```

---

## 总结

本文介绍了 Docker 的核心知识点：

- **基础概念**：镜像、容器、仓库
- **常用命令**：`run`、`exec`、`logs`、`build`
- **Dockerfile**：编写规范和最佳实践
- **Docker Compose**：多容器应用编排
- **生产实践**：安全、日志、健康检查

> 🚀 **下一步**：学习 Kubernetes，将容器编排提升到更高层次！

---

## 参考资料

1. [Docker 官方文档](https://docs.docker.com/)
2. [Docker Hub](https://hub.docker.com/)
3. [Docker 最佳实践](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
4. [Awesome Docker](https://github.com/veggiemonk/awesome-docker)

*本文最后更新于 2026 年 1 月*