---
video_url: https://www.youtube.com/watch?v=HGPJ4ekhcLg&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR
prev_url: 06-docker_chi.md
next_url: 08-summary_chi.md
---
# 部署到云端：AWS Elastic Beanstalk（可选）

本节为可选内容：我们将 churn 服务的 Docker image 部署到 AWS Elastic Beanstalk，使它可以从互联网访问。

## 为什么需要云端

目前服务只运行在自己的机器上。营销服务若要使用它，模型必须运行在双方都能访问的位置，也就是云端服务器。云服务商可以按需提供虚拟机；AWS 是最常见的云服务商。跟随本节操作需要 AWS 账号，详情请参见 [AWS 账号创建说明](https://mlbookcamp.com/article/aws)。

Elastic Beanstalk（EB）是 AWS 提供的简化部署服务：我们提供应用（这里是 Docker image），其余工作由 EB 处理。它会启动实例、保持应用运行并根据流量扩缩容：请求增多时启动更多容器实例，流量下降时再缩减实例。

整体架构与[第一节](01-intro_chi.md)中的场景相同：营销服务向 EB 环境发送请求；EB 将请求转发给运行 Flask 应用的 Docker 容器；容器返回 churn 概率，EB 再将结果传回。

![营销服务通过 Elastic Beanstalk 环境调用 churn 服务](images/07-aws-eb-02-eb-architecture-imagegen.jpg)

## 分步部署

首先安装 EB 命令行工具。它作为开发依赖安装，只需在本机上创建和管理环境；容器本身不需要它：

```bash
pipenv install awsebcli --dev
```

初始化 EB 应用。该命令会询问区域和平台，并在 `.elasticbeanstalk/config.yml` 中创建配置：

```bash
eb init -p docker -r eu-north-1 churn-serving
```

- `-p docker`：指定 Docker 平台，EB 将使用我们的 Dockerfile。
- `-r eu-north-1`：指定 AWS 区域，请选择离用户较近的区域。
- `churn-serving`：应用名称。

可以使用 `less .elasticbeanstalk/config.yml` 查看生成的配置。在部署到云端之前，可以先在本地检查应用是否正常；EB 会构建 Docker image 并运行它：

```bash
eb local run --port 9696
```

可以像[前一节](04-flask-deployment_chi.md)一样使用测试脚本。确认运行正常后，在 AWS 中创建正式环境：

```bash
eb create churn-serving-env
```

这需要几分钟：EB 会启动实例、部署容器，并为环境创建 URL。该 URL 就是预测请求的 endpoint；更新测试脚本中的主机地址即可访问。无需指定端口：EB 接收标准 HTTP 端口上的请求，并将请求路由到容器。

注意两点：环境 URL 是公开的，任何知道该地址的人都可以调用服务，因此真实生产部署需要身份验证或网络级访问控制。完成实验后请删除环境，否则 AWS 会继续对实例收费：

```bash
eb terminate churn-serving-env
```

## 资料

- [幻灯片](https://www.slideshare.net/AlexeyGrigorev/ml-zoomcamp-5-model-deployment)
- [创建 AWS 账号](https://mlbookcamp.com/article/aws)

## 笔记

### 使用 AWS Elastic Beanstalk 部署到云端

AWS 是最常见的云服务商之一。Elastic Beanstalk 提供了简单的方式来部署应用，适合部署打包在 Docker 容器中的应用，例如 churn 预测服务。

本节介绍如何使用 Docker 将机器学习模型容器化，再通过 AWS Elastic Beanstalk 部署到云端。

#### 架构概览

1. 将 churn 预测服务打包到 Docker 容器中。
2. 将容器部署到 AWS Elastic Beanstalk。
3. 营销服务向 Elastic Beanstalk 环境发送请求。
4. Elastic Beanstalk 将请求转发到 Docker 容器。
5. 容器处理请求并将响应发送回 Elastic Beanstalk。
6. Elastic Beanstalk 将响应返回给请求方。

#### Elastic Beanstalk 的扩缩容

Elastic Beanstalk 会根据流量自动扩缩容。当 churn 预测服务收到大量请求时，EB 会自动添加更多实例来处理负载；流量下降时则会缩减实例，以优化资源使用。

#### 部署步骤

1. **安装 AWS EB CLI：**

   ```bash
   pipenv install awsebcli --dev
   ```

   说明：EB CLI 作为开发依赖安装，容器内部不需要它。

2. **初始化 EB 环境：**

   ```bash
   eb init -p docker -r eu-north-1 churn-serving
   ```

   - `-p docker`：指定 Docker 平台。
   - `-r eu-north-1`：设置 AWS 区域。可根据账号和用户所在位置选择其他区域。
   - `churn-serving`：应用名称。

3. **检查 EB 配置：**

   ```bash
   less .elasticbeanstalk/config.yml
   ```

   查看生成的配置文件。

4. **本地测试：**

   ```bash
   eb local run --port 9696
   ```

   使用指定端口在本地启动应用，也可以使用 `predict.py` 测试服务。

5. **部署到 AWS：**

   ```bash
   eb create churn-serving-env
   ```

   创建 Elastic Beanstalk 环境并部署应用。

6. **访问已部署的服务：**

   部署成功后，Elastic Beanstalk 会提供服务 endpoint URL。将 `predict-test` 脚本中的主机地址更新为该 URL。访问部署后的服务时不再需要端口号。

7. **安全注意事项：**

   已部署的服务可公开访问。生产环境中必须采取适当的安全措施，将访问限制为经过授权的用户或网络。

8. **终止服务：**

   ```bash
   eb terminate churn-serving-env
   ```

   该命令会终止 Elastic Beanstalk 环境并移除相关资源。

另外，运行 EB CLI 命令前请先配置 AWS 凭据。

### 使用 Heroku 部署

我们已经了解如何在 AWS 部署应用，下面简要介绍另一种方式：Heroku。

- 首先创建 Flask Web 服务，例如 [churn_prediction.py](https://github.com/amindadgar/customer-churn-app/blob/main/churn_serving.py)。
- 创建 `requirements.txt` 并列出依赖，例如：

  ```text
  pickle
  numpy
  flask
  gunicorn
  ```

- 再创建 `Procfile`，写入要运行的应用，例如：

  ```text
  web: gunicorn churn_serving:app
  ```

  这里的 `churn_serving` 是包含应用代码的 Python 文件名。
- 创建 Heroku 账号，进入 dashboard 的 Deploy 页面。
- 按照说明通过 Heroku Git 部署。
- 完成后，应用即可通过互联网访问。

示例应用文件位于此仓库：https://github.com/amindadgar/customer-churn-app