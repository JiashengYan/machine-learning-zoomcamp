# 使用 FastAPI 和 uv 部署机器学习模型

- 视频：https://www.youtube.com/watch?v=jzGzw98Eikk

本 workshop 将复习 [Machine Learning Zoomcamp](https://github.com/DataTalksClub/machine-learning-zoomcamp) 的[模块 5](https://github.com/DataTalksClub/machine-learning-zoomcamp/tree/main/05-deployment)，并介绍较新的工具：

- 使用 Scikit-Learn Pipeline
- 使用 uv 替代 Pipenv
- 使用 FastAPI 替代 Flask
- 使用 Fly.io 替代 AWS EBS

本 workshop 按照与模块相同的顺序展开：

- 使用 pickle 保存和加载模型
- 将 notebook 转换为训练脚本
- FastAPI 简介（替代 Flask）
- 使用 FastAPI 提供模型服务
- 使用 Pydantic 验证输入（新增）
- 使用 uv 管理虚拟环境（替代 Pipenv）
- 使用 Docker 容器化
- 使用 Fly.io 部署

## 环境

可以使用 [GitHub Codespaces](https://www.youtube.com/watch?v=pqQFlV3f9Bo&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR)。

安装需要的库：

```bash
pip install jupyter scikit-learn pandas
```

然后下载 starter notebook，并将其保存为 `workshop-uv-fastapi.ipynb`。后续练习会以它为基础：

```bash
wget https://raw.githubusercontent.com/alexeygrigorev/workshops/main/mlzoomcamp-fastapi-uv/starter.ipynb -O workshop-uv-fastapi.ipynb
```

在 Jupyter 中打开它。

## 加载和保存模型

先对下面这个数据点进行预测：

```python
datapoint = {
    'gender': 'female',
    'seniorcitizen': 0,
    'partner': 'yes',
    'dependents': 'no',
    'phoneservice': 'no',
    'multiplelines': 'no_phone_service',
    'internetservice': 'dsl',
    'onlinesecurity': 'no',
    'onlinebackup': 'yes',
    'deviceprotection': 'no',
    'techsupport': 'no',
    'streamingtv': 'no',
    'streamingmovies': 'no',
    'contract': 'month-to-month',
    'paperlessbilling': 'yes',
    'paymentmethod': 'electronic_check',
    'tenure': 1,
    'monthlycharges': 29.85,
    'totalcharges': 29.85
}
```

先使用 dictionary vectorizer 对其进行转换：

```python
X = dv.transform(datapoint)
```

然后获得预测结果：

```python
model.predict_proba(X)[0, 1]
```

使用 pickle 保存：

```python
with open('model.bin', 'wb') as f_out:
    pickle.dump((dv, model), f_out)
```

加载方式如下：

```python
with open('model.bin', 'rb') as f_in:
    (dv, model) = pickle.load(f_in)
```

## Scikit-Learn Pipeline

同时处理两个对象 `dv` 和 `model` 不太方便。我们可以将它们组合成一个对象：

```python
from sklearn.pipeline import make_pipeline

pipeline = make_pipeline(
    DictVectorizer(),
    LogisticRegression(solver='liblinear')
)

pipeline.fit(train_dict, y_train)
```

预测也会更简单：

```python
pipeline.predict_proba(datapoint)[0, 1]
```

## 将 notebook 转换为脚本

现在可以将 notebook 转换为训练脚本：

```bash
jupyter nbconvert --to=script workshop-uv-fastapi.ipynb
mv workshop-uv-fastapi.py train.py
```

接下来编辑脚本。最终代码会类似 [train.py](train.py)：

```python
df = load_data()
pipeline = train_model(df)
save_model(pipeline, 'model.bin')

print('Model saved to model.bin')
```

现在加载已保存的模型。创建 [predict.py](predict.py)，并在其中加载模型：

```python
import pickle

with open('model.bin', 'rb') as f_in:
    pipeline = pickle.load(f_in)

# apply the model
```

## FastAPI

接下来将 `predict.py` 改造成 Web 服务。

安装 FastAPI 和 uvicorn：

```bash
pip install fastapi uvicorn
```

最简单的 FastAPI 应用（由 ChatGPT 根据[旧版 Flask 代码](https://github.com/DataTalksClub/machine-learning-zoomcamp/blob/main/05-deployment/code/ping.py)转换而来）：

创建 `ping.py`：

```python
from fastapi import FastAPI
import uvicorn

app = FastAPI(title="ping")

@app.get("/ping")
def ping():
    return "PONG"

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=9696)
```

运行：

```bash
python ping.py
```

更常用的启动方式是：

```bash
uvicorn ping:app --host 0.0.0.0 --port 9696 --reload
```

现在可以在浏览器中打开 http://localhost:9696/ping，也可以使用 curl 发送请求：

```bash
curl localhost:9696/ping
```

目前它与 Flask 没有明显区别。不过 FastAPI 还能提供 Flask 不具备的交互式 API 文档：

http://localhost:9696/docs

现在将脚本改为 Web 应用：

```python
import pickle
from fastapi import FastAPI
import uvicorn

app = FastAPI(title="customer-churn-prediction")

with open('model.bin', 'rb') as f_in:
    pipeline = pickle.load(f_in)


def predict_single(customer):
    result = pipeline.predict_proba(customer)[0, 1]
    return float(result)


@app.post("/predict")
def predict(customer):
    prob = predict_single(customer)

    return {
        "churn_probability": prob,
        "churn": bool(prob >= 0.5)
    }


if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=9696)
```

运行：

```bash
uvicorn predict:app --host 0.0.0.0 --port 9696 --reload
```

目前 FastAPI 还无法识别输入为 JSON，因此添加类型提示：

```python
from typing import Dict, Any

@app.post("/predict")
def predict(customer: Dict[str, Any]):
    prob = predict_single(customer)

    return {
        "churn_probability": prob,
        "churn": bool(prob >= 0.5)
    }
```

打开 API 文档并发送请求：

```json
{
    "gender": "female",
    "seniorcitizen": 0,
    "partner": "yes",
    "dependents": "no",
    "phoneservice": "no",
    "multiplelines": "no_phone_service",
    "internetservice": "dsl",
    "onlinesecurity": "no",
    "onlinebackup": "yes",
    "deviceprotection": "no",
    "techsupport": "no",
    "streamingtv": "no",
    "streamingmovies": "no",
    "contract": "month-to-month",
    "paperlessbilling": "yes",
    "paymentmethod": "electronic_check",
    "tenure": 1,
    "monthlycharges": 29.85,
    "totalcharges": 29.85
}
```

也可以使用 curl：

```bash
curl -X 'POST' 'http://localhost:9696/predict' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
    "gender": "female",
    "seniorcitizen": 0,
    "partner": "yes",
    "dependents": "no",
    "phoneservice": "no",
    "multiplelines": "no_phone_service",
    "internetservice": "dsl",
    "onlinesecurity": "no",
    "onlinebackup": "yes",
    "deviceprotection": "no",
    "techsupport": "no",
    "streamingtv": "no",
    "streamingmovies": "no",
    "contract": "month-to-month",
    "paperlessbilling": "yes",
    "paymentmethod": "electronic_check",
    "tenure": 1,
    "monthlycharges": 29.85,
    "totalcharges": 29.85
}'
```

请求跨越 `/predict` 服务边界，并返回预测响应：

![Python 客户端向 FastAPI /predict endpoint 发送客户数据，并收到 churn 概率和判定。](images/fastapi-request-response-imagegen.jpg)

需要包含请求头。FastAPI 对 schema 和验证比 Flask 更严格。

如果要从脚本发送请求，可以使用 `requests` 库。安装它：

```bash
pip install requests
```

创建 [`test.py`](test.py)：

```python
import requests

url = 'http://localhost:9696/predict'

customer = {
    'gender': 'female',
    'seniorcitizen': 0,
    'partner': 'yes',
    'dependents': 'no',
    'phoneservice': 'no',
    'multiplelines': 'no_phone_service',
    'internetservice': 'dsl',
    'onlinesecurity': 'no',
    'onlinebackup': 'yes',
    'deviceprotection': 'no',
    'techsupport': 'no',
    'streamingtv': 'no',
    'streamingmovies': 'no',
    'contract': 'month-to-month',
    'paperlessbilling': 'yes',
    'paymentmethod': 'electronic_check',
    'tenure': 1,
    'monthlycharges': 29.85,
    'totalcharges': 29.85
}

response = requests.post(url, json=customer)
predictions = response.json()

print(predictions)
if predictions['churn']:
    print('customer is likely to churn, send promo')
else:
    print('customer is not likely to churn')
```

## Pydantic 与验证

FastAPI 的另一个优点是支持输入和输出验证，Flask 本身没有内置该功能。

为设计下面的 schema，我使用了 ChatGPT，并提供了一个示例以及以下代码的输出：

```python
for n in numerical:
    print(df[n].describe())
    print()

for c in categorical:
    print(df[c].value_counts())
    print()
```

输入和输出模型如下：

```python
from typing import Literal
from pydantic import BaseModel, Field

class Customer(BaseModel):
    gender: Literal["male", "female"]
    seniorcitizen: Literal[0, 1]
    partner: Literal["yes", "no"]
    dependents: Literal["yes", "no"]
    phoneservice: Literal["yes", "no"]
    multiplelines: Literal["no", "yes", "no_phone_service"]
    internetservice: Literal["dsl", "fiber_optic", "no"]
    onlinesecurity: Literal["no", "yes", "no_internet_service"]
    onlinebackup: Literal["no", "yes", "no_internet_service"]
    deviceprotection: Literal["no", "yes", "no_internet_service"]
    techsupport: Literal["no", "yes", "no_internet_service"]
    streamingtv: Literal["no", "yes", "no_internet_service"]
    streamingmovies: Literal["no", "yes", "no_internet_service"]
    contract: Literal["month-to-month", "one_year", "two_year"]
    paperlessbilling: Literal["yes", "no"]
    paymentmethod: Literal[
        "electronic_check",
        "mailed_check",
        "bank_transfer_(automatic)",
        "credit_card_(automatic)",
    ]
    tenure: int = Field(..., ge=0)
    monthlycharges: float = Field(..., ge=0.0)
    totalcharges: float = Field(..., ge=0.0)


class PredictResponse(BaseModel):
    churn_probability: float
    churn: bool
```

现在可以明确声明期望的输入和生成的输出：

```python
@app.post("/predict")
def predict(customer: Customer) -> PredictResponse:
    prob = predict_single(customer.model_dump())

    return PredictResponse(
        churn_probability=prob,
        churn=prob >= 0.5
    )
```

注意：如果使用 `customer.dict()` 而不是 `model_dump()`，可能会看到以下警告：

```
PydanticDeprecatedSince20: The `dict` method is deprecated; use `model_dump` instead. Deprecated in Pydantic V2.0 to be removed in V3.0. See https://errors.pydantic.dev/2.11/migration/
```

现在可以测试错误输入的处理方式。在 `test.py` 中加入字段 `"whatever": 31337` 并运行。

此时看起来没有变化，服务仍然像之前一样运行。若要让 Pydantic 对额外字段报错，需要添加 `model_config`：

```python
from pydantic import ConfigDict


class Customer(BaseModel):
    model_config = ConfigDict(extra="forbid")

    ... # rest of the fields
```

现在会收到错误：

```python
response: {'detail': [{'type': 'extra_forbidden', 'loc': ['body', 'whatever'], 'msg': 'Extra inputs are not permitted', 'input': 31337}]}
```

如果传入模型未定义的取值，例如：

```json
{
    ...
    "streamingtv": "maybe"
    ...
}
```

此时行为符合预期：请求会报错：

```python
response: {'detail': [{'type': 'literal_error', 'loc': ['body', 'streamingtv'], 'msg': "Input should be 'no', 'yes' or 'no_internet_service'", 'input': 'maybe', 'ctx': {'expected': "'no', 'yes' or 'no_internet_service'"}}]}
```

## 环境管理

现在应用可以运行了，但它可能与其他项目产生版本冲突，因此需要隔离该项目的环境。

本 workshop 不详细介绍虚拟环境的理论背景。可参考[模块 5](https://github.com/DataTalksClub/machine-learning-zoomcamp/blob/main/05-deployment/05-pipenv.md)。

这里使用 [`uv`](https://docs.astral.sh/uv/) 管理依赖和环境。

安装：

```bash
pip install uv
```

初始化项目：

```bash
uv init
```

我们不需要 `main.py`，可以删除：

```bash
rm main.py
```

可以看到生成了一些文件：

- `.python-version`
- `pyproject.toml`

项目需要 Scikit-Learn 和 FastAPI，因此添加它们：

```bash
uv add scikit-learn fastapi uvicorn
```

还会生成：

- 包含所有依赖的 `.venv`
- 详细描述依赖的 `uv.lock`

我们还需要一个开发依赖，生产环境中不会用到它：

```bash
uv add --dev requests
```

要在该虚拟环境中运行程序，只需在命令前加上 `uv run`：

```bash
uv run uvicorn predict:app --host 0.0.0.0 --port 9696 --reload
uv run python test.py
```

获取一个已经使用 uv 的项目副本时，可以使用 sync 命令安装所有依赖：

```bash
uv sync
```

## Docker

下面使用 Docker 实现完整隔离。若想了解 Docker 的更多内容，请参考[模块 5](https://github.com/DataTalksClub/machine-learning-zoomcamp/blob/main/05-deployment/06-docker.md)。

本 workshop 会调整模块中创建的 [Dockerfile](https://github.com/DataTalksClub/machine-learning-zoomcamp/blob/main/05-deployment/code/Dockerfile)。

首先确认需要的 Python 版本。可以使用以下命令查看：

```bash
$ python -V
Python 3.13.5
```

因此使用 Python 3.13.5 image：

```dockerfile
# Use the official Python 3.13.5 slim version based on Debian Bookworm as the base image
FROM python:3.13.5-slim-bookworm

# Copy the 'uv' and 'uvx' executables from the latest uv image into /bin/ in this image
# 'uv' is a fast Python package installer and environment manager
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

# Set the working directory inside the container to /code
# All subsequent commands will be run from here
WORKDIR /code

# Add the virtual environment's bin directory to the PATH so Python tools work globally
ENV PATH="/code/.venv/bin:$PATH"

# Copy the project configuration files into the container
# pyproject.toml     → project metadata and dependencies
# uv.lock            → locked dependency versions (for reproducibility)
# .python-version    → Python version specification
COPY "pyproject.toml" "uv.lock" ".python-version" ./

# Install dependencies exactly as locked in uv.lock, without updating them
RUN uv sync --locked

# Copy application code and model data into the container
COPY "predict.py" "model.bin" ./

# Expose TCP port 9696 so it can be accessed from outside the container
EXPOSE 9696

# Run the application using uvicorn (ASGI server)
# predict:app → refers to 'app' object inside predict.py
# --host 0.0.0.0 → listen on all interfaces
# --port 9696    → listen on port 9696
ENTRYPOINT ["uvicorn", "predict:app", "--host", "0.0.0.0", "--port", "9696"]
```

（注释由 [ChatGPT](https://chatgpt.com/share/6899ebd9-96f8-800a-819e-093fccadaf7e) 添加。）

构建 image：

```bash
docker build -t predict-churn .
```

运行：

```bash
docker run -it --rm -p 9696:9696 predict-churn
```

## 部署

应用容器化后，就可以部署到不同平台。

课程中演示了 Elastic Beanstalk，其他选择包括：

- Google Cloud Run
- AWS App Runner
- Fly.io
- 还可以查看课程学员的分享，那里有许多其他选择。

根据 ChatGPT 的建议，Fly.io 的流程很简单：

```bash
# for other OS, check https://fly.io/docs/flyctl/install/
# you may also need to replace fly with flyctl
curl -L https://fly.io/install.sh | sh

fly auth signup
fly launch --generate-name
fly deploy
```

从日志中获取 URL，形式大致如下：

```
Visit your newly deployed app at https://mlzoomcamp-flask-uv.fly.dev/
```

将该 URL 写入 `test.py` 并确认服务可用。

部署流程是：将服务打包为 Docker image，部署到 Fly.io，再使用生成的可访问 endpoint：

![Docker 化的预测服务部署到 Fly.io，并通过 endpoint 对外提供访问。](images/docker-fly-deployment-flow-imagegen.jpg)

现在可以销毁部署：

```bash
fly apps destroy <app-name>
```

使用 `apps list` 命令可以查看应用列表。

注意查看定价信息。

## 总结

在本 workshop 中，我们将机器学习模型放入 Docker 容器并部署到云端。

如果想进一步了解机器学习工程，请查看 [ML Zoomcamp 课程](https://github.com/DataTalksClub/machine-learning-zoomcamp/)。