# Imagegen 来源记录：回归章节截图批次

本批次替换了恰好十个已发布的、由截图衍生的素材。每次重绘都向内置 imagegen 工具提供未经修改的原始 JPG，以及明确限定范围的 `-imagegen-crop.jpg` 参考图。裁剪图是构图或参考输入，并非最终素材，也不是缩放或锐化后的衍生图。原始 JPG 保持不变。

所有最终 PNG 都经过原生分辨率检查，以及缩放至 608px 宽后的检查；后者是本批次采用的课程显示尺寸。浏览器、摄像头、光标、选区、播放控件和手写叠加内容均已移除。代码、标签、数值、坐标轴、曲线和图示关系均与来源参考图进行核对。

| 目标文件 | 原始来源 | 原图裁剪坐标 (x, y, width, height) | 裁剪参考文件 | 最终 imagegen 输出 ID |
|---|---|---:|---|---|
| `01-car-price-intro-01-select-best-price-crisp.jpg` | `01-car-price-intro-01-select-best-price.jpg` | `(90, 70, 450, 240)` | `01-car-price-intro-01-select-best-price-imagegen-crop.jpg` | `exec-03448b54-7beb-45ee-8b27-2f4078ab88e5` |
| `01-car-price-intro-02-kaggle-dataset-crisp.jpg` | `01-car-price-intro-02-kaggle-dataset.jpg` | `(15, 50, 570, 280)` | `01-car-price-intro-02-kaggle-dataset-imagegen-crop.jpg` | `exec-e8d79784-b4b7-4735-88f0-1d414800768b` |
| `01-car-price-intro-03-kaggle-data-explorer-crisp.jpg` | `01-car-price-intro-03-kaggle-data-explorer.jpg` | `(0, 65, 598, 285)` | `01-car-price-intro-03-kaggle-data-explorer-imagegen-crop.jpg` | `exec-2d6a0af7-ba01-42a9-b7a1-87d9071f1a56` |
| `01-car-price-intro-04-msrp-column-crisp.jpg` | `01-car-price-intro-04-msrp-column.jpg` | `(0, 50, 598, 300)` | `01-car-price-intro-04-msrp-column-imagegen-crop.jpg` 以及旧版清晰度裁剪图 | `exec-f77aeb51-4dbe-4612-b5c0-e980999e927d` |
| `01-car-price-intro-06-github-repo-crisp.jpg` | `01-car-price-intro-06-github-repo.jpg` | `(0, 50, 598, 285)` | `01-car-price-intro-06-github-repo-imagegen-crop.jpg` | `exec-ce9d07be-34ce-439b-8d07-1aff25a68d7e` |
| `01-car-price-intro-07-chapter-files-crisp.jpg` | `01-car-price-intro-07-chapter-files.jpg` | `(0, 175, 598, 95)` | `01-car-price-intro-07-chapter-files-imagegen-crop.jpg` | `exec-cff023f5-31d4-4eae-a892-b2d4d84f5b99` |
| `03-eda-03-long-tail-distribution-crisp.jpg` | `03-eda-03-long-tail-distribution.jpg` | `(0, 90, 598, 270)` | `03-eda-03-long-tail-distribution-imagegen-crop.jpg` | `exec-8d3c75e9-6545-4806-b3b4-81f76c042425` |
| `03-eda-04-zoom-below-100k-crisp.jpg` | `03-eda-04-zoom-below-100k.jpg` | `(0, 150, 598, 210)` | `03-eda-04-zoom-below-100k-imagegen-crop.jpg` | `exec-3902e0e0-f706-461e-9a35-86bba0476ef8` |
| `03-eda-06-log1p-normal-distribution-crisp.jpg` | `03-eda-06-log1p-normal-distribution.jpg` | `(0, 95, 598, 265)` | `03-eda-06-log1p-normal-distribution-imagegen-crop.jpg` | `exec-84d080e6-1f37-497f-96e1-4319c09e5a53` |
| `04-validation-framework-01-train-val-test-split-crisp.jpg` | `04-validation-framework-01-train-val-test-split.jpg` | `(0, 60, 570, 300)` | `04-validation-framework-01-train-val-test-split-imagegen-crop.jpg` | `exec-e2875f78-747b-4437-aa19-5a68c32b082a` |

## 来源完整性

原始的非 crisp 版本 JPG 逐字节保留。以下记录了它们在本批次处理前后的 SHA-256 哈希值：

| 来源文件 | SHA-256 |
|---|---|
| `01-car-price-intro-01-select-best-price.jpg` | `ecef46bc06c7c0bf5a74b18f96997fef75649d6bd6ac498e0313dc0b86ce802e` |
| `01-car-price-intro-02-kaggle-dataset.jpg` | `8afdab00cde83da167e7b9a07072d693da3c02f98c62a568204e6a76ed8fc4c2` |
| `01-car-price-intro-03-kaggle-data-explorer.jpg` | `38cfc8b184356e53aac1f8efeda9614cd1cc33bfe4e139e902624f70d97d841b` |
| `01-car-price-intro-04-msrp-column.jpg` | `ad9207ebeb9bb73756d48d30409d2e3ac3141b98f7778b1aa12d6a422acd4c51` |
| `01-car-price-intro-06-github-repo.jpg` | `2ec66758aebaec9ed51e7777e116caf93bdf39402415520d4897d96252f0ab2d` |
| `01-car-price-intro-07-chapter-files.jpg` | `ac98f22d1ced6008328b07c7826e89a5f1d6a30040f4ff3fb5ffa7de8f1409ea` |
| `03-eda-03-long-tail-distribution.jpg` | `2e958c803ca9dbaee1ba93b68c6eefea1f306661ce958aeef8526cbd3b938c0e` |
| `03-eda-04-zoom-below-100k.jpg` | `23d4513b8d37850ae4051abef5958451aed6d22451a3f24382df182a69750874` |
| `03-eda-06-log1p-normal-distribution.jpg` | `3d02d0d251ae1cfb570268559e4a1461cb4263a2f0d58c5b58a34bc8f5437fe5` |
| `04-validation-framework-01-train-val-test-split.jpg` | `41ea5bba9757bbcabfa249a5521ae3756a2e8f04bd5ddfcb034cffe32cc14a45` |

视觉检查中被拒绝的输出包括：第一张验证框架重绘图使用了类似下标的标签；一张 MSRP 重绘图改变了可见表格数值；两张低于 100,000 的直方图重绘图包含多余标点或线条瑕疵。这些输出均未发布。

## 当前参考图专项审计：2026-09-09

当前 `*imagegen*.png` 参考图集合在模块 02 和 03 中共有十二个已发布目标。本节记录四个回归目标；八个分类目标记录在对应的分类台账中。标记为 `REGENERATED` 的三行现已具有可长期保留、针对各目标的证据：未经修改的原始 JPG、限定范围的原生裁剪图，以及新的内置 imagegen 输出。此前发布的 PNG 未作为 imagegen 输入。

| 已发布目标 | 原始来源 | 裁剪 `(x,y,width,height)` | 保留的裁剪图 | Imagegen 输出 | 来源 SHA-256 | 裁剪图 SHA-256 | 最终 SHA-256 | 最终尺寸 | 处理结果 |
|---|---|---:|---|---|---|---|---|---:|---|
| `05-linear-regression-simple-01-one-car-one-price-imagegen-pilot.jpg` | `05-linear-regression-simple-01-one-car-one-price.jpg` | `(25,0,470,340)` | `05-linear-regression-simple-01-one-car-one-price-imagegen-crop.jpg` | `exec-65bf2408-9fe3-4f95-aaae-1a171db00530` | `194180f3da215af459ed369029319d44d309ebb4559aaac314c6d39510b20d91` | `61a5b901691f74323965cea5284ac9e785c6fcb21bf02d419b1bda9ea3ea7752` | `88953eb79f8c0ca8eb7a35b6f8f0259ef71c716ac36004356cdc74bb13bb7daf` | `1475x1067` | **已重新生成 / 接受** |
| `11-feature-engineering-01-feature-engineering-imagegen.jpg` | `11-feature-engineering-01-year-column.jpg` | `(15,80,480,260)` | `11-feature-engineering-01-feature-engineering-imagegen-crop.jpg` | `exec-30b04bdd-9000-45de-b5c6-b42035e2e835` | `f5df307802221f1da8e1fc9537db31d8e578584cdb6c07b19ee18c75be22d737` | `13dec6cff48bc8c60cee880aa031de9b29a618c94a526c5d7668bbf63445f18d` | `09eb22ad57c57f260cf95296f8d45908ab04cf6e90fe0218acf1688a23e24f98` | `1942x809` | **已重新生成 / 接受** |
| `15-using-model-05-website-request-diagram-imagegen.jpg` | `15-using-model-05-website-request-diagram.jpg` | `(20,5,485,330)` | `15-using-model-05-website-request-diagram-imagegen-crop.jpg` | `exec-a898c763-66c5-4454-bd0c-998124bb9591` | `745465a2ca409edfc6fdaca5602d1a2fe98c8d35f7a9590fec0ec5182ce09c32` | `fdd65bee01a701c29481c290985c1e845c86247dcc6f0271de71dac03352f9ec` | `384bc5828616c1e4814d1a83c31d733f0cd2588e22369991317e4e39bab04fc0` | `1677x938` | **已重新生成 / 接受** |
| `17-explore-more-01-feature-experiments-imagegen.jpg` | `不适用 — 独立生成的素材，没有对应的原始 JPG` | `N/A` | `不适用 — 没有来源裁剪图` | `C2PA 声明 `urn:c2pa:97ea80b6-608a-4041-97f5-334a5122b325`; 执行 ID 未保留 | `N/A` | `N/A` | `2a61cbd5a94153f2d5d1a6a7999890eb657e3ec576e1355dbf3b5149998d382e` | `1672x941` | **未修改 / 接受** — 现有的 OpenAI C2PA imagegen 输出；未使用先前的目标图片 |

### 检查与不变条件

- 三个重新生成的输出均从内置 imagegen 结果逐字节复制，并在原生尺寸和模拟的 608px 课程宽度下检查：`05` 为 `608x440`，`11` 为 `608x253`，`15` 为 `608x340`。
- `05` 保留了 `g(x_i) ≈ y_i`、`A CAR` 和 `ITS PRICE` 标签、箭头，以及底部可见片段 `x_i = (x_i1`。`11` 保留了 `year` 及其值 `2008`、`2012`、`2016`，准确的 `2017 − year` 变换，`age` 及其值 `9`、`5`、`1`，以及 `Model` / `Input Matrix` 端点。`15` 保留了 `TOYOTA`、`SIENNA`、空表单字段、箭头、花括号和省略号。
- 人脸、摄像头画面、浏览器或编辑器界面、光标、播放控件、选区、黑边和水印叠加内容均已移除。创建已发布目标时，未使用放大、Lanczos 缩放或锐化。
- `17` 已是有效的独立内置 imagegen 输出，带有 OpenAI C2PA 元数据，因此逐字节保持不变。其原生尺寸和 608px 检查均已通过；按照这张独立插图的生成方式，本就没有原始 JPG 或限定范围的裁剪图。
- 三个原始 JPG 和三个限定范围的原生裁剪图仍保留在本目录中。上面的来源和裁剪图哈希值为重新生成的目标提供了可复现的输入链。
