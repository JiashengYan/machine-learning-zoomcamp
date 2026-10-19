---
video_url: https://www.youtube.com/watch?v=rawGPXg2ofE&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR&index=21
code:
  - label: Notebook
    path: notebook.ipynb
prev_url: 09-rmse_chi.md
next_url: 11-feature-engineering_chi.md
---
# 在验证数据上计算 RMSE

上一节实现了均方根误差 RMSE，并用它衡量基准模型的质量。但存在一个问题：我们是在训练数据上衡量的，而模型已经从这些数据中学习过。本节改为在验证数据上评估模型。

## 验证框架

回顾验证过程：将数据集划分为训练集、验证集和测试集。在训练集上训练模型 g，再将它应用于验证集，观察其在未见过的数据上的表现。

![训练集、验证集和测试集：模型 g 在训练集上训练，在验证集上应用](images/10-car-price-validation-01-split-diagram-crisp.jpg)

两节之前，我们用五个数值特征——`engine_hp`、`engine_cylinders`、`highway_mpg`、`city_mpg` 和 `popularity`——建立了第一个基准模型，随后编写 `rmse` 函数来客观衡量模型质量。但计算误差时，使用的仍是训练模型时的数据。

正确做法是将模型应用于验证数据，再查看验证集上的均方根误差。

## 用 prepare_X 准备特征矩阵

取出之前编写的代码。下面是模型训练代码，其中这一行用于准备特征矩阵 X：

```python
X_train = df_train[base].fillna(0).values
```

这一行包含多项操作，因此为它编写一个专用函数 `prepare_X`，接收 DataFrame：

```python
def prepare_X(df):
    df_num = df[base]
    df_num = df_num.fillna(0)
    X = df_num.values
    return X
```

函数完成三件事：选择数值列、填充缺失值，最后提取并返回 NumPy 数组形式的特征矩阵。虽然从一行变成五行，但更容易理解。

这个函数的意义在于，无论输入的是训练集、验证集还是测试集，都以相同方式准备数据。它适用于任意 DataFrame，因此参数名用 `df`，而不是 `df_train`。


## 在验证数据上计算 RMSE

现在可以训练并验证模型。训练阶段：从训练集 DataFrame 准备矩阵，并训练模型。验证阶段：用同一函数准备验证矩阵，应用模型，再计算真实目标值与预测值之间的 RMSE：

```python
X_train = prepare_X(df_train)
w0, w = train_linear_regression(X_train, y_train)

X_val = prepare_X(df_val)
y_pred = w0 + X_val.dot(w)
rmse(y_val, y_pred)
```

得到结果：

```
0.7616530991301577
```


这个数值与训练数据上的结果很接近。这正是我们希望看到的：模型在未见过的数据上，与在学习过的数据上表现相近。

从代码可以清楚地区分两个部分：


- 训练部分只使用训练集：准备矩阵并学习权重。
- 验证部分以同样方式准备验证集，应用上一步训练好的模型，再计算它的均方根误差。

现在可以用验证集上的均方根误差衡量模型质量，也能检查某项修改是否确实改善模型。接下来将从[特征工程](11-feature-engineering_chi.md)开始尝试。

## 笔记

在汽车价格预测数据集的验证部分计算 RMSE，从而获得评估模型表现的指标。

