# MySQL 参考指南

> 范围：基础查询、JOIN、聚合、索引、性能
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
brew install mysql

# Ubuntu
sudo apt-get install mysql-server

# Docker
docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=password -p 3306:3306 mysql:latest
```

### 基础连接

```bash
# 本地连接
mysql -u root -p

# 远程连接
mysql -h hostname -u username -p databasename

# 无密码连接
mysql -u root -h localhost
```

### 基础操作

| 操作 | 命令 | 说明 |
|------|------|------|
| 查看数据库 | `SHOW DATABASES;` | 列出所有数据库 |
| 创建数据库 | `CREATE DATABASE dbname;` | 创建数据库 |
| 选择数据库 | `USE dbname;` | 切换数据库 |
| 查看表 | `SHOW TABLES;` | 列出所有表 |
| 查看表结构 | `DESC tablename;` | 显示表结构 |
| 删除数据库 | `DROP DATABASE dbname;` | 删除数据库 |
| 删除表 | `DROP TABLE tablename;` | 删除表 |

---

## 核心命令

### 分类 1: 查询操作

#### 命令 1.1: SELECT - 基础查询

**用途**: 从表中查询数据

**基础用法**:
```sql
-- 查询所有列
SELECT * FROM users;

-- 查询特定列
SELECT id, name, email FROM users;

-- WHERE 条件查询
SELECT * FROM users WHERE age > 18;

-- 多条件查询
SELECT * FROM users WHERE status = 'active' AND age >= 18;

-- 或条件
SELECT * FROM users WHERE status = 'active' OR vip = true;
```

**参数说明**:
- `WHERE condition` - 过滤条件
- `>`, `>=`, `<`, `<=`, `=`, `!=` - 比较运算符
- `AND`, `OR` - 逻辑运算符
- `IN (value1, value2)` - 包含值
- `BETWEEN x AND y` - 范围
- `LIKE 'pattern'` - 模式匹配
- `IS NULL` / `IS NOT NULL` - 空值判断

**常见用例**:
```sql
-- 范围查询
SELECT * FROM orders WHERE amount BETWEEN 100 AND 500;

-- 模糊查询
SELECT * FROM users WHERE name LIKE 'A%';

-- 日期查询
SELECT * FROM orders WHERE created_at >= DATE_SUB(NOW(), INTERVAL 7 DAY);

-- 排序和限制
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;

-- 去重
SELECT DISTINCT city FROM users;
```

**相关命令**: `SELECT COUNT`, `SELECT SUM`, `SELECT AVG`

---

#### 命令 1.2: JOIN - 表关联

**用途**: 从多个表查询数据

**语法**:
```sql
-- INNER JOIN（内联接，只返回匹配的行）
SELECT u.id, u.name, o.amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN（左联接，保留左表所有行）
SELECT u.id, u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id;

-- RIGHT JOIN（右联接，保留右表所有行）
SELECT u.id, u.name, o.amount
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;
```

**示例**:
```sql
-- 查询用户及其订单
SELECT u.name, o.id, o.amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id
WHERE o.status = 'completed';

-- 多表关联
SELECT u.name, p.product_name, oi.quantity
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
LEFT JOIN order_items oi ON o.id = oi.order_id
LEFT JOIN products p ON oi.product_id = p.id;
```

---

#### 命令 1.3: GROUP BY - 分组和聚合

**用途**: 按字段分组统计数据

**用法**:
```sql
-- 按状态分组统计
SELECT status, COUNT(*) as count
FROM orders
GROUP BY status;

-- 按用户分组计算总消费
SELECT user_id, SUM(amount) as total_spent, COUNT(*) as order_count
FROM orders
WHERE status = 'completed'
GROUP BY user_id
ORDER BY total_spent DESC;

-- 按日期分组统计
SELECT DATE(created_at) as date, COUNT(*) as daily_count
FROM orders
GROUP BY DATE(created_at)
ORDER BY date DESC;

-- HAVING 过滤分组结果
SELECT user_id, SUM(amount) as total
FROM orders
GROUP BY user_id
HAVING SUM(amount) > 1000;
```

---

### 分类 2: 插入、更新、删除

#### 命令 2.1: INSERT - 插入数据

**用途**: 向表中添加新行

**用法**:
```sql
-- 插入单行
INSERT INTO users (name, email, age) 
VALUES ('Alice', 'alice@example.com', 25);

-- 插入多行
INSERT INTO users (name, email, age) VALUES 
('Bob', 'bob@example.com', 30),
('Charlie', 'charlie@example.com', 28),
('Diana', 'diana@example.com', 26);

-- 从另一个表复制数据
INSERT INTO users_backup 
SELECT * FROM users WHERE status = 'inactive';
```

---

#### 命令 2.2: UPDATE - 更新数据

**用途**: 修改表中的数据

**用法**:
```sql
-- 更新特定行
UPDATE users 
SET status = 'inactive' 
WHERE id = 1;

-- 多字段更新
UPDATE users 
SET status = 'active', last_login = NOW() 
WHERE id IN (1, 2, 3);

-- 条件更新（按月日期）
UPDATE orders 
SET status = 'expired' 
WHERE status = 'pending' AND created_at < DATE_SUB(NOW(), INTERVAL 30 DAY);
```

---

#### 命令 2.3: DELETE - 删除数据

**用途**: 删除表中的数据

**用法**:
```sql
-- 删除特定行
DELETE FROM users WHERE id = 1;

-- 删除多行
DELETE FROM logs WHERE created_at < DATE_SUB(NOW(), INTERVAL 90 DAY);

-- 删除所有数据（谨慎使用）
DELETE FROM temp_table;

-- 截断表（清空数据并重置ID）
TRUNCATE TABLE temp_table;
```

---

## 高级用法

### 高级 1: 子查询和 CTE

**场景**: 复杂的嵌套查询

**子查询示例**:
```sql
-- 查询消费超过平均值的用户
SELECT * FROM users 
WHERE id IN (
  SELECT user_id FROM orders 
  GROUP BY user_id 
  HAVING SUM(amount) > (
    SELECT AVG(total) FROM (
      SELECT SUM(amount) as total FROM orders GROUP BY user_id
    ) t
  )
);

-- EXISTS 子查询
SELECT u.id, u.name FROM users u 
WHERE EXISTS (
  SELECT 1 FROM orders o WHERE o.user_id = u.id AND o.status = 'completed'
);
```

**CTE（公用表表达式）**:
```sql
-- WITH 语句
WITH user_totals AS (
  SELECT user_id, SUM(amount) as total_spent
  FROM orders
  WHERE status = 'completed'
  GROUP BY user_id
)
SELECT u.name, ut.total_spent
FROM users u
JOIN user_totals ut ON u.id = ut.user_id
WHERE ut.total_spent > 1000;
```

---

### 高级 2: 窗口函数（MySQL 8.0+）

**场景**: 排名、分析、累计计算

**示例**:
```sql
-- ROW_NUMBER 排名
SELECT 
  user_id, 
  amount, 
  created_at,
  ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) as rn
FROM orders
WHERE rn <= 5;

-- RANK 并列排名
SELECT 
  user_id, 
  SUM(amount) as total,
  RANK() OVER (ORDER BY SUM(amount) DESC) as rank
FROM orders
GROUP BY user_id
LIMIT 10;

-- LAG/LEAD 前后数据
SELECT 
  id, 
  amount, 
  LAG(amount) OVER (ORDER BY created_at) as prev_amount,
  LEAD(amount) OVER (ORDER BY created_at) as next_amount
FROM orders
WHERE user_id = 1;
```

---

## 实战例子

### 例子 1: 用户管理

**需求**: 创建用户表、查询活跃用户、统计信息

**解决方案**:
```sql
-- 创建表
CREATE TABLE users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  username VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  age INT,
  status ENUM('active', 'inactive', 'banned') DEFAULT 'active',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  last_login TIMESTAMP NULL
);

-- 添加索引
CREATE INDEX idx_email ON users(email);
CREATE INDEX idx_status ON users(status);

-- 插入数据
INSERT INTO users (username, email, age) VALUES ('john', 'john@example.com', 30);

-- 查询活跃用户
SELECT * FROM users WHERE status = 'active' ORDER BY created_at DESC;

-- 统计用户
SELECT status, COUNT(*) as count FROM users GROUP BY status;
```

**关键点**:
- 使用 UNIQUE 保证字段唯一性
- 使用 ENUM 限制状态值
- 为常用查询字段创建索引

---

### 例子 2: 订单统计系统

**问题**: 统计每天的销售额、用户消费排名

**步骤**:
1. 创建订单表
   ```sql
   CREATE TABLE orders (
     id INT PRIMARY KEY AUTO_INCREMENT,
     user_id INT NOT NULL,
     amount DECIMAL(10, 2),
     status ENUM('pending', 'completed', 'cancelled'),
     created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
     FOREIGN KEY (user_id) REFERENCES users(id)
   );
   ```

2. 日销售额统计
   ```sql
   SELECT 
     DATE(created_at) as date,
     SUM(amount) as daily_total,
     COUNT(*) as order_count,
     AVG(amount) as avg_order
   FROM orders
   WHERE status = 'completed'
   GROUP BY DATE(created_at)
   ORDER BY date DESC;
   ```

3. 用户消费排名
   ```sql
   SELECT 
     u.id,
     u.name,
     SUM(o.amount) as total_spent,
     COUNT(o.id) as order_count,
     RANK() OVER (ORDER BY SUM(o.amount) DESC) as rank
   FROM users u
   LEFT JOIN orders o ON u.id = o.user_id AND o.status = 'completed'
   GROUP BY u.id
   LIMIT 10;
   ```

---

## 性能优化

### 优化 1: 创建合适的索引

**问题**: 查询速度很慢

**方案**:
```sql
-- 创建单列索引
CREATE INDEX idx_user_id ON orders(user_id);

-- 创建复合索引（注意字段顺序）
CREATE INDEX idx_user_status ON orders(user_id, status);

-- 查看表的索引
SHOW INDEX FROM orders;

-- 分析查询性能
EXPLAIN SELECT * FROM orders WHERE user_id = 1 AND status = 'completed';
```

**性能提升**: ~10-100x（根据数据量）

---

### 优化 2: 查询优化

**说明**: 写出高效的 SQL 语句

**代码**:
```sql
-- ❌ 慢：全表扫描
SELECT * FROM orders WHERE YEAR(created_at) = 2024;

-- ✅ 快：使用范围条件
SELECT * FROM orders 
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';

-- ❌ 慢：在字段上使用函数
SELECT * FROM orders WHERE YEAR(created_at) = 2024;

-- ✅ 快：直接比较
SELECT * FROM orders 
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
```

---

## 故障排查

### 问题 1: "Access denied for user"

**症状**: 无法连接数据库

**原因**: 用户名或密码错误

**解决**:
```bash
# 重新连接，检查凭证
mysql -u username -p databasename

# 重置 root 密码（如忘记）
sudo mysql -u root
> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
> FLUSH PRIVILEGES;
```

---

### 问题 2: "Table doesn't exist"

**症状**: 执行查询时报表不存在

**原因**: 表名拼写错误或未切换到正确的数据库

**解决**:
```sql
-- 检查当前数据库
SELECT DATABASE();

-- 列出所有表
SHOW TABLES;

-- 检查表是否存在
DESC tablename;

-- 验证表名大小写
```

---

## 快速速查表

| 任务 | 命令 | 说明 |
|------|------|------|
| 查询所有 | `SELECT * FROM table;` | 返回所有行 |
| 条件查询 | `SELECT * FROM table WHERE ...;` | 按条件查询 |
| 统计数量 | `SELECT COUNT(*) FROM table;` | 统计行数 |
| 插入 | `INSERT INTO table VALUES (...);` | 插入新行 |
| 更新 | `UPDATE table SET ... WHERE ...;` | 更新数据 |
| 删除 | `DELETE FROM table WHERE ...;` | 删除数据 |
| 关联 | `SELECT ... FROM t1 JOIN t2 ON ...;` | 多表关联 |
| 分组 | `SELECT ... GROUP BY ...;` | 分组统计 |

---

## 更新日志

- **2025-11-23**: 初始版本，添加基础和高级命令
- **TBD**: 添加事务和锁的说明
- **TBD**: 补充复制和备份命令