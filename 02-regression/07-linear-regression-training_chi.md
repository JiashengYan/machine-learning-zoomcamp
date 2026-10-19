---
video_url: "https://www.youtube.com/watch?v=hx6nak-Y11g&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR&index=18"
code:
  - label: "notebook.ipynb"
    path: "notebook.ipynb"
prev_url: 06-linear-regression-vector_chi.md
next_url: 08-baseline-model_chi.md
---
# 训练线性回归：正规方程

上一节学习了在已有权重时如何应用线性回归模型。本节回答剩下的问题：权重从哪里来？我们将推导正规方程，即计算权重的闭式公式，并将它实现为 `train_linear_regression` 函数。

## 求解权重

给定特征矩阵 `X`，我们希望预测值 `g(X) = Xw` 接近真实目标值 `y`：

```text
Xw ≈ y
```

理想情况下，`Xw` 恰好等于 `y`，可以尝试求解这个方程组中的 `w`。

暂时假设 `X` 可逆，即存在矩阵 `X⁻¹`，使 `X⁻¹X` 等于单位矩阵 `I`。此时可以在方程两边左乘 `X⁻¹`：

```text
X⁻¹Xw = X⁻¹y
   Iw = X⁻¹y
    w = X⁻¹y
```

![利用 X 的逆矩阵推导解](images/07-linear-regression-training-01-inverse-solution-crisp.jpg)

逆矩阵将 `X` “抵消”，于是直接得到权重。

但这里存在一个问题：`X` 是长方形矩阵，有 `m` 行（每辆汽车一行）和 `n+1` 列（每个特征一列，再加上偏置项）。本数据集中，`m` 远大于 `n+1`，因此它显然不是方阵，而只有方阵才可能有通常意义上的逆矩阵。这种直接求逆的方法不可用；一般也无法精确满足所有方程。

## 格拉姆矩阵与正规方程

即使没有精确解，也可以求出最接近的近似解。在 `Xw ≈ y` 两边左乘 `X` 的转置：

```text
XᵀXw = Xᵀy
```

矩阵 `XᵀX` 称为格拉姆矩阵（Gram matrix）：

![格拉姆矩阵 X 转置乘 X 是 (n+1) × (n+1) 的方阵](images/07-linear-regression-training-02-gram-matrix-crisp.jpg)

与 `X` 不同，格拉姆矩阵是方阵，具有 `n+1` 行和 `n+1` 列。当它可逆时（并不总是如此，稍后会处理不可逆的情况），可以在两边左乘 `(XᵀX)⁻¹`：

```text
(XᵀX)⁻¹XᵀXw = (XᵀX)⁻¹Xᵀy
```

左边的 `(XᵀX)⁻¹XᵀX` 是单位矩阵，而 `Iw = w`。因此得到：

```text
w = (XᵀX)⁻¹Xᵀy
```

这就是正规方程：

![正规方程：w = (XᵀX)⁻¹Xᵀy](images/07-linear-regression-training-03-normal-equation-crisp.jpg)

这个 `w` 不是原方程组的精确解，而是在最小二乘意义下最接近的解。这一结论有严格的数学证明，推导涉及较多数学知识。如果感兴趣，可以阅读《The Elements of Statistical Learning》中的相关推导。

## 实现正规方程

用 NumPy 逐步实现公式。为了简单起见，先使用一个小型示例矩阵，包含九行和三个特征。

```python
X = [
    [148, 24, 1385],
    [132, 25, 2031],
    [453, 11, 86],
    [158, 24, 185],
    [172, 25, 201],
    [413, 11, 86],
    [38,  54, 185],
    [142, 25, 431],
    [453, 31, 86],
]
X = np.array(X)
```

正规方程的第一步是计算格拉姆矩阵：

```python
XTX = X.T.dot(X)
```


然后求其逆矩阵：

```python
XTX_inv = np.linalg.inv(XTX)
```

可以快速检查逆矩阵是否正确：格拉姆矩阵与其逆矩阵相乘，应得到单位矩阵。

```python
XTX.dot(XTX_inv).round(1)
```

```text
array([[ 1., -0.,  0.],
       [-0.,  1.,  0.],
       [ 0.,  0.,  1.]])
```


对角线上的值为 1，其他位置非常接近零。它们不完全等于零，是因为浮点数精度有限，计算中会累积微小的舍入误差，但可以将其视为零。

现在需要目标向量 `y`。先随意设定九个价格：

```python
y = [100, 200, 150, 250, 100, 200, 150, 250, 120]
```

将各部分组合起来，通过正规方程得到权重：

```python
w_full = XTX_inv.dot(X.T).dot(y)
```

不过，这个矩阵遗漏了一项：偏置项。偏置很重要，它是基准预测，表示不了解汽车其他信息时的价格。为了纳入偏置，给 `X` 添加一列 1，再重复相同步骤：

```python
ones = np.ones(X.shape[0])
X = np.column_stack([ones, X])
XTX = X.T.dot(X)
XTX_inv = np.linalg.inv(XTX)
w_full = XTX_inv.dot(X.T).dot(y)
```

`np.column_stack` 将全为 1 的向量与特征矩阵按列组合，使这些 1 成为 `X` 的第一列。


结果 `w_full` 包含全部权重：第一个元素是偏置项，其余是特征权重。可以将它们分开：

```python
w0 = w_full[0]
w = w_full[1:]
```

注意，这里的权重为负。之前的例子中权重为正，马力越大，价格越高。负权重表示相反的关系：特征值增加时，预测价格下降。如果该特征是车龄，就很合理：汽车越老，价格越低。


最后将所有步骤封装成函数。函数接收 `X` 和 `y`，在内部添加全为 1 的列，应用正规方程，并分别返回偏置项和特征权重：

```python
def train_linear_regression(X, y):
    ones = np.ones(X.shape[0])
    X = np.column_stack([ones, X])

    XTX = X.T.dot(X)
    XTX_inv = np.linalg.inv(XTX)
    w_full = XTX_inv.dot(X.T).dot(y)

    return w_full[0], w_full[1:]
```

由于添加全为 1 的列是在函数内部完成的，使用者无须处理偏置项，只需传入原始特征矩阵，其余由函数完成。下一节终于可以把这个函数应用到汽车价格数据集上。

## 笔记

为了让预测值尽可能接近目标值 $y$，需要从一般线性回归方程计算权重。特征矩阵不是方阵，没有通常意义上的逆矩阵，因此需要近似求解。可以使用**格拉姆矩阵**，即特征矩阵转置 $X^T$ 与特征矩阵 $X$ 的乘积。由该公式得到的权重或系数向量 $w$，是在最小二乘意义下最接近原线性回归方程组的解。

正规方程：

$w$ = $(X^TX)^{-1}X^Ty$

其中：

$X^TX$ 是格拉姆矩阵




