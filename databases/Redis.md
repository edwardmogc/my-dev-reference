# Redis 参考指南

> 范围：基础命令、数据结构、缓存策略、持久化
> 最后更新：2025-11-23

---

## 目录

1. [连接与基础](#连接与基础)
2. [核心命令](#核心命令)
3. [高级用法](#高级用法)
4. [实战例子](#实战例子)
5. [性能优化](#性能优化)
6. [故障排查](#故障排查)

---

## 连接与基础

### 安装和启动

```bash
# macOS
brew install redis

# Ubuntu
sudo apt-get install redis-server

# Docker
docker run -d --name redis -p 6379:6379 redis:latest

# 启动 Redis 服务
redis-server

# 或后台启动
redis-server --daemonize yes
```

### 基础连接

```bash
# 启动 Redis CLI（本地）
redis-cli

# 连接到远程 Redis
redis-cli -h hostname -p 6379

# 连接带密码的 Redis
redis-cli -h hostname -p 6379 -a password

# 选择数据库（0-15）
SELECT 0
```

### 基础操作

| 操作 | 命令 | 说明 |
|------|------|------|
| 获取键值 | `GET key` | 获取字符串值 |
| 设置键值 | `SET key value` | 设置字符串值 |
| 检查键 | `EXISTS key` | 检查键是否存在 |
| 删除键 | `DEL key` | 删除键 |
| 查看类型 | `TYPE key` | 查看值的类型 |
| 设置过期 | `EXPIRE key seconds` | 设置过期时间 |
| 查看过期 | `TTL key` | 查看剩余时间 |
| 查看所有键 | `KEYS *` | 列出所有键（谨慎使用）|

---

## 核心命令

### 分类 1: 字符串操作

#### 命令 1.1: SET 和 GET - 基本操作

**用途**: 存储和获取字符串值

**基础用法**:
```bash
# 基础 SET/GET
SET username "john"
GET username

# 设置多个键
MSET key1 "value1" key2 "value2" key3 "value3"

# 获取多个键
MGET key1 key2 key3

# 设置过期时间（秒）
SET session:123 "data" EX 3600

# 设置过期时间（毫秒）
SET cache:key "value" PX 60000

# 只在键不存在时设置
SETNX newkey "value"
```

**参数说明**:
- `EX seconds` - 设置秒级过期时间
- `PX milliseconds` - 设置毫秒级过期时间
- `NX` - 仅当键不存在时设置
- `XX` - 仅当键存在时设置

**常见用例**:
```bash
# 用户会话存储
SET user:123:session "{...json...}" EX 86400

# 计数器
INCR counter:page_views
DECR counter:stock

# 浮点数操作
INCRBYFLOAT price 0.5
```

**相关命令**: `APPEND`, `GETRANGE`, `SETRANGE`, `STRLEN`

---

#### 命令 1.2: INCR/DECR - 计数操作

**用途**: 对数字值进行增减操作

**语法**:
```bash
# 递增 1
INCR counter

# 递减 1
DECR counter

# 递增指定数值
INCRBY counter 10

# 递减指定数值
DECRBY counter 5

# 浮点数递增
INCRBYFLOAT score 1.5
```

**示例**:
```bash
# 页面浏览计数
INCR page:home:views

# 库存减少
DECRBY product:123:stock 2

# 评分累加
INCRBYFLOAT user:score 3.5
```

---

### 分类 2: 列表操作

#### 命令 2.1: LPUSH/RPUSH 和 LPOP/RPOP

**用途**: 使用列表实现队列和栈

**用法**:
```bash
# 从左侧添加元素
LPUSH mylist "value1" "value2" "value3"

# 从右侧添加元素
RPUSH mylist "value4"

# 从左侧弹出元素
LPOP mylist

# 从右侧弹出元素
RPOP mylist

# 获取列表长度
LLEN mylist

# 获取列表范围
LRANGE mylist 0 -1
```

---

#### 命令 2.2: 队列应用

**用途**: 实现消息队列

**用法**:
```bash
# 生产者：添加任务到队列
RPUSH task:queue "{job_data}"

# 消费者：从队列取出任务
LPOP task:queue

# 阻塞弹出（等待数据）
BLPOP task:queue 0
```

---

### 分类 3: 哈希表操作

#### 命令 3.1: HSET/HGET - 哈希操作

**用途**: 存储对象的字段和值

**用法**:
```bash
# 设置单个字段
HSET user:123 name "john" email "john@example.com"

# 获取单个字段
HGET user:123 name

# 获取所有字段和值
HGETALL user:123

# 获取所有字段名
HKEYS user:123

# 获取所有值
HVALS user:123

# 检查字段是否存在
HEXISTS user:123 email

# 获取字段数量
HLEN user:123
```

---

#### 命令 3.2: 批量哈希操作

**用途**: 效率更高的操作

**用法**:
```bash
# 一次性设置多个字段
HMSET user:123 name "john" age "30" city "NYC"

# 一次性获取多个字段
HMGET user:123 name age city

# 字段值递增
HINCRBY user:123 age 1
```

---

### 分类 4: 集合和有序集合

#### 命令 4.1: SET 操作

**用途**: 存储不重复的值

**用法**:
```bash
# 添加成员
SADD tags "python" "java" "go" "python"

# 获取所有成员
SMEMBERS tags

# 检查成员是否存在
SISMEMBER tags "python"

# 获取集合大小
SCARD tags

# 移除成员
SREM tags "java"

# 集合运算
SINTER set1 set2      # 交集
SUNION set1 set2      # 并集
SDIFF set1 set2       # 差集
```

---

#### 命令 4.2: ZSET 有序集合

**用途**: 存储带分数的有序数据（排行榜）

**用法**:
```bash
# 添加成员和分数
ZADD leaderboard 100 "player1" 200 "player2" 150 "player3"

# 按分数范围查询
ZRANGE leaderboard 0 -1           # 按分数升序
ZREVRANGE leaderboard 0 -1        # 按分数降序
ZRANGE leaderboard 0 -1 WITHSCORES  # 包含分数

# 获取成员排名
ZRANK leaderboard "player1"       # 升序排名
ZREVRANK leaderboard "player1"    # 降序排名

# 增加分数
ZINCRBY leaderboard 50 "player1"

# 按分数范围查询
ZCOUNT leaderboard 100 200
```

---

## 高级用法

### 高级 1: 键过期和淘汰策略

**场景**: 实现缓存自动失效、会话管理

**操作**:
```bash
# 设置过期时间（秒）
EXPIRE key 3600

# 设置过期时间戳
EXPIREAT key 1672531200

# 查看剩余 TTL（秒）
TTL key

# 查看剩余 TTL（毫秒）
PTTL key

# 移除过期时间
PERSIST key

# 查看所有键的过期情况
KEYS *
TTL key
```

---

### 高级 2: 发布订阅

**场景**: 实时消息推送、事件通知

**操作**:
```bash
# 订阅频道
SUBSCRIBE channel1 channel2

# 发布消息
PUBLISH channel1 "message content"

# 取消订阅
UNSUBSCRIBE channel1

# 发布到多个频道（模式）
PSUBSCRIBE news:*
PUNSUBSCRIBE news:*
```

---

### 高级 3: 事务

**场景**: 原子性执行多个命令

**操作**:
```bash
# 开始事务
MULTI

# 添加命令
INCR counter1
INCR counter2
DECR counter3

# 执行事务
EXEC

# 或取消事务
DISCARD
```

---

## 实战例子

### 例子 1: 用户会话管理

**需求**: 存储用户登录会话，自动过期

**解决方案**:
```bash
# 用户登录时保存会话
SET user:session:123abc "{
  'user_id': 456,
  'username': 'john',
  'login_time': '2024-01-23T10:30:00Z'
}" EX 86400

# 验证会话是否有效
GET user:session:123abc
TTL user:session:123abc

# 会话过期后，键自动删除
```

**关键点**:
- 使用 EX 设置 24 小时过期
- 使用 JSON 存储复杂数据
- TTL 检查会话是否将过期

---

### 例子 2: 排行榜系统

**问题**: 实现游戏玩家排行榜

**步骤**:
1. 添加玩家分数
   ```bash
   ZADD leaderboard 1000 "player1" 1500 "player2" 1200 "player3"
   ```

2. 获取前 10 名
   ```bash
   ZREVRANGE leaderboard 0 9 WITHSCORES
   ```

3. 玩家获得分数
   ```bash
   ZINCRBY leaderboard 100 "player1"
   ```

4. 查询玩家排名
   ```bash
   ZREVRANK leaderboard "player1"
   ```

---

### 例子 3: 访问计数和限流

**问题**: 统计页面访问量，实现速率限制

**步骤**:
1. 页面访问计数
   ```bash
   INCR page:home:views
   GET page:home:views
   ```

2. 实现速率限制（每分钟最多 100 次）
   ```bash
   INCR rate:limit:user123:minute
   EXPIRE rate:limit:user123:minute 60
   
   # 检查是否超限
   GET rate:limit:user123:minute  # 超过 100 则拒绝
   ```

---

## 性能优化

### 优化 1: 使用管道批量操作

**问题**: 频繁的网络往返导致速度慢

**方案**:
```bash
# ❌ 慢：逐个发送命令
GET key1
GET key2
GET key3

# ✅ 快：使用管道一次发送
(echo -e "GET key1\nGET key2\nGET key3") | redis-cli

# 程序中使用管道（示例）
pipeline = redis.pipeline()
pipeline.get('key1')
pipeline.get('key2')
pipeline.get('key3')
results = pipeline.execute()
```

**性能提升**: ~10x（减少网络往返）

---

### 优化 2: 选择合适的数据结构

**说明**: 根据应用场景选择最优的数据结构

**代码**:
```bash
# ✅ 会话存储：使用 Hash（字段多）
HSET user:123 name "john" email "john@example.com" age "30"

# ✅ 队列：使用 List
RPUSH queue:tasks "{task_data}"

# ✅ 标签/分类：使用 Set
SADD user:123:tags "python" "golang"

# ✅ 排行榜：使用 ZSet
ZADD leaderboard 1000 "player1"
```

---

## 故障排查

### 问题 1: 连接被拒绝

**症状**:
- `Connection refused`
- `Unable to connect`

**原因**: Redis 服务未启动或地址/端口错误

**解决**:
```bash
# 检查 Redis 是否运行
ps aux | grep redis

# 启动 Redis
redis-server

# 检查端口是否监听
netstat -tuln | grep 6379

# 验证连接
redis-cli ping
```

---

### 问题 2: 内存占用过高

**症状**:
- Redis 占用内存不断增加
- 性能下降

**原因**: 键未过期、没有清理策略

**解决**:
```bash
# 检查内存使用
INFO memory

# 查看最大内存设置
CONFIG GET maxmemory

# 设置最大内存和淘汰策略
CONFIG SET maxmemory 1000000000
CONFIG SET maxmemory-policy allkeys-lru

# 查找大键
KEYS *
STRLEN key_name
```

---

## 快速速查表

| 任务 | 命令 | 说明 |
|------|------|------|
| 存储字符串 | `SET key value` | 存储值 |
| 获取字符串 | `GET key` | 获取值 |
| 递增计数 | `INCR counter` | 计数 +1 |
| 添加列表元素 | `RPUSH list value` | 添加到队列 |
| 弹出列表 | `LPOP list` | 取出队列 |
| 哈希字段 | `HSET hash field value` | 存储对象 |
| 集合成员 | `SADD set member` | 添加集合 |
| 有序集合 | `ZADD zset 100 member` | 添加排行榜 |
| 设置过期 | `EXPIRE key 3600` | 过期时间 |

---

## 更新日志

- **2025-11-23**: 初始版本，添加基础和高级命令
- **TBD**: 添加 Lua 脚本示例
- **TBD**: 补充集群和哨兵模式