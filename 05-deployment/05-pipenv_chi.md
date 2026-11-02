---
video_url: https://www.youtube.com/watch?v=BMXh8JGROHM&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR
code:
  - label: Pipfile
    path: code/Pipfile
prev_url: 04-flask-deployment_chi.md
next_url: 06-docker_chi.md
---
# Python 虚拟环境：Pipenv

本节使用 Pipenv 将项目依赖安装到隔离的虚拟环境中，确保服务始终使用经过测试的库版本。

## 问题：库版本冲突

使用 `pip install` 安装的 Python 包通常会进入系统级 Python 的共享目录。当两个项目需要同一个库的不同版本时，就会出现问题。例如，旧项目需要 scikit-learn 0.24，而新项目需要最新的 1.0；升级其中一个项目的版本可能会导致另一个项目无法运行。

![两个服务共用系统 Python：为一个服务升级 scikit-learn 会导致另一个服务出错](images/05-pipenv-02-version-conflict-imagegen.jpg)

解决办法是使用虚拟环境：为每个项目创建一套独立的包。每个项目使用自己的库版本，不会与其他项目或系统级 Python 相互干扰。

![每个服务都有自己的虚拟环境，不会发生依赖冲突](images/05-pipenv-03-isolated-environments-imagegen.jpg)

可用于创建环境的工具有很多，例如 `venv`、`conda` 和 `poetry`。本课程使用 Pipenv，因为它不仅能创建环境，还能记录准确的包版本，便于稍后在服务器上复现环境。

## 使用 Pipenv 安装库

先在系统范围内安装一次 Pipenv：

```bash
pip install pipenv
```

然后在项目目录中添加依赖。我们的 churn 服务需要 numpy、scikit-learn、flask 和 gunicorn：

```bash
pipenv install numpy scikit-learn flask gunicorn
```

此命令会创建虚拟环境并在其中安装这些包，同时在项目目录下创建两个文件：

- `Pipfile`：记录项目依赖，以及我们指定的版本。示例：

  ```toml
  [[source]]
  url = "https://pypi.org/simple"
  verify_ssl = true
  name = "pypi"

  [packages]
  numpy = "*"
  scikit-learn = "==0.24.2"
  flask = "*"
  gunicorn = "*"

  [dev-packages]

  [requires]
  python_version = "3.8"
  ```

- `Pipfile.lock`：记录每个已安装包的确切版本，包括依赖项的依赖项，以及对应的哈希值。有了 lock 文件，就能在其他机器上逐项复现相同环境。

Lock 文件让部署更可预测：服务器会安装我们实际测试过的依赖，不再因为“在我的机器上能运行”而产生不确定性。

## 使用 Pipenv 运行程序

在另一台机器上，或重新克隆项目后，只需运行一条命令即可根据这两个文件重建环境：

```bash
pipenv install
```

进入虚拟环境：

```bash
pipenv shell
```

在这个 shell 中，`python` 和其他命令都会使用虚拟环境中的包，因此可以像往常一样启动服务：

```bash
gunicorn --bind localhost:9696 predict:app
```

输入 `exit` 可以退出环境。如果不想先激活 shell，也可以直接通过 Pipenv 运行命令：

```bash
pipenv run gunicorn --bind localhost:9696 predict:app
```

现在，服务已经使用固定且可复现的依赖运行。不过 Python 本身以及 Python 包之外的内容仍与当前机器绑定。[下一节](06-docker_chi.md)将使用 Docker 把整个环境（包括 Python 版本）放进容器中。

## 笔记

本节介绍如何为项目创建虚拟环境。

- 直接从目录运行文件时，使用的是系统级可执行文件和全局安装的库。安装 Python 后，可执行文件通常位于类似 `/home/username/python/bin/` 的目录中，`pip` 也可能位于 `/home/username/python/bin/pip`。
- 库版本冲突可能导致项目无法运行或产生大量错误。例如，旧项目使用 sklearn 0.24.1，而另一个项目需要 sklearn 1.0.0，升级版本后旧项目可能出现问题。
- 可以通过虚拟环境隔离系统库和项目所需的特定版本。本课程使用 Pipenv：
  - 使用 `pip install pipenv` 安装。
  - 使用 `pipenv install numpy scikit-learn==0.24.1 flask` 安装项目所需依赖。
  - 执行后会生成 `Pipfile` 和 `Pipfile.lock`。前者列出依赖，后者记录确切版本和哈希信息，便于在另一台机器上复现。
  - 在新机器上运行 `pipenv install` 可依据这两个文件安装指定版本。
  - 运行 `pipenv shell` 进入虚拟环境，随后可使用 `gunicorn --bind localhost:9696 predict:app` 启动 Web 服务；也可以直接运行 `pipenv run gunicorn --bind localhost:9696 predict:app`。
- 本节创建了依赖版本固定的虚拟环境。下一节将介绍 Docker，以进一步隔离系统环境，并让服务更容易在不同机器上运行。