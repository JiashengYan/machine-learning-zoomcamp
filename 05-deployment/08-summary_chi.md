---
video_url: https://www.youtube.com/watch?v=sSAqYSk7Br4&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR
prev_url: 07-aws-eb_chi.md
next_url: 09-explore-more_chi.md
---
# 总结

这是本模块的最后一节。我们将 notebook 中的 churn 预测模型部署成了其他系统可以调用的 Web 服务。

## 本模块内容回顾

我们从一个只存在于 notebook 中的模型开始，逐步移除了它对本地电脑的依赖：

- 使用 pickle 保存和加载模型，使其无需重新训练即可使用，并将 notebook 转换为 Python 脚本。
- 学习 Web 服务如何通过 HTTP 通信，并使用 Flask 将模型变成一个服务。`/predict` endpoint 接收 JSON 格式的客户数据并返回 churn 概率。
- 使用 gunicorn 替代 Flask 开发服务器；Windows 用户可以使用 waitress。
- 使用 Pipenv 管理依赖和环境，为项目创建隔离环境并锁定准确的库版本。
- 使用 Docker 管理环境，将服务、Python 及所有依赖打包成 image，使其在不同机器上以相同方式运行。
- 使用 AWS Elastic Beanstalk 将容器部署到云端，让服务可以通过互联网访问。

整个模块可以概括为：

- 使用 pickle 保存模型。
- 使用 Flask 将模型封装为 Web 服务。
- 使用 Pipenv 管理依赖和环境。
- 使用 Docker 打包服务。
- 使用 AWS Elastic Beanstalk 部署到云端。

[下一节](09-explore-more_chi.md)没有视频，列出了一些可以自行尝试的工具。之后是作业，你将亲自部署一个模型。

下一模块将学习基于树的模型，包括决策树、随机森林和梯度提升，它们是构建预测模型的另一种方法。

## 笔记

本章学习了以下内容：

- 保存和加载模型，以便后续重复使用，而不必重新运行之前的代码。
- 将模型部署为 Web 服务。
- 创建虚拟环境。
- 创建容器，使代码能在支持相应容器技术的不同操作系统上运行。
- 将代码部署到公开的 Web 服务，使其可以从本地计算机之外访问。

下一章将学习决策树、随机森林和梯度提升等算法，作为组合决策树的另一种方式。