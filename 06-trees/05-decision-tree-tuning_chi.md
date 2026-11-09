---
video_url: "https://www.youtube.com/watch?v=XJaxwH50Qok&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR"
code:
  - label: Notebook
    path: notebook.ipynb
prev_url: 04-decision-tree-learning_chi.md
next_url: 06-random-forest_chi.md
---
# 决策树参数调优

本节将调整决策树：尝试不同的 `max_depth` 和 `min_samples_leaf`，选出验证集 AUC 最好的组合，并查看最终的树。

## 要调整哪些参数

`DecisionTreeClassifier` 有很多参数，例如不纯度度量 `criterion`、`max_depth`、`min_samples_leaf` 等。其中最重要的两个是：

- `max_depth` - 最大层数，限制规则可以变得多具体。
- `min_samples_leaf` - 叶节点中的最少样本数，避免叶节点只包含一两个客户。

调参就是选择能在验证集上获得最佳分数的参数值；本例使用 AUC。

## 选择 max_depth

先单独调整 `max_depth`。我们尝试从 1 到 20 的若干深度，以及 `None`（不限制），并记录每种情况下的验证集 AUC：

```python
depths = [1, 2, 3, 4, 5, 6, 10, 15, 20, None]

for depth in depths: 
    dt = DecisionTreeClassifier(max_depth=depth)
    dt.fit(X_train, y_train)
    
    y_pred = dt.predict_proba(X_val)[:, 1]
    auc = roc_auc_score(y_val, y_pred)
    
    print('%4s -> %.3f' % (depth, auc))
```

```text
   1 -> 0.606
   2 -> 0.669
   3 -> 0.739
   4 -> 0.761
   5 -> 0.767
   6 -> 0.744
  10 -> 0.683
  15 -> 0.654
  20 -> 0.654
None -> 0.662
```

深度为 4、5 和 6 时效果最好（AUC 约为 0.76）。树太浅会欠拟合；树太深会过拟合，最终表现甚至不如较简单的树。

## 加入 min_samples_leaf

接下来选取表现较好的深度 4、5 和 6，分别尝试多个 `min_samples_leaf` 值，并将结果以元组形式存入列表：

```python
scores = []

for depth in [4, 5, 6]:
    for s in [1, 5, 10, 15, 20, 500, 100, 200]:
        dt = DecisionTreeClassifier(max_depth=depth, min_samples_leaf=s)
        dt.fit(X_train, y_train)

        y_pred = dt.predict_proba(X_val)[:, 1]
        auc = roc_auc_score(y_val, y_pred)
        
        scores.append((depth, s, auc))
```

然后将分数放入 DataFrame：

```python
columns = ['max_depth', 'min_samples_leaf', 'auc']
df_scores = pd.DataFrame(scores, columns=columns)
```

排序后的 DataFrame 不方便浏览。我们可以重新整理数据：行对应 `min_samples_leaf`，列对应 `max_depth`，单元格中显示 AUC。`pivot` 方法可以完成这项操作：

```python
df_scores_pivot = df_scores.pivot(index='min_samples_leaf', columns=['max_depth'], values=['auc'])
df_scores_pivot.round(3)
```

```text
                    auc              
max_depth             4      5      6
min_samples_leaf                     
1                 0.761  0.767  0.759
5                 0.761  0.768  0.759
10                0.761  0.762  0.778
15                0.764  0.772  0.785
20                0.761  0.774  0.774
100               0.756  0.763  0.776
200               0.747  0.759  0.768
500               0.680  0.680  0.680
```

热力图更容易阅读，颜色最浅的单元格代表最佳结果：

```python
sns.heatmap(df_scores_pivot, annot=True, fmt=".3f")
```

![AUC 热力图：max_depth=6 且 min_samples_leaf=15 时效果最好](images/05-decision-tree-tuning-05-heatmap-imagegen.jpg)

最佳组合是 `max_depth=6` 和 `min_samples_leaf=15`，AUC 为 0.785，比只调 `max_depth` 得到的结果更好。

请注意，第二个参数的搜索只使用了深度 4 到 6。在大型数据集上，尝试所有组合会很慢，因此我们先选择 `max_depth`，再调整其他参数。这个小型数据集允许我们检查更多组合；若这样做，也会发现其他表现接近的组合。一个好习惯是先进行粗略搜索，再逐步缩小范围。

![在更广的深度范围内搜索，会发现多个表现接近的组合](images/05-decision-tree-tuning-07-wider-search-imagegen.jpg)

## 最终决策树

使用调好的参数训练决策树，并查看它学到的规则：

```python
dt = DecisionTreeClassifier(max_depth=6, min_samples_leaf=15)
dt.fit(X_train, y_train)

print(export_text(dt, feature_names=list(dv.get_feature_names_out())))
```

```text
|--- records=no <= 0.50
|   |--- seniority <= 6.50
|   |   |--- amount <= 862.50
|   |   |   |--- price <= 925.00
|   |   |   |   |--- amount <= 525.00
|   |   |   |   |   |--- class: 1
|   |   |   |   |--- amount >  525.00
|   |   |   |   |   |--- class: 1
|   |   |   |--- price >  925.00
|   |   |   |   |--- price <= 1382.00
|   |   |   |   |   |--- class: 0
|   |   |   |--- price >  1382.00
|   |   |   |   |   |--- class: 0
|   |   |--- amount >  862.50
|   |   |   |--- assets <= 8250.00
...
```

这棵树仍然易于阅读，不像之前不受限制的树那样复杂：它先检查 `records`，然后依次查看 `seniority`、`amount`、`price` 和 `assets`，以判断客户是否可能违约。

需要注意的是，查看结果时要留意没有意义的参数值，例如搜索网格中出现 `nan`。某个组合可能偶然取得很高的分数；应确认所选参数确实能按照预期控制树的大小。

## 笔记

本课将介绍决策树（DT）的不同控制参数。其中 `max_depth` 和 `min_samples_leaf` 最为重要。我们会先调整 `max_depth`，然后调整其他参数。接着创建一个 DataFrame，记录 `max_depth`、`min_sample_leaf` 的所有组合及其对应的 AUC 分数。通过对 DataFrame 进行透视并绘制热力图，可以更方便地找出最佳参数组合。最后使用选定的参数重新训练决策树，并查看树的规则。

### 步骤

- **调优流程**：通过反复尝试找到合适的参数设置。
  - 先尝试不同的 `max_depth`，确定一组表现较好的深度。
  - 再基于这组深度尝试不同的 `min_samples_leaf`，进一步调优模型。

  这种方法对**大型数据集**而言计算效率较高，但对小型数据集来说未必是最优方法。

- **用热力图可视化**：将调参得到的分数（例如 AUC）存入透视表，并用 `seaborn` 绘制热力图，便于识别高分区域，从而找到合适的 `max_depth` 和 `min_samples_leaf` 组合。

**注意：**选择能够有效控制树大小的参数值；即使 `nan`（非数字）看起来能获得较高分数，也应避免使用。

### `max_depth` 和 `min_samples_leaf` 的重要性

- **控制过拟合**：这两个参数对于防止过拟合很重要。
  - `max_depth` 限制树的复杂度，避免树长得过深并记住训练数据。
  - `min_samples_leaf` 确保叶节点包含足够的样本，降低节点过于贴合训练数据的风险。
- **对偏差和方差的影响**：这两个参数也会影响模型的偏差和方差。
  - 增大 `max_depth`、减小 `min_samples_leaf`，通常会使模型更复杂、偏差更低但方差更高。
  - 减小 `max_depth`、增大 `min_samples_leaf`，通常会使模型更简单、偏差更高但方差更低。

因此，需要在 `max_depth` 和 `min_samples_leaf` 之间取得平衡。最佳取值取决于具体数据集和问题。
