# 配置参考指南

> 范围：环境配置、系统集成、部署配置  
> 最后更新：2024-01-23

---

## 目录

1. [快速开始](#快速开始)
2. [配置文件](#配置文件)
3. [环境变量](#环境变量)
4. [集成指南](#集成指南)
5. [配置验证](#配置验证)

---

## 快速开始

### 初始化

```bash
# 初始化项目
npm init -y
# 或
python -m venv venv
# 或
go mod init github.com/user/project

# 创建配置文件
touch .env .env.example config/application.yml
```

### 文件结构

```
project-root/
├── config/
│   ├── application.yml          # 应用主配置
│   ├── application-dev.yml      # 开发环境
│   ├── application-prod.yml     # 生产环境
│   └── database.yml             # 数据库配置
├── .env                         # 环境变量（不提交 Git）
├── .env.example                 # 环境变量模板（提交 Git）
├── docker-compose.yml           # Docker 编排
└── .gitignore                   # Git 忽略规则
```

---

## 配置文件

### 配置文件 1: application.yml

**位置**: `config/application.yml`

**格式**: YAML

**用途**: Spring Boot 应用主配置文件

```yaml
# 应用配置
spring:
  application:
    name: gaming-api
    version: 1.0.0
  
  # 服务器配置
  server:
    port: 8080
    servlet:
      context-path: /api/v1
  
  # 数据库配置
  datasource:
    url: jdbc:mysql://localhost:3306/gamedb?useSSL=false&serverTimezone=UTC
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
  
  # JPA 配置
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL8Dialect
        format_sql: true

# 自定义配置
app:
  jwt:
    secret: your_jwt_secret_key_min_32_characters_long
    expiry: 86400000
  payment:
    api-key: sk_live_xxxx
  database:
    pool-size: 20
    timeout: 30000
```

**关键字段说明**:
- `server.port`: 应用监听端口
- `datasource.url`: 数据库连接地址
- `jpa.hibernate.ddl-auto`: 自动创建表策略（validate/update/create）

---

### 配置文件 2: docker-compose.yml

**位置**: 项目根目录

**格式**: YAML

**用途**: Docker 容器编排

```yaml
version: '3.8'

services:
  # Node.js 应用
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: gaming_api
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=mongodb://mongo:27017/gamedb
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=your_secret_key
    depends_on:
      - mongo
      - redis
    volumes:
      - ./logs:/app/logs
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # MongoDB
  mongo:
    image: mongo:5.0
    container_name: mongo_db
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password
    volumes:
      - mongo_data:/data/db
    networks:
      - app-network

  # Redis
  redis:
    image: redis:7-alpine
    container_name: redis_cache
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - app-network
    command: redis-server --appendonly yes

  # MySQL
  mysql:
    image: mysql:8.0
    container_name: mysql_db
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: gamedb
      MYSQL_USER: gameuser
      MYSQL_PASSWORD: gamepass
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - app-network

volumes:
  mongo_data:
  redis_data:
  mysql_data:

networks:
  app-network:
    driver: bridge
```

**关键字段说明**:
- `services`: 定义各个容器
- `depends_on`: 容器启动顺序
- `environment`: 环境变量
- `volumes`: 数据持久化
- `networks`: 容器通信网络

---

### 配置文件 3: Dockerfile

**位置**: 项目根目录

**格式**: Dockerfile 语法

**用途**: 构建应用容器镜像

```dockerfile
# 多阶段构建 - 生产环境
FROM node:18-alpine as builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# 最终镜像
FROM node:18-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./

EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD node healthcheck.js

CMD ["node", "dist/index.js"]
```

**常用参数**:
- `FROM`: 基础镜像
- `WORKDIR`: 工作目录
- `RUN`: 执行命令
- `COPY`: 复制文件
- `EXPOSE`: 暴露端口
- `CMD`: 启动命令

---

### 配置文件 4: .gitignore

**位置**: 项目根目录

**格式**: 纯文本

**用途**: 定义 Git 忽略文件

```
# 环境变量
.env
.env.local
.env.*.local

# 依赖
node_modules/
__pycache__/
vendor/
*.pyc

# 构建输出
dist/
build/
*.o
*.a

# IDE
.vscode/
.idea/
*.swp
*.swo
.DS_Store

# 日志
logs/
*.log

# 临时文件
tmp/
temp/
.cache/

# 系统文件
.DS_Store
Thumbs.db
```

---

## 环境变量

### 设置方法

```bash
# Linux/macOS - 导出环境变量
export DATABASE_URL="mongodb://localhost:27017/gamedb"
export NODE_ENV="production"

# Windows PowerShell - 导出环境变量
$env:DATABASE_URL="mongodb://localhost:27017/gamedb"
$env:NODE_ENV="production"

# Windows CMD - 设置环境变量
set DATABASE_URL=mongodb://localhost:27017/gamedb
set NODE_ENV=production

# 使用 .env 文件（推荐）
# 在项目根目录创建 .env 文件，在启动时自动加载
```

### 常用变量

```env
# 应用配置
NODE_ENV=production
PORT=3000
DEBUG=false
LOG_LEVEL=info

# 数据库 - MongoDB
MONGODB_URL=mongodb://localhost:27017/gamedb
MONGODB_USER=admin
MONGODB_PASSWORD=password

# 数据库 - MySQL
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_DATABASE=gamedb
MYSQL_USER=root
MYSQL_PASSWORD=password

# 缓存 - Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=

# 安全
JWT_SECRET=your_jwt_secret_key_min_32_characters
API_KEY=sk_live_xxxxx
REFRESH_TOKEN_SECRET=refresh_secret_key

# 第三方服务
STRIPE_SECRET_KEY=sk_test_xxxxx
SENDGRID_API_KEY=SG.xxxxx
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=xxxxx

# 消息队列
RABBITMQ_URL=amqp://localhost:5672
RABBITMQ_USER=guest
RABBITMQ_PASSWORD=guest

# 服务注册
NACOS_SERVER=localhost:8848
NACOS_NAMESPACE=dev
NACOS_GROUP=DEFAULT_GROUP
```

### .env 文件示例

```bash
# .env - 本地开发环境（不提交到 Git）
# 复制 .env.example 为 .env，然后填入真实值

NODE_ENV=development
PORT=3000
DEBUG=true
LOG_LEVEL=debug

# 本地数据库
MONGODB_URL=mongodb://localhost:27017/gamedb_dev
MYSQL_HOST=localhost
MYSQL_DATABASE=gamedb_dev
REDIS_HOST=localhost

# 开发用密钥（非真实密钥）
JWT_SECRET=dev_secret_key_for_testing_only_min_32_chars
API_KEY=test_key_xxxxxx
```

### .env.example 模板

```bash
# .env.example - 提交到 Git，供其他开发者参考
# 复制此文件为 .env，然后填入真实值

# ========== 应用配置 ==========
NODE_ENV=development
PORT=3000
DEBUG=false
LOG_LEVEL=info

# ========== 数据库配置 ==========
# MongoDB
MONGODB_URL=mongodb://localhost:27017/dbname
MONGODB_USER=
MONGODB_PASSWORD=

# MySQL
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_DATABASE=
MYSQL_USER=
MYSQL_PASSWORD=

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=

# ========== 安全配置 ==========
JWT_SECRET=
API_KEY=
REFRESH_TOKEN_SECRET=

# ========== 第三方服务 ==========
STRIPE_SECRET_KEY=
SENDGRID_API_KEY=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=

# ========== 消息队列 ==========
RABBITMQ_URL=amqp://localhost:5672
RABBITMQ_USER=guest
RABBITMQ_PASSWORD=guest

# ========== 服务配置 ==========
NACOS_SERVER=localhost:8848
NACOS_NAMESPACE=dev
NACOS_GROUP=DEFAULT_GROUP
```

---

## 集成指南

### 集成 1: Node.js + dotenv 加载环境变量

**前置条件**:
- [ ] 已安装 Node.js
- [ ] 已创建 .env 文件

**步骤**:

1. 安装依赖
   ```bash
   npm install dotenv
   ```

2. 在应用启动时加载 .env
   ```javascript
   // main.js 或 index.js
   require('dotenv').config();
   
   const port = process.env.PORT || 3000;
   const dbUrl = process.env.MONGODB_URL;
   
   console.log(`Server running on port ${port}`);
   ```

3. 使用环境变量
   ```javascript
   // config.js
   module.exports = {
     port: process.env.PORT,
     database: {
       url: process.env.MONGODB_URL,
       user: process.env.MONGODB_USER,
     },
     jwt: {
       secret: process.env.JWT_SECRET,
     }
   };
   ```

4. 测试
   ```bash
   node index.js
   ```

---

### 集成 2: Docker Compose 启动完整环境

**前置条件**:
- [ ] 已安装 Docker 和 Docker Compose
- [ ] 已创建 docker-compose.yml

**步骤**:

1. 启动所有服务
   ```bash
   docker-compose up -d
   ```

2. 验证服务状态
   ```bash
   docker-compose ps
   ```

3. 查看日志
   ```bash
   docker-compose logs -f app
   ```

4. 进入容器
   ```bash
   docker-compose exec mongo mongosh
   docker-compose exec mysql mysql -u root -p
   docker-compose exec redis redis-cli
   ```

5. 停止所有服务
   ```bash
   docker-compose down
   ```

6. 删除数据卷
   ```bash
   docker-compose down -v
   ```

---

### 集成 3: Spring Boot 多环境配置

**前置条件**:
- [ ] Spring Boot 项目已创建
- [ ] 已创建配置文件

**步骤**:

1. 创建配置文件
   ```
   src/main/resources/
   ├── application.yml            # 默认配置
   ├── application-dev.yml        # 开发环境
   ├── application-prod.yml       # 生产环境
   └── application-test.yml       # 测试环境
   ```

2. application.yml
   ```yaml
   spring:
     profiles:
       active: dev
   ```

3. application-dev.yml
   ```yaml
   spring:
     datasource:
       url: jdbc:mysql://localhost:3306/gamedb_dev
       username: root
       password: root
   ```

4. 启动时指定环境
   ```bash
   # 开发环境
   java -jar app.jar --spring.profiles.active=dev
   
   # 生产环境
   java -jar app.jar --spring.profiles.active=prod
   
   # 通过环境变量
   export SPRING_PROFILES_ACTIVE=prod
   java -jar app.jar
   ```

---

## 配置验证

### 验证命令

```bash
# Node.js - 验证 .env 加载
node -e "require('dotenv').config(); console.log(process.env.PORT)"

# Docker Compose - 验证配置
docker-compose config

# Docker - 验证镜像构建
docker build -t myapp:test .

# 启动并验证
docker run -it myapp:test npm run test
```

### 常见错误

#### 错误 1: "Cannot find module 'dotenv'"

**症状**: 应用启动时报找不到 dotenv 模块

**原因**: 未安装 dotenv 包

**修复**:
```bash
npm install dotenv
```

---

#### 错误 2: "Port already in use"

**症状**: 启动应用时报端口已被使用

**原因**: 该端口已被其他应用占用

**修复**:
```bash
# Linux/macOS - 查看占用进程
lsof -i :3000

# 杀死进程
kill -9 <PID>

# 或使用不同的端口
PORT=3001 npm start
```

---

#### 错误 3: "Connection refused" - 无法连接数据库

**症状**: 应用启动但无法连接到数据库

**原因**: 数据库未启动或连接地址错误

**修复**:
```bash
# 使用 Docker Compose 启动数据库
docker-compose up -d mongo

# 验证连接
docker-compose exec mongo mongosh

# 检查环境变量
echo $MONGODB_URL

# 测试连接
node -e "
  const MongoClient = require('mongodb').MongoClient;
  MongoClient.connect(process.env.MONGODB_URL)
    .then(() => console.log('Connected'))
    .catch(err => console.error(err));
"
```

---

## 快速参考

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|-------|------|
| NODE_ENV | string | development | 运行环境 |
| PORT | number | 3000 | 应用端口 |
| DEBUG | boolean | false | 调试模式 |
| MONGODB_URL | string | - | MongoDB 连接地址 |
| MYSQL_HOST | string | localhost | MySQL 主机 |
| REDIS_HOST | string | localhost | Redis 主机 |
| JWT_SECRET | string | - | JWT 密钥 |
| LOG_LEVEL | string | info | 日志级别 |

---

## 部署配置示例

### 开发环境 (.env)

```env
NODE_ENV=development
PORT=3000
DEBUG=true
LOG_LEVEL=debug

MONGODB_URL=mongodb://localhost:27017/gamedb_dev
MYSQL_HOST=localhost
REDIS_HOST=localhost

JWT_SECRET=dev_secret_for_testing_only
```

### 生产环境 (.env.prod)

```env
NODE_ENV=production
PORT=80
DEBUG=false
LOG_LEVEL=warn

MONGODB_URL=mongodb+srv://user:pass@cluster.mongodb.net/gamedb
MYSQL_HOST=prod-mysql.example.com
REDIS_HOST=prod-redis.example.com

JWT_SECRET=production_secret_key_xxxxx
```

---

## 更新日志

- **2025-11-23**: 初始版本，添加常见配置文件和环境变量
- **TBD**: 添加 Kubernetes ConfigMap 示例
- **TBD**: 补充 CI/CD 环境变量配置