---
video_url: https://www.youtube.com/watch?v=sGLAToAAMa4&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR&index=23
code:
  - label: Notebook
    path: notebook.ipynb
prev_url: 11-feature-engineering_chi.md
next_url: 13-regularization_chi.md
---
# 类别变量

类别变量包含的是类别，而不是连续数值。本节将它们加入汽车价格模型，并发现新增类别特征使模型效果严重恶化。这正是下一节要解决的问题。

## 表示类别的变量

目前只使用了数值特征，但数据集中还有不少类别变量。它们通常是字符串，例如 `make`、`model`、`engine_fuel_type`、`transmission_type`、`driven_wheels` 等。查看数据类型，本数据集中所有 `object` 类型的列都是类别变量。


还有一个看起来是数值、实际上可以按类别处理的变量：`number_of_doors`。它包含数字 2、3 和 4，但这些数字代表不同类型的汽车：两门、三门和四门汽车有明显差别。只是变量值恰好是数字，所以 pandas 将它们当作普通数值。

我们希望在模型中使用这些变量，因为它们可能很重要。例如，两门汽车可能比四门汽车更贵。

## 将类别编码为二元列

机器学习模型需要数值输入，因此要编码类别变量。常见方法是：为列中的每个类别创建一个独立的二元列。假设原列依次包含 2、3、4、2，就用三列表示，每种车门数量对应一列；与该行匹配的列填 1，其余列填 0：

| number_of_doors | num_doors_2 | num_doors_3 | num_doors_4 |
| --- | ---: | ---: | ---: |
| 2 | 1 | 0 | 0 |
| 3 | 0 | 1 | 0 |
| 4 | 0 | 0 | 1 |
| 2 | 1 | 0 | 0 |

一个类别列变成多个二元列，每一行恰好有一个 1。

在 pandas 中，比较操作符可以完成大部分工作。使用 `df.number_of_doors == 2` 时，每辆两门汽车对应的结果都是 `True`：

```python
df_train.number_of_doors == 2
```

结果是布尔值 Series，接下来只需用 `astype(int)` 将其转换为整数 1 和 0。

为了创建这些列，可以遍历 `[2, 3, 4]`，并用字符串模板生成列名，使每个类别都有自己的列：

```python
for v in [2, 3, 4]:
    df_train['num_doors_%s' % v] = (df_train.number_of_doors == v).astype('int')
```

## 在 prepare_X 中添加车门数量

再次修改 `prepare_X` 函数。和之前一样，先复制 DataFrame，避免意外将新列写入原始数据。同时复制 `base` 列表：如果直接在 `base` 上调用 `append`，每次运行函数都会向它追加一次 `age`。

```python
def prepare_X(df):
    df = df.copy()
    features = base.copy()

    df['age'] = 2017 - df.year
    features.append('age')

    for v in [2, 3, 4]:
        df['num_doors_%s' % v] = (df.number_of_doors == v).astype('int')
        features.append('num_doors_%s' % v)

    df_num = df[features]
    df_num = df_num.fillna(0)
    X = df_num.values
    return X
```

现在特征列表包含基础特征、`age`，以及三个新特征：`num_doors_2`、`num_doors_3` 和 `num_doors_4`。


用之前相同的代码验证模型：

```python
X_train = prepare_X(df_train)
w0, w = train_linear_regression(X_train, y_train)

X_val = prepare_X(df_val)
y_pred = w0 + X_val.dot(w)
rmse(y_val, y_pred)
```

之前的结果是 0.5172，现在得到：

```
0.5157995641502978
```

效果只略有改善，提升几乎可以忽略。车门数量这个特征并没有带来很大帮助。


## 添加品牌

不过，我认为 `make` 应该很有用。查看该列，发现有 48 个不同品牌。可以用 `value_counts` 选出出现次数最多的品牌：

```python
df.make.value_counts().head()
```


输出值是计数，品牌名称位于索引中，因此将索引转换为列表：

```python
makes = list(df.make.value_counts().head().index)
```

这样得到五个最常见的品牌：chevrolet、ford、volkswagen、toyota 和 dodge。与车门数量一样，为每个品牌创建一个二元列：

```python
for v in makes:
    df['make_%s' % v] = (df.make == v).astype('int')
    features.append('make_%s' % v)
```

再次验证，这次结果改善了约一个百分点，从 0.5158 降到 0.5077。虽然不如添加 `age` 时显著，但仍是不错的改进。

## 对所有类别变量执行相同操作

其他类别变量也可以用同样方法处理：`engine_fuel_type`、`transmission_type`、`driven_wheels`、`market_category`、`vehicle_size` 和 `vehicle_style`。这里不添加 `model`，因为车型数量实在太多。

首先，将每个变量最常见的五个值收集到字典中：

```python
categorical_variables = [
    'make', 'engine_fuel_type', 'transmission_type', 'driven_wheels',
    'market_category', 'vehicle_size', 'vehicle_style'
]

categories = {}

for c in categorical_variables:
    categories[c] = list(df_train[c].value_counts().head().index)
```


随后修改 `prepare_X`，加入所有这些类别。需要两层循环：外层遍历字典的键值对，内层遍历每个变量选中的类别值：

```python
for c, values in categories.items():
    for v in values:
        df['%s_%s' % (c, v)] = (df[c] == v).astype('int')
        features.append('%s_%s' % (c, v))
```


## 出现了问题

再次执行同样的验证代码，现在的结果远高于之前。原先约为 0.5，突然变成了 41：

```
41.45414769975256
```

显然出了问题。查看 `train_linear_regression` 输出的权重，发现它们非常大，有些约为 10 的 15 次方。我们本想通过增加变量改善模型，却使它变差了。


[下一节](13-regularization_chi.md)将解释原因，并介绍解决方法。

## 笔记

类别变量通常以字符串表示，pandas 将其识别为 object 类型。但有些看似数值的变量实际上也可以按类别处理，例如汽车的车门数量。需要将这些类别变量转换为数值形式，因为模型接收的是数值特征。可以只纳入某个特征中的部分类别，而不必包含全部类别。这种从类别到数值的转换称为独热编码（One-Hot encoding）。

## 补充说明

这种编码类别特征的方法称为“独热编码”。第三部分会进一步介绍。
