# FastAPI Workshop 视觉审查记录 - 2026-09-09

## 审查结果

我审查了 [FastAPI 和 uv workshop](README.md) 及其源视频 [jzGzw98Eikk](https://www.youtube.com/watch?v=jzGzw98Eikk)。视频通过文档记录的 DataImpulse 路径获取，使用 `ffprobe` 验证，并在相关教学时刻逐帧检查。

有两张基于源视频、适合发布的概念插图：

- 客户端到 `/predict` 的请求及预测响应；
- 从 Docker image 经 Fly.io 部署到可访问应用 endpoint 的流程。

生成的插图有意不复现具体代码、URL、概率值、终端输出或临时 UI。这些细节保留在原生 Markdown 和代码块中。原始视频、源帧和裁剪图保留在被忽略的临时目录中，用于来源记录和审查；它们不会作为课程素材提交。

## 来源与临时材料

以下是审查使用的来源和临时路径：

- 来源 URL：`https://www.youtube.com/watch?v=jzGzw98Eikk`
- 视频 ID：`jzGzw98Eikk`
- 缓存的 transcript：`~/.cache/youtube_transcripts/jzGzw98Eikk.txt`
- 缓存 transcript 的 SHA-256：`ad491d4780a9aff8b47a1e3038ebc5834324e9ed0b801c2e0942f7884b007610`
- 获取的源视频：`/home/alexey/git/.tmp/ml-workshop-videos/jzGzw98Eikk.mp4`
- 源视频 SHA-256：`8c20deeb5c0552f9c28198643882c83a3c6d1d42077c6b9901c238124ffdfb2a`
- 源视频属性：`640x360`、`30 fps`、`6083.674558` 秒、`178855244` 字节
- 候选帧目录：`/home/alexey/git/.tmp/ml-workshop-frames/jzGzw98Eikk/`
- 最终输出目录：`images/`

请求响应候选帧在 `00:53:11` 提取为 `response.jpg`，并按 `(x=45, y=15, width=540, height=300)` 裁剪为 `response-crop.png`。原始帧 SHA-256 为 `f9a87c95c8cf31e81ab748a9d06a5cd090feddcdf5ae187e03573609edfb7c1c`；裁剪图 SHA-256 为 `4f9c7d0823c7514d4fbfeaaa47a8e5a2188dc36f4affb63b2b56cc4c90f467e2`。

部署候选帧在 `01:33:22` 提取为 `fly-deployed.jpg`，并按 `(x=155, y=195, width=470, height=155)` 裁剪为 `fly-deployed-crop.png`。原始帧 SHA-256 为 `1facd7c23eaa443b9938b3fecdd57214184ca208bb6de5c592fd8644f06dba68`；裁剪图 SHA-256 为 `e66d670e3aff47ba6af6a1ce0d0e69ab355f0bca7871fabdfb979ae3efb17f96`。

每张最终采用的插图都以原始非 crisp 帧和局部裁剪图作为图像生成输入。生成文件的哈希如下：

- `images/fastapi-request-response-imagegen.jpg`：`4892a2e5ce31e5e75b759c2eeb4f846e39a64ceec104fc181820c77d91449e0e`
- `images/docker-fly-deployment-flow-imagegen.jpg`：`ca20b6ae853e47d997a5f44a13d413a0029062f2c69231323d0970eced3ddf07`

使用 DataImpulse 获取来源的检查已通过：`yt-dlp` 使用 Android client 和 360p 格式下载了非空 MP4，`ffprobe` 确认了视频时长和尺寸。没有将缩略图或仅依据 transcript 的说法用作课程素材。

## 候选审查

评分标准包含贡献度、相关性、可读性、互补性、持久性和标题/无障碍性，每项 0 到 2 分，总分 12 分。首先必须确认源帧可验证。具体代码、命令、URL、数值、表格和临时 UI 保留在原生 Markdown 或确定性渲染内容中；imagegen 仅用于表达持久的概念关系。

### 候选 1：服务边界

- 时间戳：`00:36:30`
- 源帧：`service-boundary.jpg`
- 得分：`4/12`
- 决定：拒绝。该帧主要展示实现代码，无法在附近说明和代码之外提供清晰的服务关系信息。

### 候选 2：交互式 API 文档

- 时间戳：`00:42:20`
- 源帧：`api-docs.jpg`
- 得分：`3/12`
- 决定：拒绝。它只是通用的 `/ping` Swagger UI；具体 UI 会变化，endpoint 细节也已由原生文字说明。

### 候选 3：客户端请求和预测响应

- 时间戳：`00:53:11`
- 源帧：`response.jpg`；局部裁剪图：`response-crop.png`
- 得分：`11/12`
- 决定：保留。该帧清晰展示客户端向 `/predict` 发送 JSON 并接收 churn 预测这一教学要点。imagegen 将有源帧依据的关系转换为清晰、持久的示意图，没有虚构源素材中的具体代码或数值输出。
- 发布素材：`images/fastapi-request-response-imagegen.jpg`
- README 引用位置：紧接 `curl /predict` 示例之后

### 候选 4：无效输入

- 时间戳：`00:55:40`
- 源帧：`invalid-input.jpg`
- 得分：`3/12`
- 决定：拒绝。截图没有清晰展示验证错误；准确的错误 payload 应放在已有原生代码/文本中。

### 候选 5：远程部署状态

- 时间戳：`01:33:22`
- 源帧：`fly-deployed.jpg`；局部裁剪图：`fly-deployed-crop.png`
- 得分：`10/12`
- 决定：保留。该帧支持容器化服务、Fly.io 和可访问部署之间的关系。imagegen 移除了临时部署 URL 以及编辑器/Zoom 覆盖层，同时保留了这一关系。
- 发布素材：`images/docker-fly-deployment-flow-imagegen.jpg`
- README 引用位置：紧接 Fly.io 部署 URL 说明之后

### 候选 6：部署响应

- 时间戳：`01:36:51`
- 源帧：`deploy-response.jpg`
- 得分：`4/12`
- 决定：拒绝。它与部署候选重复，而且主要是临时的终端/URL 输出；已采用的部署插图足以表达持久流程。

## 发布决定

- 最终采用且仅采用两项有来源支持的素材。
- 在 `README.md` 中添加了这两项引用。
- 两张图都使用原始非 crisp 帧和局部裁剪图作为图像生成输入。
- 从发布图中移除人脸/摄像头画面以及编辑器/Zoom 覆盖层。
- 将准确代码、命令、URL、数值和验证 payload 保留在原生 Markdown 中，而不是要求 imagegen 重绘。
- 拒绝通用、重复、临时或无法诊断的截图。