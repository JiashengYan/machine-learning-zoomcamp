---
video_url: https://www.youtube.com/watch?v=EJpqZ7OlwFU&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR
code:
  - label: 训练脚本
    path: code/train.py
prev_url: 01-intro_chi.md
next_url: 03-flask-intro_chi.md
---
# 保存和加载模型

本节使用 pickle 将训练好的 churn 模型保存到文件中，再重新加载，并将 notebook 转换为 Python 脚本。这样之后就可以使用模型，而不必再次训练。

## 为什么要保存模型

训练过程会产生两个对象：将客户记录转换为特征矩阵的 `DictVectorizer`，以及逻辑回归模型本身。它们都只存在于 notebook 进程的内存中；关闭 notebook 后，这些对象就消失了。

每次预测都重新训练模型既浪费时间，也可能因数据划分和随机状态不同而得到略有差异的模型。更好的做法是训练一次，将结果保存到文件中，然后由 Web 服务加载该文件来进行预测。

## 使用 pickle 保存

Pickle 是 Python 标准库中的对象序列化工具：它可以将 Python 对象写入二进制文件，之后再读取回来。我们将 vectorizer 和模型一起保存为一个 tuple：

```python
import pickle

output_file = f'model_C={C}.bin'

with open(output_file, 'wb') as f_out:
    pickle.dump((dv, model), f_out)
```

`'wb'` 表示以二进制写入模式打开文件。Pickle 生成的是二进制数据，因此不能用文本模式打开。文件名包含所使用的 `C` 值；经过上一模块的调参后，我们知道这个参数，因此把它记录在文件名中也很有用，例如 `model_C=1.0.bin`。

Vectorizer 必须和模型一起保存。它是在训练数据上拟合的，并记住了哪些类别取值对应哪些特征列；没有它，就无法用相同方式准备新的客户记录。

## 加载模型

例如，在 Web 服务中使用模型时，可以从文件中加载：

```python
import pickle

with open('model_C=1.0.bin', 'rb') as f_in:
    dv, model = pickle.load(f_in)
```

这里的 `'rb'` 表示以二进制读取模式打开文件。`pickle.load` 返回保存的 tuple，再将其中内容解包到 `dv` 和 `model`。从此以后，就可以直接为客户打分，而不必运行任何训练代码。

注意：不要反序列化来源不可信的文件。Pickle 文件可能包含代码，加载时会执行这些代码；只加载自己创建的文件，或来自可信来源的文件。

## 从 notebook 转为脚本

最后一步是把训练代码从 notebook 移到普通 Python 脚本中。模块 [code/](code/) 目录里的 [train.py](code/train.py) 包含了我们在 notebook 中开发的相同步骤：

- 加载并清理 churn 数据集；
- 将数据划分为训练集和测试集；
- 定义 `train` 和 `predict` 函数；
- 使用 `KFold` 进行交叉验证，并报告 `C=1.0` 时的 AUC；
- 训练最终模型并用 pickle 保存。

脚本才是我们实际会在服务器上运行的形式：它具有可复现性，可以由调度程序或 CI 作业执行，也不需要启动 Jupyter。

在[下一节](03-flask-intro_chi.md)中，我们会使用 Flask 创建第一个 Web 服务。

## 笔记

本节介绍如何在未来使用模型，而不必重新运行训练和评估代码。

- 可以使用 pickle 库保存之前训练的模型：
  - 训练模型并完成预测准备后，将它保存起来以便以后使用：

    ```python
    import pickle

    with open('model.bin', 'wb') as f_out:
        pickle.dump((dict_vectorizer, model), f_out)
    ```

  - 上面的代码会创建名为 `model.bin` 的二进制文件，并将 one-hot 编码用的 `dict_vectorizer` 和模型保存在其中。使用二进制格式是为了保存对象，而不是生成供人直接阅读的文本。
  - 之后可以读取该二进制文件：

    ```python
    import pickle

    with open('model.bin', 'rb') as f_in:
        dict_vectorizer, model = pickle.load(f_in)
    ```

  - 注意：不要打开来源不可信的二进制文件。
  - 解包模型和 `dict_vectorizer` 后，就可以为新输入进行预测，而不必重新训练模型。