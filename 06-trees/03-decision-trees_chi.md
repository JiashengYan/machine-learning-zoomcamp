---
video_url: "https://www.youtube.com/watch?v=YGiQvFbSIg8&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR"
code:
  - label: Notebook
    path: notebook.ipynb
prev_url: 02-data-prep_chi.md
next_url: 04-decision-tree-learning_chi.md
---
# 决策树

本节将使用 scikit-learn 在信贷评分数据上训练第一棵决策树，观察过拟合，并学习如何控制树的大小。

## 决策树的样子

从本质上说，决策树就是一组 if/else 规则。对于信贷风险问题，我们可以手动编写一组规则：

```python
def assess_risk(client):
    if client['records'] == 'yes':
        if client['job'] == 'parttime':
            return 'default'
        else:
            return 'ok'
    else:
        if client['assets'] > 6000:
            return 'ok'
        else:
            return 'default'
```

从上往下读：如果客户有过往违约记录并且是兼职工作，我们预测其会 `default`。如果客户有记录但工作类型不同，则预测 `ok`。如果客户没有记录，我们就查看资产：超过 6000 则预测 `ok`，否则预测 `default`。

![将相同规则画成决策树](images/03-decision-trees-01-risk-rules-tree-imagegen.jpg)

从训练数据中取一位客户，看看这些规则会如何判断：

```python
xi = df_train.iloc[0].to_dict()
assess_risk(xi)
```

```text
'ok'
```

手写规则无法扩展。决策树算法的作用，就是自动从数据中学习这些规则。

## 训练决策树

我们使用 scikit-learn 中的 `DecisionTreeClassifier`。特征包含数值列和已编码为字符串的类别列。先将每行转换为字典，把缺失值填为 0，再用 `DictVectorizer` 把字典转换为特征矩阵：

```python
from sklearn.tree import DecisionTreeClassifier
from sklearn.feature_extraction import DictVectorizer
from sklearn.metrics import roc_auc_score
from sklearn.tree import export_text
```

```python
train_dicts = df_train.fillna(0).to_dict(orient='records')

dv = DictVectorizer(sparse=False)
X_train = dv.fit_transform(train_dicts)
```

`DictVectorizer` 会对类别特征进行独热编码，例如创建 `job=partime` 和 `records=no` 这样的列；数值特征则保持原样。

现在训练决策树：

```python
dt = DecisionTreeClassifier()
dt.fit(X_train, y_train)
```

然后检查模型在验证集上的表现：

```python
val_dicts = df_val.fillna(0).to_dict(orient='records')
X_val = dv.transform(val_dicts)

y_pred = dt.predict_proba(X_val)[:, 1]
roc_auc_score(y_val, y_pred)
```

```text
0.6548400377806302
```

## 过拟合

AUC 为 0.65，表现并不理想。模型在训练数据上的表现如何？

```python
y_pred = dt.predict_proba(X_train)[:, 1]
roc_auc_score(y_train, y_pred)
```

```text
1.0
```

训练集 AUC 为 1.0，验证集却只有 0.65：这棵树记住了训练数据。它为每位客户学到了一套特定规则，因此在见过的客户上表现完美，但无法泛化到没见过的客户。这就是过拟合：记住数据，却不能泛化。

![过拟合的树用非常具体的规则记住每位客户](images/03-decision-trees-05-memorizing-imagegen.jpg)

原因是我们没有限制树的生长，因此树变得很深：它不断分裂，直到每个叶节点都变得纯净。

## 控制树的大小

我们可以用 `max_depth` 参数限制树的深度。较浅的树无法记住每位客户，只能学习更宽泛的规则：

```python
dt = DecisionTreeClassifier(max_depth=2)
dt.fit(X_train, y_train)
```

```python
y_pred = dt.predict_proba(X_train)[:, 1]
auc = roc_auc_score(y_train, y_pred)
print('train:', auc)

y_pred = dt.predict_proba(X_val)[:, 1]
auc = roc_auc_score(y_val, y_pred)
print('val:', auc)
```

```text
train: 0.7054989859726213
val: 0.6685264343319367
```

训练集 AUC 降低了，但验证集 AUC 提高了。我们牺牲了模型在已知客户上的部分准确度，换来了更好的泛化能力。

## 查看树的内部规则

我们使用 `export_text` 查看树学到的规则：

```python
print(export_text(dt, feature_names=list(dv.get_feature_names_out())))
```

```text
|--- records=no <= 0.50
|   |--- seniority <= 6.50
|   |   |--- class: 1
|   |--- seniority >  6.50
|   |   |--- class: 0
|--- records=no >  0.50
|   |--- job=partime <= 0.50
|   |   |--- class: 0
|   |--- job=partime >  0.50
|   |   |--- class: 1
```

这是树的文本表示：没有违约记录且资历超过 6.5 年的客户会被预测为 `ok`（类别 0）；有记录且兼职的客户会被预测为 `default`（类别 1），其他情况以此类推。

![学到的决策树：先按 records 分裂，再按 job 分裂](images/03-decision-trees-06-learned-rules-imagegen.jpg)

深度为 1、只有一个条件的树称为决策树桩（decision stump）。它更像一次分裂，而不是完整的树，是最简单的决策树。

![深树与只有一次分裂的决策树桩](images/03-decision-trees-07-decision-stump-imagegen.jpg)

接下来要问的是：学习算法如何决定每次分裂使用哪个条件和阈值？这是下一节的主题。

## 笔记

决策树是一种强大的算法，能够拟合复杂数据集。决策树通过一系列 *if/else* 语句进行预测，将节点分裂成两个或更多子节点。

虽然决策树用途广泛，但也容易过拟合。常见原因之一是树的深度过大：模型会记住训练数据中的各种模式，却难以在未见过的数据（验证集或测试集）上表现良好。

要缓解过拟合，可以通过降低树的深度来减少模型复杂度。

深度为 1 的决策树称为 `decision stump`，只有从根节点开始的一次分裂。

**类、函数和方法**：

- `DecisionTreeClassifier`：`sklearn.tree` 中的分类模型。
- `max_depth`：用于控制决策树最大深度的超参数。
- `export_text`：`sklearn.tree` 中用于输出决策树规则文本的方法。

*注：* 我们已分别在第 3 节和第 4 节介绍过 `DictVectorizer` 与 `roc_auc_score`。
