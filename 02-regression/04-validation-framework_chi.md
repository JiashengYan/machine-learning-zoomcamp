---
video_url: "https://www.youtube.com/watch?v=ck0IfiPaQi0&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR&index=15"
code:
  - label: "notebook.ipynb"
    path: "notebook.ipynb"
prev_url: 03-eda_chi.md
next_url: 05-linear-regression-simple_chi.md
---
# 建立验证框架

数据集已经清理并探索过，现在建立验证框架：将数据划分为训练集、验证集和测试集，并准备各部分的目标变量。我们只用 pandas 和 NumPy 手动完成这些步骤。

## 训练、验证和测试

通过前面的课程，我们已经知道为什么需要验证模型。为此将数据集分成三部分：第一部分用于训练，第二部分用于验证，最后一部分用于测试。训练模型后，检查它在验证集上的表现，而测试集暂时保留，只在最后偶尔使用，以确认模型效果良好。

随后，为每一部分创建特征矩阵 X 和目标变量 y：

![数据划分为训练集、验证集和测试集，各自拥有 X 和 y](images/04-validation-framework-01-train-val-test-split-crisp.jpg)

接下来用 pandas 实现这一过程。

## 计算各部分的大小

我们按 60/20/20 的比例划分，因此先计算 20% 对应多少条记录。DataFrame 中有近 12,000 条记录，其 20% 约为 2,400。记录数必须为整数，不能是 2382.8 这样的数，因此用 `int()` 去掉小数部分：

```python
n = len(df)

n_val = int(n * 0.2)
n_test = int(n * 0.2)
n_train = n - n_val - n_test
```

理想情况下也可以写 `n_train = int(n * 0.6)`，但由于取整，三部分之和可能不再等于 `n`，从而意外遗漏一些记录。因此，我们先扣除验证集和测试集，剩余记录全部用于训练。

本数据集各部分的大小如下：

```python
n, n_val, n_test, n_train
```

```
(11914, 2382, 2382, 7150)
```


## 用 iloc 提取 DataFrame 的一部分

知道各部分的大小后，需要提取相应数量的记录。为此使用 pandas 入门中介绍的 `iloc`。通常我们向它传入索引列表，但它也接受范围。要获取前十条记录：

```python
df.iloc[:10]
```

这里从开头取到 10，但不包含 10，因此得到第 0 至第 9 条记录。同样，`df.iloc[10:20]` 获取第 10 至第 19 条，`df.iloc[10:]` 获取第 10 条到末尾的所有记录。

最简单的方法就是按顺序切分 DataFrame：

```python
df_val = df.iloc[:n_val]
df_test = df.iloc[n_val:n_val+n_test]
df_train = df.iloc[n_val+n_test:]
```

但这样有一个问题：划分是顺序进行的。数据集中的汽车按品牌排列，因此验证集开头包含所有 BMW，末尾包含所有 Porsche，而训练集中完全没有 BMW：


可见数据集中存在某种顺序，我们需要打破它。通常打乱数据是个好主意：如果数据中存在偶然的排序，我们要确保它不会影响划分。

## 打乱记录

思路如下：可以向 `iloc` 传入任意数字序列，它会按该顺序返回行。因此，我们可以生成 0 到 n-1 的数字，打乱顺序，然后将打乱后的前 20% 用于验证，接下来的 20% 用于测试，剩余部分用于训练：


使用 NumPy 的 `np.arange` 生成数字序列，用 `np.random.shuffle` 打乱它：

```python
idx = np.arange(n)
np.random.shuffle(idx)
```

现在，用打乱后的数字索引 DataFrame，而不再使用连续范围：

```python
df_train = df.iloc[idx[:n_train]]
df_val = df.iloc[idx[n_train:n_train+n_val]]
df_test = df.iloc[idx[n_train+n_val:]]
```

前 `n_train` 个数字用于训练，接下来的 `n_val` 个用于验证，其余全部用于测试：


这段代码还有一个问题：在你的电脑上运行时，会得到不同的记录，第一辆汽车可能不是 Porsche 或 GMC，而是其他品牌。为使结果可复现，我们在打乱前设置随机种子。NumPy 入门中介绍过这一点：

```python
np.random.seed(2)

idx = np.arange(n)
np.random.shuffle(idx)
```

随机种子设为 2 后，训练集的第一条记录是 Chevrolet。如果使用相同版本的 NumPy，你也会在电脑上得到完全相同的子集。

## 重置索引

检查三个 DataFrame 的长度：

```python
len(df_train), len(df_val), len(df_test)
```

```
(7150, 2382, 2382)
```

查看 `df_train`，可以看到索引保留了原始行号，例如 2735、6720 等，并且顺序随机。有时这不太方便，因为我们并不需要知道每条记录原来的索引。因此重置索引，并将结果赋回 DataFrame。传入 `drop=True`，是因为不需要将旧索引保留为一列：

```python
df_train = df_train.reset_index(drop=True)
df_val = df_val.reset_index(drop=True)
df_test = df_test.reset_index(drop=True)
```

现在索引重新从 0 连续递增到 7149。


## 准备 y 并删除 msrp

这些 DataFrame 将用于生成特征矩阵 X。目标 y 来自 `msrp` 列。由于其分布存在长尾，我们需要应用 log1p 变换。不必保留 pandas Series，而是立即通过 `.values` 提取数值，得到 NumPy 数组，因为 y 不需要索引等 pandas 信息：

```python
y_train = np.log1p(df_train.msrp.values)
y_val = np.log1p(df_val.msrp.values)
y_test = np.log1p(df_test.msrp.values)
```

最后，用 `del` 操作符从各个 DataFrame 中删除 `msrp`：

```python
del df_train['msrp']
del df_val['msrp']
del df_test['msrp']
```

删除它是为了防止意外使用。如果价格进入特征，就会变成用价格预测价格，模型当然会表现得完美。之后我们可能花费大量时间寻找问题。我曾多次遇到这种情况，所以划分数据后，总会把目标提取到独立变量中，并彻底从 DataFrame 中删除。


这就是完整的验证框架。我们没有使用专门的划分库，而是仅用 pandas 和 NumPy 手动实现。现在已学会将数据分为三部分，可以进入模型训练阶段。下一节讨论[线性回归](05-linear-regression-simple_chi.md)。

## 笔记

通常将数据集划分为训练集、验证集和测试集。每一部分都需要对应的特征矩阵（X）和目标向量（y）。先计算各部分大小，再打乱记录，确保各部分包含原数据集中非连续的记录，最后利用打乱后的索引创建各个子集。

**Pandas 属性和方法：**

* `df.iloc[]` -> 按数值索引选择 DataFrame 的记录子集
* `df.reset_index()` -> 重置原有索引
* `del df[col]` -> 删除指定列

**NumPy 方法：**

* `np.arange()` -> 生成数字数组
* `np.random.shuffle()` -> 原地打乱数组
* `np.random.seed()` -> 设置随机种子以便复现结果

