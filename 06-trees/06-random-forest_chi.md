---
video_url: https://www.youtube.com/watch?v=FZhcmOfNNZE&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR
code:
  - label: Notebook
    path: notebook.ipynb
prev_url: 05-decision-tree-tuning_chi.md
next_url: 07-boosting_chi.md
---
# 集成学习与随机森林

本节将多个决策树组合成一个模型。我们先从背后的想法开始：与其只相信一位专家，不如询问专家委员会。然后用 scikit-learn 构建随机森林，并调整它的三个主要参数。

## 专家委员会

到目前为止，我们用一棵决策树来评估贷款申请：输入客户特征，树输出违约概率，银行再根据这个分数批准或拒绝申请。

现在想象一下，由五位专家组成委员会，而不是只问一位专家。客户提交申请后，申请会交给每一位专家。他们分别查看申请并决定批准或拒绝，最终结果由多数投票决定。与单独听取一位专家的意见相比，五位专家的集体意见可能更可靠。

![随机森林：每位专家查看随机抽取的特征子集，给出违约概率，最终预测是所有概率的平均值](images/06-random-forest-01-board-of-experts-imagegen.jpg)

模型也可以采用相同的做法。我们用五个模型（g1、g2 等）代替五个人，每个模型都会返回违约概率。组合这些概率时，取它们的平均值：

```text
(1/n) * (p1 + p2 + ... + pn)
```

这种组合多个模型的方法称为集成学习，各个模型通常称为“弱学习器”：单独使用时表现一般，但组合在一起后会比任何一个单独模型更强。

## 随机森林

如果集成的模型是决策树，这种集成就称为随机森林。但树不能只是随意组成一组，它们需要彼此不同。如果在相同数据、相同参数下重复训练同一棵树，就会得到完全相同的树和预测结果；对它们取平均与只用一棵树的结果完全一样。

随机森林中的“随机”使树彼此具有差异：每棵树会使用随机抽取的特征子集；根据 `bootstrap` 参数，每棵树还可能使用从训练数据中随机重采样得到的数据副本。假设有三个特征：`assets`、`debt` 和 `price`，我们训练三棵树。第一棵树可能使用 `assets` 和 `debt`，第二棵使用 `assets` 和 `price`，第三棵使用 `debt` 和 `price`。每棵树都会给出自己的预测 p1、p2、p3，最终预测为它们的平均值：1/3 (p1 + p2 + p3)。

![由三棵决策树组成的专家委员会：每棵树使用不同的特征对，最后对预测取平均值](images/06-random-forest-02-random-forest-imagegen.jpg)

在 scikit-learn 中，随机森林位于 `ensemble` 包中。我们使用信贷评分数据，训练一个包含不同数量决策树的随机森林：树的数量从 10 到 200，每次增加 10，然后查看树的数量如何影响验证集 AUC：

```python
from sklearn.ensemble import RandomForestClassifier

scores = []

for n in range(10, 201, 10):
    rf = RandomForestClassifier(n_estimators=n, random_state=1)
    rf.fit(X_train, y_train)

    y_pred = rf.predict_proba(X_val)[:, 1]
    auc = roc_auc_score(y_val, y_pred)

    scores.append((n, auc))
```

接着将分数放入 DataFrame 并绘图：

```python
df_scores = pd.DataFrame(scores, columns=['n_estimators', 'auc'])
plt.plot(df_scores.n_estimators, df_scores.auc)
```

![验证集 AUC 与树的数量：开始时迅速上升，随后趋于稳定](images/06-random-forest-03-auc-vs-trees-imagegen.jpg)

曲线一开始快速上升，然后逐渐稳定：当树的数量达到一定程度后，继续增加树的数量不会提升分数。不过，更多的树通常不会损害模型质量，曲线会保持平稳；额外的成本主要是训练时间，而不是准确度。因此，`n_estimators` 通常会设为“在可承受范围内尽可能多”，然后优先调整其他参数。

## 调整 max_depth

接下来调整 `max_depth`，也就是森林中每棵树的最大深度。使用相同的循环，这次尝试 5、10 和 15 三种深度，并分别遍历不同的树数量：

```python
scores = []

for d in [5, 10, 15]:
    for n in range(10, 201, 10):
        rf = RandomForestClassifier(n_estimators=n,
                                    max_depth=d,
                                    random_state=1)
        rf.fit(X_train, y_train)

        y_pred = rf.predict_proba(X_val)[:, 1]
        auc = roc_auc_score(y_val, y_pred)

        scores.append((d, n, auc))
```

为了比较三种深度，我们将每种深度的结果分别画成一条曲线：

```python
columns = ['max_depth', 'n_estimators', 'auc']
df_scores = pd.DataFrame(scores, columns=columns)

for d in [5, 10, 15]:
    df_subset = df_scores[df_scores.max_depth == d]

    plt.plot(df_subset.n_estimators, df_subset.auc,
             label='max_depth=%d' % d)

plt.legend()
```

![比较 max_depth 为 5、10、15 时的验证集 AUC：深度 10 的曲线最好](images/06-random-forest-04-tuning-max-depth-imagegen.jpg)

`max_depth=10` 和 `max_depth=15` 的曲线明显优于 `max_depth=5`，其中 `max_depth=10` 最好。因此固定这个参数：

```python
max_depth = 10
```

## 调整 min_samples_leaf

第三个参数是 `min_samples_leaf`，也就是决策树叶节点允许的最少观测数。我们重复相同的步骤：固定 `max_depth=10`，尝试 `min_samples_leaf` 为 1、3、5、10 和 50，并为每种取值遍历不同的树数量：

```python
scores = []

for s in [1, 3, 5, 10, 50]:
    for n in range(10, 201, 10):
        rf = RandomForestClassifier(n_estimators=n,
                                    max_depth=max_depth,
                                    min_samples_leaf=s,
                                    random_state=1)
        rf.fit(X_train, y_train)

        y_pred = rf.predict_proba(X_val)[:, 1]
        auc = roc_auc_score(y_val, y_pred)

        scores.append((s, n, auc))
```

将五条曲线分别设置颜色并绘制：

```python
columns = ['min_samples_leaf', 'n_estimators', 'auc']
df_scores = pd.DataFrame(scores, columns=columns)

colors = ['black', 'blue', 'orange', 'red', 'grey']
values = [1, 3, 5, 10, 50]

for s, col in zip(values, colors):
    df_subset = df_scores[df_scores.min_samples_leaf == s]

    plt.plot(df_subset.n_estimators, df_subset.auc,
             color=col,
             label='min_samples_leaf=%d' % s)

plt.legend()
```

![比较 min_samples_leaf 为 1、3、5、10 和 50 时的验证集 AUC](images/06-random-forest-05-tuning-min-samples-leaf-imagegen.jpg)

取值较小的曲线（1、3 和 5）位于上方且彼此接近；10 和特别是 50 的结果更差。我们选择 `min_samples_leaf=3`：

```python
min_samples_leaf = 3
```

## 最终模型

使用选定的参数，训练包含 200 棵树的最终随机森林：

```python
rf = RandomForestClassifier(n_estimators=200,
                            max_depth=max_depth,
                            min_samples_leaf=min_samples_leaf,
                            random_state=1)
rf.fit(X_train, y_train)
```

随机森林还有其他值得了解的参数，例如 `max_features`，它控制每棵树可以看到的随机特征子集大小；`bootstrap` 控制每棵树是否使用从数据中随机重采样得到的副本进行训练。本节不调整这些参数，但值得了解。完整参数列表请参阅 [RandomForestClassifier 文档](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html)。

## 笔记

**集成学习**是一种机器学习范式，它会有策略地组合多个模型（通常称为“弱学习器”）来解决计算智能问题。与单独使用一个模型相比，这种方法往往能获得更好的预测性能。

**随机森林**是集成学习的一种：其中的模型都是决策树，通过汇总它们的预测来确定最终结果。随机森林只会从原始数据中随机选择一部分特征来进行预测。随机森林的“随机性”主要来自两个方面：

- 每棵树可能使用原始数据的 bootstrap 样本进行训练，使行采样具有随机性。
- 构建树的每个节点时，只考虑随机选择的一部分特征进行分裂。这种特征随机性有助于降低树之间的相关性，从而避免过拟合并提升对未见数据的泛化能力。

**Bootstrap（自助抽样）**是一种重采样技术，通过有放回地从原始数据中抽样来创建多个子集。这意味着一个数据点可能在同一个 bootstrap 样本中出现多次，也可能完全不被抽中。在随机森林中，每棵决策树都基于不同的 bootstrap 样本进行训练，从而进一步增加集成的多样性和稳健性。

**参数调优**对于优化随机森林模型性能非常重要。两个关键参数是：控制每棵树最大深度的 `max_depth`，以及决定森林中树数量的 `n_estimators`。增大 `max_depth` 会使模型更复杂，并可能导致过拟合。增加 `n_estimators` 通常能提升模型表现，但也会增加计算成本。

随机森林中的决策树彼此独立地训练。

**类、函数和方法**：

- `from sklearn.ensemble import RandomForestClassifier`：从 sklearn 的 ensemble 模块导入随机森林分类器。
- `plt.plot(x, y)`：根据 x 值绘制 y 值的折线图。
