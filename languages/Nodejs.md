# Node.js 完整参考指南

> 范围：Node.js安装、包管理、应用运行、框架使用、性能监控、部署管理  
> 版本：Node.js 16+, npm 8+, Yarn 3+  
> 最后更新：2025-11-24

---

## 目录

1. [Node.js安装和配置](#nodejs安装和配置)
2. [NPM包管理](#npm包管理)
3. [Yarn包管理](#yarn包管理)
4. [应用运行](#应用运行)
5. [Express框架](#express框架)
6. [PM2进程管理](#pm2进程管理)
7. [性能监控](#性能监控)
8. [应用部署](#应用部署)
9. [常见问题](#常见问题)
10. [快速参考](#快速参考)

---

## Node.js安装和配置

### 直接安装（官方仓库）

```bash
# 更新包列表
sudo apt update

# 安装Node.js和npm
sudo apt install -y nodejs npm

# 验证安装
node --version
npm --version

# 查看安装位置
which node
which npm
```

### 使用NVM管理Node.js版本（推荐）

```bash
# 安装NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# 重新加载环境变量
source ~/.bashrc
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# 验证NVM安装
nvm --version

# 查看可用的Node.js版本
nvm list-remote

# 安装特定版本
nvm install 16.18.0
nvm install 18.17.0
nvm install 20.3.0

# 安装LTS版本
nvm install --lts

# 查看已安装的版本
nvm list

# 切换到特定版本
nvm use 18.17.0

# 设置默认版本
nvm alias default 18.17.0

# 在项目中使用特定版本（创建.nvmrc文件）
echo "18.17.0" > .nvmrc
nvm use  # 会自动读取.nvmrc

# 卸载某个版本
nvm uninstall 16.18.0

# 查看当前使用的版本
node --version
nvm current
```

### 使用NodeSource PPA（获得最新版本）

```bash
# 添加NodeSource仓库
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -

# 安装Node.js
sudo apt-get install -y nodejs

# 验证
node --version
npm --version
```

### 配置Node.js全局目录

```bash
# 检查npm全局目录
npm config get prefix

# 设置全局目录到用户目录
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'

# 添加到PATH（编辑~/.bashrc）
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# 验证
npm config get prefix
which npm

# 解决权限问题的方法
# 方法1：修改npm权限（不推荐）
sudo chown -R $(whoami) $(npm config get prefix)/{lib/node_modules,bin,share}

# 方法2：使用sudo安装全局包（不推荐）
# 方法3：使用nvm（推荐）
```

### 配置npm源

```bash
# 查看当前源
npm config get registry

# 设置为淘宝源
npm config set registry https://registry.npmmirror.com

# 设置为官方源
npm config set registry https://registry.npmjs.org/

# 设置为阿里源
npm config set registry https://npm.aliyun.com

# 查看全部配置
npm config list -l

# 查看配置文件位置
cat ~/.npmrc

# 为特定包设置源
npm config set @scope:registry https://registry.npmjs.org/

# 安装时临时使用其他源
npm install --registry https://registry.npmjs.org/
```

---

## NPM包管理

### 项目初始化

```bash
# 交互式初始化项目
npm init

# 快速初始化（使用默认值）
npm init -y

# 初始化时指定参数
npm init -y \
  --scope=@myorg \
  --name=myapp \
  --version=1.0.0 \
  --description="My application" \
  --main=index.js \
  --author="Your Name" \
  --license=MIT

# 查看生成的package.json
cat package.json

# 从模板初始化（需要安装create-react-app等）
npx create-react-app myapp
npx create-next-app myapp
```

### 安装依赖

```bash
# 安装package.json中的所有依赖
npm install

# 安装特定包
npm install express

# 安装特定版本
npm install express@4.18.2

# 安装最新版本
npm install express@latest

# 安装版本范围
npm install "express@^4.18.0"  # 4.18.0 ~ 5.0.0
npm install "express@~4.18.0"  # 4.18.0 ~ 4.19.0
npm install "express@1.x"      # 1.0.0 ~ 2.0.0

# 安装生产依赖（添加到dependencies）
npm install --save express
npm install -S express
npm install --production express

# 安装开发依赖（添加到devDependencies）
npm install --save-dev webpack
npm install -D webpack
npm install --development webpack

# 全局安装
npm install -g pm2
npm install -g nodemon
npm install -g typescript

# 安装可选依赖
npm install --optional package-name

# 安装精确版本（不使用^或~）
npm install --save-exact express@4.18.2

# 在CI环境中安装（严格遵循package-lock.json）
npm ci

# 使用淘宝源加速安装
npm install --registry https://registry.npmmirror.com
```

### 卸载和更新依赖

```bash
# 卸载包
npm uninstall express
npm remove express
npm rm express

# 全局卸载
npm uninstall -g pm2

# 删除package.json中的记录
npm uninstall --save express

# 更新所有依赖
npm update

# 更新特定包
npm update express

# 检查过期的包
npm outdated

# 显示过期的主版本
npm outdated --depth=0

# 升级包到最新版本
npm install express@latest

# 查看包的版本信息
npm view express

# 查看package的所有版本
npm view express versions

# 查看已安装的包
npm list
npm list --depth=0

# 查看全局安装的包
npm list -g
npm list -g --depth=0

# 查看特定包的信息
npm list express

# 导出依赖版本列表
npm list --production > production-deps.txt
```

### 包版本和更新

```bash
# 导出依赖（freeze当前版本）
npm freeze > dependencies.txt

# 导出当前依赖到package.json（已默认导出）

# 查看npm更新
npm update --save

# 检查安全漏洞
npm audit

# 显示漏洞详情
npm audit --long

# 只显示高风险漏洞
npm audit --audit-level=high

# 自动修复漏洞
npm audit fix

# 只修复主要版本不变的漏洞
npm audit fix --force

# 生成审计报告
npm audit --json > audit.json

# 检查特定包的漏洞
npm audit --package=express
```

### 搜索和发布

```bash
# 搜索包（npm官网更便捷）
npm search express

# 查看包详情
npm view express
npm info express

# 查看包的README
npm view express readme

# 查看包的依赖
npm view express dependencies

# 登录npm账户（用于发布）
npm login

# 查看当前登录用户
npm whoami

# 发布包到npm仓库
npm publish

# 发布为特定版本
npm version major|minor|patch
npm publish

# 发布到私有仓库
npm publish --registry https://npm.example.com/

# 设置包的访问权限为public
npm access public [<package>]

# 设置包为私有
npm access private [<package>]

# 登出
npm logout
```

### NPM脚本

```bash
# 在package.json中定义脚本示例
cat > package.json << 'EOF'
{
  "name": "myapp",
  "version": "1.0.0",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest",
    "build": "webpack",
    "lint": "eslint .",
    "format": "prettier --write .",
    "prepare": "husky install"
  }
}
EOF

# 运行脚本
npm start        # 运行start脚本
npm run dev      # 运行dev脚本
npm test         # 运行test脚本
npm run build    # 运行build脚本

# 查看所有可用脚本
npm run

# 运行单个npm脚本
npm run-script start

# 传递参数给脚本
npm run dev -- --port 3001

# 生命周期脚本（自动运行）
# - preinstall: install前运行
# - postinstall: install后运行
# - prestart: start前运行
# - poststart: start后运行
```

---

## Yarn包管理

### Yarn安装和配置

```bash
# 安装Yarn
npm install -g yarn

# 验证Yarn安装
yarn --version

# 使用curl安装
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update
sudo apt install yarn

# 设置Yarn源
yarn config set registry https://registry.npmmirror.com

# 查看Yarn配置
yarn config list

# 设置Yarn全局目录
yarn config set global-folder ~/.yarn/global
```

### Yarn常用命令

```bash
# 初始化项目
yarn init
yarn init -y

# 安装依赖
yarn install
yarn

# 安装特定包
yarn add express

# 安装开发依赖
yarn add --dev webpack

# 安装全局包
yarn global add pm2

# 删除包
yarn remove express

# 更新包
yarn upgrade
yarn upgrade express

# 检查过期的包
yarn outdated

# 检查安全漏洞
yarn audit

# 修复安全漏洞
yarn audit --fix

# 清理缓存
yarn cache clean

# 导出依赖
yarn list

# 导出树形依赖
yarn tree

# 查看包信息
yarn info express

# 搜索包
yarn search express

# 发布包
yarn publish

# 登录
yarn login

# 登出
yarn logout
```

---

## 应用运行

### 直接运行Node.js

```bash
# 运行脚本文件
node app.js

# 运行npm脚本
npm start

# 带参数运行
node app.js --port 3001

# 使用环境变量
NODE_ENV=production node app.js

# 启用inspector（调试）
node --inspect app.js

# 启用inspector并暂停
node --inspect-brk app.js

# 指定inspect主机和端口
node --inspect=0.0.0.0:9229 app.js

# 监听文件变化自动重启（需要nodemon）
npm install -g nodemon
nodemon app.js

# 使用nodemon运行npm脚本
nodemon npm run dev

# 在后台运行
node app.js &

# 后台运行并保存PID
node app.js > app.log 2>&1 &
echo $! > app.pid

# 使用nohup后台运行（进程不受终端关闭影响）
nohup node app.js > app.log 2>&1 &

# 启用多线程（Worker Threads）
node --experimental-worker app.js

# 启用ES modules
node --experimental-modules app.js

# 查看Node.js版本和信息
node --version
node -v
node --verbose-version

# 进入REPL（交互式shell）
node

# 执行Node.js代码
node -e "console.log('Hello')"

# 检查脚本语法
node --check app.js

# 查看所有Node.js选项
node --help
```

### 环境变量配置

```bash
# 设置环境变量（临时）
export NODE_ENV=production
export DATABASE_URL=mongodb://localhost:27017
export API_KEY=secret123

# 验证环境变量
echo $NODE_ENV
echo $DATABASE_URL

# 在启动时设置环境变量
NODE_ENV=production DATABASE_URL=mongodb://localhost PORT=3000 node app.js

# 使用.env文件（需要dotenv包）
npm install dotenv

# 创建.env文件
cat > .env << 'EOF'
NODE_ENV=development
DATABASE_URL=mongodb://localhost:27017
PORT=3000
API_KEY=secret123
REDIS_URL=redis://localhost:6379
EOF

# 在应用中加载.env文件
# 在app.js顶部添加
require('dotenv').config();

# 查看所有环境变量
env
printenv

# 永久设置环境变量（编辑~/.bashrc）
echo 'export NODE_ENV=production' >> ~/.bashrc
source ~/.bashrc
```

### Node.js性能参数

```bash
# 增加堆内存
node --max-old-space-size=4096 app.js

# 详解：
# --max-old-space-size=4096  设置最大堆为4GB
# --max-semi-space-size=1024 设置新生代最大为1GB

# 启用代码缓存
node --code-cache-path=/tmp/cache app.js

# 启用性能钩子
node --enable-fips app.js

# 禁用deprecation警告
node --no-deprecation app.js

# 显示deprecation警告
node --trace-deprecation app.js

# 启用experimental功能
node --experimental-modules app.js

# 多线程Worker
node --enable-worker-threads app.js

# 典型的生产环境配置
NODE_ENV=production \
NODE_OPTIONS="--max-old-space-size=4096 --enable-source-maps" \
node app.js
```

---

## Express框架

### 创建Express应用

```bash
# 方式1：使用express-generator
npm install -g express-generator
express myapp --view=ejs
cd myapp
npm install

# 方式2：手动创建
npm init -y
npm install express

# 创建app.js文件
cat > app.js << 'EOF'
const express = require('express');
const app = express();

// 中间件
app.use(express.json());
app.use(express.static('public'));

// 路由
app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.get('/api/users', (req, res) => {
  res.json([
    { id: 1, name: 'User 1' },
    { id: 2, name: 'User 2' }
  ]);
});

// 错误处理
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something went wrong!');
});

// 启动服务器
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
EOF

# 运行应用
npm start
```

### Express常用中间件

```bash
# 安装常用中间件
npm install body-parser
npm install morgan
npm install cors
npm install helmet
npm install compression
npm install multer
npm install session-parser

# 使用示例
const express = require('express');
const bodyParser = require('body-parser');
const morgan = require('morgan');
const cors = require('cors');
const helmet = require('helmet');
const compression = require('compression');

const app = express();

// 日志中间件
app.use(morgan('combined'));

// 安全中间件
app.use(helmet());

// CORS中间件
app.use(cors());

// 压缩响应
app.use(compression());

// 解析JSON
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));

// 静态文件
app.use(express.static('public'));
```

### Express路由

```bash
# 创建路由文件
mkdir routes
cat > routes/users.js << 'EOF'
const express = require('express');
const router = express.Router();

// GET所有用户
router.get('/', (req, res) => {
  res.json([{ id: 1, name: 'User 1' }]);
});

// GET单个用户
router.get('/:id', (req, res) => {
  res.json({ id: req.params.id, name: 'User' });
});

// POST创建用户
router.post('/', (req, res) => {
  res.json({ id: 1, name: req.body.name });
});

// PUT更新用户
router.put('/:id', (req, res) => {
  res.json({ id: req.params.id, name: req.body.name });
});

// DELETE删除用户
router.delete('/:id', (req, res) => {
  res.json({ message: 'User deleted' });
});

module.exports = router;
EOF

# 在app.js中使用路由
const usersRouter = require('./routes/users');
app.use('/api/users', usersRouter);
```

### Express应用部署

```bash
# 创建Dockerfile
cat > Dockerfile << 'EOF'
FROM node:18-alpine

WORKDIR /app

# 复制package文件
COPY package*.json ./

# 安装依赖
RUN npm ci --only=production

# 复制应用代码
COPY . .

# 暴露端口
EXPOSE 3000

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD node healthcheck.js

# 启动应用
CMD ["npm", "start"]
EOF

# 构建镜像
docker build -t myapp:1.0 .

# 运行容器
docker run -d -p 3000:3000 --name myapp myapp:1.0
```

---

## PM2进程管理

### PM2安装和基础命令

```bash
# 全局安装PM2
npm install -g pm2

# 验证安装
pm2 --version

# 启动应用
pm2 start app.js

# 带名字启动
pm2 start app.js --name "myapp"

# 后台启动多个应用
pm2 start app.js --name "web"
pm2 start worker.js --name "worker"

# 使用npm脚本启动
pm2 start npm -- start

# 启动package.json中的main字段
pm2 start package.json

# 多进程模式（利用多核CPU）
pm2 start app.js -i max
pm2 start app.js -i 4  # 启动4个进程

# 启用自动重启
pm2 start app.js --watch

# 启用自动重启（仅监听特定文件）
pm2 start app.js --watch -- --ignore=logs
```

### PM2常用命令

```bash
# 列出所有应用
pm2 list
pm2 ls

# 查看应用详细信息
pm2 show myapp

# 查看应用日志
pm2 logs
pm2 logs myapp

# 查看最后100行日志
pm2 logs myapp --lines 100

# 实时监控应用
pm2 monit

# 停止应用
pm2 stop myapp

# 启动已停止的应用
pm2 start myapp

# 重启应用
pm2 restart myapp

# 重载应用（零停机重启）
pm2 reload myapp

# 重载所有应用
pm2 reload all

# 删除应用
pm2 delete myapp

# 删除所有应用
pm2 delete all

# 启用应用开机自启
pm2 startup
pm2 save

# 禁用开机自启
pm2 unstartup

# 保存进程列表
pm2 save

# 恢复保存的进程列表
pm2 resurrect

# 查看PM2日志
pm2 logs PM2

# 清理PM2日志
pm2 flush
```

### PM2配置文件

```bash
# 创建ecosystem.config.js配置文件
cat > ecosystem.config.js << 'EOF'
module.exports = {
  apps: [
    {
      name: 'web',
      script: 'app.js',
      instances: 'max',
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'development',
        PORT: 3000
      },
      env_production: {
        NODE_ENV: 'production',
        PORT: 3000
      },
      // 自动重启
      watch: true,
      ignore_watch: ['node_modules', 'logs'],
      // 日志
      out_file: 'logs/out.log',
      error_file: 'logs/error.log',
      // 重启策略
      max_restarts: 10,
      min_uptime: '10s',
      // 优雅关闭
      kill_timeout: 5000,
      listen_timeout: 3000,
      // 资源限制
      max_memory_restart: '500M'
    },
    {
      name: 'worker',
      script: 'worker.js',
      instances: 1,
      exec_mode: 'fork',
      cron_restart: '0 0 * * *'  // 每天0点重启
    }
  ]
};
EOF

# 使用配置文件启动
pm2 start ecosystem.config.js

# 使用特定环境启动
pm2 start ecosystem.config.js --env production

# 重新加载配置
pm2 restart ecosystem.config.js
```

### PM2监控和日志

```bash
# 启用PM2监控
pm2 web          # 访问 http://localhost:9615

# 设置警告阈值
pm2 trigger myapp "value > 90" "Email alert"

# 监控内存和CPU
pm2 monit

# 查看每个进程的资源使用
pm2 list

# 详细的应用统计
pm2 statsd  # 启用statsd集成

# 设置日志级别
pm2 logs myapp --level info

# 导出日志
pm2 logs myapp > app-logs.txt

# 定期清理日志
pm2 install pm2-logrotate

# 查看PM2配置
cat ~/.pm2/conf.js
```

---

## 性能监控

### 内置Inspector

```bash
# 启用Node.js Inspector
node --inspect app.js

# 指定Inspector地址和端口
node --inspect=0.0.0.0:9229 app.js

# 自动打开Chrome DevTools
node --inspect --inspect-publish-uid=http app.js

# 使用break point（代码执行前暂停）
node --inspect-brk app.js

# 在Chrome中打开
# 1. 打开Chrome
# 2. 输入 chrome://inspect
# 3. 点击inspect

# 使用命令行debugger
node inspect app.js
```

### 性能分析

```bash
# 启用CPU性能分析
node --prof app.js

# 处理prof文件
node --prof-process isolate-*.log > profile.txt

# 使用clinic.js进行性能诊断
npm install -g clinic
clinic doctor -- node app.js

# 查看性能瓶颈
clinic bubbleprof -- node app.js

# 性能对比
clinic flame -- node app.js

# 内存分析
node --max-old-space-size=4096 app.js

# 使用0x可视化性能
npm install -g 0x
0x ./app.js
```

### 监控指标

```bash
# 查看进程监控信息
pm2 show myapp

# 实时监控
pm2 monit

# 收集监控数据
pm2 save-dump

# 导出日志和统计
pm2 logs myapp --lines 1000 > logs.txt

# CPU和内存使用
ps aux | grep node

# 详细的进程信息
ps -p <PID> -o %cpu,%mem,vsz,rss,etime,comm

# 实时监控指标
watch -n 1 'ps aux | grep node'
```

---

## 应用部署

### Systemd服务配置

```bash
# 创建systemd服务文件
sudo tee /etc/systemd/system/myapp.service > /dev/null << 'EOF'
[Unit]
Description=My Node.js Application
After=network.target

[Service]
Type=simple
User=nodeuser
WorkingDirectory=/home/nodeuser/myapp

# 使用nvm管理的Node.js
ExecStart=/home/nodeuser/.nvm/versions/node/v18.17.0/bin/node /home/nodeuser/myapp/app.js

# 环境变量
Environment="NODE_ENV=production"
Environment="PORT=3000"

# 自动重启
Restart=on-failure
RestartSec=10

# 日志
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp

# 优雅关闭时间
TimeoutStopSec=30

[Install]
WantedBy=multi-user.target
EOF

# 重载systemd
sudo systemctl daemon-reload

# 启用服务（开机自启）
sudo systemctl enable myapp

# 启动服务
sudo systemctl start myapp

# 查看状态
sudo systemctl status myapp

# 查看日志
journalctl -u myapp -f
```

### Docker部署

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

# 复制package文件
COPY package*.json ./

# 安装依赖
RUN npm ci --only=production

# 复制应用代码
COPY . .

# 暴露端口
EXPOSE 3000

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD node healthcheck.js

# 环境变量
ENV NODE_ENV=production

# 启动应用
CMD ["node", "app.js"]
```

```bash
# 构建镜像
docker build -t myapp:1.0 .

# 运行容器
docker run -d \
  --name myapp \
  -p 3000:3000 \
  -e "NODE_ENV=production" \
  -e "PORT=3000" \
  myapp:1.0

# 查看日志
docker logs -f myapp
```

### Kubernetes部署

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: PORT
          value: "3000"
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
```

---

## 常见问题

### 问题1: 内存泄漏

**症状**: 应用内存持续增长，最终被杀死

**诊断**:
```bash
# 使用clinic.js诊断
clinic doctor -- node app.js

# 导出堆文件
node --heap-prof app.js

# 分析堆文件
node --heap-prof-dir=. app.js
```

**解决**:
```bash
# 增加内存限制
node --max-old-space-size=4096 app.js

# 使用PM2重启应用
pm2 start app.js --max-memory-restart 500M

# 检查代码中的内存泄漏
# - 检查全局变量
# - 检查事件监听器是否解绑
# - 检查定时器是否清理
# - 检查缓存大小
```

### 问题2: 应用启动缓慢

**症状**: 应用需要很长时间才能启动

**诊断**:
```bash
# 测量启动时间
time node app.js

# 使用clinic.js分析
clinic doctor -- node app.js

# 启用NODE_DEBUG查看启动过程
NODE_DEBUG=http node app.js
```

**解决**:
```bash
# 异步加载模块
const lazyRequire = (path) => {
  let module;
  return () => module || (module = require(path));
};

# 启用代码缓存
node --code-cache-path=/tmp app.js

# 减少启动时加载的依赖
```

### 问题3: CPU占用率高

**症状**: Node.js进程CPU占用接近100%

**诊断**:
```bash
# 使用flamegraph分析
npm install -g 0x
0x ./app.js

# 使用clinic.js
clinic flame -- node app.js

# 查看进程信息
ps aux | grep node
```

**解决**:
```bash
# 使用cluster模式
pm2 start app.js -i max

# 优化代码（避免热循环）
# 使用缓存避免重复计算
# 异步处理CPU密集任务

# 使用Worker Threads处理CPU密集任务
const { Worker } = require('worker_threads');
```

### 问题4: 端口被占用

**症状**: EADDRINUSE错误

**诊断**:
```bash
# 查看占用端口的进程
lsof -i :3000
ss -tlnp | grep :3000

# 获取进程PID
netstat -tlnp | grep :3000
```

**解决**:
```bash
# 杀死占用端口的进程
kill -9 <PID>

# 改变应用端口
PORT=3001 node app.js

# 在应用中优雅处理端口
const PORT = process.env.PORT || 3000;
const server = app.listen(PORT, () => {
  console.log(`Server on port ${PORT}`);
});
```

---

## 快速参考

### 最常用的15个命令

| 用途 | 命令 |
|------|------|
| 查看版本 | `node --version` |
| 运行应用 | `npm start` |
| 运行开发模式 | `npm run dev` |
| 安装依赖 | `npm install` |
| 添加包 | `npm install express` |
| 删除包 | `npm uninstall express` |
| 更新包 | `npm update` |
| 检查漏洞 | `npm audit` |
| 修复漏洞 | `npm audit fix` |
| 启动PM2应用 | `pm2 start app.js` |
| 查看PM2应用 | `pm2 list` |
| 查看PM2日志 | `pm2 logs` |
| 停止PM2应用 | `pm2 stop app.js` |
| 启用Inspector | `node --inspect app.js` |
| 后台运行 | `nohup node app.js &` |

### NPM常用命令速查

```bash
npm init -y              # 快速初始化
npm install              # 安装依赖
npm install express      # 安装包
npm install -D webpack   # 安装开发依赖
npm install -g pm2       # 全局安装
npm uninstall express    # 卸载包
npm update               # 更新包
npm audit                # 检查漏洞
npm audit fix            # 修复漏洞
npm list                 # 列出依赖
npm outdated             # 检查过期
npm run dev              # 运行脚本
npm start                # 启动应用
npm test                 # 运行测试
```

### PM2常用命令速查

```bash
pm2 start app.js              # 启动应用
pm2 start app.js -i max       # 多进程启动
pm2 list                      # 列出应用
pm2 logs                      # 查看日志
pm2 monit                     # 实时监控
pm2 stop app                  # 停止应用
pm2 restart app               # 重启应用
pm2 reload app                # 零停机重启
pm2 delete app                # 删除应用
pm2 startup                   # 开机自启
pm2 save                      # 保存配置
```

---

**版本**: 1.0.0  
**最后更新**: 2025-11-24  
**命令数**: 150+  
**配置示例**: 8个