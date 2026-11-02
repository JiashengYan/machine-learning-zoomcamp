---
video_url: https://www.youtube.com/watch?v=wAtyYZ6zvAs&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR
code:
  - label: Dockerfile
    path: code/Dockerfile
prev_url: 05-pipenv_chi.md
next_url: 07-aws-eb_chi.md
---
# 环境管理：Docker

本节将 churn 服务打包到 Docker 容器中，使其无论运行在哪里，都使用相同的 Python 版本和依赖，并以相同方式运行，不受宿主机差异影响。

## 安装 Docker

Docker 可运行于 Linux、Windows 和 macOS。安装方式如下：

### Ubuntu

```bash
sudo apt-get install docker.io
```

如果希望不使用 `sudo` 运行 Docker，请参考[此说明](https://docs.docker.com/engine/install/linux-postinstall/)。

### Windows

参考 Andrew Lock 的[安装说明](https://andrewlock.net/installing-docker-desktop-for-windows/)。

如果使用子系统，并且首次运行 Docker 时集成失败，请检查资源设置，确认已启用对应的发行版。

### macOS

按照 [Docker 文档](https://docs.docker.com/desktop/install/mac-install/)中的步骤安装。

## 为什么使用 Docker

Pipenv 可以隔离项目的 Python 包，但操作系统、Python 版本和系统库仍与其他项目共享。在 Python 3.10 的笔记本上运行正常的服务，可能无法在 Python 3.6 的服务器上运行。

Docker 将应用及其完整运行环境（操作系统、Python、系统依赖、Python 依赖、代码和模型文件）打包成 image。基于这个 image 启动的容器在任何安装了 Docker 的机器上都以相同方式运行。

![每个容器都携带自己的操作系统和 Python 版本，实现完整隔离](images/06-docker-02-containers-on-host-imagegen.jpg)

## Dockerfile

Docker image 根据 Dockerfile 构建。Dockerfile 是一份构建配方：从基础 image 开始，并描述后续添加内容的每一步。以下是 churn 服务的 Dockerfile：[code/Dockerfile](code/Dockerfile)：

```dockerfile
FROM python:3.8.12-slim

RUN pip install pipenv

WORKDIR /app

COPY ["Pipfile", "Pipfile.lock", "./"]

RUN pipenv install --system --deploy

COPY ["predict.py", "model_C=1.0.bin", "./"]

EXPOSE 9696

ENTRYPOINT ["gunicorn", "--bind=0.0.0.0:9696", "predict:app"]
```

逐条说明：

- `FROM python:3.8.12-slim`：基础 image，基于精简版 Debian，包含 Python 3.8.12。`slim` 版本体积更小，因此构建和下载更快。
- `RUN pip install pipenv`：在 image 中安装 Pipenv，以便安装项目依赖。
- `WORKDIR /app`：创建 `/app` 目录，并将其设为后续指令的工作目录。
- `COPY ["Pipfile", "Pipfile.lock", "./"]`：将两个依赖文件复制到 image 中。
- `RUN pipenv install --system --deploy`：严格安装 lock 文件中指定的依赖。`--system` 将包安装到 image 的系统 Python 中，而不是再创建一个虚拟环境（容器本身已经是隔离环境）；`--deploy` 会在 lock 文件不是最新状态时使 Pipenv 失败退出。
- `COPY ["predict.py", "model_C=1.0.bin", "./"]`：将服务代码和模型文件复制到 image 中。
- `EXPOSE 9696`：声明容器监听 9696 端口。容器网络是隔离的，启动容器时仍需发布该端口。
- `ENTRYPOINT ["gunicorn", "--bind=0.0.0.0:9696", "predict:app"]`：容器启动时执行的命令，也就是使用生产服务器提供 Flask 应用。如果没有 ENTRYPOINT，容器会启动 Python shell 后退出。

注意，ENTRYPOINT 使用双引号和 exec 格式的 JSON 数组。这种写法让 Docker 直接执行命令，而不通过 shell 包装。

## 构建并运行

从 Dockerfile 构建 image：

```bash
docker build -t churn-prediction .
```

`-t` 为 image 指定名称（tag）：`churn-prediction`。然后基于它启动容器：

```bash
docker run -it -p 9696:9696 churn-prediction:latest
```

参数说明：

- `-it`：让终端连接到容器，以便查看输出并通过 Ctrl-C 停止容器。
- `-p 9696:9696`：将容器的 9696 端口发布为宿主机上的 9696 端口。冒号前是宿主机端口，冒号后是容器端口。

![端口发布：测试脚本访问宿主机的 9696 端口，该端口映射到容器内部](images/06-docker-05-port-mapping-imagegen.jpg)

现在，[上一节](04-flask-deployment_chi.md)中的测试脚本会访问容器化服务。URL 和响应都相同，但服务运行所需的内容都来自 Docker image。在[下一节](07-aws-eb_chi.md)中，我们会将容器部署到云端。

## 笔记

- 将项目打包进 Docker 容器后，就可以在支持 Docker 的机器上运行。
- 首先需要创建 Docker image，其中包含项目所需的设置和依赖。可以在 [Docker Hub](https://hub.docker.com/search?type=image) 上查找需要的 image。

下面是一个 Dockerfile 示例（复制时请移除注释）：

```dockerfile
FROM python:3.8.12-slim

RUN pip install pipenv

WORKDIR /app

COPY ["Pipfile", "Pipfile.lock", "./"]

RUN pipenv install --deploy --system

COPY ["*.py", "churn-model.bin", "./"]

EXPOSE 9696

ENTRYPOINT ["gunicorn", "--bind=0.0.0.0:9696", "churn_serving:app"]
```

`--deploy` 和 `--system` 会将依赖直接安装到 Docker 容器中，而不额外创建 Pipenv 默认会创建的虚拟环境。

如果省略最后的 `ENTRYPOINT`，容器会进入 Python shell。注意，entrypoint 命令使用双引号。

创建 Dockerfile 后，构建 image：

```bash
docker build -t churn-prediction .
```

运行 image：

```bash
docker run -it -p 9696:9696 churn-prediction:latest
```

参数说明：

- `-t`：用于指定 tag 名称 `churn-prediction`。
- `-it`：允许我们连接容器的终端。
- `--rm`：容器结束后自动从系统中移除。
- `-p`：将 Docker 容器的 9696 端口映射到本机的 9696 端口（前者是本机端口，后者是容器端口）。
- `--entrypoint=bash`：启动容器后进入 bash，以便像使用终端一样与容器交互。默认 entrypoint 是 `python`。

到这里，你已经将预测应用部署在 Docker 容器中。恭喜！