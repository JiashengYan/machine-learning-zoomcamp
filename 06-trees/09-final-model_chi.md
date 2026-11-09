---
video_url: https://www.youtube.com/watch?v=lqdnyIVQq-M&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR
code:
  - label: Notebook
    path: notebook.ipynb
prev_url: 08-xgb-tuning_chi.md
next_url: 10-summary_chi.md
---
# 选择最佳模型

本节将比较本模块训练的三类模型——决策树、随机森林和 XGBoost，选出最佳模型，然后使用完整训练数据训练最终模型，并在测试集上评估。

## 比较三种模型

在前几节中，我们已分别为三种模型调优参数。现在在训练集上训练它们，并在同一个验证集上比较 AUC。

![在同一个验证集上比较三种调优后的模型](images/09-final-model-01-comparing-validation-imagegen.jpg)

调优后的决策树使用 `max_depth=6` 和 `min_samples_leaf=15`，结果如下：

```python
dt = DecisionTreeClassifier(max_depth=6, min_samples_leaf=15)
dt.fit(X_train, y_train)
```

```python
y_pred = dt.predict_proba(X_val)[:, 1]
roc_auc_score(y_val, y_pred)
```

```text
0.7850802838390931
```

调优后的随机森林包含 200 棵树，使用 `max_depth=10` 和 `min_samples_leaf=3`，结果如下：

```python
rf = RandomForestClassifier(n_estimators=200,
                            max_depth=10,
                            min_samples_leaf=3,
                            random_state=1)
rf.fit(X_train, y_train)
```

```python
y_pred = rf.predict_proba(X_val)[:, 1]
roc_auc_score(y_val, y_pred)
```

```text
0.8249709379767989
```

最后是调优后的 XGBoost 模型：使用 `eta=0.1`、`max_depth=3`、`min_child_weight=1`，训练 175 轮：

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

```python
y_pred = model.predict(dval)
roc_auc_score(y_val, y_pred)
```

```text
0.8360387251459157
```

XGBoost 的 AUC 约为 0.836，优于随机森林的约 0.825 和单棵决策树的约 0.785。对于表格数据，这是很常见的结果：梯度提升树表现很强。代价是 XGBoost 有许多超参数，而且容易快速过拟合，因此调参时需要比随机森林投入更多精力；随机森林通常开箱即用，调参需求较少。

## 训练最终模型

选定最佳模型和参数后，就可以按常规流程进行最后一步：将训练集与验证集合并为 `df_full_train`，使用相同参数重新训练，然后在之前保留的测试集上评估。此前我们没有使用测试集：

```python
df_full_train = df_full_train.reset_index(drop=True)

y_full_train = (df_full_train.status == 'default').astype(int).values
del df_full_train['status']
```

使用新拟合的 `DictVectorizer` 将 `df_full_train` 和测试集都转换为特征矩阵，再将它们封装成 DMatrix。注意，测试集的 DMatrix 不需要标签：

```python
dicts_full_train = df_full_train.to_dict(orient='records')

dv = DictVectorizer(sparse=False)
X_full_train = dv.fit_transform(dicts_full_train)

dicts_test = df_test.to_dict(orient='records')
X_test = dv.transform(dicts_test)
```

```python
dfulltrain = xgb.DMatrix(X_full_train, label=y_full_train,
                    feature_names=dv.get_feature_names_out())

dtest = xgb.DMatrix(X_test, feature_names=dv.get_feature_names_out())
```

然后训练最终模型，并计算它在测试数据上的 AUC：

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

model = xgb.train(xgb_params, dfulltrain, num_boost_round=175)
```

```python
y_pred = model.predict(dtest)
roc_auc_score(y_test, y_pred)
```

```text
0.8322662626460096
```

测试集 AUC 约为 0.832，与验证集上的 0.836 接近，两者之间没有明显差距，说明模型很好地泛化了：它学到的模式也适用于从未见过的数据。

## 笔记

我们根据 AUC 分数，从决策树、随机森林和 XGBoost 中选择最终模型。之后准备 `df_full_train` 和 `df_test`，用前者训练最终模型，再用后者评估。如果模型在训练集与测试集上的 AUC 没有太大差别，说明它较好地泛化了所学到的模式。

一般来说，XGBoost 在表格数据上的表现优于其他机器学习模型，但它有许多超参数，因此容易过拟合。为了优化模型，XGBoost 需要投入更多精力调参。
