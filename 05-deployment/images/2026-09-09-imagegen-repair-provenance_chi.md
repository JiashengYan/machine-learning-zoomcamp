# Imagegen 修复来源记录 - 部署模块

## `05-pipenv-02-version-conflict-imagegen.jpg`

- 课程引用位置：`cohorts/2026/05-deployment/05-pipenv.md:21`
- 修复原因：先前的重绘包含一个未标注的黄色标记，并遗漏了源图注释 `INSTALLS THE LATEST`。
- 原始来源 JPG：`05-pipenv-02-version-conflict.jpg`
- 源图尺寸：`598x360`
- 源图 SHA-256：`6f579e3ac4154e9aeb56536e6b29f001b3be442fafd7bbd66270902ce195910c`
- 保留的限定裁剪图：`05-pipenv-02-version-conflict-imagegen-crop.jpg`
- 裁剪坐标：`555x300+23+0`（`x=23`、`y=0`、`width=555`、`height=300`）
- 裁剪图 SHA-256：`e5f10b6ee2f18f6857129841a068b4be888c63dad18966f89ff944536e3bb231`
- Imagegen 输出：`05-pipenv-02-version-conflict-imagegen.jpg`
- 输出尺寸：`1706x922`
- 输出 SHA-256：`64e6751a8bd29772b7209767927ba47b98c899ea7ab9433312d974544a11fce3`
- C2PA 元数据：存在；URN 为 `urn:c2pa:8cfcb601-1235-4936-bfed-e98bb58fbe30`
- 生成方式：使用内置 imagegen，并将原始 JPG 和保留的限定裁剪图作为参考。这是一次重绘，不是放大或锐化裁剪图。
- 仅调整尺寸的对比：与调整到 `1706x922` 的限定裁剪图相比，归一化 RMSE 为 `0.349921`，证实输出并非仅做尺寸调整的副本。

### 保留的不变量

- 明确保留共享系统 Python 中的依赖冲突：`CHURN SERVICE` 使用 `scikit-learn == 0.24.2`，而 `LEAD SCORING SERVICE` 使用 `scikit-learn == 1.0`。
- 保留共享安装路径和包流转信息：`$PATH`、`~/anaconda3/bin/pip`、`python`、`pypi.org` 和 `.wheel`。
- 源图注释 `INSTALLS THE LATEST` 已逐字恢复。
- 保留标题 `5.5 ENVIRONMENT & DEPENDENCY MANAGEMENT`。

### 移除的素材

- 演讲者面部/摄像头画面
- 屏幕录制光标和控制项
- 黑色边框和右下角仪表
- 无法解释的黄色涂鸦/标记

### 验证

- 检查了原生输出 `1706x922`。
- 检查了按课程宽度缩放的版本 `608x329`。
- 在两个视图中检查了必需标签和版本值。
- 不再包含人脸、摄像头画面、浏览器/录屏边框、光标、水印或未标注的黄色标记。

## 尚待严格审查的来源记录队列 - 2026-09-09

严格审查将以下 8 个非修复类部署素材标记为 `PROVENANCE-BLOCKED`。其中 7 个原本有非 crisp JPG 来源，已使用内置 imagegen，并将原始 JPG 和对应的限定原生裁剪图作为参考重新生成。剩余的 `09-explore-more` 信息图本身就是带有 C2PA 元数据的直接 imagegen 输出，没有底层 JPG；该图保持字节级不变，并在单独的记录中说明。

可复现的裁剪脚本为 [`2026-09-09-remaining-provenance-crops.sh`](2026-09-09-remaining-provenance-crops.sh)，SHA-256 为 `1641fb5702a321e5c74f73a2e44a5aec2d32f43ec5affbc1a731edc63df1b2a9`。脚本只使用 ImageMagick 原生 `-crop WIDTHxHEIGHT+X+Y`，不调整尺寸、不使用 Lanczos、不锐化，也不对源图进行其他增强。先前发布的 PNG 没有作为 imagegen 的参考输入。我们检查了原生输出和临时生成的 608px 课程宽度预览图；临时预览图不会发布。

### 基于来源重绘的素材

`Crop` 是源 JPG 像素坐标中的 `(x, y, width, height)`。`RMSE` 是相对于临时缩放裁剪图计算的归一化对比值，仅用于证明发布图并非只调整了尺寸。

| 发布 PNG | 课程引用位置 | 源 JPG | 裁剪 JPG | 裁剪范围 | 源图 SHA-256 | 裁剪图 SHA-256 | 旧 PNG SHA-256 | 当前 PNG SHA-256 | 原生尺寸 | 608px 尺寸 | Imagegen 执行记录 | C2PA | RMSE |
|---|---|---|---|---|---|---|---|---|---:|---:|---|---|---:|
| `01-intro-02-model-deployment-diagram-imagegen.jpg` | `01-intro.md:29` | `01-intro-02-model-deployment-diagram.jpg` | `01-intro-02-model-deployment-diagram-imagegen-crop.jpg` | `(20, 10, 430, 330)` | `1f0815984949b48a85c7f2c390c2981ad2f3e6ce480aeed532e6521449ae96c6` | `ccfe5763c47680ed6b9d02208bce5bb1eaa8cd850ce74b7c0e8439ff1a010818` | `c945dbc1cc07bf12deda18ef518c359c0b989058b3bf9db477fb8ffbbe2946ea` | `112018141d685aa9eb4df4e53757e832ec9c99491f65f5fdc59f8c55d72cec15` | `1536x1024` | `608x405` | `exec-fa955f65-56dc-449a-828a-feaa09f7b9e2` | `urn:c2pa:42dee4d3-5718-4ecc-9830-6d4da71a82a7` | `0.253346` |
| `01-intro-05-environments-imagegen.jpg` | `01-intro.md:53` | `01-intro-05-environments.jpg` | `01-intro-05-environments-imagegen-crop.jpg` | `(20, 10, 430, 330)` | `647357fe288431f542cbf342b3374493191247faaa8f63ad36a8ca256cb18ac7` | `6331d35940beb53cf06ebec91f0e34ebf7444ab1c331e920fe6ab10071057fce` | `01692c845b397f009694dda20c79149125f4e1a8cdac584550135452f2fb8fbf` | `a763c89e085440a8b72bb8ab97b517a3c1ff79328e726b7c02a68129352b3973` | `1536x1024` | `608x405` | `exec-eb0499e7-b1a3-445f-ad79-74f5b128ba59` | `urn:c2pa:74b99fc6-31fc-40f1-bfed-a17776c5d8d7` | `0.270182` |
| `03-flask-intro-02-request-response-imagegen.jpg` | `03-flask-intro.md:19` | `03-flask-intro-02-request-response.jpg` | `03-flask-intro-02-request-response-imagegen-crop.jpg` | `(25, 10, 450, 330)` | `8e5dff072baf56ad3846c929aaa692a124037a51efc84949d1910899cfb24fd4` | `082e79da8431669c813335d2aafd3ab520ebc5712e8b61bb5d38104a0e03a55c` | `6d4825726690f2b9c7f328e3341208c0f8d1170db2298033a2663bfc859108bc` | `8030ba17e3b8c534ef78ff4a7096a1821f38ac1605e3f933150e60bd0cb8dc29` | `1617x973` | `608x366` | `exec-d6ecd660-1ae1-4ae4-8be6-d35f6ded627c` | `urn:c2pa:386272e9-719f-414d-ab3f-9d28dcb234d4` | `0.208263` |
| `05-pipenv-03-isolated-environments-imagegen.jpg` | `05-pipenv.md:27` | `05-pipenv-03-isolated-environments.jpg` | `05-pipenv-03-isolated-environments-imagegen-crop.jpg` | `(8, 5, 475, 335)` | `2e409d6d3d8694253d616be362e05c146faf101c718a05d0e51206f5dd3b0d3c` | `8b519dab1932d8a3bc8dccc43ebe76baa7df34a76fd02bab1b88cd020c4af079` | `2fb501d045b462980c9ae79c267c4a3dcc7e7fc96cfd1950df1d67f8c29e1cf4` | `7a4022a90b3fb40511ef9f4f933c90f2498e1aefec0b88bf21f5c05668978e4f` | `1536x1024` | `608x405` | `exec-cdba492d-a64f-474d-8d23-e5f571fe31d9` | `urn:c2pa:95356c54-44fa-42ab-97ee-4f0968b7e2da` | `0.318491` |
| `06-docker-02-containers-on-host-imagegen.jpg` | `06-docker.md:49` | `06-docker-02-containers-on-host.jpg` | `06-docker-02-containers-on-host-imagegen-crop.jpg` | `(15, 5, 455, 335)` | `5a9d8eb7f6fe20da378cf8e0a4fd97ba6962f8c4928876a6eeef302921d647cf` | `92f6830c9343dec60cac3fdfddc2ac081cce7818727bf90335efbe6979b18a3e` | `284950d40d34c85640b52b0cd29c33f9c8758b44e89c1485eda099b6a365484f` | `efdb8e1a66b2c5d712d808f978fda00957c3ae8fb0af809785b0bead3c918ce1` | `1616x973` | `608x366` | `exec-07574b49-88b3-4e7b-8a32-0731467c4c6c` | `urn:c2pa:1691bed8-65ff-4ddd-b88e-4a3ee1b3acda` | `0.310580` |
| `06-docker-05-port-mapping-imagegen.jpg` | `06-docker.md:127` | `06-docker-05-port-mapping.jpg` | `06-docker-05-port-mapping-imagegen-crop.jpg` | `(25, 10, 430, 320)` | `13ef781550bfb1acb83ca149d00eadcb975084358b41aa4bec11fa887e1c6bf2` | `a637247b1bd030658c04c5457ddcd70ff2174a22c60d47501fded355e51fa493` | `bda48ba4b5fa19861ae6285da3ef00e8a51bd766ac736a4feb99aa61fda94282` | `305c67a4c2a6ca7196d6fb460aeb1fb70a6f6bf7f0cc9226cfad057324c9bd8d` | `1617x973` | `608x366` | `exec-43160290-6473-4bb8-b0a8-1a145653cd35` | `urn:c2pa:747ed203-092a-46c7-85a8-f1405aa628fd` | `0.288315` |
| `07-aws-eb-02-eb-architecture-imagegen.jpg` | `07-aws-eb.md:29` | `07-aws-eb-02-eb-architecture.jpg` | `07-aws-eb-02-eb-architecture-imagegen-crop.jpg` | `(25, 0, 430, 340)` | `472f8939e324ca9a52aa3161605458e5738c0e4f60b1a2ab979022512453e008` | `54d8a0560cf758406f1721a63635a160cdd9f4f932d8a0141516d8440b1a8918` | `6869b17e15363be29a3e0b972c54c869f30cebc951363f37fd6ab49cce20d99e` | `ef01c5f7c52d80453c6238a758dbb00683785907287ea24b0485900a72188e87` | `1617x973` | `608x366` | `exec-23878f94-6994-4840-b31b-5c8625983e8a` | `urn:c2pa:f643cc35-c541-49cf-a7d8-622ffd49d457` | `0.251506` |

### 保留的不变量和移除的内容

- 部署架构图保留 notebook → `model.bin` → churn 服务的流程，以及营销服务的请求/响应箭头。
- 环境图保留嵌套的 Flask、Pipenv 和 Docker 层，以及准确文本 `ENVIRONMENT FOR PYTHON DEPS` 和 `ENVIRONMENT — SYSTEM DEPENDENCIES`。
- Flask 插图保留请求/响应箭头，以及准确注释 `ml+zoomcamp` 和 `q = web service`。
- Pipenv 插图保留两条准确的可执行文件路径和 `NO CONFLICTS`。
- Docker 插图保留准确的操作系统版本、服务名称、端口、`9696:9696`、`host : container` 和完整命令 `docker run -it -p 9696:9696 churn-prediction:latest`。
- AWS 插图保留 `190`、`100`、`100`、两个 `S3` 框、Docker churn 服务，以及营销服务的请求/响应路径。
- 七张重绘图均移除了摄像头画面、色轮控件、黑色录制边框、浏览器/编辑器界面、光标、播放控件和选中状态覆盖层。

### 保留的直接 imagegen 输出

| 发布 PNG | 课程引用位置 | 当前 PNG SHA-256 | 原生尺寸 | 608px 尺寸 | C2PA | 决定 |
|---|---|---|---:|---:|---|---|
| `09-explore-more-01-deployment-choices-imagegen.jpg` | `09-explore-more.md:6` | `b3c35230788a9dc861b6f96d65bf127fc98b4e988e553f7e02d75b8742b29b75` | `1672x941` | `608x342` | `urn:c2pa:04369cae-5b1e-467a-b459-fb9099728d8d` | 按字节原样保留。这是一张全新的概念信息图，模块中没有对应的非 crisp JPG 原图。嵌入的 C2PA 记录证明它是直接由 imagegen 生成的；原生尺寸和 608px 视图均已检查，未发现人脸、摄像头画面、浏览器/编辑器界面、光标、播放控件或选中状态覆盖层。 |

没有为这张直接生成的 imagegen 素材伪造源图/裁剪图组合。已有的语义修复素材 `05-pipenv-02-version-conflict-imagegen.jpg` 也保持不变；其单独的 provenance 和语义修复记录仍为权威记录。