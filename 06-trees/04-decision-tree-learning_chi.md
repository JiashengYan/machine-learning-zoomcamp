---
video_url: "https://www.youtube.com/watch?v=XODz6LwKY7g&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR"
code:
  - label: Notebook
    path: notebook.ipynb
prev_url: 03-decision-trees_chi.md
next_url: 05-decision-tree-tuning_chi.md
---
# 决策树学习算法

本节将深入了解决策树学习算法：它如何寻找最佳分裂、什么是不纯度，以及算法会在何时停止分裂。

## 决策树术语

首先了解决策树各部分的名称：

- 节点包含一个条件，例如 `assets > 3000`。
- 节点向外延伸出两条分支，分别对应“真”和“假”。
- 树顶部的节点是根节点；除了叶节点以外，每个节点都是另外两个节点的父节点。
- 树底部的节点是叶节点（决策节点），树在这里停止并给出预测。
- 树的深度是层数，也就是从根节点到叶节点的最长路径长度。

为了理解算法如何学习这样的树，我们使用一个小型示例数据集，而不是完整的信贷评分数据。

## 为单个列寻找最佳分裂

我们选取 8 位客户，使用一个特征 `assets` 和目标变量 `status`：

```python
data = [
    [8000, 'default'],
    [2000, 'default'],
    [   0, 'default'],
    [5000, 'ok'],
    [5000, 'ok'],
    [4000, 'ok'],
    [9000, 'ok'],
    [3000, 'default'],
]

df_example = pd.DataFrame(data, columns=['assets', 'status'])
```

我们想建立一个决策树桩——只有一个条件 `assets > T` 的树。问题是：哪个阈值 `T` 最好？

这个条件会把数据分成两部分：条件为假时位于左侧（`assets <= T`），条件为真时位于右侧（`assets > T`）。候选阈值取观测值之间的中间位置。按 `assets` 排序后，候选值为：

```python
Ts = [0, 2000, 3000, 4000, 5000, 8000]
```

阈值 0 和 8000 不太合理，因为其中一侧会没有样本，不过算法仍会尝试它们。我们用 `T = 4000` 进行分裂：

```python
T = 4000
df_left = df_example[df_example.assets <= T]
df_right = df_example[df_example.assets > T]

display(df_left)
print(df_left.status.value_counts(normalize=True))
display(df_right)
print(df_right.status.value_counts(normalize=True))
```

左侧有 4 位客户：3 位 `default`、1 位 `ok`。右侧也有 4 位客户：3 位 `ok`、1 位 `default`。

每一侧都预测多数类别，也就是出现频率最高的状态。左侧预测 `default`，右侧预测 `ok`。左侧错误率为 1/4 = 25%（那位 `ok` 客户），右侧同样为 1/4 = 25%。

这个错误率称为分类错误率，是衡量一个群组有多不纯的一种方法。纯群组只包含一个类别，其分类错误率为 0。我们可以通过 `value_counts(normalize=True)` 计算两侧的错误率，再按群组大小进行加权平均：

```text
T    decision LEFT   impurity LEFT   decision RIGHT  impurity RIGHT  AVG
0    default         0%              ok              43%             21%
2000 default         0%              ok              33%             16%
3000 default         0%              ok              20%             10%
4000 default         25%             ok              25%             25%
5000 default         50%             ok              50%             50%
8000 default         43%             ok              0%              21%
```

最佳阈值是 `T = 3000`，因为平均不纯度最低，只有 10%。因此，这个特征的最佳分裂条件是 `assets > 3000`。

```text
       ASSETS > 3000 
        /         \
   FALSE           TRUE
   DEFAULT         OK
```

## 检查多个特征

真实数据通常有很多特征，算法需要在所有特征中寻找最佳分裂。我们增加第二个特征 `debt`：

```python
data = [
    [8000, 3000, 'default'],
    [2000, 1000, 'default'],
    [   0, 1000, 'default'],
    [5000, 1000, 'ok'],
    [5000, 1000, 'ok'],
    [4000, 1000, 'ok'],
    [9000,  500, 'ok'],
    [3000, 2000, 'default'],
]

df_example = pd.DataFrame(data, columns=['assets', 'debt', 'status'])
```

对于 `debt`，候选阈值为 `[500, 1000, 2000]`。我们把所有阈值放入字典，然后遍历每个特征及其阈值：

```python
thresholds = {
    'assets': [0, 2000, 3000, 4000, 5000, 8000],
    'debt': [500, 1000, 2000]
}

for feature, Ts in thresholds.items():
    print('#####################')
    print(feature)
    for T in Ts:
        print(T)
        df_left = df_example[df_example[feature] <= T]
        df_right = df_example[df_example[feature] > T]

        display(df_left)
        print(df_left.status.value_counts(normalize=True))
        display(df_right)
        print(df_right.status.value_counts(normalize=True))

        print()
    print('#####################')
```

对于 `debt`，最佳分裂的平均不纯度是 16%，比使用 `assets` 得到的 10% 更差。因此，整体最佳分裂仍然是 `assets > 3000`。如果有更多特征，只需在比较中添加相应的行即可。

## 寻找分裂点的算法

用伪代码表示，寻找最佳分裂的流程如下：

```text
FOR each feature in FEATURES:
    FIND all thresholds for the feature
    FOR each threshold in thresholds:
        SPLIT the dataset using "feature > threshold" condition
        COMPUTE the impurity of this split
SELECT the condition with the LOWEST IMPURITY
```

分类错误率并不是唯一的不纯度度量。Scikit-learn 使用更敏感的准则：基尼不纯度和熵。对于回归树，对应的度量是 MSE。核心思路不变：选择不纯度最低的分裂。虽然本节讨论的是分类，但决策树也可以解决回归问题。

## 停止条件

根节点分裂后，算法会在左右两侧递归地应用相同的流程。什么时候停止？

- 群组已经纯净——分类错误率为 0%，没有必要继续分裂。
- 树已达到最大深度限制，由 `max_depth` 控制。
- 群组太小，无法继续分裂，由 `min_samples_leaf` 控制。
- 已达到叶节点（决策节点）的最大数量。

这些停止条件可以防止树过拟合。

## 完整算法

综合起来：

- 寻找最佳分裂：对每个特征尝试所有可能的阈值，选择不纯度最低的条件。
- 如果尚未满足停止条件（未达到最大深度、群组仍足够大且尚未纯净），就对左侧和右侧重复上述步骤。

下一节将为信贷评分项目调整 `max_depth` 和 `min_samples_leaf`。

## 笔记

本课首先回顾上一课的内容：如何使用 scikit-learn 训练决策树，以及如何处理决策树因过拟合而无法良好泛化的问题。

本课将学习决策树如何进行最佳分裂，以及可用于分裂的不同分类准则。我们会通过示例深入了解使用 `misclassification`（分类错误率）进行树分裂的过程，并讨论迭代分裂的不同停止条件。

- **决策树的结构**：决策树是一种数据结构，由包含条件的**节点**和表示条件取值（真或假）的**分支**组成。树从**根节点**开始，根节点有两个子节点，每个子节点也可以继续拥有子节点。树的末层是终止节点，也称为**叶节点**。
- **决策树的深度**：树的**深度**是树所包含的层数，也可以理解为根节点到叶节点的最长路径长度。
- **规则、条件与阈值**：决策树学习算法会在每个节点确定最佳分裂条件，以得到尽可能好的分类器。如果有很多**特征**，算法会考虑每个特征及其候选**阈值**，选择适用于当前节点的最佳特征。在每个节点，算法会评估每个特征的所有可能阈值，并计算分裂后的分类错误率，然后选择不纯度最低的条件（特征和阈值）。
- **分类错误率**：每次分裂的目标是把数据划分成两个尽可能纯净的集合，也就是说，每个集合中的数据应主要属于同一个类别。我们也可以把它理解为尽量降低**分类错误率**（不纯度）。分类错误率是分裂后两个集合错误率的加权平均。每个集合的预测类别由其中的**多数类别**决定。
- **不纯度准则**：常见的不纯度度量包括**基尼不纯度**和**熵**。回归问题也可以使用 **MSE**。
- **决策树也可用于回归问题**：本节重点讨论了决策树分类器，但决策树也可通过决策树回归器解决回归问题。
- **停止条件**：在子节点上递归分裂数据的过程会依据停止条件终止。这些条件可以避免模型过拟合，包括：
  - 群组已经**纯净**：不纯度为 0%。
  - 已达到**最大深度**。
  - 群组**小于**设定的最小大小。
  - 已达到叶节点/终止节点的最大数量。

#### 决策树学习算法概要

- 在一个节点处寻找最佳分裂。
- 如果达到 `max_depth`，则停止。
- 对每个子节点，如果节点样本量足够且尚未纯净，就从头重复这个过程。
