---
video_url: "https://www.youtube.com/watch?v=SvPpMMYtYbU&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR&index=19"
code:
  - label: "notebook.ipynb"
    path: "notebook.ipynb"
prev_url: 07-linear-regression-training_chi.md
next_url: 09-rmse_chi.md
---
# 汽车价格预测项目的基准模型

上一节实现了 `train_linear_regression`，用于求出线性回归模型的权重。现在终于将它应用到汽车价格数据上，构建第一个模型：基准模型。这个模型刻意保持简单，只使用数值特征，其效果将作为后续改进的参照。

## 选择基础特征

首先回顾训练集 DataFrame 中有哪些列：

```python
df_train.columns
```


我们实现的线性回归只能处理数字，因此基准模型选择以下数值列：

- `engine_hp` - 发动机马力
- `engine_cylinders` - 气缸数
- `highway_mpg` - 高速公路上每加仑行驶英里数
- `city_mpg` - 城市路况下每加仑行驶英里数
- `popularity` - 受欢迎程度

将这五个列名放入一个列表：

```python
base = ['engine_hp', 'engine_cylinders', 'highway_mpg',
        'city_mpg', 'popularity']
```


用这个列表选择 DataFrame 的列子集，再将数值提取为 NumPy 数组：

```python
X_train = df_train[base].values
```

## 处理缺失值

尝试直接训练模型：

```python
w0, w = train_linear_regression(X_train, y_train)
```

结果不符合预期：偏置项和权重都变成了 `nan`，即“非数值”。


数据存在问题。检查缺失值：

```python
df_train[base].isnull().sum()
```

可以看到，`engine_hp` 和 `engine_cylinders` 包含缺失值。最简单的处理方法是用零填充：

```python
X_train = df_train[base].fillna(0).values
```


用零填充可能看起来奇怪，因为汽车不可能没有马力，发动机也不可能没有气缸。要理解为什么这样仍然可行，可以看缺失值在线性回归公式中的作用。假设有两个特征，第一个缺失：

```text
g(xi) = w0 + xi1·w1 + xi2·w2      (xi1 is missing)
     = w0 +   0·w1 + xi2·w2
     = w0 +          xi2·w2
```

当 `xi1` 为零时，`xi1 · w1` 这一项消失，模型实际上忽略了该特征。从常识角度看，用该特征的均值替换往往更合理，作业中已经见过这种方法。但从实践角度看，用零能让处理过程保持简单，有时效果也不错。基准模型采用零填充。

## 训练模型

现在特征矩阵中没有缺失值，可以完成训练：

```python
X_train = df_train[base].fillna(0).values
w0, w = train_linear_regression(X_train, y_train)
```

得到偏置项 `w0 ≈ 7.93`。注意，我们预测的是价格的对数，因此这是对数尺度上的值。它与五个特征权重一起组成基准模型。

## 比较预测值与实际价格

现在可以将训练好的模型应用于训练数据。使用矩阵乘法，再单独加上偏置项：

```python
y_pred = w0 + X_train.dot(w)
```

为了观察预测效果，将预测值绘制成直方图，并与真实目标值的直方图比较。使用探索性数据分析中已经用过的 seaborn `histplot` 函数：

```python
sns.histplot(y_pred, color='red', alpha=0.5, bins=50)
sns.histplot(y_train, color='blue', alpha=0.5, bins=50)
```

`alpha` 参数控制柱子的透明度，使同一幅图中的两组直方图都能看见。

![比较预测值（红色）与实际价格（蓝色）的分布](images/08-baseline-model-06-prediction-histogram-crisp.jpg)

观察图表，预测分布与真实值分布并不吻合：红色峰值位于蓝色峰值左侧，说明许多情况下模型预测的值低于实际价格。

仅凭这张图，我们怀疑模型并不理想，但这不是客观的衡量方式。开始改进模型时，需要确认效果确实提高。因此需要一个数值，量化模型的好坏。下一节将通过 RMSE 完成这一点。

## 笔记

* 本节构建基准模型，使用 `df_train` 数据集求出偏置项（w0）和特征权重（w）。为此调用上一节的 `train_linear_regression(X, y)` 函数。
* 当前实现的线性回归仅处理数值特征，因此特征矩阵只使用 `df_train` 中的数值特征。
* 发现 `df_train` 中某些特征值为 `nan`。为简单起见，将它们设为 `0`，使模型可以求解；也可以用非零值填充，例如该特征的均值。
* 计算权重后，将其代入$$\\\\ \large g(X) = w_0 + X \cdot w$$以得到预测向量 y。
* 随后将预测 y 和真实 y 的直方图绘制在同一幅图上，进行视觉比较。

