# Pandas 基础知识与金融数据处理

## 1. Pandas 基础概念

### 1.1 核心数据结构
```python
import pandas as pd
import numpy as np

# Series - 一维数组
s = pd.Series([1, 3, 5, np.nan, 6, 8])
print("Series:")
print(s)

# DataFrame - 二维表格
df = pd.DataFrame({
    'customer_id': [1, 2, 3, 4, 5],
    'customer_name': ['张三', '李四', '王五', '赵六', '钱七'],
    'balance': [50000, 120000, 80000, 30000, 200000]
})
print("\nDataFrame:")
print(df)
```

### 1.2 基本操作
```python
# 读取数据
df = pd.read_csv('bank_data.csv')           # CSV文件
df = pd.read_excel('bank_data.xlsx')        # Excel文件
df = pd.read_sql(query, connection)         # SQL查询结果

# 查看数据
print(df.head())          # 前5行
print(df.tail())          # 后5行
print(df.info())          # 数据类型和内存使用
print(df.describe())      # 统计摘要

# 选择数据
df['column_name']         # 选择列
df.loc[0]                # 按标签选择行
df.iloc[0:3]             # 按位置选择行
df[df['balance'] > 100000]  # 条件选择
```

## 2. 数据清洗

### 2.1 处理缺失值
```python
# 检查缺失值
print(df.isnull().sum())

# 删除缺失值
df_clean = df.dropna()                 # 删除包含缺失值的行
df_clean = df.dropna(axis=1)           # 删除包含缺失值的列

# 填充缺失值
df['balance'] = df['balance'].fillna(0)                    # 填充0
df['balance'] = df['balance'].fillna(df['balance'].mean())  # 填充平均值
df['balance'].fillna(method='ffill', inplace=True)         # 前向填充
```

### 2.2 处理重复值
```python
# 检查重复值
print(df.duplicated().sum())

# 删除重复值
df_clean = df.drop_duplicates()
df_clean = df.drop_duplicates(subset=['customer_id'])  # 按列去重
```

### 2.3 数据类型转换
```python
# 类型转换
df['transaction_date'] = pd.to_datetime(df['transaction_date'])
df['amount'] = pd.to_numeric(df['amount'], errors='coerce')
df['customer_type'] = df['customer_type'].astype('category')

# 字符串处理
df['customer_name'] = df['customer_name'].str.strip()      # 去空格
df['id_card'] = df['id_card'].str.upper()                   # 转大写
```

## 3. 金融数据处理示例

### 3.1 时间序列分析
```python
# 交易数据示例
transactions = pd.DataFrame({
    'date': pd.date_range('2024-01-01', periods=100, freq='D'),
    'amount': np.random.normal(5000, 2000, 100),
    'type': np.random.choice(['存款', '取款', '转账'], 100)
})

# 按时间聚合
daily_sum = transactions.groupby('date')['amount'].sum()
monthly_avg = transactions.resample('M', on='date')['amount'].mean()

# 计算移动平均
transactions['ma_7d'] = transactions['amount'].rolling(window=7).mean()
transactions['ma_30d'] = transactions['amount'].rolling(window=30).mean()
```

### 3.2 客户行为分析
```python
# 客户交易数据
customer_data = pd.DataFrame({
    'customer_id': [1, 1, 1, 2, 2, 3, 3, 3, 3],
    'transaction_date': pd.date_range('2024-01-01', periods=9),
    'amount': [1000, 5000, 2000, 8000, 3000, 15000, 5000, 7000, 12000]
})

# 客户行为统计
customer_stats = customer_data.groupby('customer_id').agg({
    'amount': ['sum', 'mean', 'count', 'std'],
    'transaction_date': ['min', 'max']
}).round(2)

# 扁平化列名
customer_stats.columns = ['_'.join(col).strip() for col in customer_stats.columns]
```

### 3.3 风险指标计算
```python
def calculate_risk_indicators(df):
    """计算风险指标"""

    # 交易频率（每天交易次数）
    daily_count = df.groupby('customer_id')['transaction_date'].count().reset_index()
    daily_count.columns = ['customer_id', 'daily_frequency']

    # 大额交易比例（>100万）
    large_transactions = df[df['amount'] > 1000000]
    large_ratio = large_transactions.groupby('customer_id').size() / df.groupby('customer_id').size()

    # 交易金额波动性
    amount_volatility = df.groupby('customer_id')['amount'].std().fillna(0)

    # 合并指标
    risk_indicators = pd.DataFrame({
        'daily_frequency': daily_count.set_index('customer_id')['daily_frequency'],
        'large_transaction_ratio': large_ratio,
        'amount_volatility': amount_volatility
    }).fillna(0)

    return risk_indicators

# 使用示例
risk_data = calculate_risk_indicators(transactions)
```

## 4. 数据聚合与分组

### 4.1 GroupBy 操作
```python
# 基础分组
grouped = df.groupby('customer_type')
balance_mean = grouped['balance'].mean()
transaction_count = grouped.size()

# 多列分组
multi_grouped = df.groupby(['customer_type', 'account_status'])
stats = multi_grouped.agg({
    'balance': ['mean', 'sum', 'std'],
    'customer_id': 'count'
})

# 自定义聚合函数
def cv(x):
    return x.std() / x.mean() if x.mean() != 0 else 0

custom_stats = df.groupby('customer_type')['balance'].agg(['mean', 'std', cv])
```

### 4.2 Pivot Table
```python
# 透视表
pivot_table = pd.pivot_table(
    df,
    values='amount',
    index='customer_type',
    columns='transaction_type',
    aggfunc='sum',
    fill_value=0,
    margins=True
)

# 交叉表
cross_tab = pd.crosstab(
    df['customer_type'],
    df['risk_level'],
    margins=True
)
```

## 5. 数据合并与拼接

### 5.1 合并操作
```python
# 客户表
customers = pd.DataFrame({
    'customer_id': [1, 2, 3, 4],
    'customer_name': ['张三', '李四', '王五', '赵六'],
    'phone': ['13800138001', '13800138002', '13800138003', '13800138004']
})

# 账户表
accounts = pd.DataFrame({
    'account_id': [101, 102, 103, 104, 105],
    'customer_id': [1, 1, 2, 3, 3],
    'balance': [50000, 120000, 80000, 30000, 200000]
})

# 内连接
inner_join = pd.merge(customers, accounts, on='customer_id', how='inner')

# 左连接
left_join = pd.merge(customers, accounts, on='customer_id', how='left')

# 右连接
right_join = pd.merge(customers, accounts, on='customer_id', how='right')

# 外连接
outer_join = pd.merge(customers, accounts, on='customer_id', how='outer')
```

### 5.2 数据拼接
```python
# 垂直拼接
df1 = pd.DataFrame({'A': ['A0', 'A1'], 'B': ['B0', 'B1']})
df2 = pd.DataFrame({'A': ['A2', 'A3'], 'B': ['B2', 'B3']})
vertical_concat = pd.concat([df1, df2], ignore_index=True)

# 水平拼接
df3 = pd.DataFrame({'C': ['C0', 'C1', 'C2', 'C3']})
horizontal_concat = pd.concat([df1, df3], axis=1)
```

## 6. 金融分析实战案例

### 6.1 客户分群分析
```python
def customer_segmentation(df):
    """客户分群分析"""

    # 计算RFM指标
    current_date = pd.Timestamp.now()

    rfm = df.groupby('customer_id').agg({
        'transaction_date': lambda x: (current_date - x.max()).days,  # Recency
        'transaction_id': 'count',                                    # Frequency
        'amount': 'sum'                                               # Monetary
    }).rename(columns={
        'transaction_date': 'recency',
        'transaction_id': 'frequency',
        'amount': 'monetary'
    })

    # 分箱打分
    rfm['r_score'] = pd.qcut(rfm['recency'], 5, labels=[5,4,3,2,1])
    rfm['f_score'] = pd.qcut(rfm['frequency'].rank(method='first'), 5, labels=[1,2,3,4,5])
    rfm['m_score'] = pd.qcut(rfm['monetary'], 5, labels=[1,2,3,4,5])

    # 计算综合分
    rfm['rfm_score'] = rfm['r_score'].astype(str) + rfm['f_score'].astype(str) + rfm['m_score'].astype(str)

    # 客户分群
    def segment_customer(row):
        if row['r_score'] >= 4 and row['f_score'] >= 4 and row['m_score'] >= 4:
            return '重要价值客户'
        elif row['r_score'] >= 4 and row['f_score'] >= 4:
            return '重要保持客户'
        elif row['r_score'] >= 4 and row['m_score'] >= 4:
            return '重要发展客户'
        elif row['f_score'] >= 4 and row['m_score'] >= 4:
            return '重要挽留客户'
        else:
            return '一般客户'

    rfm['segment'] = rfm.apply(segment_customer, axis=1)

    return rfm
```

### 6.2 异常交易检测
```python
def detect_anomalies(transactions, window=30, threshold=3):
    """检测异常交易"""

    # 按客户分组计算统计指标
    def detect_customer_anomalies(customer_trans):
        # 按时间排序
        customer_trans = customer_trans.sort_values('transaction_date')

        # 计算移动平均和标准差
        customer_trans['amount_mean'] = customer_trans['amount'].rolling(window=window, min_periods=1).mean()
        customer_trans['amount_std'] = customer_trans['amount'].rolling(window=window, min_periods=1).std()

        # 计算Z-score
        customer_trans['z_score'] = (customer_trans['amount'] - customer_trans['amount_mean']) / customer_trans['amount_std']
        customer_trans['z_score'] = customer_trans['z_score'].fillna(0)

        # 标记异常
        customer_trans['is_anomaly'] = abs(customer_trans['z_score']) > threshold

        return customer_trans

    # 应用到每个客户
    anomalies = transactions.groupby('customer_id').apply(detect_customer_anomalies).reset_index(drop=True)

    return anomalies[anomalies['is_anomaly'] == True]
```

## 7. Pandas 练习题目

1. **数据清洗**：处理包含缺失值和重复值的客户数据
2. **时间序列**：分析每日交易量的趋势和季节性
3. **客户分析**：计算客户的RFM指标并分群
4. **风险监控**：识别异常交易模式
5. **报表生成**：按月生成分行的业务统计报表