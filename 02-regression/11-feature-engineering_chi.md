---
video_url: https://www.youtube.com/watch?v=-aEShw4ftB0&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR&index=22
code:
  - label: Notebook
    path: notebook.ipynb
prev_url: 10-car-price-validation_chi.md
next_url: 12-categorical-variables_chi.md
---
# 特征工程

特征工程是从已有特征创建新特征的过程。本节创建第一个新特征——车龄，并观察它如何明显降低模型的 RMSE。

![特征工程在建模前将年份列转换为车龄特征](images/11-feature-engineering-01-feature-engineering-imagegen.jpg)

*图：用 2017 减去每辆汽车的年份，得到进入模型输入矩阵的 `age` 特征。*

## 从年份到车龄

再次查看训练集，其中有一列 `year`。这是预测汽车价格最重要的变量之一：汽车越旧，通常越便宜；越新，通常越贵。

不直接使用年份，而是计算车龄。为此需要知道数据的采集时间，这份数据采集于 2017 年。因此，本例中的“现在”是 2017 年，车龄就是 `2017 - year`。有些汽车车龄为 0 年，有些为 9 年，有些为 26 年：

```python
2017 - df_train.year
```

我们希望将这个车龄作为模型特征。

## 在 prepare_X 中添加车龄特征

修改[上一节](10-car-price-validation_chi.md)编写的 `prepare_X` 函数。新特征叫 `age`，计算方式与刚才相同，只是使用函数接收到的 DataFrame，而不是 `df_train`。输入可以是训练集、验证集或测试集。

构建矩阵时也要纳入新特征。因此不再直接使用 `base`，而是创建名为 `features` 的列表，包含基准数值特征和新的 `age` 特征：

```python
def prepare_X(df):
    df['age'] = 2017 - df.year
    features = base + ['age']

    df_num = df[features]
    df_num = df_num.fillna(0)
    X = df_num.values
    return X
```

## 函数不应修改输入数据

运行此函数时，第一行会添加一列。再次查看 `df_train`，可以发现它多出了一列。这说明我们修改了传入的 DataFrame。

这个函数不应产生这种副作用。作为使用者，我不希望它改变我的数据，因为有些操作可能无法撤销。更好的做法是不修改接收到的 DataFrame。

解决方法很简单：在任何操作之前，先复制 DataFrame，函数内部只处理副本。原始 DataFrame 保持不变。函数返回后，不再需要这个副本，因为我们最终只关心 `X`。

```python
def prepare_X(df):
    df = df.copy()

    df['age'] = 2017 - df.year
    features = base + ['age']

    df_num = df[features]
    df_num = df_num.fillna(0)
    X = df_num.values
    return X
```

现在 `X_train` 有六列：五个基础特征，加上最后一列 `age`。

## 检查改进效果

使用与之前相同的代码验证模型：准备矩阵、训练模型、应用于验证集并计算 RMSE：

```python
X_train = prepare_X(df_train)
w0, w = train_linear_regression(X_train, y_train)

X_val = prepare_X(df_val)
y_pred = w0 + X_val.dot(w)
rmse(y_val, y_pred)
```

模型有了明显改善，RMSE 从 0.76 降到了 0.51：

```
0.5172055461058291
```

还可以像之前一样，将预测值与真实值绘制在同一幅直方图中，观察改进幅度。注意，现在与 `y_val` 比较，因为预测的是验证集，而不是此前的训练集。

```python
sns.histplot(y_pred, label='prediction', color='red', alpha=0.5, bins=50)
sns.histplot(y_val, label='target', color='blue',  alpha=0.5, bins=50)
plt.legend()
```

两个分布的形状现在更加接近。仍有较大改进空间，例如模型完全没有捕捉到图中的某根柱子，但总体表现好得多。

还可以添加更多特征。[下一节](12-categorical-variables_chi.md)将讨论类别变量，例如 `model`、`make` 等列。

## 笔记

在数据集中加入车龄特征，由数据中的最大年份减去每辆汽车的年份得到。这个新特征改善了模型表现，可以通过 RMSE 以及目标变量 y 与预测值的分布比较来衡量。

