---
video_url: "https://www.youtube.com/watch?v=YkyevnYyAww&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR&index=17"
code:
  - label: "notebook.ipynb"
    path: "notebook.ipynb"
prev_url: 05-linear-regression-simple_chi.md
next_url: 07-linear-regression-training_chi.md
---
# 线性回归：向量形式

上一节实现了单辆汽车的线性回归。现在将其推广：先用点积表示一辆汽车的预测，再用矩阵与向量的乘法一次性预测数据集中的所有汽车。

## 点积

回顾当前的问题：g 是线性回归模型，X 是特征矩阵（训练数据），y 是目标，也就是价格：

![g(X) 近似等于 y：模型、特征矩阵和目标](images/06-linear-regression-vector-01-g-x-approx-y-crisp.jpg)

先再次写出单辆汽车的公式：偏置项加上从 1 到 n 的求和，每一项是特征与其权重的乘积：

$$g(x_i) = w_0 + \sum_{j=1}^{n} x_{ij} \cdot w_j$$

观察这个求和可以发现，它就是点积，即向量与向量的乘法。NumPy 入门中已经讨论过点积，因此可以用相应记号改写公式：将向量 x<sub>i</sub> 转置，再与权重向量相乘：

$$g(x_i) = w_0 + x_i^T w$$

![求和就是点积：g(xi) = w0 + xiT w](images/06-linear-regression-vector-02-dot-product-notation-crisp.jpg)

这种写法更紧凑，接下来实现它。已有 `linear_regression` 函数，先把求和部分提取成独立函数。这与之前实现的向量点积相同：结果初始化为零，然后逐个将对应元素相乘并累加：

```python
def dot(xi, w):
    n = len(xi)

    res = 0.0

    for j in range(n):
        res = res + xi[j] * w[j]

    return res
```

现在线性回归函数变短了，只需将偏置项加到特征向量与权重向量的点积上：

```python
def linear_regression(xi):
    return w0 + dot(xi, w)
```


## 虚拟特征

还可以进一步简化。偏置项 w0 没有对应特征，单独放在外面。将它并入点积：假设汽车多了一个特征 x<sub>i0</sub>，其值始终为 1。

权重向量就变为 w0、w1、w2，直到 wn，维数变成 n+1。特征向量则变为 x<sub>i0</sub>、x<sub>i1</sub>、x<sub>i2</sub> 等，其中 x<sub>i0</sub> 为 1：

![添加虚拟特征：w 和 xi 都增加一个元素](images/06-linear-regression-vector-04-fake-feature-crisp.jpg)

为什么这样可行？计算点积时，w0 乘以 1，数值不变，其余部分与之前相同。因此结果等价，可以用点积表示整个线性回归公式。

在 Python 中，用 `+` 连接两个列表会创建新列表，将第一个列表的元素放在前面，这正是我们需要的：

```python
w_new = [w0] + w
```

```python
[7.17, 0.01, 0.04, 0.002]
```

对特征也做同样处理：在 xi 前面添加 1，然后计算两个新向量的点积：

```python
def linear_regression(xi):
    xi = [1] + xi
    return dot(xi, w_new)
```

结果应该与之前一致：

```python
linear_regression(xi)
```

```
12.312
```

确实如此，仍然是上一节得到的 12.312。

## 对所有汽车进行线性回归

现在重新考虑所有样本，而不只是一辆汽车。我们有矩阵 X。由于添加了虚拟特征，矩阵每行都以 1 开始，随后是该汽车的各项特征：第一行对应第一辆汽车，第二行对应第二辆，依此类推，直到第 m 行。

![特征矩阵 X：每行都以 1 开始](images/06-linear-regression-vector-06-matrix-vector-multiplication-crisp.jpg)

X 有 m 行、n+1 列。对每一行与权重向量 w 计算点积，就能得到该汽车的预测。对数据集中所有行执行这一操作，就得到预测向量 y。

你可能已经看出，这与矩阵乘法很相似。实际上，它是矩阵与向量的乘法：应用线性回归时，只需将矩阵 X 与向量 w 相乘：

$$Xw \approx y$$

开始实现：取三辆汽车，在各自的特征向量前添加 1，再将它们组成一个矩阵，也就是列表的列表，并转换成二维 NumPy 数组：

```python
x1  = [1, 148, 24, 1385]
x2  = [1, 132, 25, 2031]
x10 = [1, 453, 11, 86]

X = [x1, x2, x10]
X = np.array(X)
```

```python
array([[   1,  148,   24, 1385],
       [   1,  132,   25, 2031],
       [   1,  453,   11,   86]])
```

权重向量仍使用之前的 w_new：

```python
w0 = 7.17
w = [0.01, 0.04, 0.002]
w_new = [w0] + w
```

现在只需将矩阵与向量相乘。NumPy 数组提供了 `dot` 方法：

```python
def linear_regression(X):
    return X.dot(w_new)
```

```python
linear_regression(X)
```

```
array([12.38 , 13.552, 12.312])
```

三辆汽车各得到一个预测值，也就是各自的价格：


这就是线性回归。回顾推导过程：从单辆汽车的 for 循环开始，识别出它是特征向量与权重向量的点积，再通过添加值为 1 的虚拟特征简化表达，最后推广到完整的特征矩阵。此时线性回归就是一次矩阵与向量的乘法。

你可能一直在想权重从哪里来，该如何确定它们的值。下一节就讨论这个问题。

## 笔记

线性回归公式可以概括为特征与权重的点积。通过增加一个值为 1 的虚拟特征，可以把*偏置*项也纳入点积，例如原文记号 $w_{0}^{x_{i0}},\ where\ x_{i0} = 1\ for\ w_0$。

包含所有记录时，可通过***特征矩阵***与***权重向量***相乘计算线性回归，得到预测向量 `y`。

