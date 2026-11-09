---
prev_url: 10-summary_chi.md
next_url: ../08-deep-learning/01-fashion-classification.md
---

# 进一步探索

![数据集可以分支到 EDA、特征重要性、随机森林和梯度提升等方向，帮助我们发现更多洞见。](images/11-explore-more-01-ensemble-experiments-imagegen.jpg)

*图：尝试多种分析方法和集成模型，从同一份数据中获得更多认识。*

- 本项目没有进行 EDA 或特征工程。可以尝试这些方法，从数据中获得更多洞见。
- 随机森林还有其他可以调整的参数。可以查看 `max_features` 和 `bootstrap`。
- 随机森林有一种变体叫极端随机树（extra trees）。它不会在所有可能的阈值中选择最佳分裂，而是随机选取几个阈值，再从中挑选最佳者。因此，极端随机树不会过拟合。在 Scikit-Learn 中，它们由 `ExtraTreesClassifier` 实现。可以尝试用于本项目。
- XGBoost 可以处理 NA 值，因此不一定需要对数据执行 `fillna`。可以试试不填充 NA 是否能改善表现。
- 尝试调整其他 XGBoost 参数：`subsample` 和 `colsample_bytree`。
- 决策树选择最佳分裂时会找到最有用的特征。我们可以利用这一信息了解哪些特征比其他特征更重要。可查看随机森林和 XGBoost 的特征重要性示例（决策树也适用）。
- 树也可以解决回归问题：可以尝试 `DecisionTreeRegressor`、`RandomForestRegressor`，以及 XGBoost 的 `objective=reg:squarederror` 参数。

## 笔记

欢迎通过 Pull Request 补充视频笔记。
