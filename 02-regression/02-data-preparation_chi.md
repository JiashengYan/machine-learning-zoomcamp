---
video_url: "https://www.youtube.com/watch?v=Kd74oR4QWGM&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR&index=13"
code:
  - label: "notebook.ipynb"
    path: "notebook.ipynb"
prev_url: 01-car-price-intro_chi.md
next_url: 03-eda_chi.md
---
# 数据准备

现在，我们为机器学习准备汽车价格数据集：下载数据，用 pandas 加载，并统一列名和字符串值的格式。

## 获取数据

数据集位于 Kaggle 上，mlbookcamp-code 仓库的 chapter-02-car-price 文件夹中也有一份副本。打开 data.csv，获取原始文件的 URL 并下载。可以在终端中使用 wget，也可以直接通过浏览器保存文件：

```python
data = 'https://raw.githubusercontent.com/alexeygrigorev/mlbookcamp-code/master/chapter-02-car-price/data.csv'
!wget $data
```

wget 会获取文件并将其放入当前目录，因此现在本地已有 data.csv。


## 加载数据

在 pandas 中，我们使用 read_csv 函数加载 CSV 文件。它读取数据并返回一个 DataFrame：

```python
import pandas as pd

df = pd.read_csv('data.csv')
```

加载后，我通常首先查看前五行，这正是 head() 的作用：

```python
df.head()
```


我们可以看到汽车制造商、车型、年份和许多其他特征。而 MSRP，即制造商建议零售价，就是我们想要预测的值。

## 统一列名

观察列名会发现格式不一致：有时使用大写字母（Make、Model），有时包含空格（Engine Fuel Type），有时使用下划线（Driven_Wheels）。

空格尤其麻烦。通常，如果想访问变速箱类型这一列，就不能使用点号语法，因为 df.Transmission Type 会产生语法错误。必须改用方括号语法 df['Transmission Type']。为了方便使用，我们清理列名：全部转换为小写，并将空格替换为下划线。

在 pandas 中，每个 DataFrame 都有 columns 属性，其中包含列名。它是一个索引，即 pandas 中一种与 Series 很相似的特殊数据结构。与 Series 一样，它也提供用于字符串操作的 str 访问器。因此，我们可以一次对所有列名应用字符串函数：

```python
df.columns.str.lower()
```

还可以链式调用多个字符串操作。另一个要使用的方法是 replace，用于将空格替换为下划线：

```python
df.columns.str.lower().str.replace(' ', '_')
```

这样就得到了新的列名：

```
Index(['make', 'model', 'year', 'engine_fuel_type', 'engine_hp',
       'engine_cylinders', 'transmission_type', 'driven_wheels',
       'number_of_doors', 'market_category', 'vehicle_size', 'vehicle_style',
       'highway_mpg', 'city_mpg', 'popularity', 'msrp'],
      dtype='object')
```

仅执行上述表达式并不会改变原始数据，我们需要将结果赋回 columns 属性：

```python
df.columns = df.columns.str.lower().str.replace(' ', '_')
```

现在列名统一了：全部小写，没有空格，更加整洁。


## 找出字符串列

列中的值也存在相同问题：有些全部大写（BMW、MANUAL），有些则不是。我们用同样的方法统一字符串值。

首先需要确定哪些列是字符串，因为不能对数字应用字符串方法。为此使用 dtypes 属性，它会告诉我们每一列的数据类型：

```python
df.dtypes
```


我们关注的类型是 object，pandas 中的字符串通常以这种类型存储。从技术上说，它也可以包含其他对象，但在这里从 CSV 文件读取的数据中，object 列存储的是字符串。

为了仅选择 object 类型的列，我们将 dtypes 与 'object' 比较。这样会得到一个 Series，其值为数据类型，索引为列名：

```python
df.dtypes[df.dtypes == 'object']
```

这些值都是 "object"，我们并不关心它们。我们需要的是列名，因此取出该 Series 的索引，并转换成 Python 列表。这没有特别的原因，只是看起来更清楚：

```python
strings = list(df.dtypes[df.dtypes == 'object'].index)
strings
```

```
['make',
 'model',
 'engine_fuel_type',
 'transmission_type',
 'driven_wheels',
 'market_category',
 'vehicle_size',
 'vehicle_style']
```



## 统一字符串值

已经取得所有字符串列的列名，接下来遍历这些列，应用与列名相同的转换：全部转为小写，将空格替换为下划线，再将结果写回 DataFrame：

```python
for col in strings:
    df[col] = df[col].str.lower().str.replace(' ', '_')
```

循环结束后，数据更加整洁。所有字符串都是小写，其中的空格也已替换为下划线：

```python
df.head()
```


数据集已经准备好，下一节将通过[探索性数据分析](03-eda_chi.md)进一步了解它。

## 笔记

**Pandas 属性和方法：**

* `pd.read_csv(<file_path_string>)` -> 读取 CSV 文件
* `df.head()` -> 查看 DataFrame 的前几行
* `df.columns` -> 获取 DataFrame 的列名
* `df.columns.str.lower()` -> 将所有字母转为小写
* `df.columns.str.replace(' ', '_')` -> 将空格分隔符替换为下划线
* `df.dtypes` -> 获取所有特征的数据类型
* `df.index` -> 获取 DataFrame 的索引

