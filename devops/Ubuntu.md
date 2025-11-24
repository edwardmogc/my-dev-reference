# Ubuntu 系统参考指南

> 范围：系统管理、服务运行、容器编排、监控部署  
> 版本：Ubuntu 20.04 LTS / 22.04 LTS  
> 最后更新：2025-11-24

---

## 目录

1. [安装和启动](#安装和启动)
2. [核心概念](#核心概念)
3. [系统服务管理](#系统服务管理)
4. [常用命令](#常用命令)
5. [配置示例](#配置示例)
6. [实战场景](#实战场景)
7. [监控和调试](#监控和调试)
8. [故障排查](#故障排查)

---

## 安装和启动

### 系统更新和基础工具安装

```bash
# 更新包列表
sudo apt update

# 升级已安装的包
sudo apt upgrade -y

# 安装基础工具
sudo apt install -y curl wget git vim htop net-tools
```

### 验证系统信息

```bash
# 查看系统版本
lsb_release -a
uname -a

# 查看内核版本
uname -r

# 查看系统架构
dpkg --print-architecture
```

---

## 核心概念

### 概念 1: Systemd 和 Systemctl

**定义**: systemd是现代Linux系统的系统和服务管理器，取代了传统的init系统。systemctl是与systemd交互的主要命令行工具。

**作用**: 管理系统服务的启动、停止、重启、启用、禁用，以及查看服务状态和日志。

**关键属性**:
- 单元文件位置: `/etc/systemd/system/` 和 `/lib/systemd/system/`
- 用户单元位置: `~/.config/systemd/user/`
- 服务状态: enabled/disabled, active/inactive
- 启动类型: Type=simple, Type=forking, Type=oneshot, Type=notify

### 概念 2: 服务单元（Service Unit）

**定义**: systemd中的服务单元定义了如何启动和管理特定的应用程序或服务。

**结构**:
```
[Unit]              # 单元定义
  Description=      # 服务描述
  After=            # 启动依赖关系
  Wants=            # 弱依赖关系

[Service]           # 服务配置
  Type=             # 服务类型
  ExecStart=        # 启动命令
  ExecStop=         # 停止命令
  Restart=          # 重启策略
  RestartSec=       # 重启延迟

[Install]           # 安装配置
  WantedBy=         # 在什么目标下启用
```

### 概念 3: 服务依赖和目标（Target）

**定义**: Target是systemd中的一个逻辑分组，用于组织系统启动序列和依赖关系。常见的有multi-user.target（多用户模式）、graphical.target（图形界面）等。

**关键目标**:
- multi-user.target: 多用户模式（无图形界面）
- graphical.target: 图形用户界面模式
- rescue.target: 救援模式
- emergency.target: 紧急模式

---

## 系统服务管理

### 基础服务命令

```bash
# 启动服务
sudo systemctl start <service-name>

# 停止服务
sudo systemctl stop <service-name>

# 重启服务
sudo systemctl restart <service-name>

# 重载配置（不停止服务）
sudo systemctl reload <service-name>

# 重新加载配置并重启
sudo systemctl reload-or-restart <service-name>

# 使服务在系统启动时自动启动
sudo systemctl enable <service-name>

# 禁用服务开机自启
sudo systemctl disable <service-name>

# 查看服务是否启用
sudo systemctl is-enabled <service-name>
```

### 服务查看和监控

```bash
# 查看指定服务状态
sudo systemctl status <service-name>

# 列出所有运行中的服务
systemctl list-units --type=service --state=running

# 列出所有已启用的服务
systemctl list-unit-files --type=service --state=enabled

# 列出所有禁用的服务
systemctl list-unit-files --type=service --state=disabled

# 列出所有服务单元（包括运行和已停止）
systemctl list-units --type=service

# 列出所有已安装的服务单元文件
systemctl list-unit-files --type=service

# 查看服务失败状态
systemctl list-units --failed

# 实时监控服务状态变化
systemctl monitor
```

### 服务日志查看

```bash
# 查看服务的全部日志
journalctl -u <service-name>

# 查看最近的日志（最后20行）
journalctl -u <service-name> -n 20

# 实时跟随服务日志
journalctl -u <service-name> -f

# 查看特定时间范围的日志
journalctl -u <service-name> --since "2025-11-24 10:00:00" --until "2025-11-24 11:00:00"

# 查看昨天的日志
journalctl -u <service-name> --since "yesterday"

# 查看过去一小时的日志
journalctl -u <service-name> --since "1 hour ago"

# 仅查看错误日志
journalctl -u <service-name> -p err

# 以JSON格式查看日志
journalctl -u <service-name> -o json

# 查看系统所有日志（不指定服务）
journalctl -n 50 -f

# 按优先级查看日志（emerg, alert, crit, err, warning, notice, info, debug）
journalctl -p warning
```

---

## 常用命令

### 进程和服务查询

```bash
# 查看进程树
ps aux | grep <process-name>

# 使用pstree查看进程树
pstree

# 查看特定进程信息
ps -ef | grep <service-name>

# 查看进程内存占用
ps aux --sort=-%mem | head -10

# 查看进程CPU占用
ps aux --sort=-%cpu | head -10

# 获取进程PID
pgrep <process-name>

# 查找并杀死进程
pkill <process-name>

# 强制杀死进程
pkill -9 <process-name>
```

### 系统资源监控

```bash
# 实时监控系统资源（CPU、内存、网络等）
top

# 改进的top版本
htop

# 查看内存使用情况
free -h

# 查看磁盘使用情况
df -h

# 查看磁盘IO状态
iostat -x 1

# 查看网络连接状态
netstat -tlnp

# 现代化的netstat替代品
ss -tlnp

# 查看网络接口信息
ip addr show

# 查看路由表
ip route show

# 监控网络流量
iftop -i eth0
```

### 文件和目录操作

```bash
# 查看目录大小
du -sh <directory>

# 查看详细的文件列表
ls -lh

# 递归查看目录树
tree -L 3

# 查找文件
find / -name <filename>

# 查看文件内容
cat <filename>
tail -f <filename>
head -20 <filename>

# 统计文件行数
wc -l <filename>

# 查看文件权限
stat <filename>
```

### 用户和权限管理

```bash
# 查看当前用户
whoami

# 查看用户信息
id

# 列出所有用户
cut -d: -f1 /etc/passwd

# 添加新用户
sudo useradd -m -s /bin/bash <username>

# 设置用户密码
sudo passwd <username>

# 删除用户
sudo userdel -r <username>

# 修改文件权限
chmod 755 <file>

# 修改文件所有者
chown user:group <file>

# 给用户添加sudo权限
sudo usermod -aG sudo <username>
```

---

## 配置示例

### 配置 1: 自定义Java应用服务

**用途**: 将Java应用注册为系统服务，支持开机自启和systemctl管理

```bash
# 创建服务文件
sudo vim /etc/systemd/system/myapp.service
```

```ini
[Unit]
Description=My Java Application Service
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp

# Java应用启动命令
ExecStart=/usr/bin/java -Xmx2g -Xms2g -jar /opt/myapp/application.jar --server.port=8080

# 如果应用崩溃，自动重启
Restart=on-failure
RestartSec=10

# 优雅关闭的超时时间
TimeoutStopSec=30

# 标准输出和错误日志
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp

# 环境变量
Environment="JAVA_OPTS=-Dfile.encoding=UTF-8"

[Install]
WantedBy=multi-user.target
```

启用和管理：
```bash
# 重新加载systemd配置
sudo systemctl daemon-reload

# 启用服务（开机自启）
sudo systemctl enable myapp.service

# 启动服务
sudo systemctl start myapp.service

# 查看状态
sudo systemctl status myapp.service

# 查看日志
journalctl -u myapp.service -f
```

### 配置 2: Node.js应用服务

**用途**: 将Node.js应用作为后台服务运行

```ini
[Unit]
Description=Node.js Application
After=network.target

[Service]
Type=simple
User=nodeuser
WorkingDirectory=/home/nodeuser/myapp

# 使用nvm管理的Node.js
ExecStart=/home/nodeuser/.nvm/versions/node/v18.17.0/bin/node /home/nodeuser/myapp/server.js

Restart=on-failure
RestartSec=5

StandardOutput=journal
StandardError=journal
SyslogIdentifier=nodeapp

Environment="NODE_ENV=production"

[Install]
WantedBy=multi-user.target
```

### 配置 3: Python应用服务（使用虚拟环境）

**用途**: 运行Python应用并使用虚拟环境

```ini
[Unit]
Description=Python Application with Gunicorn
After=network.target

[Service]
Type=notify
User=pyuser
WorkingDirectory=/home/pyuser/myapp

ExecStart=/home/pyuser/myapp/venv/bin/gunicorn \
    --workers 4 \
    --worker-class sync \
    --bind 0.0.0.0:8000 \
    --timeout 30 \
    wsgi:app

Restart=on-failure
RestartSec=10

StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

### 配置 4: Go应用服务

**用途**: 运行Go编译的可执行文件

```ini
[Unit]
Description=Go Application Service
After=network.target

[Service]
Type=simple
User=gouser
WorkingDirectory=/opt/goapp

ExecStart=/opt/goapp/main

Restart=on-failure
RestartSec=5

StandardOutput=journal
StandardError=journal
SyslogIdentifier=goapp

# Go应用的环境变量
Environment="GIN_MODE=release"

[Install]
WantedBy=multi-user.target
```

### 配置 5: Docker容器化应用服务

**用途**: 使用systemd启动和管理Docker容器

```ini
[Unit]
Description=Docker Container - My App
After=docker.service
Requires=docker.service

[Service]
Type=simple

ExecStart=/usr/bin/docker run \
    --name myapp-container \
    --restart=no \
    -p 8080:8080 \
    -v /data:/app/data \
    -e "APP_ENV=production" \
    myapp:latest

# 停止容器
ExecStop=/usr/bin/docker stop myapp-container
ExecStopPost=/usr/bin/docker rm myapp-container

Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

---

## 实战场景

### 场景 1: 启动、管理和监控自定义Java服务

**需求**: 部署一个Spring Boot应用，需要开机自启、自动重启和实时日志监控

**步骤**:

1. 创建应用用户和目录
   ```bash
   sudo useradd -m -s /bin/bash appuser
   sudo mkdir -p /opt/myapp
   sudo chown -R appuser:appuser /opt/myapp
   ```

2. 上传应用JAR文件
   ```bash
   # 将application.jar复制到/opt/myapp/
   sudo cp application.jar /opt/myapp/
   sudo chown appuser:appuser /opt/myapp/application.jar
   ```

3. 创建systemd服务文件
   ```bash
   sudo tee /etc/systemd/system/myapp.service > /dev/null << 'EOF'
   [Unit]
   Description=Spring Boot Application
   After=network.target

   [Service]
   Type=simple
   User=appuser
   WorkingDirectory=/opt/myapp
   ExecStart=/usr/bin/java -Xmx2g -Xms2g -jar application.jar
   Restart=on-failure
   RestartSec=10
   StandardOutput=journal
   StandardError=journal

   [Install]
   WantedBy=multi-user.target
   EOF
   ```

4. 启用并启动服务
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable myapp.service
   sudo systemctl start myapp.service
   ```

5. 验证服务状态
   ```bash
   sudo systemctl status myapp.service
   ```

6. 实时监控日志
   ```bash
   journalctl -u myapp.service -f
   ```

---

### 场景 2: 管理多个容器化微服务

**需求**: 同时运行多个Docker容器（如Java API服务、Node.js前端、Python后台任务），需要统一管理

**步骤**:

1. 为每个容器创建systemd服务文件

   **Java API服务**:
   ```bash
   sudo tee /etc/systemd/system/api-service.service > /dev/null << 'EOF'
   [Unit]
   Description=Java API Service
   After=docker.service
   Requires=docker.service

   [Service]
   ExecStart=/usr/bin/docker run --rm --name api-service -p 8080:8080 \
       -e "SPRING_PROFILES_ACTIVE=prod" api-service:latest
   ExecStop=/usr/bin/docker stop api-service
   Restart=on-failure
   RestartSec=5

   [Install]
   WantedBy=multi-user.target
   EOF
   ```

   **Node.js前端服务**:
   ```bash
   sudo tee /etc/systemd/system/web-service.service > /dev/null << 'EOF'
   [Unit]
   Description=Node.js Web Service
   After=docker.service
   Requires=docker.service

   [Service]
   ExecStart=/usr/bin/docker run --rm --name web-service -p 3000:3000 \
       web-service:latest
   ExecStop=/usr/bin/docker stop web-service
   Restart=on-failure

   [Install]
   WantedBy=multi-user.target
   EOF
   ```

2. 加载所有服务
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable api-service.service web-service.service
   ```

3. 一次性启动所有服务
   ```bash
   sudo systemctl start api-service.service web-service.service
   ```

4. 监控所有服务状态
   ```bash
   systemctl list-units --type=service --state=running | grep -E "api-service|web-service"
   ```

5. 查看所有服务日志
   ```bash
   # 查看特定服务日志
   journalctl -u api-service.service -f

   # 同时查看多个服务
   journalctl -u api-service.service -u web-service.service -f
   ```

---

### 场景 3: 故障恢复和日志分析

**需求**: 服务频繁崩溃，需要找出根本原因并修复

**步骤**:

1. 检查服务当前状态
   ```bash
   sudo systemctl status myapp.service
   ```

2. 查看最近的错误日志
   ```bash
   journalctl -u myapp.service -p err -n 50
   ```

3. 查看详细的启动失败信息
   ```bash
   journalctl -u myapp.service --since "1 hour ago" -o verbose
   ```

4. 手动启动服务以查看即时错误
   ```bash
   # 首先停止服务
   sudo systemctl stop myapp.service

   # 然后直接运行应用命令，查看输出
   cd /opt/myapp && /usr/bin/java -jar application.jar
   ```

5. 修复问题后重启服务
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart myapp.service
   ```

6. 验证修复
   ```bash
   journalctl -u myapp.service -f
   ```

---

### 场景 4: Kubernetes集群服务管理

**需求**: 在主机上运行kubelet和其他K8s组件，需要systemd管理

**步骤**:

1. 创建kubelet服务文件
   ```bash
   sudo tee /etc/systemd/system/kubelet.service > /dev/null << 'EOF'
   [Unit]
   Description=Kubernetes Kubelet
   After=network.target

   [Service]
   Type=notify
   ExecStart=/usr/bin/kubelet \
       --kubeconfig=/etc/kubernetes/kubelet.conf \
       --config=/etc/kubernetes/kubelet-config.yaml \
       --v=2
   Restart=always
   RestartSec=5
   StandardOutput=journal
   StandardError=journal

   [Install]
   WantedBy=multi-user.target
   EOF
   ```

2. 启用kubelet
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable kubelet.service
   sudo systemctl start kubelet.service
   ```

3. 监控kubelet状态和日志
   ```bash
   sudo systemctl status kubelet.service
   journalctl -u kubelet.service -f
   ```

---

## 监控和调试

### 系统级别监控

```bash
# 查看所有正在运行的服务
systemctl list-units --type=service --state=running

# 查看所有故障的单元
systemctl list-units --failed

# 按CPU占用排序查看进程
top -o %CPU

# 按内存占用排序查看进程
top -o %MEM

# 使用htop（更友好的界面）
htop

# 实时查看系统日志
journalctl -f

# 查看最近引导的系统日志
journalctl -b

# 统计每个进程的内存占用
ps aux | sort -k4 -rn | head -10

# 查看系统负载
uptime

# 查看僵尸进程
ps aux | grep defunct
```

### 服务级别调试

```bash
# 显示服务的详细状态信息
systemctl show <service-name>

# 显示服务的所有配置属性
systemctl show <service-name> -a

# 查看systemd对服务的依赖关系
systemctl list-dependencies <service-name>

# 查看在当前启动目标下会启动哪些服务
systemctl list-dependencies multi-user.target

# 检查服务配置文件的语法
systemd-analyze verify /etc/systemd/system/myapp.service

# 分析系统启动时间
systemd-analyze

# 显示启动过程中最慢的服务
systemd-analyze blame

# 图形化显示启动过程
systemd-analyze plot > startup.svg
```

### 网络和连接监控

```bash
# 查看所有监听的端口
ss -tlnp

# 查看特定进程的网络连接
ss -tlnp | grep <process-name>

# 监控网络连接数量的变化
watch -n 1 'ss -tlnp | wc -l'

# 查看TCP连接统计
ss -s

# 实时监控网络流量（需要iftop）
iftop -i eth0

# 使用netstat查看连接
netstat -tulpn

# 查看DNS查询
dig @8.8.8.8 google.com
```

---

## 故障排查

### 问题 1: 服务无法启动

**症状**:
- `systemctl status <service>` 显示 failed
- `journalctl -u <service>` 显示错误信息
- 应用程序文件不存在或权限不足

**原因**: 
- ExecStart路径错误或文件不存在
- 运行用户权限不足
- 依赖的服务尚未启动
- 配置文件语法错误

**解决**:
```bash
# 1. 验证服务文件语法
systemd-analyze verify /etc/systemd/system/myapp.service

# 2. 检查可执行文件是否存在且有执行权限
ls -l /usr/bin/java
which java

# 3. 检查运行用户和工作目录
id appuser
ls -l /opt/myapp

# 4. 查看详细错误日志
journalctl -u myapp.service -n 100

# 5. 尝试手动运行应用
sudo -u appuser /usr/bin/java -jar /opt/myapp/application.jar

# 6. 重新加载并重启
sudo systemctl daemon-reload
sudo systemctl restart myapp.service
```

---

### 问题 2: 服务启动后立即退出

**症状**:
- `systemctl status` 显示 active (exited)
- `Type=simple` 时应保持运行但实际退出了

**原因**:
- 应用程序正常结束
- 应用启动了后台进程后自己退出
- 缺少必要的配置参数或环境变量

**解决**:
```bash
# 1. 检查应用实际的进程树
pstree -p <process-name>

# 2. 检查应用输出的日志信息
journalctl -u myapp.service -o verbose

# 3. 如果是Java应用，检查是否有nohup进程
ps aux | grep java

# 4. 修改服务文件中的Type
# 将 Type=simple 改为 Type=forking（如果应用启动子进程）

# 5. 或者添加 RemainAfterExit=yes（适用于一次性任务）
```

---

### 问题 3: 服务占用过多系统资源

**症状**:
- CPU占用率高达100%
- 内存持续增长导致OOM
- 磁盘IO持续高占用

**原因**:
- 应用代码中存在死循环或内存泄漏
- 日志文件无限增长
- 数据库连接泄漏

**解决**:
```bash
# 1. 识别占用最多资源的进程
top -o %CPU
ps aux --sort=-%mem | head -5

# 2. 获取进程的详细信息
ps -p <PID> -o %cpu,%mem,vsz,rss

# 3. 监控资源趋势
while true; do ps -p <PID> -o %cpu,%mem,vsz,rss; sleep 1; done

# 4. 查看文件描述符数量（检查是否泄漏）
cat /proc/<PID>/limits

# 5. 查看打开的文件数
lsof -p <PID> | wc -l

# 6. 配置内存限制（在systemd服务文件中）
# [Service]
# MemoryLimit=2G
# MemoryAccounting=yes

# 7. 重启服务
sudo systemctl restart myapp.service
```

---

### 问题 4: 端口已被占用

**症状**:
- 启动时报 "Address already in use" 错误
- netstat显示端口被另一个进程占用

**原因**:
- 前一个进程未完全关闭
- 端口处于TIME_WAIT状态
- 另一个应用已在使用该端口

**解决**:
```bash
# 1. 查看占用端口的进程
ss -tlnp | grep :8080
netstat -tulpn | grep :8080

# 2. 强制杀死占用端口的进程
sudo lsof -i :8080
sudo kill -9 <PID>

# 3. 检查是否有僵尸进程
ps aux | grep <service-name> | grep defunct

# 4. 修改服务配置使用不同端口
# 在 /etc/systemd/system/myapp.service 中修改
# ExecStart=/usr/bin/java -jar application.jar --server.port=8081

# 5. 重新加载并重启
sudo systemctl daemon-reload
sudo systemctl restart myapp.service
```

---

### 问题 5: 服务日志不输出或输出不完整

**症状**:
- journalctl查看不到日志
- 只能看到systemd的日志，看不到应用日志
- 日志被截断或丢失

**原因**:
- StandardOutput/StandardError配置不正确
- 应用日志输出到文件而非stdout
- journald缓冲区满了

**解决**:
```bash
# 1. 检查服务配置中的日志设置
grep -E "StandardOutput|StandardError" /etc/systemd/system/myapp.service

# 2. 修改服务文件确保日志输出到journal
# [Service]
# StandardOutput=journal
# StandardError=journal
# SyslogIdentifier=myapp

# 3. 如果应用本身输出到文件，也要指定
# ExecStart=/usr/bin/java -jar application.jar > /var/log/myapp.log 2>&1

# 4. 查看journald的存储大小和限制
journalctl --disk-usage
journalctl --vacuum-size=1G

# 5. 查看systemd日志完整性
journalctl --verify

# 6. 重新加载并查看日志
sudo systemctl daemon-reload
sudo systemctl restart myapp.service
journalctl -u myapp.service -f
```

---

## 快速命令参考表

| 任务 | 命令 | 说明 |
|------|------|------|
| 启动服务 | `systemctl start <service>` | 立即启动服务 |
| 停止服务 | `systemctl stop <service>` | 停止运行中的服务 |
| 重启服务 | `systemctl restart <service>` | 停止后立即启动 |
| 重载配置 | `systemctl reload <service>` | 重载配置但不重启 |
| 启用自启 | `systemctl enable <service>` | 设置开机自启 |
| 禁用自启 | `systemctl disable <service>` | 关闭开机自启 |
| 查看状态 | `systemctl status <service>` | 查看当前运行状态 |
| 查看日志 | `journalctl -u <service> -f` | 实时查看服务日志 |
| 列出运行服务 | `systemctl list-units --type=service --state=running` | 列出所有运行中的服务 |
| 列出故障单元 | `systemctl list-units --failed` | 列出启动失败的单元 |
| 查看所有日志 | `journalctl -f` | 实时查看系统所有日志 |
| 查找进程 | `pgrep <name>` | 按名称查找进程PID |
| 监控资源 | `htop` | 交互式资源监控 |
| 查看开放端口 | `ss -tlnp` | 列出所有监听的端口 |
| 查看进程网络 | `ss -tlnp \| grep <pid>` | 查看进程的网络连接 |

---

## 最佳实践

1. **总是验证服务文件语法**: `systemd-analyze verify /path/to/service`
2. **使用User=指定运行用户**: 避免以root身份运行应用
3. **设置合理的Restart策略**: `Restart=on-failure` + `RestartSec=10` 
4. **配置StandardOutput和StandardError**: 便于日志查看和调试
5. **设置TimeoutStopSec**: 确保应用有足够时间优雅关闭
6. **使用After和Wants**: 明确指定服务依赖关系
7. **定期检查日志**: 及时发现和解决问题
8. **使用资源限制**: 防止单个服务消耗过多系统资源
9. **建立监控告警**: 及时发现服务故障
10. **保持文档更新**: 记录服务配置和常见问题

---

## 更新日志

- **2025-11-24**: 初版完成，包含系统管理、服务管理、监控调试、故障排查等完整内容
- **2025-11-24**: 添加Docker、K8s、Java、Node.js、Python等常见服务配置示例
- **2025-11-24**: 补充实战场景和快速参考表