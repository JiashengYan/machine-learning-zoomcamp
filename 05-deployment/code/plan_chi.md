# 5. 部署机器学习模型

我们将使用之前训练并评估过的 churn 预测模型，并把它部署为 Web 服务。

## 5.1 简介 / 本节概览

- 本周要学习的内容

## 5.2 保存和加载模型

- 使用 pickle 保存模型
- 使用 pickle 加载模型
- 将 notebook 转换为 Python 脚本

## 5.3 Web 服务：Flask 简介

- 编写简单的 ping/pong 应用
- 使用 `curl` 和浏览器查询服务

## 5.4 使用 Flask 提供 churn 模型服务

- 将预测脚本封装为 Flask 应用
- 使用 `requests` 发送请求
- 为生产环境做准备：gunicorn
- 在 Windows 上使用 waitress 运行

## 5.5 Python 虚拟环境：Pipenv

- 管理依赖和环境
- 了解为什么需要虚拟环境
- 安装 Pipenv
- 使用 Pipenv 安装库
- 使用 Pipenv 运行程序

## 5.6 环境管理：Docker

- 了解为什么需要 Docker
- 使用 Docker 运行 Python image
- 编写 Dockerfile
- 构建 Docker image
- 运行 Docker image

## 5.7 部署到云端：AWS Elastic Beanstalk（可选）

- 安装 EB CLI
- 在本地运行 EB
- 部署模型

## 5.8 总结

- 使用 pickle 保存模型
- 使用 Flask 将模型变成 Web 服务
- 使用依赖和环境管理工具
- 使用 Docker 打包
- 部署到云端

## 5.9 继续探索

- Flask 不是创建 Web 服务的唯一框架，也可以尝试 FastAPI。
- 尝试其他环境管理工具，例如 virtualenv、conda、poetry。
- 探索其他 Web 服务部署平台，例如 GCP、Azure、Heroku、PythonAnywhere 等。