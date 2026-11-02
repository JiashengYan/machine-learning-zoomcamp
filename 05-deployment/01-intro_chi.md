---
video_url: https://www.youtube.com/watch?v=agIFak9A3m8&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR
prev_url: ../04-evaluation/09-explore-more.md
next_url: 02-pickle_chi.md
---
# 简介 / 本节概览

本模块将把之前训练的 churn 预测模型改造成一个服务，让其他系统可以通过网络调用它。

## 问题：模型目前只存在于 notebook 中

在模块 3 中，我们训练了一个模型，用来预测电信公司的客户是否会流失，也就是是否会停止使用服务。只有当人们真正能够使用它时，模型才有价值。目前模型只存在于 notebook 中：要获得预测结果，我们必须打开 notebook、运行所有单元，再查看输出。这对我们自己来说没问题，但公司里的其他系统无法这样工作。

想象一下，营销团队希望向可能流失的客户发送促销邮件。他们的系统需要自动询问模型：“这个客户流失的概率是多少？”并且要能处理数千名客户，而不需要任何人打开 notebook。

解决办法是部署模型：在服务器上运行模型，并公开一个 API endpoint。其他服务（例如营销服务）向这个 endpoint 发送客户信息，服务器在响应中返回预测结果。营销服务随后可以根据响应决定是否发送促销邮件。

![营销服务向 churn 服务请求预测，churn 服务使用模型完成预测](images/01-intro-02-model-deployment-diagram-imagegen.jpg)

## 本模块计划

我们会逐步把 notebook 中的 churn 模型变成可运行的 Web 服务：

- 使用 pickle 保存和加载模型，并将 notebook 转换为 Python 脚本，使模型无需每次重新训练即可使用。
- 使用 Flask 创建 Web 服务：先实现简单的 ping/pong 服务，再实现真正处理预测请求的 churn 服务。
- 使用 gunicorn（Windows 上使用 waitress）替代 Flask 开发服务器，为生产环境做好准备。
- 使用 Pipenv 管理项目依赖，确保服务使用经过测试的库版本。
- 将所有内容打包到 Docker 容器中，让它在不同机器上以相同方式运行。
- 最后，可选地使用 AWS Elastic Beanstalk 将容器部署到云端，使服务可通过互联网访问。

每一步都在消除模型对本地电脑的依赖：notebook、开发服务器、系统级 Python 包、操作系统，最后是本地机器本身。

![Web 服务封装模型，Pipenv 管理 Python 依赖，Docker 封装整个环境](images/01-intro-05-environments-imagegen.jpg)

## 笔记

本节回顾了第 3 章中构建的 churn 预测模型，并介绍了模型部署。若希望不必重新运行 notebook 就能对新数据进行预测，可以将模型部署到服务器上。服务器运行模型并提供 API endpoints，其他机器就可以连接服务器并请求预测结果。

当模型需要被不同机器或应用程序使用时，部署非常重要。将模型部署为 Web 服务后，外部系统（例如营销服务）可以向服务器发送请求，获取客户是否可能流失等预测，并自动触发后续操作，例如发送促销优惠。

部署到服务器通常包含以下步骤：

1. **训练并保存模型**：训练模型后将其保存为文件，以便之后进行预测（第 02 节）。
2. **创建 API endpoints**：使用 Flask 创建 API endpoints，供其他服务请求预测（第 03 和第 04 节）。
3. **管理服务运行环境**（第 5 至第 9 节）：
   - **Pipenv**：创建隔离环境并管理 Web 服务的 Python 依赖，避免它们与机器上的其他服务冲突。
   - **Docker**：将服务及其系统依赖、Python 依赖打包到容器中，从而在不同环境中保持一致。
4. **部署到云端**：最后，将 Docker 容器部署到 AWS 等云服务，让模型可以被全球访问，并获得可扩展性和可靠性。