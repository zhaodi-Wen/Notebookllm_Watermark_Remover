# PDF Watermark Remover

自动检测并去除 PDF 文件中的水印，支持 NotebookLM 等工具导出的图片型 PDF 和传统结构型 PDF。提供 Web UI 界面，支持上传、预览、导出 PDF 和每页图片。

## 功能

- **像素级水印清除** — 针对每页为整张图片的 PDF（NotebookLM、Google Slides 导出等），水印烧录在像素中
- **结构层水印清除** — 针对含文本/图片对象的传统 PDF 水印
- **Web UI** — 拖放上传、灵敏度可调、逐页预览
- **导出 PDF** — 下载去水印后的 PDF 文件
- **导出图片** — 每页导出一张图片（PNG/JPG），DPI 可选，ZIP 打包下载

## 快速开始

### 1. 安装依赖

```bash
pip install -r requirements.txt
```

### 2. 启动服务

```bash
python app.py
```

访问 http://127.0.0.1:5000

### 3. 使用

1. 拖放或点击上传 PDF 文件（最大 100MB）
2. 选择检测灵敏度（低 / 中 / 高）
3. 点击「开始去除水印」
4. 预览结果，翻页浏览
5. 导出 PDF 或图片

## 项目结构

```
├── app.py                 # Flask 后端 + WatermarkRemover 引擎
├── templates/
│   └── index.html         # Web 前端 UI
├── requirements.txt       # Python 依赖
├── uploads/               # 上传文件临时目录
└── outputs/               # 处理结果输出目录
```

## 技术栈

| 组件 | 技术 |
|------|------|
| 后端 | Python + Flask |
| PDF 引擎 | PyMuPDF (fitz) |
| 图像处理 | Pillow + NumPy |
| 前端 | 原生 HTML/CSS/JS，暗色主题 |

## 核心算法

### 图片型 PDF（如 NotebookLM 导出）

1. **类型检测** — 前 3 页无文本且有图片 → 图片型 PDF
2. **逐页扫描** — 右下角 30px × 250px 区域
3. **智能背景采样** — 找区域内前景最少的 5 行取中位色
4. **前景覆盖** — 低阈值（diff > 8）检测所有非背景像素
5. **平滑过渡** — 膨胀 3px + 高斯模糊 1px 边缘过渡
6. **写回 PDF** — `page.replace_image()` 替换原始图片

### 结构型 PDF

1. **品牌关键词命中** — NotebookLM 等直接判定
2. **综合评分** — 关键词、颜色、字号多维度打分
3. **Redaction 覆盖** — 使用 PDF redaction annotation 删除水印

## API 接口

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/api/upload` | 上传 PDF 文件 |
| POST | `/api/process` | 处理去水印（参数：task_id, filename, sensitivity） |
| GET | `/api/preview/{task_id}/{page_num}` | 预览指定页面 |
| GET | `/api/page_count/{task_id}` | 获取总页数 |
| GET | `/api/export/pdf/{task_id}` | 下载去水印 PDF |
| GET | `/api/export/images/{task_id}` | 下载每页图片 ZIP（参数：dpi, format） |

## 灵敏度说明

| 等级 | 说明 |
|------|------|
| 低（low） | 仅去除明显水印，误删风险最低 |
| 中（medium） | 推荐，平衡效果与安全 |
| 高（high） | 激进去除，可能误删部分浅色内容 |

## 依赖

```
flask==3.1.0
PyMuPDF==1.25.3
Pillow==11.1.0
gunicorn==23.0.0
```

## License

MIT
