# SQL 基础知识与银行应用

## 1. SQL 基础语法

### 1.1 基本查询
```sql
-- 基本查询语句
SELECT 列名 FROM 表名;

-- 查询所有列
SELECT * FROM customers;

-- 条件查询
SELECT * FROM accounts WHERE balance > 10000;

-- 多条件查询
SELECT * FROM transactions
WHERE amount > 1000
AND transaction_date >= '2024-01-01';
```

### 1.2 聚合函数
```sql
-- 统计客户数量
SELECT COUNT(*) FROM customers;

-- 计算平均账户余额
SELECT AVG(balance) FROM accounts;

-- 计算总交易额
SELECT SUM(amount) FROM transactions
WHERE transaction_type = 'withdrawal';

-- 分组统计
SELECT customer_id, COUNT(*), SUM(amount)
FROM transactions
GROUP BY customer_id;
```

### 1.3 JOIN 操作
```sql
-- 连接客户表和账户表
SELECT c.customer_name, a.account_number, a.balance
FROM customers c
JOIN accounts a ON c.customer_id = a.customer_id;

-- 左连接（保留左表所有记录）
SELECT c.customer_name, COUNT(a.account_id) as account_count
FROM customers c
LEFT JOIN accounts a ON c.customer_id = a.customer_id
GROUP BY c.customer_id;
```

## 2. 银行场景示例数据结构

### 2.1 客户表 (customers)
```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    id_card VARCHAR(18),
    phone VARCHAR(20),
    address VARCHAR(200),
    customer_type VARCHAR(20),  -- 个人/企业
    created_date DATE
);
```

### 2.2 账户表 (accounts)
```sql
CREATE TABLE accounts (
    account_id INT PRIMARY KEY,
    account_number VARCHAR(30) UNIQUE,
    customer_id INT,
    account_type VARCHAR(20),  -- 储蓄/支票/信用
    balance DECIMAL(18,2),
    currency VARCHAR(10),
    status VARCHAR(20),         -- 活跃/冻结/销户
    created_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

### 2.3 交易表 (transactions)
```sql
CREATE TABLE transactions (
    transaction_id BIGINT PRIMARY KEY,
    from_account_id INT,
    to_account_id INT,
    amount DECIMAL(18,2),
    transaction_type VARCHAR(20),  -- 转账/存款/取款
    transaction_date TIMESTAMP,
    description VARCHAR(200),
    FOREIGN KEY (from_account_id) REFERENCES accounts(account_id),
    FOREIGN KEY (to_account_id) REFERENCES accounts(account_id)
);
```

## 3. 实际银行业务查询

### 3.1 风险监控查询
```sql
-- 查找大额交易
SELECT t.transaction_id,
       c.customer_name,
       t.amount,
       t.transaction_date
FROM transactions t
JOIN accounts a ON t.from_account_id = a.account_id
JOIN customers c ON a.customer_id = c.customer_id
WHERE t.amount > 500000
AND t.transaction_date >= CURRENT_DATE - INTERVAL '7 days'
ORDER BY t.amount DESC;
```

### 3.2 关联客户分析
```sql
-- 查找同一电话号码的多个客户
SELECT phone, COUNT(*) as customer_count,
       GROUP_CONCAT(customer_name) as customer_names
FROM customers
WHERE phone IS NOT NULL
GROUP BY phone
HAVING COUNT(*) > 1;

-- 查找同一地址的多个客户
SELECT address, COUNT(*) as customer_count
FROM customers
WHERE address IS NOT NULL
GROUP BY address
HAVING COUNT(*) > 1;
```

### 3.3 资金流向分析
```sql
-- 统计每个客户的资金净流入
SELECT c.customer_name,
       SUM(CASE WHEN t.to_account_id = a.account_id
                THEN t.amount ELSE 0 END) as total_inflow,
       SUM(CASE WHEN t.from_account_id = a.account_id
                THEN t.amount ELSE 0 END) as total_outflow,
       SUM(CASE WHEN t.to_account_id = a.account_id
                THEN t.amount ELSE -t.amount END) as net_flow
FROM customers c
JOIN accounts a ON c.customer_id = a.customer_id
LEFT JOIN transactions t ON a.account_id IN (t.from_account_id, t.to_account_id)
WHERE t.transaction_date >= '2024-01-01'
GROUP BY c.customer_id, c.customer_name
ORDER BY ABS(net_flow) DESC;
```

## 4. 高级查询技巧

### 4.1 窗口函数
```sql
-- 计算客户交易排名
SELECT customer_name,
       transaction_amount,
       ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY amount DESC) as rank
FROM transaction_analysis;

-- 计算累计余额
SELECT account_number,
       transaction_date,
       amount,
       SUM(amount) OVER (PARTITION BY account_id ORDER BY transaction_date) as cumulative_balance
FROM transaction_details;
```

### 4.2 公用表表达式 (CTE)
```sql
WITH high_value_customers AS (
    SELECT customer_id
    FROM accounts
    GROUP BY customer_id
    HAVING SUM(balance) > 1000000
)
SELECT c.customer_name, SUM(a.balance) as total_balance
FROM customers c
JOIN accounts a ON c.customer_id = a.customer_id
JOIN high_value_customers h ON c.customer_id = h.customer_id
GROUP BY c.customer_id, c.customer_name;
```

## 5. SQL 练习题目

1. **基础查询**：查询所有账户余额大于50万的客户信息
2. **统计分析**：统计每个客户的账户数量和总余额
3. **时间序列**：查询最近30天内交易额最大的10笔交易
4. **关联分析**：查找相互转账频率最高的客户对
5. **风险识别**：找出24小时内累计转出超过100万的客户