---
video_url: "https://www.youtube.com/watch?v=k6k8sQ0GhPM&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR&index=14"
code:
  - label: "notebook.ipynb"
    path: "notebook.ipynb"
prev_url: 02-data-preparation_chi.md
next_url: 04-validation-framework_chi.md
---
# 探索性数据分析

上一节我们整理了 DataFrame 中各列的格式。现在进行探索性数据分析（EDA）：检查每一列包含哪些值，在训练模型之前先了解数据和问题。

## 查看每一列

我们已经知道有些列是字符串，有些列是数值。为了了解每列的内容，我们遍历所有列，并输出一些统计信息：列名、几个示例值以及不同值的数量：

```python
for col in df.columns:
    print(col)
    print(df[col].unique()[:5])
    print(df[col].nunique())
    print()
```

unique() 返回一列中的不同值，这里只取前五个。nunique() 返回不同值的数量。输出的开头如下：

```
make
['bmw' 'audi' 'fiat' 'mercedes-benz' 'chrysler']
48

model
['1_series_m' '1_series' '100' '124_spider' '190-class']
914

year
[2011 2012 2013 1992 1993]
28

engine_fuel_type
['premium_unleaded_(required)' 'regular_unleaded'
 'premium_unleaded_(recommended)' 'flex-fuel_(unleaded/e85)' 'diesel']
10

engine_hp
[335. 300. 230. 320. 172.]
356
```


逐项查看输出：make 表示汽车制造商，如 BMW、Audi、Fiat、Mercedes-Benz 和 Chrysler，共有 48 家。model 表示更细的车型，每个制造商有多种车型，所以共有 914 个不同值。year 是数值列，包含 28 个不同年份。engine_fuel_type 表示发动机使用的燃料类型，包括柴油等，共十种。engine_hp 表示发动机马力，即动力强弱。此外还有气缸数、变速箱类型、驱动方式、市场类别、车辆尺寸和车身类型等我们可能掌握的汽车特征。

highway_mpg 表示汽车在高速公路上每加仑燃料能行驶多少英里，city_mpg 则表示城市路况下的相同指标。popularity 是数据集作者计算的受欢迎程度：他们统计了汽车在 Twitter 上被提及的次数，认为越受欢迎的汽车被提及得越多。最后是 MSRP，即汽车价格。这是我们的预测目标，其中自然有许多不同值。

## 价格分布

价格列值得进一步观察。仅查看数字难以把握整体情况，因此需要可视化。我们使用两个绘图库：matplotlib 是较底层的绘图库，seaborn 建立在它之上，使用更方便。%matplotlib inline 这一行确保图表显示在 notebook 中：

```python
import matplotlib.pyplot as plt
import seaborn as sns

%matplotlib inline
```


我们想了解价格的分布，也就是各个价格区间有多少辆汽车。直方图正好能展示这些信息：它将数值范围划分成若干区间，每个区间对应一根柱子，柱子的高度表示落入该区间的数值数量。bins 参数控制柱子的数量：

```python
sns.histplot(df.msrp, bins=50)
```

![价格直方图呈现长尾](images/03-eda-03-long-tail-distribution-crisp.jpg)

横轴上的 1e6 是科学计数法，表示 10 的 6 次方，也就是一百万。可以看到，大量汽车价格较低，大多数集中在左侧。只有少量汽车非常昂贵：可能有一辆价值 200 万，一辆价值 150 万，还有几辆价值约 100 万。它们数量不多，但价格很高。

这种分布称为长尾分布：大部分数据集中在一个区域，一条低矮的尾部向右延伸很远。价格经常呈现长尾分布，因为大多数商品价格较低，符合普通消费者的购买力，但也有少量非常昂贵的商品，只有少数人买得起。

为了更清楚地观察形状，我们只查看价格低于 100,000 的部分：

```python
sns.histplot(df.msrp[df.msrp < 100000], bins=50)
```

![放大查看低于 100,000 的价格](images/03-eda-04-zoom-below-100k-crisp.jpg)

这是上一幅直方图的左侧部分，易读得多。价格为 1,000 的汽车出现了一个奇怪的峰值，这可能是平台允许填写的最低价格，所以很多汽车恰好标价 1,000。之后汽车数量逐渐增加，在约 25,000 的位置达到大约 700 辆，然后随价格升高而下降。除了 1,000 处的峰值，这是一种比较合理的价格分布。

## 用对数变换消除长尾

虽然这种分布并不意外，但它不太适合机器学习：长尾可能干扰模型，影响效果。我们希望减弱它的影响，常见做法是对价格取对数。

对数会压缩较大的数值。对于很大的数字，其对数并没有那么大：从 10 增加到 1,000 是很大的变化，但两者对数之间的差距小得多。因此，对数能把非常大的数值压缩到较小的范围。

直接取对数有一个问题：零的对数不存在。如果数据中出现零，NumPy 会返回负无穷并输出警告：

```python
np.log([0 + 1, 1 + 1, 10 + 1, 1000 + 1, 100000 + 1])
```

```
RuntimeWarning: divide by zero encountered in log
array([-inf, 0., 2.30258509, 6.90775528, 11.51292546])
```

在本例中，价格都不低于 1,000，因此不会遇到这个问题。不过，为了避免问题，通常会先给所有值加一，再取对数。NumPy 提供了专门的快捷函数 log1p，其中 "1p" 表示 "plus one"（加一）。它先给每个值加一，再取对数：

```python
np.log1p([0, 1, 10, 1000, 100000])
```

```
array([ 0.        ,  0.69314718,  2.39789527,  6.90875478, 11.51293546])
```

将它应用于价格，并把结果命名为 price_logs：

```python
price_logs = np.log1p(df.msrp)
```

现在可以绘制取对数后价格的直方图：

```python
sns.histplot(price_logs, bins=50)
```

![对数变换后长尾消失](images/03-eda-06-log1p-normal-distribution-crisp.jpg)

长尾消失了。所有高价汽车被压缩到右侧的一小片区域，普通消费者购买的汽车仍集中在中间附近。现在的形状类似钟形曲线：有明显的分布中心，向左右两侧逐渐下降。这称为正态分布。（价格 1,000 对应的奇怪峰值依然存在，但除此之外，整体接近正态分布。）

这种情况更适合模型：当目标变量接近正态分布时，模型通常比面对长尾分布时表现更好。因此我们通常希望消除长尾，而像刚才那样对价格取对数，就是一种方法。

## 缺失值

结束前还要检查缺失值。数据集中的某些值没有被记录。pandas 在某个位置没有值时，会显示 NaN，即 not a number（非数值）。我们查看数据时已经见过：有些汽车的车门数量列是 NaN。

为了统计缺失值数量，我们使用 isnull()，它会判断每个单元格是否缺失。仅得到这些判断还不够实用，因此接着调用 sum()，按列求和，得到每列的缺失值数量：

```python
df.isnull().sum()
```

```
make                    0
model                   0
year                    0
engine_fuel_type        3
engine_hp              69
engine_cylinders       30
transmission_type       0
driven_wheels           0
number_of_doors         6
market_category      3742
vehicle_size            0
vehicle_style           0
highway_mpg             0
city_mpg                0
popularity              0
msrp                    0
dtype: int64
```


有不少汽车缺少燃料类型、市场类别、马力或气缸数的信息。需要记住：训练模型之前，必须处理这些缺失值。

回顾一下：我们查看了每列的不同值，随后观察价格分布，发现长尾并用对数变换减弱其影响，最后检查了缺失值。下一节将建立验证框架，因为训练之前必须确保能够验证模型，详见[建立验证框架](04-validation-framework_chi.md)。

## 笔记

**Pandas 属性和方法：**

* `df[col].unique()` -> 返回 Series 中的不同值
* `df[col].nunique()` -> 返回 Series 中不同值的数量
* `df.isnull().sum()` -> 返回 DataFrame 中各列的缺失值数量

**Matplotlib 和 seaborn 方法：**

* `%matplotlib inline` -> 确保图表显示在 Jupyter Notebook 的单元格中
* `sns.histplot()` -> 绘制 Series 的直方图
   
**NumPy 方法：**
* `np.log1p()` -> 先给每个输入值加一，再对变量进行对数变换。

长尾分布通常会干扰机器学习模型，因此建议在可能的情况下，将目标变量的分布转换为接近正态的分布。

