---
video_url: "https://www.youtube.com/watch?v=tfuQdI3YO2c&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR"
code:
  - label: Notebook
    path: notebook.ipynb
prev_url: 01-credit-risk_chi.md
next_url: 03-decision-trees_chi.md
---
# 数据清理与准备

本节将下载信贷评分数据集并为建模做好准备：重新编码类别列、修正经过编码的缺失值，并划分训练集、验证集和测试集。

## 下载数据集

首先导入项目所需的库：

```python
import pandas as pd
import numpy as np

import seaborn as sns
from matplotlib import pyplot as plt
%matplotlib inline
```

数据集是一个 CSV 文件。我们将 URL 保存到变量中，并使用 `wget` 下载：

```python
data = 'https://raw.githubusercontent.com/alexeygrigorev/mlbookcamp-code/master/chapter-06-trees/CreditScoring.csv'
```

```bash
!wget $data
```

文件很小（约 178 KB），下载很快。用 `head` 看一下文件内容：

```text
"Status","Seniority","Home","Time","Age","Marital","Records","Job","Expenses","Income","Assets","Debt","Amount","Price"
1,9,1,60,30,2,1,3,73,129,0,0,800,846
1,17,1,60,58,3,1,1,48,131,0,0,1000,1658
2,10,2,36,46,2,2,3,90,200,3000,0,2000,2985
1,0,1,60,24,1,1,1,63,182,2500,0,900,1325
...
```

每一行代表一位客户。现在用 Pandas 加载数据，并将列名转换为小写，以保持一致：

```python
df = pd.read_csv(data)
df.columns = df.columns.str.lower()
```

## 重新编码类别变量

`status`、`home`、`marital`、`records` 和 `job` 列是类别变量，但文件中用数字编码。例如，`status` 的取值是 1、2 或 0。这些编码不够直观，因此我们将它们解码为字符串。

先查看 `status` 的取值：

```python
df.status.value_counts()
```

```text
1    3200
2    1254
0       1
Name: status, dtype: int64
```

因此，有 3200 位客户偿还了贷款，1254 位客户违约，还有一条记录的值为 0，表示状态未知。我们用字典和 Pandas 的 `map` 方法将数字映射为字符串：

```python
status_values = {
    1: 'ok',
    2: 'default',
    0: 'unk'
}

df.status = df.status.map(status_values)
```

对其他类别列也执行相同的操作：

```python
home_values = {
    1: 'rent',
    2: 'owner',
    3: 'private',
    4: 'ignore',
    5: 'parents',
    6: 'other',
    0: 'unk'
}

df.home = df.home.map(home_values)

marital_values = {
    1: 'single',
    2: 'married',
    3: 'widow',
    4: 'separated',
    5: 'divorced',
    0: 'unk'
}

df.marital = df.marital.map(marital_values)

records_values = {
    1: 'no',
    2: 'yes',
    0: 'unk'
}

df.records = df.records.map(records_values)

job_values = {
    1: 'fixed',
    2: 'partime',
    3: 'freelance',
    4: 'others',
    0: 'unk'
}

df.job = df.job.map(job_values)
```

现在 DataFrame 更容易阅读了：

```python
df.head()
```

```text
    status  seniority   home  time  age  marital records        job  expenses  ...
0       ok          9   rent    60   30  married      no  freelance        73
1       ok         17   rent    60   58    widow      no      fixed        48
2  default         10  owner    36   46  married     yes  freelance        90
3       ok          0   rent    60   24   single      no      fixed        63
```

## 缺失值

并非所有问题都会表现为缺失值。先看汇总统计信息：

```python
df.describe().round()
```

```text
       seniority    time     age  expenses      income      assets  ...
count     4455.0  4455.0  4455.0    4455.0      4455.0      4455.0
mean         8.0    46.0    37.0      56.0    763317.0   1060341.0
std          8.0    15.0    11.0      20.0   8703625.0  10217569.0
min          0.0     6.0    18.0      35.0         0.0         0.0
max         48.0    72.0    68.0     180.0  99999999.0  99999999.0
```

`income`、`assets` 和 `debt` 的最大值看起来很可疑：99999999。这个数据集用一串 9 来编码缺失值，也就是说，未知值并没有记录为 NaN，而是记录为 99999999。

我们将这些值替换为 NaN：

```python
for c in ['income', 'assets', 'debt']:
    df[c] = df[c].replace(to_replace=99999999, value=np.nan)
```

替换后，运行 `df.describe().round()` 会看到更真实的统计结果：记录数减少（`income` 从 4455 条降至 4421 条，`assets` 降至 4408 条，`debt` 降至 4437 条），最大值也变得合理。

## 移除唯一一条状态未知的记录

数据中还有一位客户的状态为 `unk`。由于我们只关心两个类别——`ok` 和 `default`——因此移除这行：

```python
df = df[df.status != 'unk'].reset_index(drop=True)
```

这样还剩 4454 行。

## 划分训练集、验证集和测试集

现在划分数据集。与之前的模块一样，我们分两步进行：先划出 20% 作为测试集，再将剩余数据按 75/25 划分为训练集和验证集，最终比例为 60/20/20：

```python
from sklearn.model_selection import train_test_split

df_full_train, df_test = train_test_split(df, test_size=0.2, random_state=11)
df_train, df_val = train_test_split(df_full_train, test_size=0.25, random_state=11)
```

重置索引，使每个 DataFrame 的索引从头开始且各自唯一：

```python
df_train = df_train.reset_index(drop=True)
df_val = df_val.reset_index(drop=True)
df_test = df_test.reset_index(drop=True)
```

## 准备目标变量

模型会输出概率，因此目标变量需要是数字。我们将 `status` 编码为：`ok` 对应 0，`default` 对应 1：

```python
y_train = (df_train.status == 'default').astype('int').values
y_val = (df_val.status == 'default').astype('int').values
y_test = (df_test.status == 'default').astype('int').values
```

最后，从 DataFrame 中删除 `status`，否则模型会把目标变量当作特征，从而“作弊”：

```python
del df_train['status']
del df_val['status']
del df_test['status']
```

数据已经准备就绪。下一节将训练第一棵决策树。

## 笔记

本节清理并准备[数据集](https://github.com/gastonstat/CreditScoring/raw/master/CreditScoring.csv)，步骤如下：

- 从给定链接下载数据。
- 使用对应的取值映射重新编码类别列（`status`、`home`、`marital`、`records` 和 `job`）。
- 将 `income`、`assets` 和 `debt` 列中的最大值替换为 NaN。
- 将 DataFrame 中的 NaN 替换为 `0`（将在下一课介绍）。
- 只保留 `status` 为 `ok` 或 `default` 的行。
- 分两步划分数据，最终得到 60% 训练集、20% 验证集和 20% 测试集，随机种子设为 `11`。
- 准备目标变量 `status`，将其从类别值转换为二元值，其中 0 表示 `ok`，1 表示 `default`。
- 最后，从训练集、验证集和测试集 DataFrame 中删除目标变量。
