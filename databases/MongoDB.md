# MongoDB 参考指南

> 范围：基础查询、聚合、索引、连接、权限
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
brew install mongodb-community

# Ubuntu
sudo apt-get install -y mongodb-org

# Docker
docker run -d --name mongodb -p 27017:27017 mongo:latest
```

### 基础连接

```bash
# 本地连接（无认证）
mongosh

# 远程连接
mongosh "mongodb://username:password@host:27017/dbname"

# MongDB Atlas（云服务）
mongosh "mongodb+srv://username:password@cluster.mongodb.net/dbname"
```

### 基础操作

| 操作 | 命令 | 说明 |
|------|------|------|
| 查看数据库 | `show dbs` | 列出所有数据库 |
| 切换数据库 | `use dbname` | 选择或创建数据库 |
| 查看集合 | `show collections` | 列出当前库的所有集合 |
| 查看集合数量 | `db.collection.countDocuments()` | 统计文档数 |
| 删除数据库 | `db.dropDatabase()` | 删除当前数据库 |
| 删除集合 | `db.collection.drop()` | 删除集合 |

---

## 核心命令

### 分类 1: 查询操作

#### 命令 1.1: find - 查找文档

**用途**: 查询集合中的文档

**基础用法**:
```javascript
// 查询所有文档
db.users.find()

// 条件查询
db.users.find({ status: "active" })

// 多条件查询（AND）
db.users.find({ status: "active", age: { $gt: 18 } })

// 多条件查询（OR）
db.users.find({ $or: [{ status: "active"}, { vip: true }] })
```

**参数说明**:
- `{ field: value }` - 相等条件
- `{ $gt: value }` - 大于
- `{ $gte: value }` - 大于等于
- `{ $lt: value }` - 小于
- `{ $lte: value }` - 小于等于
- `{ $ne: value }` - 不等于
- `{ $in: [v1, v2] }` - 包含任一值
- `{ $nin: [v1, v2] }` - 不包含任何值

**常见用例**:
```javascript
// 范围查询
db.orders.find({ amount: { $gte: 100, $lte: 500 } })

// 数组查询
db.users.find({ tags: "vip" })

// 模糊查询
db.users.find({ name: { $regex: "^A" } })

// 排序和限制
db.users.find().sort({ createdAt: -1 }).limit(10)

// 字段投影
db.users.find({}, { name: 1, email: 1, _id: 0 })
```

**相关命令**: `findOne`, `findOneAndUpdate`, `countDocuments`

---

#### 命令 1.2: findOne - 查找单个文档

**用途**: 查询并返回第一个匹配的文档

**语法**:
```javascript
db.users.findOne({ email: "user@example.com" })

// 带字段投影
db.users.findOne({ _id: ObjectId("...") }, { password: 0 })
```

**示例**:
```javascript
// 获取最新创建的用户
db.users.findOne({}, { sort: { createdAt: -1 } })

// 获取特定用户
const user = db.users.findOne({ username: "john" })
```

---

### 分类 2: 插入和更新操作

#### 命令 2.1: insertOne 和 insertMany - 插入文档

**用途**: 向集合添加新文档

**用法**:
```javascript
// 插入单个文档
db.users.insertOne({
  name: "Alice",
  email: "alice@example.com",
  age: 25,
  createdAt: new Date()
})

// 插入多个文档
db.users.insertMany([
  { name: "Bob", age: 30 },
  { name: "Charlie", age: 28 },
  { name: "Diana", age: 26 }
])
```

---

#### 命令 2.2: updateOne 和 updateMany - 更新文档

**用途**: 修改集合中的文档

**用法**:
```javascript
// 更新单个文档
db.users.updateOne(
  { _id: ObjectId("...") },
  { $set: { status: "inactive" } }
)

// 更新多个文档
db.users.updateMany(
  { status: "active" },
  { $set: { lastLogin: new Date() } }
)

// 使用操作符
db.accounts.updateOne(
  { _id: 1 },
  {
    $set: { name: "New Name" },      // 设置字段
    $inc: { balance: 100 },           // 递增
    $push: { tags: "new-tag" },       // 向数组添加
    $pull: { tags: "old-tag" }        // 从数组删除
  }
)
```

---

#### 命令 2.3: deleteOne 和 deleteMany - 删除文档

**用途**: 删除集合中的文档

**用法**:
```javascript
// 删除单个文档
db.users.deleteOne({ _id: ObjectId("...") })

// 删除多个文档
db.logs.deleteMany({ createdAt: { $lt: new Date("2024-01-01") } })

// 删除所有文档
db.temp.deleteMany({})
```

---

## 高级用法

### 高级 1: Aggregation Pipeline - 数据聚合

**场景**: 复杂的数据分析、统计、转换

**基础示例**:
```javascript
// 按状态分组统计
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $group: {
    _id: "$status",
    count: { $sum: 1 },
    totalAmount: { $sum: "$amount" }
  }},
  { $sort: { totalAmount: -1 } }
])
```

**关键 Stages**:
```javascript
// $match - 过滤文档（类似WHERE）
{ $match: { status: "active" } }

// $group - 分组聚合
{ $group: { _id: "$category", total: { $sum: "$amount" } } }

// $project - 字段投影
{ $project: { name: 1, email: 1, _id: 0 } }

// $sort - 排序
{ $sort: { createdAt: -1 } }

// $limit - 限制数量
{ $limit: 10 }

// $skip - 跳过数量
{ $skip: 20 }

// $lookup - 关联查询（JOIN）
{ $lookup: { from: "users", localField: "userId", foreignField: "_id", as: "user" } }

// $unwind - 展开数组
{ $unwind: "$tags" }
```

---

### 高级 2: 索引和性能

**创建索引**:
```javascript
// 创建单字段索引
db.users.createIndex({ email: 1 })

// 创建复合索引
db.orders.createIndex({ userId: 1, createdAt: -1 })

// 创建唯一索引
db.users.createIndex({ username: 1 }, { unique: true })

// 创建文本索引
db.products.createIndex({ name: "text", description: "text" })

// 查看索引
db.users.getIndexes()

// 删除索引
db.users.dropIndex("email_1")
```

---

## 实战例子

### 例子 1: 用户管理系统

**需求**: 创建用户、更新用户信息、查询活跃用户

**解决方案**:
```javascript
// 创建用户
db.users.insertOne({
  username: "john_doe",
  email: "john@example.com",
  passwordHash: "...",
  status: "active",
  createdAt: new Date(),
  lastLogin: new Date(),
  profile: {
    firstName: "John",
    lastName: "Doe",
    avatar: "url"
  }
})

// 更新最后登录时间
db.users.updateOne(
  { username: "john_doe" },
  { $set: { lastLogin: new Date() } }
)

// 查询活跃用户（7天内登录）
db.users.find({
  status: "active",
  lastLogin: { $gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000) }
}).limit(100)
```

**关键点**:
- 使用 `createdAt` 和 `lastLogin` 追踪时间
- 分离 `profile` 为嵌套文档便于扩展
- 使用 `$set` 只更新需要改动的字段

---

### 例子 2: 电商订单系统

**问题**: 统计每个用户的总消费和订单数

**步骤**:
1. 创建订单
   ```javascript
   db.orders.insertOne({
     userId: ObjectId("..."),
     items: [
       { productId: "...", quantity: 2, price: 99.99 },
       { productId: "...", quantity: 1, price: 49.99 }
     ],
     totalAmount: 249.97,
     status: "completed",
     createdAt: new Date()
   })
   ```

2. 查询用户消费统计
   ```javascript
   db.orders.aggregate([
     { $match: { status: "completed" } },
     { $group: {
       _id: "$userId",
       totalSpent: { $sum: "$totalAmount" },
       orderCount: { $sum: 1 },
       avgOrder: { $avg: "$totalAmount" }
     }},
     { $sort: { totalSpent: -1 } },
     { $limit: 10 }
   ])
   ```

---

## 性能优化

### 优化 1: 使用索引加速查询

**问题**: 频繁查询某个字段很慢

**方案**:
```javascript
// 创建索引
db.orders.createIndex({ userId: 1, createdAt: -1 })

// 使用 explain 分析查询
db.orders.find({ userId: "..." }).explain("executionStats")
```

**性能提升**: ~100x（根据数据量）

---

### 优化 2: 数据聚合前过滤

**说明**: 在 aggregation pipeline 最前面使用 $match 过滤数据

**代码**:
```javascript
// ❌ 慢：先 $group 再 $match
db.logs.aggregate([
  { $group: { _id: "$userId", count: { $sum: 1 } } },
  { $match: { count: { $gt: 100 } } }
])

// ✅ 快：先 $match 再 $group
db.logs.aggregate([
  { $match: { status: "error" } },
  { $group: { _id: "$userId", count: { $sum: 1 } } },
  { $match: { count: { $gt: 100 } } }
])
```

---

## 故障排查

### 问题 1: 找不到集合或字段

**症状**:
- `collection does not exist`
- 查询返回空结果

**原因**: 集合或字段名拼写错误，或数据不存在

**解决**:
```bash
# 检查数据库
show databases

# 检查集合
show collections

# 查看集合结构
db.collection.findOne()

# 验证字段名
db.collection.find({}).limit(1)
```

---

### 问题 2: 查询性能差

**症状**: 查询很慢，无法及时返回结果

**原因**: 缺少索引、查询不优化

**解决**:
```javascript
// 分析查询性能
db.collection.find({...}).explain("executionStats")

// 创建适合的索引
db.collection.createIndex({ field1: 1, field2: 1 })

// 使用投影减少返回数据
db.collection.find({}, { field1: 1, field2: 1 })
```

---

## 快速速查表

| 任务 | 命令 | 说明 |
|------|------|------|
| 查询所有 | `db.col.find()` | 返回所有文档 |
| 条件查询 | `db.col.find({...})` | 按条件查询 |
| 统计数量 | `db.col.countDocuments()` | 统计文档数 |
| 插入 | `db.col.insertOne({...})` | 插入单个 |
| 更新 | `db.col.updateOne({...})` | 更新单个 |
| 删除 | `db.col.deleteOne({...})` | 删除单个 |
| 聚合 | `db.col.aggregate([...])` | 数据聚合 |
| 创建索引 | `db.col.createIndex({...})` | 创建索引 |

---

## 更新日志

- **2025-11-23**: 初始版本，添加基础和高级命令
- **TBD**: 添加更多实战例子
- **TBD**: 补充复制和备份命令