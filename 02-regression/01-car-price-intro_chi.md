---
video_url: "https://www.youtube.com/watch?v=vM3SqPNlStE&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR&index=12"
prev_url: ../01-intro/10-summary.md
next_url: 02-data-preparation_chi.md
---
# 汽车价格预测项目

第二部分是一个实践项目：构建预测汽车价格的模型。本节首先介绍应用场景、所用数据集、项目计划，以及代码和数据的位置。

## 应用场景

回顾导论中的场景：用户想在在线分类信息网站上出售汽车，网站要求输入价格。如果用户不知道汽车值多少钱，就只能猜。我们希望提供帮助：用户描述汽车，我们的模型为其推荐合适的价格。

![如何帮助用户选择合适的价格](images/01-car-price-intro-01-select-best-price-crisp.jpg)

## 数据集

训练模型需要数据。我们使用 Kaggle 上的汽车价格数据集，其中包含汽车的各种特征，例如品牌（制造商）、车型、年份、发动机类型、燃料类型及其他属性。

![Kaggle 上的汽车特征与 MSRP 数据集](images/01-car-price-intro-02-kaggle-dataset-crisp.jpg)

Kaggle 的数据浏览器展示了数据集的内容。每一行代表一辆汽车，每一列代表这辆汽车的一项特征。

![Kaggle 数据浏览器中的汽车数据列](images/01-car-price-intro-03-kaggle-data-explorer-crisp.jpg)

我们特别关注 MSRP 这一列。MSRP 是 Manufacturer Suggested Retail Price 的缩写，意为制造商建议零售价，也就是汽车的价格。这正是我们要预测的目标。我们将使用品牌、车型、年份、发动机等其他特征来预测这个价格。

![MSRP 列包含每辆汽车的价格](images/01-car-price-intro-04-msrp-column-crisp.jpg)

## 项目计划

我们将分几步完成项目：

- 获取数据并进行探索性数据分析：观察数据，尝试加深对数据的了解。
- 准备数据集，在其上训练线性回归模型来预测汽车价格。
- 深入了解线性回归的实现细节，并亲自实现它。
- 使用 RMSE 评估模型质量。RMSE 指均方根误差，是衡量模型预测质量的指标。
- 进行特征工程：创建模型可以使用的新特征或新属性。
- 处理数值稳定性问题，了解如何通过正则化解决这些问题。
- 最后，使用模型。


## 代码的位置

本项目的所有代码都位于 GitHub 的 [mlbookcamp-code](https://github.com/alexeygrigorev/mlbookcamp-code) 仓库中，这是《Machine Learning Bookcamp》一书的代码仓库。我们需要其中的 chapter-02-car-price 文件夹。

![mlbookcamp-code 仓库中的 chapter-02-car-price 文件夹](images/01-car-price-intro-06-github-repo-crisp.jpg)

该文件夹中有两个文件。第一个是包含所有代码的 notebook，涵盖本部分的全部操作。第二个是 data.csv，也就是用于训练模型的数据集。本期课程将逐节讲解这个 notebook；本模块对应的文件是 [notebook.ipynb](notebook.ipynb)。

![chapter-02-car-price 中的 notebook 和数据文件](images/01-car-price-intro-07-chapter-files-crisp.jpg)

接下来，我们将读取 CSV 文件并做一些数据准备，这就是下一节的内容：[数据准备](02-data-preparation_chi.md)。

## 笔记

本项目旨在创建帮助用户预测汽车价格的模型。数据集来自[这个 Kaggle 竞赛](https://www.kaggle.com/CooperUnion/cardataset)。

**项目计划：**

* 数据准备与探索性数据分析（EDA）
* 使用线性回归预测价格
* 理解线性回归的内部原理
* 使用 RMSE 评估模型
* 特征工程
* 正则化
* 使用模型

代码和数据集可通过[此链接](https://github.com/alexeygrigorev/mlbookcamp-code/tree/master/chapter-02-car-price)获取。
