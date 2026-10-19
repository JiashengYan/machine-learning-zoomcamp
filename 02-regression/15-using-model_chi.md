---
video_url: https://www.youtube.com/watch?v=KT--uIJozes&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR&index=26
code:
  - label: Notebook
    path: notebook.ipynb
prev_url: 14-tuning-model_chi.md
next_url: 16-summary_chi.md
---
# 使用模型

[上一节](14-tuning-model_chi.md)找到了合适的正则化参数。本节训练最终模型，在测试数据上检查表现，再按照实际应用方式，用它预测一辆汽车的价格。

## 训练最终模型

回顾数据划分：训练集、验证集和测试集。目前在训练集上训练，在验证集上计算 RMSE。对于最终模型，做法有所不同：将训练集与验证集合并为 `full_train`，使用全部合并数据训练，再在测试集上做最终评估。

思路很简单：参数已经调好，希望尽可能多地利用数据训练模型，同时保留未使用的测试集作为最终检验。测试集上的 RMSE 应该与验证集上的结果相差不大。

我们有两个 DataFrame，`df_train` 和 `df_val`，现在将它们合并。pandas 提供 `concat` 函数，名称是 concatenate（连接）的缩写。它接收 DataFrame 列表，并将它们连接起来：

```python
df_full_train = pd.concat([df_train, df_val])
```


查看结果，虽然现在约有 9500 行，索引仍保留了验证集 DataFrame 原来的值。可以通过重置索引解决：

```python
df_full_train = df_full_train.reset_index(drop=True)
```

现在索引连续排列，可以用 `prepare_X` 获取特征矩阵：

```python
X_full_train = prepare_X(df_full_train)
```


还需要目标值 `y`。NumPy 的 `concatenate` 函数对数组执行相同的连接操作。数组没有 pandas 行索引，因此无须重置：

```python
y_full_train = np.concatenate([y_train, y_val])
```

取出模型训练代码，将 `X_train` 替换为 `X_full_train`，将 `y_train` 替换为 `y_full_train`，并使用之前选定的 `r`：

```python
w0, w = train_linear_regression_reg(X_full_train, y_full_train, r=0.001)
```

这就是最终模型及其权重。


## 在测试数据上检查模型

为了检查最终模型，以与验证集完全相同的方式准备测试集，只需将验证数据替换为测试数据：

```python
X_test = prepare_X(df_test)
y_pred = w0 + X_test.dot(w)
score = rmse(y_test, y_pred)
score
```

RMSE 与验证集上的结果几乎相同，四舍五入到小数点后三位时一致：

```
0.4600753970266562
```


这是一个很好的信号，说明模型具有良好的泛化表现，先前的评分并非仅靠偶然获得。

## 预测一辆汽车的价格

现在按照实际需求使用模型预测汽车价格：提取一辆汽车的全部特征，将特征向量输入模型，得到价格。

可以从测试集中任取一辆汽车，假装它是新样本。这样做是合理的，因为模型没有用这些数据训练。这里选一辆 Toyota Sienna：

```python
car = df_test.iloc[20].to_dict()
car
```

得到一个包含汽车全部信息的字典：

```python
{'make': 'toyota',
 'model': 'sienna',
 'year': 2015,
 'engine_fuel_type': 'regular_unleaded',
 'engine_hp': 266.0,
 'engine_cylinders': 6.0,
 'transmission_type': 'automatic',
 'driven_wheels': 'front_wheel_drive',
 'number_of_doors': 4.0,
 'market_category': nan,
 'vehicle_size': 'large',
 'vehicle_style': 'passenger_minivan',
 'highway_mpg': 25,
 'city_mpg': 18,
 'popularity': 2031}
```


实际预测时，输入通常并不是 DataFrame。现实场景可能是网站或应用：用户输入汽车信息，网站向模型发送包含这些信息的请求，模型返回价格。因此将汽车转换成字典，因为请求数据通常采用这样的结构。

![实际应用场景：包含汽车信息的网站表单将字典发送给模型](images/15-using-model-05-website-request-diagram-imagegen.jpg)

但 `prepare_X` 需要接收 DataFrame。因此创建一个只有一行的小 DataFrame。pandas 可以从字典列表创建 DataFrame，这里列表中只包含请求中的这一辆汽车：

```python
df_small = pd.DataFrame([car])
X_small = prepare_X(df_small)
```

然后将模型应用于这个只有一行的特征矩阵：

```python
y_pred = w0 + X_small.dot(w)
y_pred = y_pred[0]
y_pred
```

预测结果为：

```
10.63249250912739
```


因为只有一辆汽车，所以取数组的第一个数。注意，这仍是价格的对数，而不是价格本身。要撤销对数变换，使用相应的指数逆变换：

```python
np.expm1(y_pred)
```

这是我们的预测，即具有这些特征的汽车应当价值：

```
41459.336786653585
```

再看看这辆汽车实际多少钱：

```python
np.expm1(y_test[20])
```

实际价格为：

```
35000.00000000001
```


因此，预测有一定偏差：汽车实际价格为 35,000 美元，而预测约为 41,500 美元。虽然不完美，但相对而言是不错的预测。

这就是先训练完整模型，再用它预测单辆汽车价格的过程。[下一节](16-summary_chi.md)将总结本部分学到的全部内容。

## 笔记

找到最佳模型及其参数后，使用训练集和验证集共同训练，并在测试集上计算最终 RMSE。

最后，用最终模型预测新汽车的价格。

