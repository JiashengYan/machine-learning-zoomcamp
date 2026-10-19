---
video_url: https://www.youtube.com/watch?v=lW-YVxPgzQw&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR&index=25
code:
  - label: Notebook
    path: notebook.ipynb
prev_url: 13-regularization_chi.md
next_url: 15-using-model_chi.md
---
# 模型调优

[上一节](13-regularization_chi.md)看到，正则化参数 `r` 会影响模型质量。模型调优就是寻找这个参数的最佳值，而验证集正是完成这一任务的工具。

## 尝试不同的 r 值

我们将尝试一组不同的 `r`：从零开始，再尝试很小的数，逐渐增大，甚至到 10：

```python
for r in [0.0, 0.00001, 0.0001, 0.001, 0.1, 1, 10]:
    X_train = prepare_X(df_train)
    w0, w = train_linear_regression_reg(X_train, y_train, r=r)

    X_val = prepare_X(df_val)
    y_pred = w0 + X_val.dot(w)
    score = rmse(y_val, y_pred)

    print(r, w0, score)
```

遍历列表，对每个 `r` 训练模型，并计算验证数据上的 RMSE。每次输出正则化参数、偏置项和评分：


## 解读结果


可以看到，正则化为零时，偏置项非常大，RMSE 也很大，达到 266。这就是存在重复列问题的失效模型。即使只加入少量正则化，效果也立即改善，此后评分变化不大。随着正则化增强，效果开始略微变差，到 10 时更差。

观察偏置项，正则化越强，它就越小。0.001 似乎是不错的取值：模型表现尚未下降，参数也不大。实际上，这里选它或相邻值的差别不明显，因此采用 0.001。


## 训练最终模型

使用选定值再次训练模型：

```python
r = 0.001
X_train = prepare_X(df_train)
w0, w = train_linear_regression_reg(X_train, y_train, r=r)

X_val = prepare_X(df_val)
y_pred = w0 + X_val.dot(w)
score = rmse(y_val, y_pred)
score
```


我们选定了合适的正则化参数，用它训练模型，并确认模型在验证集上有效：

```
0.46081585838957173
```

接下来还需要在测试集上检查，这就是[下一节](15-using-model_chi.md)的内容。

## 笔记

模型调优通过数据集的验证部分寻找最佳正则化超参数，然后使用这个正则化值训练模型。

