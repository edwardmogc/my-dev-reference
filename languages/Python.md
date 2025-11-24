# Python 完整参考指南

> 范围：Python安装、包管理、虚拟环境、框架使用、性能监控、部署管理   
> 版本：Python 3.9+, pip 21+  
> 最后更新：2025-11-24

---

## 目录

1. [Python环境设置](#python环境设置)
2. [虚拟环境管理](#虚拟环境管理)
3. [Pip包管理](#pip包管理)
4. [应用运行](#应用运行)
5. [Flask框架](#flask框架)
6. [Django框架](#django框架)
7. [Gunicorn WSGI服务器](#gunicorn-wsgi服务器)
8. [性能监控](#性能监控)
9. [应用部署](#应用部署)
10. [常见问题](#常见问题)
11. [快速参考](#快速参考)

---

## Python环境设置

### Python安装

```bash
# 查看系统中的Python版本
python3 --version
python3 -V

# 查看Python可执行路径
which python3

# 查看pip版本
pip3 --version

# 更新包列表
sudo apt update

# 安装Python 3.9
sudo apt install -y python3.9 python3.9-venv python3.9-dev

# 安装Python 3.10
sudo apt install -y python3.10 python3.10-venv python3.10-dev

# 安装Python 3.11（推荐）
sudo apt install -y python3.11 python3.11-venv python3.11-dev

# 安装Python开发工具
sudo apt install -y python3-dev

# 安装pip
sudo apt install -y python3-pip

# 升级pip
python3 -m pip install --upgrade pip

# 安装setuptools和wheel
python3 -m pip install setuptools wheel

# 安装常用开发工具
sudo apt install -y build-essential
```

### Python版本管理

```bash
# 查看所有已安装的Python版本
update-alternatives --list python3

# 设置Python命令的默认版本
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.9 1
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.10 2
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 3

# 交互式选择默认版本
sudo update-alternatives --config python3

# 删除某个版本的链接
sudo update-alternatives --remove python3 /usr/bin/python3.9

# 查看当前默认的Python版本
python3 --version

# 临时使用特定版本
python3.11 --version

# 创建python别名指向python3
sudo ln -sf /usr/bin/python3 /usr/bin/python

# 验证python命令
python --version
```

### 配置pip源

```bash
# 查看当前pip源
pip3 config get global.index-url

# 设置为淘宝源
pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

# 设置为官方源
pip3 config set global.index-url https://pypi.org/simple

# 设置为阿里源
pip3 config set global.index-url https://mirrors.aliyun.com/pypi/simple

# 设置为腾讯源
pip3 config set global.index-url https://mirrors.cloud.tencent.com/pypi/simple

# 查看全部配置
pip3 config list -v

# 查看配置文件位置
cat ~/.pip/pip.conf

# 创建配置文件
mkdir -p ~/.pip
cat > ~/.pip/pip.conf << 'EOF'
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host = pypi.tuna.tsinghua.edu.cn
EOF

# 为特定源设置可信主机
pip3 config set global.trusted-host "pypi.tuna.tsinghua.edu.cn"

# 安装时临时使用其他源
pip3 install -i https://pypi.org/simple/ package-name
```

---

## 虚拟环境管理

### 使用venv创建虚拟环境

```bash
# 创建虚拟环境
python3 -m venv venv

# 创建特定Python版本的虚拟环境
python3.11 -m venv venv

# 列出虚拟环境内容
ls -la venv/

# 激活虚拟环境（Linux/Mac）
source venv/bin/activate

# 激活虚拟环境（Windows）
venv\Scripts\activate.bat

# 查看激活状态
which python
pip list

# 在虚拟环境中升级pip
python -m pip install --upgrade pip

# 在虚拟环境中安装包
pip install django flask

# 导出虚拟环境依赖
pip freeze > requirements.txt

# 查看requirements.txt
cat requirements.txt

# 从requirements.txt安装依赖
pip install -r requirements.txt

# 删除虚拟环境（只需删除venv目录）
rm -rf venv/

# 退出虚拟环境
deactivate

# 在虚拟环境中运行Python文件
python app.py

# 在虚拟环境中使用pip
./venv/bin/pip install django
```

### 使用virtualenvwrapper增强虚拟环境管理

```bash
# 安装virtualenvwrapper
pip3 install virtualenvwrapper

# 配置bash配置文件（~/.bashrc或~/.zshrc）
export WORKON_HOME=~/.virtualenvs
mkdir -p $WORKON_HOME
source /usr/local/bin/virtualenvwrapper.sh

# 重新加载配置
source ~/.bashrc

# 创建虚拟环境
mkvirtualenv myenv
mkvirtualenv -p python3.11 myenv

# 列出所有虚拟环境
lsvirtualenv

# 激活虚拟环境
workon myenv

# 自动激活虚拟环境（进入目录时）
setvirtualenvproject

# 退出虚拟环境
deactivate

# 删除虚拟环境
rmvirtualenv myenv

# 复制虚拟环境
cpvirtualenv myenv newenv

# 查看虚拟环境信息
lsvirtualenv -d

# 在虚拟环境中执行命令
workon myenv && python script.py
```

### 使用Poetry管理项目依赖（推荐）

```bash
# 安装Poetry
curl -sSL https://install.python-poetry.org | python3 -

# 验证安装
poetry --version

# 初始化项目
poetry init

# 创建项目
poetry new myproject

# 添加依赖
poetry add flask
poetry add -D pytest

# 安装项目依赖
poetry install

# 更新依赖
poetry update

# 显示依赖信息
poetry show
poetry show --outdated

# 导出依赖到requirements.txt
poetry export -f requirements.txt > requirements.txt

# 激活虚拟环境
poetry shell

# 在虚拟环境中运行脚本
poetry run python app.py

# 检查依赖冲突
poetry check

# 发布项目
poetry publish
```

---

## Pip包管理

### 基础pip命令

```bash
# 显示pip版本
pip --version
pip3 --version

# 升级pip
python -m pip install --upgrade pip

# 安装包
pip install package-name

# 安装特定版本
pip install package-name==1.0.0

# 安装版本范围
pip install "package-name>=1.0.0"
pip install "package-name>=1.0.0,<2.0.0"

# 安装最新版本
pip install package-name --upgrade

# 全局安装（不推荐）
sudo pip3 install package-name

# 在虚拟环境中安装
source venv/bin/activate
pip install package-name

# 查看已安装的包
pip list

# 查看包信息
pip show package-name

# 检查过期的包
pip list --outdated

# 更新包
pip install --upgrade package-name

# 卸载包
pip uninstall package-name

# 卸载多个包
pip uninstall package1 package2 package3

# 使用requirements文件安装
pip install -r requirements.txt

# 生成requirements文件
pip freeze > requirements.txt

# 搜索包（建议在pip网站搜索）
pip search package-name  # 已弃用

# 安装时忽略版本冲突
pip install package-name --no-deps
```

### requirements文件管理

```bash
# 创建requirements.txt
cat > requirements.txt << 'EOF'
flask==2.3.0
django==4.2.0
requests>=2.28.0
numpy>=1.20.0,<2.0.0
pandas==1.5.3
pytest==7.3.0
EOF

# 安装requirements文件中的所有包
pip install -r requirements.txt

# 分离开发和生产依赖
cat > requirements-dev.txt << 'EOF'
-r requirements.txt
pytest==7.3.0
black==23.3.0
flake8==6.0.0
EOF

# 安装开发依赖
pip install -r requirements-dev.txt

# 只安装生产依赖
pip install -r requirements.txt

# 生成锁定版本的requirements文件
pip freeze > requirements-frozen.txt

# 更新requirements文件中的所有包
pip install --upgrade -r requirements.txt

# 查看requirements文件中哪些包需要更新
pip list --outdated -r requirements.txt

# 检查requirements文件中的问题
pip install -r requirements.txt --dry-run
```

### 依赖和安全

```bash
# 检查安全漏洞
pip install safety
safety check

# 更详细的安全检查
safety check --full-report

# 使用pip-audit检查漏洞
pip install pip-audit
pip-audit

# 修复漏洞
pip-audit --fix

# 显示漏洞详情
pip-audit --desc

# 检查license兼容性
pip install pip-licenses
pip-licenses

# 查看包的依赖树
pip install pipdeptree
pipdeptree

# 显示特定包的依赖
pipdeptree -p package-name

# 找出哪些包依赖于某个包
pipdeptree -r package-name
```

---

## 应用运行

### 直接运行Python

```bash
# 运行Python脚本
python app.py
python3 app.py

# 带参数运行
python app.py --port 3000 --debug

# 运行Python模块
python -m flask run

# 在虚拟环境中运行
source venv/bin/activate
python app.py

# 指定Python版本运行
python3.11 app.py

# 后台运行
python app.py &

# 后台运行并保存PID
python app.py > app.log 2>&1 &
echo $! > app.pid

# 使用nohup后台运行
nohup python app.py > app.log 2>&1 &

# 进入Python交互式shell
python3

# 执行Python代码
python -c "print('Hello World')"

# 检查Python语法
python -m py_compile app.py

# 查看Python信息
python --version
python -V
python --verbose

# 启用调试模式
python -m pdb app.py

# 性能分析
python -m cProfile -s cumulative app.py

# 内存分析
pip install memory-profiler
python -m memory_profiler app.py
```

### 环境变量配置

```bash
# 设置环境变量（临时）
export FLASK_ENV=development
export FLASK_DEBUG=1
export DATABASE_URL=postgresql://user:pass@localhost/db

# 验证环境变量
echo $FLASK_ENV

# 在启动时设置环境变量
FLASK_ENV=production FLASK_DEBUG=0 python app.py

# 使用.env文件（需要python-dotenv包）
pip install python-dotenv

# 创建.env文件
cat > .env << 'EOF'
FLASK_ENV=development
FLASK_DEBUG=1
DATABASE_URL=postgresql://user:pass@localhost/db
SECRET_KEY=your-secret-key-here
API_KEY=api-key
REDIS_URL=redis://localhost:6379
EOF

# 在应用中加载.env文件
from dotenv import load_dotenv
import os
load_dotenv()
env = os.getenv('FLASK_ENV')

# 查看所有环境变量
env
printenv

# 永久设置环境变量（编辑~/.bashrc）
echo 'export FLASK_ENV=production' >> ~/.bashrc
source ~/.bashrc
```

### Python性能参数

```bash
# 优化python执行
python -O app.py  # 移除assert和__debug__

# 启用哈希随机化（安全性）
python -R app.py

# 显示导入和模块加载的信息
python -v app.py

# 限制字节码缓存
python -B app.py

# 启用warnings
python -W all app.py

# 禁用site包
python -S app.py

# 搜索路径
python -c "import sys; print(sys.path)"

# 修改搜索路径
PYTHONPATH=/custom/path python app.py

# 使用PyPy运行（JIT编译，提高性能）
pypy3 app.py

# 使用Cython编译关键代码
pip install cython

# 启用profile mode
python -m profile -s cumulative app.py

# 查看所有Python选项
python --help
```

---

## Flask框架

### 创建Flask应用

```bash
# 安装Flask
pip install flask

# 创建基础Flask应用
cat > app.py << 'EOF'
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello World!'

@app.route('/api/users', methods=['GET'])
def get_users():
    return jsonify([
        {'id': 1, 'name': 'User 1'},
        {'id': 2, 'name': 'User 2'}
    ])

@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.json
    return jsonify({'id': 1, 'name': data.get('name')}), 201

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=3000)
EOF

# 运行Flask应用
python app.py

# 指定端口和主机
python app.py --host 0.0.0.0 --port 8000

# 禁用调试模式
python app.py --no-debug

# 重载模式
python app.py --reload

# 线程模式
python app.py --threaded

# 使用flask run命令
export FLASK_APP=app.py
export FLASK_ENV=development
flask run

# 指定主机和端口
flask run --host 0.0.0.0 --port 8000
```

### Flask常用扩展

```bash
# 数据库操作
pip install flask-sqlalchemy

# 表单验证
pip install flask-wtf

# 认证和授权
pip install flask-login
pip install flask-jwt-extended

# CORS支持
pip install flask-cors

# RESTful API
pip install flask-restful

# API文档
pip install flask-swagger
pip install flasgger

# 缓存
pip install flask-caching

# 任务队列
pip install celery

# 配置管理
pip install python-dotenv

# 日志
pip install python-json-logger
```

### Flask应用示例

```bash
# 创建完整的Flask应用
cat > app.py << 'EOF'
from flask import Flask, jsonify, request
from flask_cors import CORS
from functools import wraps
import logging

app = Flask(__name__)
CORS(app)

# 日志配置
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# 错误处理
@app.errorhandler(404)
def not_found(error):
    return jsonify({'error': 'Not found'}), 404

@app.errorhandler(500)
def server_error(error):
    return jsonify({'error': 'Server error'}), 500

# 装饰器：认证
def require_auth(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        token = request.headers.get('Authorization')
        if not token:
            return jsonify({'error': 'Unauthorized'}), 401
        return f(*args, **kwargs)
    return decorated

# 路由
@app.route('/health', methods=['GET'])
def health():
    return jsonify({'status': 'ok'})

@app.route('/api/users', methods=['GET'])
def get_users():
    users = [{'id': 1, 'name': 'User 1'}]
    return jsonify(users)

@app.route('/api/protected', methods=['GET'])
@require_auth
def protected():
    return jsonify({'data': 'secret'})

if __name__ == '__main__':
    app.run(debug=True)
EOF

python app.py
```

---

## Django框架

### 创建Django项目

```bash
# 安装Django
pip install django

# 创建新项目
django-admin startproject myproject

# 创建应用
cd myproject
python manage.py startapp myapp

# 迁移数据库
python manage.py migrate

# 创建超级用户
python manage.py createsuperuser

# 启动开发服务器
python manage.py runserver

# 指定主机和端口
python manage.py runserver 0.0.0.0:8000

# 运行测试
python manage.py test

# 收集静态文件（生产环境）
python manage.py collectstatic

# 创建数据库备份
python manage.py dumpdata > backup.json

# 恢复数据库
python manage.py loaddata backup.json
```

### Django迁移和数据库

```bash
# 生成迁移文件
python manage.py makemigrations

# 生成迁移并显示SQL
python manage.py sqlmigrate myapp 0001

# 应用迁移
python manage.py migrate

# 查看迁移状态
python manage.py showmigrations

# 回滚迁移
python manage.py migrate myapp 0001

# 清空迁移
python manage.py migrate myapp zero

# 创建数据库表
python manage.py migrate

# 删除所有数据
python manage.py flush

# 导出数据
python manage.py dumpdata myapp.Model > data.json

# 导入数据
python manage.py loaddata data.json
```

### Django管理后台

```bash
# 在models.py中注册模型
from django.contrib import admin
from .models import MyModel

admin.site.register(MyModel)

# 自定义管理后台
class MyModelAdmin(admin.ModelAdmin):
    list_display = ['id', 'name', 'created_at']
    search_fields = ['name']
    list_filter = ['created_at']

admin.site.register(MyModel, MyModelAdmin)

# 创建超级用户
python manage.py createsuperuser

# 访问管理后台
# http://localhost:8000/admin
```

---

## Gunicorn WSGI服务器

### Gunicorn安装和基础使用

```bash
# 安装Gunicorn
pip install gunicorn

# 验证安装
gunicorn --version

# 运行Flask应用
gunicorn app:app

# 指定worker数量
gunicorn -w 4 app:app

# 指定绑定地址和端口
gunicorn -b 0.0.0.0:8000 app:app

# 指定worker类型
gunicorn -w 4 --worker-class sync app:app

# 后台运行
gunicorn -D -w 4 -b 0.0.0.0:8000 app:app

# 保存PID文件
gunicorn -w 4 --pid /tmp/gunicorn.pid -b 0.0.0.0:8000 app:app

# 指定日志文件
gunicorn -w 4 \
  --access-logfile /var/log/gunicorn/access.log \
  --error-logfile /var/log/gunicorn/error.log \
  -b 0.0.0.0:8000 app:app

# 启用访问日志到stdout
gunicorn -w 4 --access-logfile - -b 0.0.0.0:8000 app:app

# 设置超时时间
gunicorn -w 4 --timeout 30 -b 0.0.0.0:8000 app:app

# 设置保活超时
gunicorn -w 4 --keepalive 5 -b 0.0.0.0:8000 app:app

# 优雅重启（reload workers）
gunicorn -w 4 --reload -b 0.0.0.0:8000 app:app

# 设置进程名
gunicorn -w 4 --proc_name gunicorn -b 0.0.0.0:8000 app:app

# 设置日志级别
gunicorn -w 4 --log-level debug -b 0.0.0.0:8000 app:app
```

### Gunicorn配置文件

```bash
# 创建gunicorn配置文件
cat > gunicorn.conf.py << 'EOF'
import os
from pathlib import Path

# 基础配置
workers = 4
worker_class = 'sync'
worker_connections = 1000
timeout = 30
keepalive = 2
max_requests = 1000
max_requests_jitter = 100

# 绑定配置
bind = '0.0.0.0:8000'
backlog = 1024

# 进程命名
proc_name = 'gunicorn'

# 日志配置
accesslog = '/var/log/gunicorn/access.log'
errorlog = '/var/log/gunicorn/error.log'
access_log_format = '%(h)s %(l)s %(u)s %(t)s "%(r)s" %(s)s %(b)s "%(f)s" "%(a)s"'
loglevel = 'info'

# 服务器机制
daemon = False
pidfile = '/var/run/gunicorn.pid'
preload_app = False

# SSL配置
# keyfile = '/path/to/keyfile'
# certfile = '/path/to/certfile'

# 环境
raw_env = ['DJANGO_SETTINGS_MODULE=myproject.settings']
EOF

# 使用配置文件启动
gunicorn -c gunicorn.conf.py app:app

# 查看可用的配置选项
gunicorn --help
```

### Gunicorn与systemd集成

```bash
# 创建systemd服务文件
sudo tee /etc/systemd/system/gunicorn.service > /dev/null << 'EOF'
[Unit]
Description=Gunicorn Application Server
After=network.target

[Service]
User=www-data
WorkingDirectory=/home/www-data/myapp
ExecStart=/usr/local/bin/gunicorn \
  -w 4 \
  -b unix:/run/gunicorn.sock \
  --access-logfile /var/log/gunicorn/access.log \
  --error-logfile /var/log/gunicorn/error.log \
  app:app

Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

# 重载systemd
sudo systemctl daemon-reload

# 启用和启动服务
sudo systemctl enable gunicorn
sudo systemctl start gunicorn

# 查看状态
sudo systemctl status gunicorn

# 查看日志
journalctl -u gunicorn -f
```

---

## 性能监控

### 性能分析工具

```bash
# 使用cProfile分析性能
python -m cProfile -s cumulative app.py | head -50

# 保存性能数据
python -m cProfile -o stats.prof app.py

# 分析保存的数据
python -m pstats stats.prof

# 使用line_profiler分析行性能
pip install line_profiler
kernprof -l -v app.py

# 使用memory_profiler分析内存
pip install memory_profiler
python -m memory_profiler app.py

# 使用py-spy进行采样
pip install py-spy
py-spy record -o profile.svg -- python app.py

# 生成火焰图
python -m cProfile -o stats.prof app.py
python -c "import pstats; p = pstats.Stats('stats.prof'); p.strip_dirs().sort_stats('cumulative').print_stats(20)"
```

### 监控指标

```bash
# 查看进程内存和CPU使用
ps aux | grep python

# 详细的进程信息
ps -p <PID> -o %cpu,%mem,vsz,rss,etime,comm

# 实时监控
top -p <PID>

# 监控网络连接
netstat -tlnp | grep python
ss -tlnp | grep python

# 监控打开的文件
lsof -p <PID>

# 监控进程线程数
cat /proc/<PID>/status | grep Threads

# 查看进程栈追踪
gdb -ex "thread apply all bt full" -batch -p <PID>
```

---

## 应用部署

### Systemd服务配置

```bash
# 创建systemd服务文件
sudo tee /etc/systemd/system/myapp.service > /dev/null << 'EOF'
[Unit]
Description=My Python Application
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/home/appuser/myapp

# 激活虚拟环境
Environment="PATH=/home/appuser/myapp/venv/bin"

ExecStart=/home/appuser/myapp/venv/bin/gunicorn \
  -w 4 \
  -b 0.0.0.0:8000 \
  app:app

Restart=on-failure
RestartSec=10

StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp

TimeoutStopSec=30

[Install]
WantedBy=multi-user.target
EOF

# 启用和启动服务
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp

# 查看状态
sudo systemctl status myapp

# 查看日志
journalctl -u myapp -f
```

### Docker部署

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# 安装依赖
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY . .

# 暴露端口
EXPOSE 8000

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')"

# 启动应用
CMD ["gunicorn", "-w 4", "-b 0.0.0.0:8000", "app:app"]
```

```bash
# 构建镜像
docker build -t myapp:1.0 .

# 运行容器
docker run -d \
  --name myapp \
  -p 8000:8000 \
  -e "FLASK_ENV=production" \
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
        - containerPort: 8000
        env:
        - name: FLASK_ENV
          value: "production"
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
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8000
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
    targetPort: 8000
```

---

## 常见问题

### 问题1: 包版本冲突

**症状**: ImportError或版本不兼容错误

**解决**:
```bash
# 查看依赖冲突
pip check

# 重新安装依赖
pip install --force-reinstall -r requirements.txt

# 使用约束文件
pip install -c constraints.txt -r requirements.txt

# 在虚拟环境中重新安装
deactivate
rm -rf venv/
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 问题2: 内存泄漏

**症状**: 应用内存持续增长

**解决**:
```bash
# 分析内存使用
python -m memory_profiler app.py

# 检查循环引用
pip install objgraph
python -c "import objgraph; objgraph.show_most_common_types(limit=10)"

# 使用tracemalloc
python -c "
import tracemalloc
tracemalloc.start()
# ... run code ...
current, peak = tracemalloc.get_traced_memory()
print(f'Current: {current}; Peak: {peak}')
"
```

### 问题3: 启动缓慢

**症状**: 应用需要很长时间启动

**解决**:
```bash
# 分析启动时间
python -m cProfile -s cumulative app.py 2>&1 | head -20

# 异步加载模块
# 使用lazy_imports库

# 预编译字节码
python -m compileall .
```

### 问题4: 高CPU占用

**症状**: Python进程CPU占用率高

**解决**:
```bash
# 使用py-spy采样
py-spy record -o profile.svg -- python app.py

# 检查热循环
python -m cProfile -s cumulative app.py

# 优化代码或增加工作进程
gunicorn -w 8 app:app  # 增加worker数
```

---

## 快速参考

### 最常用的15个命令

| 用途 | 命令 |
|------|------|
| 查看版本 | `python3 --version` |
| 创建虚拟环境 | `python3 -m venv venv` |
| 激活虚拟环境 | `source venv/bin/activate` |
| 安装包 | `pip install flask` |
| 安装依赖 | `pip install -r requirements.txt` |
| 导出依赖 | `pip freeze > requirements.txt` |
| 运行Flask应用 | `python app.py` |
| 运行Flask开发服务器 | `flask run` |
| 运行Django服务器 | `python manage.py runserver` |
| 使用Gunicorn | `gunicorn app:app` |
| 性能分析 | `python -m cProfile app.py` |
| 检查漏洞 | `pip-audit` |
| 迁移数据库 | `python manage.py migrate` |
| 创建超级用户 | `python manage.py createsuperuser` |
| 后台运行 | `nohup python app.py &` |

### Pip常用命令速查

```bash
pip install flask               # 安装包
pip install flask==2.0.0        # 安装特定版本
pip install -r requirements.txt # 从文件安装
pip list                        # 列出已安装
pip show flask                  # 显示包信息
pip search flask                # 搜索包
pip freeze > requirements.txt   # 导出依赖
pip check                       # 检查冲突
pip-audit                       # 检查漏洞
```

### Django常用命令速查

```bash
python manage.py runserver           # 启动服务器
python manage.py migrate             # 应用迁移
python manage.py makemigrations      # 生成迁移
python manage.py createsuperuser     # 创建超级用户
python manage.py test                # 运行测试
python manage.py collectstatic       # 收集静态文件
python manage.py dumpdata > backup.json  # 备份数据
```

### Gunicorn常用命令速查

```bash
gunicorn app:app                # 基础运行
gunicorn -w 4 app:app          # 4个worker
gunicorn -b 0.0.0.0:8000 app:app  # 指定端口
gunicorn -c gunicorn.conf.py app:app  # 使用配置文件
gunicorn -w 4 --reload app:app # 启用重载
gunicorn -D -w 4 app:app       # 后台运行
```

---

**版本**: 1.0.0  
**最后更新**: 2025-11-24  
**命令数**: 150+  
**配置示例**: 10个