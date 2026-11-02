---
video_url: https://www.youtube.com/watch?v=W7ubna1Rfv8&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR
code:
  - label: Ping 应用
    path: code/ping.py
prev_url: 02-pickle_chi.md
next_url: 04-flask-deployment_chi.md
---
# Web 服务：Flask 简介

本节介绍 Web 服务，并编写第一个 Web 服务：一个接收 ping 请求并返回 “PONG” 的小型 Flask 应用。

## 什么是 Web 服务？

Web 服务是一种让应用通过 HTTP 相互通信的方式。HTTP 也是浏览器加载网站时使用的协议。一个应用（客户端）向某个地址（URL）发送请求，另一个应用（服务器）则返回响应。

![客户端向 Web 服务发送请求并收到响应](images/03-flask-intro-02-request-response-imagegen.jpg)

客户端不仅指定请求发送到哪里，还会指定要执行什么操作，这由 HTTP 方法表示。最常见的方法包括：

- GET：获取内容。在浏览器中打开页面会发送 GET 请求；在 Google 搜索猫咪图片也是 GET 请求。
- POST：向服务器发送数据，例如提交包含姓名和邮箱的注册表单，或发送需要评分的客户记录。
- PUT：与 POST 类似，但请求会指定数据应放置的位置。
- DELETE：从服务器删除内容。

在模型部署中，我们主要使用两种方法：用 GET 请求健康检查 endpoint，用 POST 向模型发送客户数据。

## 使用 Flask 创建 ping/pong 服务

Flask 是用于构建 Web 服务的 Python 框架。使用 `pip install flask` 安装。第一个服务是最精简但可用的示例，即 [ping.py](code/ping.py)：

```python
from flask import Flask

app = Flask('ping')

@app.route('/ping', methods=['GET'])
def ping():
    return "PONG"

if __name__ == "__main__":
    app.run(debug=True, host='0.0.0.0', port=9696)
```

逐行说明：

- `app = Flask('ping')` 创建 Flask 应用并为其指定名称。
- `@app.route('/ping', methods=['GET'])` 是一个装饰器，告诉 Flask：下面的函数负责处理发往 `/ping` 地址的 GET 请求。这个地址称为 route。
- `ping()` 返回响应正文，也就是字符串 `PONG`。
- `if __name__ == "__main__"` 代码块会在直接执行文件（`python ping.py`）时启动开发服务器；从其他文件导入该模块时则不会启动。`host='0.0.0.0'` 使服务器监听所有网络接口，而不仅是 localhost；`port=9696` 设置端口。

补充说明：`localhost`（也就是 `127.0.0.1`）表示本机。绑定到 `0.0.0.0` 则意味着接受来自网络的请求，这通常是服务器部署时需要的。本地开发时两者都可以使用。

## 测试服务

启动服务：

```bash
python ping.py
```

然后在另一个终端中使用 curl 查询。curl 是用于发送 HTTP 请求的命令行工具：

```bash
curl http://localhost:9696/ping
```

响应如下：

```
PONG
```

也可以在浏览器中打开 `http://localhost:9696/ping`，浏览器同样会发送 GET 请求。这样我们就运行起了一个 Web 服务。在[下一节](04-flask-deployment_chi.md)中，我们会将 `PONG` 替换为实际的 churn 预测结果。

## 资料

- [幻灯片](https://www.slideshare.net/AlexeyGrigorev/ml-zoomcamp-5-model-deployment)
- [0.0.0.0 与 localhost 的区别](https://stackoverflow.com/a/20778887/861423)
- [顶层脚本环境（`__main__`）](https://docs.python.org/3.9/library/__main__.html)
- [Route 装饰器](https://flask.palletsprojects.com/en/2.2.x/api/#flask.Flask.route)

## 笔记

本节介绍了 Web 服务的概念，以及如何创建一个简单的 Web 服务。

- Web 服务是什么：
  - Web 服务是一种用于电子设备之间通信的方法。
  - Web 服务有多种方法可以满足不同需求，常见方法如下：
    - **GET：** 用于获取资源。例如，在 Google 搜索猫咪图片时，我们实际上是在通过 GET 请求获取图片。
    - **POST：** 用于向服务器发送数据，以创建或更新资源。例如注册时，我们会把姓名、用户名、密码等数据发送到服务器。注意，POST 并不规定数据最终保存在哪里。
    - **PUT：** 与 POST 类似，但会指定数据应放置的位置。
    - **DELETE：** 请求删除数据。
  - 可以搜索 HTTP 方法了解更多信息。
- 有很多语言和库可以用来创建 Web 服务。本课程使用 Python 的 Flask 框架。
  - 如果尚未安装 Flask，可以运行 `pip install Flask`。
  - 下面的代码创建了一个简单的 Web 服务：

    ```python
    from flask import Flask

    app = Flask('ping')

    @app.route('/ping', methods=['GET'])
    def ping():
        return 'PONG'

    if __name__ == '__main__':
        app.run(debug=True, host='0.0.0.0', port=9696)
    ```

  - 这段代码创建了名为 `ping` 的服务，并定义了一个返回 `PONG` 字符串的 `/ping` route。
  - 测试时，在新终端运行 `curl http://localhost:9696/ping`，或在浏览器中打开该地址，就能看到 `PONG`。
- 要让 Web 服务预测 churn，需要继续修改它。具体方法见下一节。