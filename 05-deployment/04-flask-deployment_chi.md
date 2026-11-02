---
video_url: https://www.youtube.com/watch?v=Q7ZWPgPnRz8&list=PL3MmuxUbc_hIhxl5Ji8t4O6lPAOpHaCLR
code:
  - label: Flask 应用
    path: code/predict.py
  - label: 测试脚本
    path: code/predict-test.py
prev_url: 03-flask-intro_chi.md
next_url: 05-pipenv_chi.md
---
# 使用 Flask 提供 churn 模型服务

本节将 churn 模型封装为 Flask Web 服务：服务加载 pickle 保存的模型，监听包含客户数据的 POST 请求，并返回 churn 概率。

## Web 服务

我们在[上一节](03-flask-intro_chi.md)的 ping 应用基础上扩展。完整服务位于 [predict.py](code/predict.py)：

```python
import pickle

from flask import Flask
from flask import request
from flask import jsonify


model_file = 'model_C=1.0.bin'

with open(model_file, 'rb') as f_in:
    dv, model = pickle.load(f_in)

app = Flask('churn')

@app.route('/predict', methods=['POST'])
def predict():
    customer = request.get_json()

    X = dv.transform([customer])
    y_pred = model.predict_proba(X)[0, 1]
    churn = y_pred >= 0.5

    result = {
        'churn_probability': float(y_pred),
        'churn': bool(churn)
    }

    return jsonify(result)


if __name__ == "__main__":
    app.run(debug=True, host='0.0.0.0', port=9696)
```

值得注意的部分：

- 模型在服务启动时加载一次，而不是每个请求到来时都重新加载。加载速度比训练快，但也没有必要重复执行。
- Route 使用 POST 方法：客户数据以 JSON 形式放在请求正文中，`request.get_json()` 会将其转换为 Python 字典。
- 我们使用 vectorizer 转换客户数据。由于转换器接收的是记录列表，因此传入只包含一个字典的列表；随后从 `predict_proba` 的第二列（索引 1，即正类“churn”的概率）取得预测值。
- 使用 `0.5` 作为阈值，将概率转换为是否流失的结果，并将 NumPy 类型转换为 Python 原生的 `float` 和 `bool`。JSON 不支持 NumPy 类型，Flask 无法直接序列化它们。
- `jsonify(result)` 将字典转换为 JSON 响应。

## 测试服务

运行服务：`python predict.py`。浏览器无法直接测试它，因为浏览器发送的是 GET 请求，而此 route 要求 POST。我们使用小型客户端脚本 [predict-test.py](code/predict-test.py)，通过 `requests` 库发送请求：

```python
import requests

url = 'http://localhost:9696/predict'

customer = {
    "gender": "female",
    "seniorcitizen": 0,
    ...
}

response = requests.post(url, json=customer).json()
print(response)

if response['churn'] == True:
    print('sending promo email to %s' % customer_id)
else:
    print('not sending promo email to %s' % customer_id)
```

客户字典中的字段与模型训练时看到的字段一致。`json=customer` 会将字典序列化为 JSON，并放入请求正文。运行脚本会打印类似结果：

```
{'churn': False, 'churn_probability': 0.3257561103397851}
not sending promo email to xyz-123
```

这正是营销服务调用模型的方式：将客户数据作为 JSON 发送，接收 churn 判定，然后采取相应操作。

## 在生产环境中运行：gunicorn

使用 `python predict.py` 启动应用时，Flask 会提示其开发服务器不适合生产环境：它是单线程的，也不是为高负载设计的。生产环境通常使用 WSGI 服务器。WSGI（Web Server Gateway Interface，Web 服务器网关接口）是 Python Web 应用的标准服务接口；WSGI 服务器会导入我们的 `app` 对象，并处理网络通信相关工作。

常见选择是 gunicorn：

```bash
pip install gunicorn
gunicorn --bind=0.0.0.0:9696 predict:app
```

最后一个参数的格式是 `module:app_object`：文件名是 `predict.py`，其中 Flask 应用对象名为 `app`。服务行为不变，但现在由适合生产环境的服务器运行。

Windows 不支持 gunicorn，因为它依赖的某些库在 Windows 上不可用。Windows 上可以使用 waitress：

```bash
pip install waitress
waitress-serve --listen=0.0.0.0:9696 predict:app
```

[下一节](05-pipenv_chi.md)将解决剩余问题：如何控制项目依赖的库版本。

## 资料

- [幻灯片](https://www.slideshare.net/AlexeyGrigorev/ml-zoomcamp-5-model-deployment)
- 本节讲解的完整服务脚本也可以在 [mlbookcamp-code 仓库](https://github.com/alexeygrigorev/mlbookcamp-code/blob/master/chapter-05-deployment/churn_serving.py)中找到。

## 笔记

本节介绍了如何为 churn Web 服务添加预测功能，以及如何在开发环境中使用它。

- 要让 Web 服务预测客户是否会流失，需要将第 3 节的代码与前面章节中的模型代码结合起来。
- 首先加载之前保存的模型，再通过专门的 route 进行预测：

  ```python
  import pickle

  with open('churn-model.bin', 'rb') as f_in:
      dv, model = pickle.load(f_in)
  ```

- 预测函数可以这样编写：

  ```python
  def predict_single(customer, dv, model):
      X = dv.transform([customer])
      y_pred = model.predict_proba(X)[:, 1]
      return y_pred[0]
  ```

- 然后定义 Web 服务的预测 route：

  ```python
  @app.route('/predict', methods=['POST'])
  def predict():
      customer = request.get_json()
      prediction = predict_single(customer, dv, model)
      churn = prediction >= 0.5

      result = {
          'churn_probability': float(prediction),
          'churn': bool(churn),
      }

      return jsonify(result)
  ```

- 完整代码见 [churn_serving.py](https://github.com/alexeygrigorev/mlbookcamp-code/blob/master/chapter-05-deployment/churn_serving.py)。
- 由于应用期待 POST 请求，不能只在浏览器中打开地址。可以使用 `requests` 发送 JSON 客户数据：

  ```python
  customer = {
      'customerid': '8879-zkjof',
      'gender': 'female',
      'seniorcitizen': 0,
      'partner': 'no',
      'dependents': 'no',
      'tenure': 41,
      'phoneservice': 'yes',
      'multiplelines': 'no',
      'internetservice': 'dsl',
      'onlinesecurity': 'yes',
      'onlinebackup': 'no',
      'deviceprotection': 'yes',
      'techsupport': 'yes',
      'streamingtv': 'yes',
      'streamingmovies': 'yes',
      'contract': 'one_year',
      'paperlessbilling': 'yes',
      'paymentmethod': 'bank_transfer_(automatic)',
      'monthlycharges': 79.85,
      'totalcharges': 3320.75
  }
  import requests
  url = 'http://localhost:9696/predict'
  response = requests.post(url, json=customer)
  result = response.json()
  print(result)
  ```

到这里，我们已经创建了一个能够预测客户 churn 的 Web 服务。启动应用时会看到警告：Flask 开发服务器不是 WSGI 服务器，不适合生产环境。可以使用 gunicorn 创建 WSGI 服务器；Windows 用户可使用 waitress。安装和运行命令见上文。

下一节将介绍如何解决不同机器上的库版本冲突，并为生产环境管理依赖。