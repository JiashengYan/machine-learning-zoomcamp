---
video_url: https://www.youtube.com/watch?v=JZ6sRZ_5j_c&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR
prev_url: 09-final-model_chi.md
next_url: 11-explore-more_chi.md
---
# 总结

本模块的最后一节将回顾决策树与集成学习的内容。

## 本模块的学习内容

本模块从信贷风险评分项目开始：预测客户是否会拖欠贷款。在此过程中，我们学习了四个主要内容。

![本模块的四个要点：决策树、最佳分裂、随机森林和梯度提升](images/10-summary-01-summary-slide-imagegen.jpg)

决策树根据数据学习 if-then-else 规则。树会依次提出问题，例如“资产是否超过 3000”，并在叶节点中给出预测。为了构建树，学习算法会寻找最佳分裂，也就是能让左右两侧群组不纯度最低的条件。如果不加限制，算法会不断生长，直到记住训练数据。因此，我们通过限制最大深度和群组最小规模来控制它：在 scikit-learn 中对应 `max_depth` 和 `min_samples_leaf`。

![决策树根据 records、seniority 和 job 提出问题](images/10-summary-02-decision-tree-imagegen.jpg)

![不受限制的树在训练集上得分为 1.0，在验证集上仅为 0.654](images/10-summary-03-overfitting-auc-imagegen.png)

随机森林通过组合多棵决策树来进行预测：每棵树使用随机抽取的特征子集，所有树彼此独立、并行训练，最后对它们的预测取平均值。要让集成方法有效，各个模型需要具有多样性——一棵树犯下的错误应能被其他树纠正。我们调整了树的数量、深度和叶节点大小，显著提升了单棵树的表现，而且所需的调参工作不多。

![多棵彼此不同的决策树共同给出随机森林的平均预测](images/10-summary-04-random-forest-imagegen.jpg)

梯度提升则按顺序训练模型：每个后续模型都会根据前面模型的错误进行训练，并尝试修正这些错误。XGBoost 是这种方法最流行的实现。本模块中表现最佳的模型就是 XGBoost，但它也有最多的参数需要调整，例如 `eta`、`max_depth`、`min_child_weight` 等，因此需要最仔细的调参。

![梯度提升按顺序训练模型，每个模型都会修正先前模型的错误](images/10-summary-05-gradient-boosting-imagegen.jpg)

![XGBoost 参数 eta、max_depth 和 min_child_weight](images/10-summary-06-xgb-parameters-imagegen.jpg)

最终的 XGBoost 模型在验证集上的 AUC 约为 0.836，在测试集上约为 0.832；单棵决策树的 AUC 约为 0.785。课程将在第 8 模块继续介绍神经网络，并训练一个图像分类模型。

## 笔记

- 决策树从数据中学习 if-then-else 规则。
- 寻找最佳分裂时，选择不纯度最低的分裂。算法容易过拟合，因此需要限制树的最大深度和群组大小。
- 随机森林通过组合多棵决策树来预测。若要得到好的预测，各个模型应具有多样性。
- 梯度提升按顺序训练模型，每个模型都尝试修正前一个模型的错误。XGBoost 是梯度提升的一种实现。
