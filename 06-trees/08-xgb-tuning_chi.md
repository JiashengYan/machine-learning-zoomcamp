---
video_url: https://www.youtube.com/watch?v=VX6ftRzYROM&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR
code:
  - label: Notebook
    path: notebook.ipynb
prev_url: 07-boosting_chi.md
next_url: 09-final-model_chi.md
---
# XGBoost 参数调优

上一节中，我们使用默认参数训练了一个 XGBoost 模型，并学习了如何监控训练过程。本节将使用训练集/验证集曲线调整三个最重要的 XGBoost 参数：`eta`、`max_depth` 和 `min_child_weight`。

## 要调整的参数

本节要调整的三个参数是：

1. `eta` - 学习率，控制后续模型在修正前一个模型错误时迈出的步长。
2. `max_depth` - 每棵树的最大深度。
3. `min_child_weight` - 叶节点的最小规模，近似对应随机森林中的 `min_samples_leaf`。

![需要调整的三个 XGBoost 参数：eta 控制步长，max_depth 控制树，min_child_weight 大致对应随机森林中的 min_samples_leaf](images/08-xgb-tuning-01-parameters-imagegen.jpg)

XGBoost 还有很多其他参数；本节按以下顺序调整这三个参数：先调整 `eta`，再调整 `max_depth`，最后调整 `min_child_weight`。每种参数取值都会训练一个模型，使用上一节介绍的 `parse_xgb_output` 函数解析监控输出，并比较验证集 AUC 曲线。

## 调整 eta

我们使用字典 `scores` 保存参数值与解析后的学习曲线之间的映射。对于每种 `eta` 取值，训练模型、捕获输出，并保存解析后的 DataFrame：

```python
scores = {}
```

```python
%%capture output

xgb_params = {
    'eta': 0.01,
    'max_depth': 6,
    'min_child_weight': 1,

    'objective': 'binary:logistic',
    'eval_metric': 'auc',

    'nthread': 8,
    'seed': 1,
    'verbosity': 1,
}

model = xgb.train(xgb_params, dtrain, num_boost_round=200,
                  verbose_eval=5,
                  evals=watchlist)
```

```python
key = 'eta=%s' % (xgb_params['eta'])
scores[key] = parse_xgb_output(output)
key
```

这会得到 `'eta=0.01'`。我们重复上述过程，将 `eta` 改为 0.3（之前使用的值）、1.0、0.1 和 0.05，然后把每次运行的验证集 AUC 画在同一张图上：

```python
for key, df_score in scores.items():
    plt.plot(df_score.num_iter, df_score.val_auc, label=key)

plt.ylim(0.8, 0.84)
plt.legend()
```

![比较 eta 为 0.3、1.0、0.1、0.05 和 0.01 时的验证集 AUC：eta=0.1 达到高点后保持稳定，eta=1.0 的曲线低于图表显示范围](images/08-xgb-tuning-02-tuning-eta-imagegen.jpg)

学习率控制模型学习的速度。`eta=1.0` 时，模型会完整采用每棵新树给出的修正，因此很快达到饱和，曲线跌到图表显示范围以下。`eta=0.3` 时，验证集 AUC 上升很快、较早达到峰值，然后开始下降。`eta=0.01` 时，曲线上升很慢，需要更多轮数才能达到最佳效果。折中来看，`eta=0.1` 达到了最高 AUC 并保持稳定，因此选择它。

## 调整 max_depth

接下来固定 `eta=0.1`，用相同方法调整 `max_depth`。重置 `scores` 以开始新的比较，并分别使用基准深度 6，以及深度 3、4 和 10 进行训练：

```python
key = 'max_depth=%s' % (xgb_params['max_depth'])
scores[key] = parse_xgb_output(output)
```

绘制四种深度的结果后，`max_depth=10` 明显最差，因此从比较中删除：

```python
del scores['max_depth=10']
```

然后重新绘制剩余曲线，并通过 `plt.ylim(0.8, 0.84)` 放大纵轴范围。`max_depth=6` 是第二差的结果，而较小的深度表现最好：`max_depth=3` 的验证集 AUC 最高且最稳定。

![比较 max_depth 为 6、3 和 4 时的验证集 AUC：深度 3 持续上升并保持领先，较深的树表现下降](images/08-xgb-tuning-03-max-depth-curves-imagegen.jpg)

## 调整 min_child_weight

最后调整 `min_child_weight`。再次重置 `scores`，比较基准值 1 与 10、30。下面是参数值为 30 的训练示例：

```python
%%capture output

xgb_params = {
    'eta': 0.1,
    'max_depth': 3,
    'min_child_weight': 30,

    'objective': 'binary:logistic',
    'eval_metric': 'auc',

    'nthread': 8,
    'seed': 1,
    'verbosity': 1,
}

model = xgb.train(xgb_params, dtrain, num_boost_round=200,
                  verbose_eval=5,
                  evals=watchlist)
```

```python
key = 'min_child_weight=%s' % (xgb_params['min_child_weight'])
scores[key] = parse_xgb_output(output)
```

即使用 `plt.ylim(0.82, 0.84)` 放大图表，三条曲线仍然很接近：这个参数带来的差异较小。因此保留默认值 `min_child_weight=1`。

![min_child_weight 为 1、10 和 30 时的验证集 AUC：三条曲线非常接近](images/08-xgb-tuning-04-min-child-weight-curves-imagegen.jpg)

## 最终模型

综合起来，参数为 `eta=0.1`、`max_depth=3`、`min_child_weight=1`。从图表看，验证集 AUC 在大约 175 轮 boosting 后达到最佳，因此本节使用 `num_boost_round=175` 训练最终模型：

```python
xgb_params = {
    'eta': 0.1,
    'max_depth': 3,
    'min_child_weight': 1,

    'objective': 'binary:logistic',
    'eval_metric': 'auc',

    'nthread': 8,
    'seed': 1,
    'verbosity': 1,
}

model = xgb.train(xgb_params, dtrain, num_boost_round=175)
```

其他值得了解的参数：

- `subsample` - 每棵树所使用训练数据的比例。
- `colsample_bytree` - 每棵树所使用特征的比例，类似随机森林中的随机特征子集。
- `lambda` 和 `alpha` - 分别是权重上的 L2 和 L1 正则化项。

完整参数列表请参阅 [XGBoost 参数文档](https://xgboost.readthedocs.io/en/latest/parameter.html)。

## 笔记

XGBoost 有许多可调参数，其中最重要的三个是：

- `eta`（默认值为 0.3）
  - 也称为 `learning_rate`，用于在每轮 boosting 中对新特征的权重进行正则化，从而帮助避免过拟合。取值范围：`[0, 1]`。
- `max_depth`（默认值为 6）
  - 树的最大深度。增加这个值会让模型更复杂，也更容易过拟合。取值范围：`[0, inf]`。
- `min_child_weight`（默认值为 1）
  - 叶节点的最小样本数量。取值范围：`[0, inf]`。

还可以使用其他方法为 XGBoost 模型寻找最佳参数。本节采用的顺序是：

- 首先找到最佳 `eta`。
- 然后找到最佳 `max_depth`。
- 最后找到最佳 `min_child_weight`。

其他有用的参数：

- `subsample`（默认值为 1）
  - 训练实例的子采样比例。设为 0.5 表示在生长树之前，模型会随机抽取一半的训练数据。取值范围：`(0, 1]`。
- `colsample_bytree`（默认值为 1）
  - 类似随机森林，每棵树使用随机抽取的特征子集进行训练。
- `lambda`（默认值为 1）
  - 也称为 `reg_lambda`，是权重上的 L2 正则化项。增大该值会让模型更保守。
- `alpha`（默认值为 0）
  - 也称为 `reg_alpha`，是权重上的 L1 正则化项。增大该值会让模型更保守。

### 替代方法：使用循环调整 XGBoost

不必针对每个 `eta` 值手动重复训练过程，可以使用循环自动完成，也可以用相同方式调整其他参数：

```python
# Train XGBoost models for each eta and store AUC results
scores = {} # dictionary to store results for each eta
etas = [0.01, 0.05, 0.1, 0.3, 1.0] # list of parameter values. in this case it is 'eta'.

for eta in etas:
    evals_result = {}

    xgb_params = {
        'eta': eta,
        'max_depth': 6,
        'min_child_weight': 1,

        'objective': 'binary:logistic',
        'eval_metric': 'auc',

        'nthread': 8,
        'seed': 1,
        'verbosity': 1
    }

    model = xgb.train(
        xgb_params,
        dtrain,
        evals=watchlist,
        verbose_eval=0,
        num_boost_round=200,
        evals_result=evals_result
    )

    columns = ['iter', 'train_auc', 'val_auc']
    train_aucs = list(evals_result['train'].values())[0]
    val_aucs = list(evals_result['val'].values())[0]

    df_results = pd.DataFrame(
        list(zip(range(1, len(train_aucs) + 1), train_aucs, val_aucs)),
        columns=columns
    )

    key = f'eta={eta}'
    scores[key] = df_results
```
