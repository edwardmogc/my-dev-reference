# Docker 服务参考指南

> 范围：容器管理、镜像构建、网络、存储、监控  
> 版本：Docker 20.10+ / Docker Engine CE  
> 最后更新：2025-11-24

---

## 目录

1. [安装和启动](#安装和启动)
2. [核心概念](#核心概念)
3. [镜像管理](#镜像管理)
4. [容器管理](#容器管理)
5. [网络配置](#网络配置)
6. [存储管理](#存储管理)
7. [配置示例](#配置示例)
8. [实战场景](#实战场景)
9. [监控和调试](#监控和调试)
10. [故障排查](#故障排查)

---

## 安装和启动

### Ubuntu上安装Docker

```bash
# 删除旧版本（如果存在）
sudo apt-get remove docker docker-engine docker.io containerd runc

# 更新包列表
sudo apt-get update

# 安装依赖
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# 添加Docker官方GPG密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 添加Docker仓库
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装Docker Engine
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 启动Docker服务
sudo systemctl start docker
sudo systemctl enable docker

# 将当前用户添加到docker组（可选，避免每次使用sudo）
sudo usermod -aG docker $USER
# 需要重新登录或运行以下命令刷新组成员资格
newgrp docker
```

### 验证安装

```bash
# 检查Docker版本
docker --version
docker version

# 运行测试容器
docker run hello-world

# 查看Docker系统信息
docker info
```

### 启动和停止Docker服务

```bash
# 启动Docker服务
sudo systemctl start docker

# 停止Docker服务
sudo systemctl stop docker

# 重启Docker服务
sudo systemctl restart docker

# 查看Docker服务状态
sudo systemctl status docker

# 启用开机自启
sudo systemctl enable docker

# 禁用开机自启
sudo systemctl disable docker
```

---

## 核心概念

### 概念 1: 镜像（Image）

**定义**: Docker镜像是一个只读的文件系统，包含运行应用程序所需的所有内容（代码、运行时、系统工具、库和设置）。

**特点**:
- 分层结构：每个命令创建一个新层
- 可重用性：同一镜像可创建多个容器
- 版本管理：支持tag标记不同版本
- 便携性：可在任何Docker环境中运行

**存储位置**: `/var/lib/docker/images/`

### 概念 2: 容器（Container）

**定义**: 容器是镜像的运行实例，是一个隔离的进程环境，拥有自己的文件系统、网络和进程空间。

**特点**:
- 轻量级：共享主机内核
- 隔离性：进程、文件系统、网络相互隔离
- 临时性：容器停止时状态丢失（除非提交为镜像）
- 可配置：支持环境变量、挂载卷、端口映射等

### 概念 3: Dockerfile

**定义**: 一个包含Docker构建指令的文本文件，用于自动化创建Docker镜像。

**关键指令**:
```dockerfile
FROM <image>              # 基础镜像
RUN <command>             # 执行命令（构建时）
COPY <src> <dest>         # 复制文件
ADD <src> <dest>          # 复制文件（支持远程URL和解压）
ENV <key> <value>         # 设置环境变量
WORKDIR <path>            # 设置工作目录
EXPOSE <port>             # 声明端口
CMD <command>             # 默认执行命令
ENTRYPOINT <command>      # 容器入口点
VOLUME <path>             # 声明卷挂载点
LABEL <key> <value>       # 添加元数据
USER <user>               # 指定运行用户
ARG <key>=<value>         # 构建参数
```

---

## 镜像管理

### 查看镜像

```bash
# 列出所有镜像
docker images

# 列出所有镜像（包括中间层）
docker images -a

# 列出特定仓库的镜像
docker images myapp

# 按标签查找镜像
docker images | grep v1.0

# 查看镜像详细信息
docker inspect <image-id>

# 查看镜像历史层
docker history <image-name>

# 搜索镜像（从Docker Hub）
docker search ubuntu
docker search nginx --limit 5
```

### 构建镜像

```bash
# 从Dockerfile构建
docker build -t myapp:1.0 .

# 指定Dockerfile位置
docker build -f /path/to/Dockerfile -t myapp:1.0 .

# 构建并指定多个标签
docker build -t myapp:1.0 -t myapp:latest .

# 使用构建参数
docker build --build-arg JAVA_VERSION=17 -t myapp:1.0 .

# 不使用缓存构建
docker build --no-cache -t myapp:1.0 .

# 查看构建输出的层
docker build --progress=plain -t myapp:1.0 .

# 使用buildx进行多平台构建
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:1.0 .
```

### 镜像标签和版本管理

```bash
# 标记镜像
docker tag myapp:1.0 myapp:latest
docker tag myapp:1.0 registry.example.com/myapp:1.0

# 重命名镜像
docker tag myapp:1.0 newname:1.0

# 删除标签
docker rmi myapp:latest

# 查看镜像标签
docker images myapp
```

### 镜像仓库操作

```bash
# 登录到Docker Hub
docker login

# 登录到私有仓库
docker login registry.example.com

# 推送镜像到仓库
docker push myapp:1.0

# 推送到私有仓库
docker push registry.example.com/myapp:1.0

# 拉取镜像
docker pull ubuntu:22.04

# 拉取私有仓库镜像
docker pull registry.example.com/myapp:1.0

# 退出登录
docker logout
```

### 镜像清理

```bash
# 删除镜像
docker rmi <image-id>

# 删除多个镜像
docker rmi image1 image2 image3

# 删除未标记的镜像
docker image prune

# 删除所有未使用的镜像（谨慎使用）
docker image prune -a

# 显示要删除的镜像但不真正删除
docker image prune -a --dry-run

# 删除镜像并提示确认
docker image prune -a --force
```

---

## 容器管理

### 启动和停止容器

```bash
# 从镜像创建并运行容器
docker run -d -p 8080:8080 --name myapp myapp:1.0

# 在前台运行容器（保持输出可见）
docker run -it ubuntu:22.04 /bin/bash

# 启动已停止的容器
docker start <container-id>

# 停止运行中的容器
docker stop <container-id>

# 立即杀死容器
docker kill <container-id>

# 优雅关闭容器（等待指定秒数后强制杀死）
docker stop --time=30 <container-id>

# 重启容器
docker restart <container-id>

# 暂停容器
docker pause <container-id>

# 恢复暂停的容器
docker unpause <container-id>
```

### 查看容器

```bash
# 列出运行中的容器
docker ps

# 列出所有容器（包括已停止的）
docker ps -a

# 列出最近创建的容器
docker ps -l

# 只显示容器ID
docker ps -q

# 查看容器详细信息
docker inspect <container-id>

# 查看容器IP地址
docker inspect <container-id> | grep IPAddress

# 查看容器资源使用情况
docker stats <container-id>

# 查看所有容器的资源使用情况
docker stats

# 查看容器进程
docker top <container-id>

# 查看容器日志
docker logs <container-id>

# 实时跟踪容器日志
docker logs -f <container-id>

# 查看最后100行日志
docker logs --tail 100 <container-id>

# 查看带时间戳的日志
docker logs -t <container-id>
```

### 容器交互

```bash
# 进入运行中的容器
docker exec -it <container-id> /bin/bash

# 在容器中执行命令
docker exec <container-id> ls -la

# 在容器中执行命令并保持交互
docker exec -it <container-id> sh -c "command1 && command2"

# 附加到容器的标准输入/输出/错误
docker attach <container-id>

# 从容器复制文件到主机
docker cp <container-id>:/path/in/container /path/on/host

# 从主机复制文件到容器
docker cp /path/on/host <container-id>:/path/in/container
```

### 容器清理

```bash
# 删除容器
docker rm <container-id>

# 删除运行中的容器（强制）
docker rm -f <container-id>

# 删除多个容器
docker rm container1 container2 container3

# 删除所有已停止的容器
docker container prune

# 删除所有容器（包括运行中的，谨慎使用）
docker rm -f $(docker ps -aq)

# 显示要删除的容器
docker container prune --dry-run
```

---

## 网络配置

### Docker网络模式

```bash
# 列出所有网络
docker network ls

# 查看网络详细信息
docker inspect <network-name>

# 创建桥接网络
docker network create my-network

# 创建自定义网络（带有IP范围）
docker network create --driver bridge --subnet 172.20.0.0/16 my-network

# 删除网络
docker network rm <network-name>

# 连接容器到网络
docker network connect my-network <container-id>

# 断开容器与网络的连接
docker network disconnect my-network <container-id>
```

### 运行容器时的网络配置

```bash
# 桥接网络（默认）
docker run -d -p 8080:8080 --name myapp myapp:1.0

# 主机网络（容器使用主机网络，不进行端口映射）
docker run -d --network host --name myapp myapp:1.0

# 自定义网络
docker run -d --network my-network --ip 172.20.0.5 myapp:1.0

# 容器间通信（使用容器名作为hostname）
docker run -d --network my-network --name web nginx
docker run -d --network my-network --name app --link web:webserver myapp:1.0

# 设置DNS
docker run -d --dns 8.8.8.8 --dns 8.8.4.4 myapp:1.0

# 暴露多个端口
docker run -d -p 8080:8080 -p 8443:8443 myapp:1.0

# 端口映射到特定IP
docker run -d -p 127.0.0.1:8080:8080 myapp:1.0

# 发布所有暴露的端口
docker run -d -P myapp:1.0
```

---

## 存储管理

### 卷（Volumes）

```bash
# 创建卷
docker volume create my-volume

# 列出所有卷
docker volume ls

# 查看卷详细信息
docker inspect my-volume

# 删除卷
docker volume rm my-volume

# 删除未使用的卷
docker volume prune

# 使用卷运行容器
docker run -d -v my-volume:/data --name myapp myapp:1.0

# 卷的完整语法（可读写）
docker run -d -v my-volume:/data:rw --name myapp myapp:1.0

# 卷的只读挂载
docker run -d -v my-volume:/data:ro --name myapp myapp:1.0

# 多个卷
docker run -d \
  -v volume1:/data1 \
  -v volume2:/data2 \
  myapp:1.0

# 卷备份
docker run --rm -v my-volume:/data -v $(pwd):/backup ubuntu tar czf /backup/backup.tar.gz -C /data .

# 卷恢复
docker run --rm -v my-volume:/data -v $(pwd):/backup ubuntu tar xzf /backup/backup.tar.gz -C /data
```

### 绑定挂载（Bind Mount）

```bash
# 绑定挂载（绝对路径）
docker run -d -v /host/path:/container/path myapp:1.0

# 只读绑定挂载
docker run -d -v /host/path:/container/path:ro myapp:1.0

# 相对路径（使用$(pwd)）
docker run -d -v $(pwd)/data:/container/data myapp:1.0

# 绑定单个文件
docker run -d -v /host/config.conf:/app/config.conf:ro myapp:1.0
```

### tmpfs 挂载

```bash
# tmpfs挂载（内存中的临时存储）
docker run -d --tmpfs /tmp:size=100m myapp:1.0

# 多个tmpfs挂载
docker run -d \
  --tmpfs /tmp:size=100m \
  --tmpfs /var/run:size=50m \
  myapp:1.0
```

---

## 配置示例

### 示例 1: Dockerfile - Java应用

```dockerfile
# 使用Java基础镜像
FROM eclipse-temurin:17-jdk-jammy

# 设置工作目录
WORKDIR /app

# 设置环境变量
ENV JAVA_OPTS="-Xms512m -Xmx2g -Dfile.encoding=UTF-8"

# 复制应用JAR文件
COPY application.jar app.jar

# 声明暴露端口
EXPOSE 8080

# 声明数据卷
VOLUME /app/logs /app/data

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD curl -f http://localhost:8080/actuator/health || exit 1

# 启动应用
ENTRYPOINT ["java", "-jar", "app.jar"]
CMD []
```

### 示例 2: Dockerfile - Node.js应用

```dockerfile
FROM node:18-alpine

WORKDIR /app

# 复制package文件
COPY package*.json ./

# 安装依赖
RUN npm ci --only=production

# 复制应用代码
COPY . .

EXPOSE 3000

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD node healthcheck.js

CMD ["node", "server.js"]
```

### 示例 3: Dockerfile - Python应用（使用虚拟环境）

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# 安装系统依赖
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# 创建虚拟环境
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# 复制requirements文件
COPY requirements.txt .

# 安装Python依赖
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY . .

EXPOSE 8000

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "wsgi:app"]
```

### 示例 4: docker-compose.yml - 多容器应用

```yaml
version: '3.8'

services:
  # API服务
  api:
    build:
      context: .
      dockerfile: Dockerfile.api
    image: myapp-api:1.0
    container_name: myapp-api
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://db:3306/myapp
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=rootpassword
    volumes:
      - ./logs:/app/logs
    depends_on:
      - db
      - redis
    networks:
      - myapp-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  # Web前端
  web:
    build:
      context: ./web
      dockerfile: Dockerfile
    image: myapp-web:1.0
    container_name: myapp-web
    ports:
      - "3000:3000"
    depends_on:
      - api
    networks:
      - myapp-network
    restart: unless-stopped

  # MySQL数据库
  db:
    image: mysql:8.0
    container_name: myapp-db
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: myapp
      TZ: 'Asia/Shanghai'
    volumes:
      - db-data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "3306:3306"
    networks:
      - myapp-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis缓存
  redis:
    image: redis:7-alpine
    container_name: myapp-redis
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    ports:
      - "6379:6379"
    networks:
      - myapp-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db-data:
  redis-data:

networks:
  myapp-network:
    driver: bridge
```

---

## 实战场景

### 场景 1: 构建和运行Java应用容器

**需求**: 构建一个Spring Boot应用的Docker镜像，并以容器形式运行

**步骤**:

1. 创建Dockerfile
   ```dockerfile
   FROM eclipse-temurin:17-jdk-jammy
   WORKDIR /app
   COPY application.jar app.jar
   EXPOSE 8080
   ENTRYPOINT ["java", "-jar", "app.jar"]
   ```

2. 构建镜像
   ```bash
   docker build -t myapp:1.0 .
   ```

3. 运行容器
   ```bash
   docker run -d \
     --name myapp \
     -p 8080:8080 \
     -v myapp-logs:/app/logs \
     -e "JAVA_OPTS=-Xmx2g" \
     myapp:1.0
   ```

4. 验证容器
   ```bash
   docker logs -f myapp
   docker stats myapp
   curl http://localhost:8080
   ```

---

### 场景 2: 使用docker-compose运行多容器应用

**需求**: 同时运行API、前端、数据库、缓存等服务

**步骤**:

1. 创建docker-compose.yml（见配置示例4）

2. 启动所有服务
   ```bash
   docker-compose up -d
   ```

3. 查看服务状态
   ```bash
   docker-compose ps
   docker-compose logs -f
   ```

4. 停止所有服务
   ```bash
   docker-compose down
   ```

5. 清理数据卷
   ```bash
   docker-compose down -v
   ```

---

### 场景 3: 容器镜像优化和瘦身

**需求**: 减小镜像大小，提高构建效率

**步骤**:

```dockerfile
# 使用多阶段构建
FROM eclipse-temurin:17-jdk-jammy as builder
WORKDIR /build
COPY . .
RUN ./gradlew bootJar

# 最小化运行镜像
FROM eclipse-temurin:17-jre-jammy
WORKDIR /app
COPY --from=builder /build/build/libs/application.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## 监控和调试

### 资源监控

```bash
# 查看容器资源使用情况
docker stats <container-id>

# 实时监控所有容器
docker stats --no-stream

# 查看容器内存限制
docker inspect <container-id> | grep Memory

# 限制容器内存
docker run -d -m 512m myapp:1.0

# 限制容器CPU
docker run -d --cpus="0.5" myapp:1.0

# 同时限制内存和CPU
docker run -d -m 1g --cpus="2" myapp:1.0
```

### 日志和调试

```bash
# 查看容器日志
docker logs <container-id>

# 实时查看日志
docker logs -f <container-id>

# 查看最后100行日志
docker logs --tail 100 <container-id>

# 查看带时间戳的日志
docker logs -t <container-id>

# 进入容器进行调试
docker exec -it <container-id> /bin/bash

# 查看容器详细信息
docker inspect <container-id>

# 查看容器进程
docker top <container-id>

# 容器事件日志
docker events --filter type=container
```

---

## 故障排查

### 问题 1: 容器无法启动

**症状**: 容器立即退出，状态为 Exited

**原因**: 入口点命令失败或应用无法启动

**解决**:
```bash
# 查看容器日志
docker logs <container-id>

# 查看容器最后的状态
docker inspect <container-id> | grep -A 20 "State"

# 使用调试模式启动
docker run -it myapp:1.0 /bin/bash

# 验证镜像构建
docker history myapp:1.0
```

### 问题 2: 容器内存溢出（OOM）

**症状**: 容器被杀死，内存不足

**原因**: 应用泄漏或没有设置内存限制

**解决**:
```bash
# 设置内存限制
docker run -d -m 2g myapp:1.0

# 查看OOM日志
docker inspect <container-id> | grep OOMKilled

# 增加swap
docker run -d -m 2g --memory-swap 4g myapp:1.0
```

### 问题 3: 端口被占用

**症状**: 绑定端口时报错：Address already in use

**原因**: 另一个容器或主机进程占用了该端口

**解决**:
```bash
# 查看占用端口的容器
docker ps | xargs docker inspect | grep -B 5 "8080"

# 删除占用端口的容器
docker rm -f <container-id>

# 使用不同端口
docker run -d -p 8081:8080 myapp:1.0
```

---

## 快速命令参考

| 任务 | 命令 | 说明 |
|------|------|------|
| 查看镜像 | `docker images` | 列出所有镜像 |
| 构建镜像 | `docker build -t name:tag .` | 构建新镜像 |
| 运行容器 | `docker run -d name:tag` | 在后台运行 |
| 查看容器 | `docker ps -a` | 列出所有容器 |
| 查看日志 | `docker logs -f name` | 实时查看日志 |
| 进入容器 | `docker exec -it name bash` | 进入容器shell |
| 停止容器 | `docker stop name` | 优雅停止 |
| 删除容器 | `docker rm name` | 删除容器 |
| 删除镜像 | `docker rmi image` | 删除镜像 |
| 监控资源 | `docker stats` | 查看资源使用 |

---

## 更新日志

- **2025-11-24**: 初版完成，包含镜像管理、容器管理、网络存储等完整内容