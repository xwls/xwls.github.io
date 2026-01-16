---
date: '2026-01-12T09:15:00+08:00'
draft: false
title: Python 数据处理实战技巧
summary: 使用 Pandas、NumPy 等工具进行高效数据处理的完整指南
tags:
- Python
- Pandas
- NumPy
- 数据分析
- 数据处理
categories:
- 技术教程
- 数据科学
---

# Python 数据处理实战技巧

> 掌握 Python 数据处理，让数据分析工作事半功倍

## 概述

本文将介绍使用 Python 进行数据处理的核心技巧，包括：

- Pandas 数据操作
- NumPy 数值计算
- 数据清洗与转换
- 性能优化技巧

> "数据是新的石油，但如果不精炼，它就毫无用处。" —— *Clive Humby*

---

## 环境准备

### 创建虚拟环境

```bash
# 使用 venv
python -m venv .venv
source .venv/bin/activate  # Linux/macOS
# .venv\Scripts\activate   # Windows

# 或使用 conda
conda create -n data-env python=3.11
conda activate data-env
```

### 安装依赖

创建 `requirements.txt`：

```txt
pandas>=2.0.0
numpy>=1.24.0
matplotlib>=3.7.0
seaborn>=0.12.0
openpyxl>=3.1.0
jupyter>=1.0.0
```

安装：

```bash
pip install -r requirements.txt
```

---

## Pandas 基础

### 数据结构

Pandas 有两种核心数据结构：

| 结构 | 维度 | 描述 | 类比 |
|:----:|:----:|:-----|:-----|
| `Series` | 1D | 带标签的一维数组 | Excel 列 |
| `DataFrame` | 2D | 带标签的二维表格 | Excel 表 |

### 创建 DataFrame

```python
import pandas as pd
import numpy as np

# 方式 1：从字典创建
data = {
    '姓名': ['张三', '李四', '王五', '赵六'],
    '年龄': [25, 30, 35, 28],
    '城市': ['北京', '上海', '广州', '深圳'],
    '薪资': [15000, 25000, 20000, 18000]
}
df = pd.DataFrame(data)

# 方式 2：从列表创建
data_list = [
    ['张三', 25, '北京'],
    ['李四', 30, '上海'],
    ['王五', 35, '广州']
]
df2 = pd.DataFrame(data_list, columns=['姓名', '年龄', '城市'])

# 方式 3：从 NumPy 数组创建
arr = np.random.randn(5, 3)
df3 = pd.DataFrame(arr, columns=['A', 'B', 'C'])
```

输出示例：

```
   姓名  年龄  城市    薪资
0  张三   25  北京  15000
1  李四   30  上海  25000
2  王五   35  广州  20000
3  赵六   28  深圳  18000
```

### 读取数据

```python
# CSV 文件
df = pd.read_csv('data.csv', encoding='utf-8')
df = pd.read_csv('data.csv', 
                 sep=',',           # 分隔符
                 header=0,          # 表头行
                 index_col='id',    # 索引列
                 usecols=['col1', 'col2'],  # 只读取指定列
                 dtype={'col1': str},       # 指定数据类型
                 parse_dates=['date'],      # 解析日期列
                 nrows=1000)        # 只读取前 1000 行

# Excel 文件
df = pd.read_excel('data.xlsx', sheet_name='Sheet1')
df = pd.read_excel('data.xlsx', sheet_name=0)  # 按索引

# JSON 文件
df = pd.read_json('data.json')
df = pd.read_json('data.json', orient='records')

# SQL 数据库
from sqlalchemy import create_engine
engine = create_engine('postgresql://user:pass@localhost/db')
df = pd.read_sql('SELECT * FROM users', engine)
df = pd.read_sql_table('users', engine)
```

### 数据探索

```python
# 基本信息
df.info()           # 数据类型、非空值数量
df.describe()       # 统计摘要
df.shape            # (行数, 列数)
df.dtypes           # 各列数据类型
df.columns          # 列名列表
df.index            # 索引

# 查看数据
df.head(10)         # 前 10 行
df.tail(5)          # 后 5 行
df.sample(5)        # 随机 5 行

# 唯一值
df['城市'].unique()           # 唯一值数组
df['城市'].nunique()          # 唯一值数量
df['城市'].value_counts()     # 各值计数
```

---

## 数据选择与过滤

### 列选择

```python
# 单列（返回 Series）
df['姓名']
df.姓名  # 属性访问（列名无空格时可用）

# 多列（返回 DataFrame）
df[['姓名', '年龄']]

# 使用 loc（标签）和 iloc（位置）
df.loc[:, '姓名']           # 所有行，姓名列
df.iloc[:, 0]               # 所有行，第一列
df.loc[:, '姓名':'城市']    # 姓名到城市的所有列
```

### 行选择

```python
# 使用 loc（标签索引）
df.loc[0]                   # 第一行
df.loc[0:2]                 # 第 0-2 行（包含 2）
df.loc[[0, 2, 4]]           # 指定行

# 使用 iloc（整数位置索引）
df.iloc[0]                  # 第一行
df.iloc[0:2]                # 第 0-1 行（不包含 2）
df.iloc[[0, 2, 4]]          # 指定行
```

### 条件过滤

```python
# 单条件
df[df['年龄'] > 25]
df[df['城市'] == '北京']
df[df['姓名'].str.contains('张')]

# 多条件（使用 & 和 |，注意括号）
df[(df['年龄'] > 25) & (df['薪资'] > 18000)]
df[(df['城市'] == '北京') | (df['城市'] == '上海')]

# 使用 isin
df[df['城市'].isin(['北京', '上海'])]

# 使用 query（更简洁）
df.query('年龄 > 25 and 薪资 > 18000')
df.query('城市 in ["北京", "上海"]')

# 使用 between
df[df['年龄'].between(25, 35)]
```

---

## 数据清洗

### 处理缺失值

```python
# 检查缺失值
df.isnull()                 # 返回布尔 DataFrame
df.isnull().sum()           # 各列缺失值数量
df.isnull().sum().sum()     # 总缺失值数量

# 删除缺失值
df.dropna()                 # 删除任何包含 NaN 的行
df.dropna(how='all')        # 删除全为 NaN 的行
df.dropna(subset=['姓名'])  # 只检查指定列
df.dropna(thresh=3)         # 保留至少有 3 个非空值的行

# 填充缺失值
df.fillna(0)                # 用 0 填充
df.fillna(method='ffill')   # 前向填充
df.fillna(method='bfill')   # 后向填充
df['年龄'].fillna(df['年龄'].mean())  # 用均值填充

# 插值
df.interpolate(method='linear')  # 线性插值
```

### 处理重复值

```python
# 检查重复
df.duplicated()             # 返回布尔 Series
df.duplicated().sum()       # 重复行数量

# 删除重复
df.drop_duplicates()                    # 删除完全重复的行
df.drop_duplicates(subset=['姓名'])     # 基于指定列去重
df.drop_duplicates(keep='first')        # 保留第一个
df.drop_duplicates(keep='last')         # 保留最后一个
df.drop_duplicates(keep=False)          # 删除所有重复
```

### 数据类型转换

```python
# 转换单列
df['年龄'] = df['年龄'].astype(int)
df['薪资'] = df['薪资'].astype(float)
df['日期'] = pd.to_datetime(df['日期'])

# 转换多列
df = df.astype({
    '年龄': 'int32',
    '薪资': 'float64',
    '城市': 'category'  # 分类类型，节省内存
})

# 字符串转数值（处理错误）
df['数量'] = pd.to_numeric(df['数量'], errors='coerce')  # 无效值变 NaN
```

### 字符串处理

```python
# 字符串方法（通过 .str 访问器）
df['姓名'].str.lower()          # 小写
df['姓名'].str.upper()          # 大写
df['姓名'].str.strip()          # 去除空白
df['姓名'].str.replace('老', '小')  # 替换
df['姓名'].str.len()            # 长度

# 分割与合并
df['姓名'].str.split(' ')       # 分割为列表
df['姓名'].str.split(' ', expand=True)  # 分割为多列

# 正则表达式
df['邮箱'].str.extract(r'(\w+)@(\w+)\.com')  # 提取
df['手机'].str.match(r'1[3-9]\d{9}')         # 匹配
```

---

## 数据转换

### 添加与修改列

```python
# 直接赋值
df['新列'] = 0
df['年薪'] = df['薪资'] * 12

# 使用 assign（链式操作）
df = df.assign(
    年薪=lambda x: x['薪资'] * 12,
    税后=lambda x: x['年薪'] * 0.8
)

# 条件赋值
df['级别'] = np.where(df['薪资'] > 20000, '高级', '普通')

# 多条件赋值
conditions = [
    df['薪资'] >= 25000,
    df['薪资'] >= 18000,
    df['薪资'] < 18000
]
choices = ['高级', '中级', '初级']
df['级别'] = np.select(conditions, choices)

# 使用 apply
df['姓名长度'] = df['姓名'].apply(len)
df['级别'] = df['薪资'].apply(lambda x: '高级' if x > 20000 else '普通')
```

### 重命名与重排

```python
# 重命名列
df.rename(columns={'姓名': 'name', '年龄': 'age'})
df.columns = ['name', 'age', 'city', 'salary']

# 重排列顺序
df = df[['姓名', '城市', '年龄', '薪资']]
df = df.reindex(columns=['姓名', '城市', '年龄', '薪资'])

# 排序
df.sort_values('薪资', ascending=False)           # 降序
df.sort_values(['城市', '薪资'], ascending=[True, False])  # 多列
df.sort_index()  # 按索引排序
```

### 分组聚合

```python
# 基本分组
grouped = df.groupby('城市')
grouped['薪资'].mean()          # 各城市平均薪资
grouped['薪资'].agg(['mean', 'max', 'min', 'count'])

# 多列分组
df.groupby(['城市', '级别'])['薪资'].mean()

# 自定义聚合
df.groupby('城市').agg({
    '薪资': ['mean', 'sum'],
    '年龄': 'max',
    '姓名': 'count'
})

# 使用 transform（保持原始行数）
df['城市平均薪资'] = df.groupby('城市')['薪资'].transform('mean')
df['薪资偏差'] = df['薪资'] - df['城市平均薪资']

# 使用 apply（更灵活）
def top_n(group, n=2):
    return group.nlargest(n, '薪资')

df.groupby('城市').apply(top_n)
```

### 数据透视表

```python
# pivot_table
pd.pivot_table(
    df,
    values='薪资',
    index='城市',
    columns='级别',
    aggfunc='mean',
    fill_value=0,
    margins=True  # 添加汇总行/列
)

# 示例输出：
# 级别      中级      初级      高级       All
# 城市
# 上海   22000.0   16000.0   28000.0   22000.0
# 北京   20000.0   15000.0   25000.0   20000.0
# ...

# crosstab（交叉表）
pd.crosstab(df['城市'], df['级别'], margins=True)
```

---

## 合并数据

### concat 拼接

```python
# 纵向拼接（行）
df_all = pd.concat([df1, df2, df3])
df_all = pd.concat([df1, df2], ignore_index=True)  # 重置索引

# 横向拼接（列）
df_wide = pd.concat([df1, df2], axis=1)
```

### merge 合并

```python
# 类似 SQL JOIN
df_merged = pd.merge(df1, df2, on='id')  # 内连接
df_merged = pd.merge(df1, df2, on='id', how='left')   # 左连接
df_merged = pd.merge(df1, df2, on='id', how='right')  # 右连接
df_merged = pd.merge(df1, df2, on='id', how='outer')  # 全外连接

# 多键合并
df_merged = pd.merge(df1, df2, on=['id', 'date'])

# 不同列名
df_merged = pd.merge(df1, df2, left_on='user_id', right_on='id')
```

连接类型对比：

```
Left:  ┌─────┐         Right: ┌─────┐
       │  A  │                │  B  │
       │  B  │                │  C  │
       │  C  │                │  D  │
       └─────┘                └─────┘

Inner (交集):  Left (左全): Right (右全): Outer (并集):
┌─────┐       ┌─────┐      ┌─────┐       ┌─────┐
│  B  │       │  A  │      │  B  │       │  A  │
│  C  │       │  B  │      │  C  │       │  B  │
└─────┘       │  C  │      │  D  │       │  C  │
              └─────┘      └─────┘       │  D  │
                                         └─────┘
```

---

## NumPy 高效计算

### 数组创建

```python
import numpy as np

# 基本创建
arr = np.array([1, 2, 3, 4, 5])
arr_2d = np.array([[1, 2, 3], [4, 5, 6]])

# 特殊数组
np.zeros((3, 4))          # 全 0
np.ones((3, 4))           # 全 1
np.full((3, 4), 7)        # 全 7
np.eye(4)                 # 单位矩阵
np.empty((3, 4))          # 未初始化（快速）

# 序列
np.arange(0, 10, 2)       # [0, 2, 4, 6, 8]
np.linspace(0, 1, 5)      # [0, 0.25, 0.5, 0.75, 1]
np.logspace(0, 3, 4)      # [1, 10, 100, 1000]

# 随机数
np.random.seed(42)        # 设置种子
np.random.rand(3, 4)      # 均匀分布 [0, 1)
np.random.randn(3, 4)     # 标准正态分布
np.random.randint(0, 10, (3, 4))  # 整数
np.random.choice([1, 2, 3], 10)   # 随机选择
```

### 数组操作

```python
# 形状操作
arr.shape                 # 形状
arr.reshape(2, 3)         # 重塑（不改变数据）
arr.ravel()               # 展平为 1D
arr.T                     # 转置

# 数学运算（逐元素）
arr + 1                   # 加法
arr * 2                   # 乘法
arr ** 2                  # 幂
np.sqrt(arr)              # 平方根
np.exp(arr)               # 指数
np.log(arr)               # 对数

# 统计函数
arr.sum()                 # 求和
arr.mean()                # 均值
arr.std()                 # 标准差
arr.min(), arr.max()      # 最值
arr.argmin(), arr.argmax()  # 最值索引

# 沿轴操作
arr_2d.sum(axis=0)        # 按列求和
arr_2d.sum(axis=1)        # 按行求和
```

### 向量化操作

```python
# 避免循环！使用向量化

# ❌ 慢：Python 循环
result = []
for x in arr:
    result.append(x ** 2 + 2 * x + 1)

# ✅ 快：NumPy 向量化
result = arr ** 2 + 2 * arr + 1

# 条件操作
np.where(arr > 0, arr, 0)  # 负数替换为 0

# 布尔索引
arr[arr > 0]              # 只保留正数
arr[(arr > 0) & (arr < 5)]  # 组合条件
```

---

## 性能优化

### 内存优化

```python
# 查看内存使用
df.info(memory_usage='deep')
df.memory_usage(deep=True)

# 优化数据类型
def optimize_dtypes(df):
    """自动优化 DataFrame 数据类型"""
    for col in df.columns:
        col_type = df[col].dtype
        
        if col_type == 'object':
            # 尝试转换为 category
            if df[col].nunique() / len(df) < 0.5:
                df[col] = df[col].astype('category')
        
        elif col_type == 'int64':
            # 使用更小的整数类型
            if df[col].min() >= 0:
                if df[col].max() < 255:
                    df[col] = df[col].astype('uint8')
                elif df[col].max() < 65535:
                    df[col] = df[col].astype('uint16')
            else:
                if df[col].max() < 32767:
                    df[col] = df[col].astype('int16')
        
        elif col_type == 'float64':
            # 使用 float32
            df[col] = df[col].astype('float32')
    
    return df

df_optimized = optimize_dtypes(df.copy())
```

### 计算优化

```python
# 1. 使用向量化操作代替 apply
# ❌ 慢
df['new'] = df['col'].apply(lambda x: x ** 2)
# ✅ 快
df['new'] = df['col'] ** 2

# 2. 使用 eval 和 query
# ❌ 慢
df['result'] = df['A'] + df['B'] * df['C']
# ✅ 快（大数据集）
df.eval('result = A + B * C', inplace=True)

# 3. 分块处理大文件
chunks = pd.read_csv('large_file.csv', chunksize=100000)
result = pd.concat([process(chunk) for chunk in chunks])

# 4. 使用 Numba JIT 编译
from numba import jit

@jit(nopython=True)
def fast_function(arr):
    result = 0
    for i in range(len(arr)):
        result += arr[i] ** 2
    return result
```

### 并行处理

```python
# 使用 multiprocessing
from multiprocessing import Pool

def process_chunk(chunk):
    # 处理逻辑
    return chunk.apply(some_function)

with Pool(4) as pool:
    chunks = np.array_split(df, 4)
    results = pool.map(process_chunk, chunks)
df_result = pd.concat(results)

# 使用 pandarallel
from pandarallel import pandarallel
pandarallel.initialize(progress_bar=True)

df['new'] = df['col'].parallel_apply(complex_function)
```

---

## 实战案例

### 销售数据分析

```python
import pandas as pd
import matplotlib.pyplot as plt

# 读取数据
df = pd.read_csv('sales_data.csv', parse_dates=['date'])

# 数据清洗
df = df.dropna(subset=['amount'])
df['amount'] = df['amount'].abs()

# 特征工程
df['year'] = df['date'].dt.year
df['month'] = df['date'].dt.month
df['weekday'] = df['date'].dt.day_name()

# 分析
monthly_sales = df.groupby(['year', 'month'])['amount'].agg([
    ('总销售额', 'sum'),
    ('订单数', 'count'),
    ('平均客单价', 'mean')
]).round(2)

# 可视化
fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# 月度销售趋势
df.groupby(df['date'].dt.to_period('M'))['amount'].sum().plot(
    ax=axes[0, 0], 
    kind='line',
    title='月度销售趋势'
)

# 产品销售占比
df.groupby('product')['amount'].sum().plot(
    ax=axes[0, 1],
    kind='pie',
    autopct='%1.1f%%',
    title='产品销售占比'
)

# 星期分布
df.groupby('weekday')['amount'].mean().reindex([
    'Monday', 'Tuesday', 'Wednesday', 
    'Thursday', 'Friday', 'Saturday', 'Sunday'
]).plot(ax=axes[1, 0], kind='bar', title='每周销售分布')

# 销售额分布
df['amount'].hist(ax=axes[1, 1], bins=50, edgecolor='black')
axes[1, 1].set_title('销售额分布')

plt.tight_layout()
plt.savefig('sales_analysis.png', dpi=150)
plt.show()
```

---

## 常见问题

<details>
<summary><strong>Q: SettingWithCopyWarning 是什么？</strong></summary>

这个警告表示你可能在修改一个切片的副本，而不是原始 DataFrame。

```python
# ❌ 可能触发警告
df[df['A'] > 0]['B'] = 1

# ✅ 正确做法
df.loc[df['A'] > 0, 'B'] = 1
```
</details>

<details>
<summary><strong>Q: 如何处理大型 CSV 文件？</strong></summary>

```python
# 分块读取
for chunk in pd.read_csv('large.csv', chunksize=100000):
    process(chunk)

# 只读取需要的列
df = pd.read_csv('large.csv', usecols=['col1', 'col2'])

# 指定数据类型减少内存
df = pd.read_csv('large.csv', dtype={'id': 'int32', 'value': 'float32'})
```
</details>

<details>
<summary><strong>Q: apply 太慢怎么办？</strong></summary>

1. 优先使用向量化操作
2. 使用 `map` 代替简单的 `apply`
3. 使用 Numba JIT 编译
4. 使用 `pandarallel` 并行处理
</details>

---

## 总结

本文介绍了 Python 数据处理的核心知识：

| 主题 | 关键技术 |
|------|----------|
| 数据读取 | `read_csv`, `read_excel`, `read_sql` |
| 数据选择 | `loc`, `iloc`, 条件过滤, `query` |
| 数据清洗 | `dropna`, `fillna`, `drop_duplicates` |
| 数据转换 | `apply`, `assign`, `groupby`, `merge` |
| 性能优化 | 数据类型优化, 向量化, 并行处理 |

> 💡 **记住**：*向量化优先，避免循环！*

---

## 参考资料

1. [Pandas 官方文档](https://pandas.pydata.org/docs/)
2. [NumPy 官方文档](https://numpy.org/doc/)
3. [Python for Data Analysis](https://wesmckinney.com/book/) - Wes McKinney
4. [Pandas Cheat Sheet](https://pandas.pydata.org/Pandas_Cheat_Sheet.pdf)

[^1]: Pandas 2.0 引入了 PyArrow 后端，可以进一步提升性能。

---

*本文最后更新于 2026 年 1 月*