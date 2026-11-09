---
video_url: https://www.youtube.com/watch?v=xFarGClszEM&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR
code:
  - label: Notebook
    path: notebook.ipynb
prev_url: 06-random-forest_chi.md
next_url: 08-xgb-tuning_chi.md
---
# 梯度提升与 XGBoost

本节介绍组合多个模型的第二种方法：梯度提升。与决策树彼此独立训练的随机森林不同，梯度提升会依次训练模型，后一个模型会修正前一个模型的错误。我们将使用最流行的梯度提升实现 XGBoost 来训练第一个模型，并观察它的学习过程。

## 梯度提升与随机森林

在随机森林中，所有树都并行训练：每棵树使用随机抽取的特征子集，彼此之间不进行交互，最后对它们的预测取平均值。

梯度提升则是按顺序进行的。先训练一个模型并查看它在哪里出错，再让第二个模型学习这些错误，从而修正第一个模型的错误。第三个模型接着修正第二个模型的错误，以此类推。最终预测会综合整个序列中所有模型学到的内容。

![随机森林并行训练模型；梯度提升按顺序训练，每个模型学习并修正前一个模型的错误](images/07-boosting-01-boosting-vs-random-forest-imagegen.jpg)

如果这一序列中的模型是决策树，这种方法称为梯度提升树：

![梯度提升树：每棵树根据前一个模型的错误进行训练，最终预测综合所有树的结果](images/07-boosting-02-gradient-boosting-trees-imagegen.jpg)

## 安装 XGBoost

XGBoost 不属于 scikit-learn，因此需要先安装：

```bash
pip install xgboost
```

然后导入，按照惯例将其命名为 `xgb`：

```python
import xgboost as xgb
```

## 将数据封装为 DMatrix

XGBoost 不能直接使用 NumPy 数组。我们需要将数据封装到一种名为 `DMatrix` 的特殊数据结构中，它针对 XGBoost 训练做了优化。我们用特征矩阵和目标数组创建 `DMatrix`，同时传入特征名称：

```python
features = list(dv.get_feature_names_out())
dtrain = xgb.DMatrix(X_train, label=y_train, feature_names=features)
dval = xgb.DMatrix(X_val, label=y_val, feature_names=features)
```

## 训练第一个模型

使用 `xgb.train` 训练模型。它接收一个参数字典、训练用的 DMatrix，以及 boosting 轮数（也就是序列中的模型数量）。我们先从 10 轮开始：

```python
xgb_params = {
    'eta': 0.3,
    'max_depth': 6,
    'min_child_weight': 1,

    'objective': 'binary:logistic',
    'nthread': 8,

    'seed': 1,
    'verbosity': 1,
}

model = xgb.train(xgb_params, dtrain, num_boost_round=10)
```

其中几个参数需要说明：

- `eta` 是学习率，控制后续模型每次修正的幅度。
- `max_depth` 和 `min_child_weight` 控制树的大小，类似于决策树和随机森林中的 `max_depth` 与 `min_samples_leaf`。
- `objective` 指定要解决的任务。`binary:logistic` 表示二元分类，并让模型输出概率。

训练完成后，使用 `model.predict(dval)` 进行预测并计算评估指标：

```python
y_pred = model.predict(dval)
roc_auc_score(y_val, y_pred)
```

结果是 0.8152745150274878，仅训练 10 轮就已经优于调优后的决策树。

## 监控模型表现

10 轮是一个盲目尝试：我们不知道模型在训练过程中的表现。可以传入一个 watchlist，让 XGBoost 在每一轮后评估指定的数据集。默认情况下，XGBoost 会监控它优化的 logloss，但这个指标不容易解释，因此我们改用一直在使用的 AUC 作为评估指标。现在训练 200 轮，并要求每 5 轮输出一次指标：

```python
watchlist = [(dtrain, 'train'), (dval, 'val')]
```

```python
%%capture output

xgb_params = {
    'eta': 0.3,
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

`%%capture output` 这个 magic 命令会将单元格的打印输出捕获到名为 `output` 的变量中，之后可以对它进行解析。输出大致如下：

```text
[0]	train-auc:0.86300	val-auc:0.76818
[5]	train-auc:0.92863	val-auc:0.80606
[10]	train-auc:0.95002	val-auc:0.81558
[15]	train-auc:0.96558	val-auc:0.81680
[20]	train-auc:0.97316	val-auc:0.81775
...
```

训练集 AUC 快速增长并接近最大值，而验证集 AUC 趋于平稳。这是过拟合的表现：经过某些轮数后，模型继续改善训练集表现，但在未见过的数据上没有进一步提升。

## 解析监控输出

为了用 pandas 分析这些数字，我们将捕获的文本解析为 DataFrame。每行包含迭代次数以及两个 AUC 值：

```python
def parse_xgb_output(output):
    results = []

    for line in output.stdout.strip().split('\n'):
        it_line, train_line, val_line = line.split('\t')

        it = int(it_line.strip('[]'))
        train = float(train_line.split(':')[1])
        val = float(val_line.split(':')[1])

        results.append((it, train, val))

    columns = ['num_iter', 'train_auc', 'val_auc']
    df_results = pd.DataFrame(results, columns=columns)
    return df_results
```

现在可以根据迭代次数绘制两条 AUC 曲线：

```python
df_score = parse_xgb_output(output)

plt.plot(df_score.num_iter, df_score.train_auc, label='train')
plt.plot(df_score.num_iter, df_score.val_auc, label='val')
plt.legend()
```

![训练集 AUC 上升至 1，而验证集 AUC 趋于平稳并开始下降](images/07-boosting-03-train-val-auc-imagegen.jpg)

训练曲线最终达到 1，而验证曲线很快停止改善。这张图将在下一节用于调参：验证集 AUC 达到峰值时对应的 boosting 轮数，可以作为最终使用的轮数。

## 笔记

**梯度提升**

与随机森林中每棵决策树独立训练不同，梯度提升树会按顺序组合模型：每个模型根据前一个模型产生的预测错误进行训练，并尝试改善预测。这个过程持续 `n` 轮，最后将所有模型的预测组合起来，得到最终预测。

XGBoost 是实现梯度提升技术的常用库。使用前需要通过 `pip install xgboost` 安装。训练和评估模型时，需要把训练数据和验证数据封装为 XGBoost 的特殊数据结构 `DMatrix`。这种结构经过优化，能够加快 XGBoost 模型的训练。

**XGBoost 训练参数**

- `eta`：学习率，表示模型学习的速度。
- `max_depth`：控制树的大小。
- `min_child_weight`：控制子节点的最小权重。
- `objective`：指定要解决的问题，可以是回归或分类任务（二元分类可使用 `'binary:logistic'`）。
- `nthread`：使用 8 个线程进行并行训练。
- `seed`：设置为 1，以便复现结果。
- `verbosity`：设置为 1（`True`），在训练模型时显示警告（如果有）。

**类、函数和方法**：

- `xgb.train()`：用于训练 XGBoost 模型的方法。
- `xgb_params`：训练模型时使用的超参数键值对。
- `watchlist`：用于保存训练集和验证集的列表，以便在每轮训练后评估模型表现。列表中的元素是 DMatrix 数据集及其名称组成的元组，例如 `watchlist = [(dtrain, 'train'), (dval, 'val')]`。
- `%%capture output`：用于捕获单元格标准输出和标准错误的 IPython magic 命令。

### 提取 `xgb.train(..)` 的训练结果

视频中使用 Jupyter magic 命令 `%%capture output` 提取 `xgb.train(..)` 方法的输出。

另一种方法是使用 `xgb.train(..)` 的 `evals_result` 参数。可以将空字典传给该参数，训练方法会把结果填入字典。结果类型为 `OrderedDict`，因此需要将其转换为 DataFrame。可以使用 `zip()`，示例如下：

```python
evals_result = {}

model = xgb.train(params=xgb_params,
                  dtrain=dm_train,
                  num_boost_round=200,
                  verbose_eval=5,
                  evals=watchlist,
                  evals_result=evals_result)

columns = ['iter', 'train_auc', 'val_auc']
train_aucs = list(evals_result['train'].values())[0]
val_aucs = list(evals_result['val'].values())[0]

df_scores = pd.DataFrame(
    list(zip(
        range(1, len(train_aucs) + 1),
        train_aucs,
        val_aucs
    )), columns=columns)

plt.plot(df_scores.iter, df_scores.train_auc, label='train')
plt.plot(df_scores.iter, df_scores.val_auc, label='val')
plt.legend()
```

### 在 Mac 上安装 XGBoost

有些同学在 Mac 上安装 XGBoost 时遇到问题。

运行 `pip install xgboost` 并尝试在脚本中 `import xgboost` 时，可能会收到警告或错误，提示尚未安装 libomp，并建议在终端运行 `brew install libomp`。

请注意：这样安装的 libomp 版本可能无法与 `xgboost` 正常配合！

尝试运行 `xgb.DMatrix(X_train, label=y_train, feature_names=features)` 后，问题可能表现为以下两种情况：

- **Python 脚本：**出现 Segmentation fault: 11（段错误：11）
- **Jupyter Notebook：**单元格一直无法完成，Notebook 无响应，直到重启 kernel。不过容易让人困惑的是，有时它又能正常运行。

#### Conda

如果使用 Anaconda 或 Miniconda，可以尝试通过 conda 安装 xgboost。

如果之前使用 pip 安装过 xgboost，请先卸载：

```bash
pip uninstall xgboost
```

然后使用 conda 重新安装：

```bash
conda install -c conda-forge xgboost
```

这也会安装所需版本的 libomp。

#### 不使用 conda

如果不使用 conda，也可以手动安装另一个与 XGBoost 兼容的 libomp 版本。

已知存在问题的 libomp 版本包括 12.x.x 和 13.x.x。XGBoost issue #7039 中提供了一种解决方法：通过终端安装旧版 libomp 11。先在终端运行 `brew list --version libomp`，查看当前安装的版本（如果有）；若版本存在问题，则运行 `brew unlink libomp`。

运行以下命令安装旧版 libomp：

```bash
brew update
wget https://raw.githubusercontent.com/chenrui333/homebrew-core/0094d1513ce9e2e85e07443b8b5930ad298aad91/Formula/libomp.rb
brew install --build-from-source ./libomp.rb
```

然后运行：

```bash
brew list --version libomp
```

确认安装成功。输出应为 `libomp 11.1.0`，之后代码就应该可以正常运行。
