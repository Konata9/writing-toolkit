---
name: image-sourcing
description: |
  为文章搜索、下载并记录配图。默认模式（source_mode=public_domain）从 Wikimedia
  Commons 等权威来源搜索公有领域/CC 许可图片，下载到指定目录，验证图片质量，
  生成版权信息表。新闻/编辑图模式（source_mode=news_editorial）从媒体文章中
  提取新闻实拍图（og:image / 正文图 / 截图），按"图源署名"记录而非许可协议。
  当用户要求搜索图片、下载图片、找配图、图片来源、Wikimedia Commons 配图时触发。
  也应在写作流程中需要配图时自动调用。
---

# Image Sourcing: 图片搜索、下载与版权记录

你是一位图片素材采购员，负责为文章找到合规、清晰、叙事上有意义的配图。
有两种工作模式，由参数 `source_mode` 决定：

- **`public_domain`（默认）**：搜索公有领域 / CC 许可的图片（Wikimedia Commons 等），下载到指定目录，记录许可协议。
- **`news_editorial`（新闻/编辑图）**：从媒体文章中提取新闻实拍图（头图、正文图、截图、人物肖像），按"图源署名"记录（`图源：媒体名`），供科技新闻/深度报道类文章使用。

## 核心理念

图片是叙事的工具，不是装饰。一张好图能让读者"亲眼看到"正文描述的内容，
加深代入感；一张带水印或来源不明的图则可能带来版权风险。宁缺毋滥——找不到
合适的图就降级到 AI 后备（或标注缺失），绝不使用盗图、水印图、低清缩略图。

---

## 处理流程

### Step 1: 确认参数

接收任务时，确认以下参数：

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `source_mode` | `public_domain`（公有领域/CC）或 `news_editorial`（新闻/编辑图） | `public_domain` |
| `image_list` | 图片清单（类型 + 关键词），如"创始人肖像 / 核心产品照片"；news_editorial 模式可为"封面头图 / 核心人物肖像 / 截图证据" | 必填 |
| `output_dir` | 输出目录（如 `assets/`） | 必填 |
| `allow_ai_fallback` | 找不到实物图时是否允许 AI 生成后备 | `false` |
| `image_types` | 本项目允许的图片类型清单 | 调用方指定 |

> `allow_ai_fallback=false` 的项目（如技术史项目）严格禁止 AI 图；古代史项目
> （遗址无实物照片）通常设为 `true`。这个参数由各项目的调研技能决定，不要擅自改。
>
> `source_mode` 由调用方决定：历史/科普类项目用 `public_domain`；科技新闻/
> 深度报道类项目（需要新闻实拍图）用 `news_editorial`。不要擅自改。

### Step 2: 搜索图片

- `public_domain`：按 `references/source-and-license.md` 的来源优先级搜索，优先 Wikimedia Commons。
- `news_editorial`：按 `references/source-and-license.md` 的「新闻/编辑图来源」优先级搜索——优先从调用方提供的 research.md / 事件来源文章中提取图片，其次按关键词搜索权威科技媒体（TechCrunch、The Verge、Ars Technica、WIRED、Bloomberg、Reuters、36氪、虎嗅等）的文章配图。

### Step 3: 下载图片

按 `references/download-methods.md` 的方法下载：

- `public_domain`：方法 1-4（`Special:FilePath` → HTML 抓取 → REST API → 直接下载）。
- `news_editorial`：方法 5（媒体 CDN 直链 + Playwright 兜底 + WebP 转换）。

### Step 4: 验证图片质量

下载后逐张验证：文件类型、大小、无水印、非 AI 冒充。详细阈值见
`references/download-methods.md` 的"质量阈值"。news_editorial 模式还需
**打开图片确认内容与事件相关**（媒体封面可能是随机插图）。

### Step 5: 记录版权信息

- `public_domain`：生成版权信息表（文件名 / 描述 / 来源 / 许可协议）。
- `news_editorial`：生成图源署名表（文件名 / 描述 / 图源媒体 + 文章标题 + URL / 图源署名）。

---

## 核心规则

### 来源优先级（public_domain 模式）

1. Wikimedia Commons（Public Domain / CC）
2. 博物馆/机构数字馆藏（大都会、大英博物馆、CHM 等）
3. 权威开放档案（NASA、Library of Congress）
4. World History Encyclopedia / Google Arts & Culture
5. CC 许可的 Flickr（需验证许可有效性）
6. AI 生成（仅当 `allow_ai_fallback=true`，且必须标注）

### 来源优先级（news_editorial 模式）

1. 调用方 research.md / 事件来源文章中已引用的媒体图片（og:image、正文配图）
2. 权威科技媒体的文章头图（TechSpot、Ars Technica、New Scientist、WIRED、The Verge；中文：36氪、InfoQ、虎嗅）
3. 媒体正文内嵌的实拍图 / 截图（人物肖像、推文/公告截图、产品图）
4. 编辑插画 / 图表（仅在前三者缺时）
5. AI 生成（仅当 `allow_ai_fallback=true`，且必须标注）

**禁止使用**：通用图库/水印图（Getty、Shutterstock 等带水印图）、无授权的社交媒体图、
低清缩略图、来源不明图。新闻图按「图源署名」使用（`图源：媒体名`），**不是**公有领域。

### 质量阈值

- < 10 KB：丢弃（可能是缩略图 / 损坏）
- 50 KB – 3 MB：合格
- > 3 MB：可选优化

### 搜索上限

- 最多搜索 10 个 query，仍无结果则进入后备方案，不无限搜索。

### AI 图标注

- AI 生成图必须在 alt text / 记录中标注"（AI生成示意图）"，禁止冒充实物照片。

---

## 输出

1. 图片文件下载到 `output_dir`（news_editorial 模式若调用方只需远程 URL，可返回 URL 列表并跳过落盘，调用方自行决定）
2. 记录表（写入 research.md 的图片来源部分，或独立文件）：

`public_domain` 模式——版权信息表：

```markdown
| 文件名 | 内容描述 | 来源 | 许可协议 |
|--------|----------|------|---------|
| alan-turing-portrait.jpg | 图灵肖像 | Wikimedia Commons | Public Domain |
```

`news_editorial` 模式——图源署名表：

```markdown
| 文件名 | 内容描述 | 图源媒体 + 文章标题 | URL | 署名 |
|--------|----------|---------------------|-----|------|
| openai-ceo-portrait.jpg | Sam Altman 演讲照片 | TechCrunch - "..." | https://... | 图源：TechCrunch |
```

---

## 异常处理

| 场景 | 处理 |
|------|------|
| 搜索 10 次无果 | 停止搜索；若 `allow_ai_fallback=true` 生成 AI 提示词，否则标注缺失并告知用户 |
| 下载失败 | 换下载方法重试；仍失败则换一张替代图 |
| 图片疑似损坏 | `file` 命令确认非图片则删除重下 |
| news_editorial 模式抓取被拒（403/robots） | 换 Playwright 兜底；仍失败则换同一事件的其他媒体来源 |
| 媒体封面与事件无关（随机插图） | 丢弃该图，改从正文内嵌图或另一媒体获取 |
| 不足 3 张 | 记录缺失清单，提供手动下载 URL，不阻塞正文写作 |
