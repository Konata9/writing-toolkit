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
| `image_types` | 本项目允许的图片类型清单（**硬白名单**，类型定义见 `references/download-methods.md` 的「图片类型覆盖」）；不在清单内的图直接丢弃换图 | 调用方指定 |

> `allow_ai_fallback=false` 的项目（如技术史项目）严格禁止 AI 图；古代史项目
> （遗址无实物照片）通常设为 `true`。这个参数由各项目的调研技能决定，不要擅自改。
>
> `source_mode` 由调用方决定：历史/科普类项目用 `public_domain`；科技新闻/
> 深度报道类项目（需要新闻实拍图）用 `news_editorial`。不要擅自改。

### Step 2: 搜索图片

- `public_domain`：按 `references/source-and-license.md` 的来源优先级搜索，优先 Wikimedia Commons。
- `news_editorial`：按 `references/source-and-license.md` 的「新闻/编辑图来源」优先级搜索——优先从调用方提供的 research.md / 事件来源文章中提取图片，其次搜索权威科技媒体（**媒体清单以该文件为准，此处不重复**）。

### Step 3: 下载图片

按 `references/download-methods.md` 的方法下载：

- `public_domain`：方法 1-4（`Special:FilePath` → HTML 抓取 → REST API → 直接下载）。
- `news_editorial`：方法 5（媒体 CDN 直链 + Playwright 兜底）。

所有下载带统一超时参数（`--connect-timeout 8 --max-time 30 --retry 1`）。出现超时 /
连接失败（curl 退出码 6/7/28/35）时，先探测代理再重试；**重试有硬上限**——单 URL 一次
调用内最多 2 个请求（含 `--retry 1`，约 60 秒），失败只换方法，不再重跑同一条命令；
单图 2 种方法、整任务 10 分钟。详见 `references/download-methods.md` 的「超时、代理与重试预算」。

### Step 4: 验证与归一化

按顺序处理，命令见 `references/format-and-optimize.md`：

1. **类型与内容验证**：`file --mime-type` 确认是图片（不是 HTML / 文本）、无水印、非 AI 冒充，
   且类型在 `image_types` 白名单内（不在则丢弃换图）。news_editorial 模式还需
   **打开图片确认内容与事件相关**（媒体封面可能是随机插图）。此步不判断体积。
2. **格式归一化**：SVG → PNG；WebP 按内容分流（透明底 / 截图 / 图表 → PNG，
   连续色调照片 → JPEG）。扩展名必须与真实格式一致。
3. **压缩**：> 3 MB 或长边 > 2400px 的图片，压到长边 ≤ 2000px，目标 ≤ 1.5MB；
   文字类截图保留 PNG 不转 JPEG。
4. **复验**：重新确认尺寸、体积、宽度 ≥ 600px，并打开图片确认文字可读、无块状伪影。

### Step 5: 记录版权信息

- `public_domain`：生成版权信息表（文件名 / 描述 / 来源 / 许可协议 / 处理）。
- `news_editorial`：生成图源署名表（文件名 / 描述 / 图源媒体 + 文章标题 + URL / 图源署名 / 处理）。

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
2. 权威科技媒体的文章头图（媒体清单见 `references/source-and-license.md` 的「新闻/编辑图来源」，此处不重复）
3. 媒体正文内嵌的实拍图 / 截图（人物肖像、推文/公告截图、产品图）
4. 编辑插画 / 图表（仅在前三者缺时）
5. AI 生成（仅当 `allow_ai_fallback=true`，且必须标注）

**禁止使用**：通用图库/水印图（Getty、Shutterstock 等带水印图）、无授权的社交媒体图、
低清缩略图、来源不明图。新闻图按「图源署名」使用（`图源：媒体名`），**不是**公有领域。

### 质量阈值

完整阈值表见 `references/download-methods.md` 的「质量阈值」，此处只列判定要点：

- 宽 < 600px：丢弃（低清缩略图）
- < 10 KB：先用 `file` 确认；非图片或照片类高色数图丢弃，纯色图形 / 截图可保留
- > 3 MB 或长边 > 2400px：压缩到长边 ≤ 2000px，目标 ≤ 1.5MB

### 搜索与下载上限

- 最多搜索 10 个 query，仍无结果则进入后备方案，不无限搜索。
- 下载预算：单 URL ≤60 秒（1 次 curl 调用，含 `--retry 1`）、单图 2 种方法、单图 ≤3 分钟、
  整任务 ≤10 分钟。超预算即停止，输出已有图片 + 缺失清单，不阻塞正文写作。
- 计时口径：从第一次下载请求到最后一张图验证完成；搜索、压缩、内容确认不计入。

### AI 图标注

- AI 生成图必须在 alt text / 记录中标注"（AI生成示意图）"，禁止冒充实物照片。

---

## 输出

1. 图片文件下载到 `output_dir`（news_editorial 模式若调用方只需远程 URL，可返回 URL 列表并跳过落盘，调用方自行决定）
2. 记录表（写入 research.md 的图片来源部分，或独立文件）：

`public_domain` 模式——版权信息表：

```markdown
| 文件名 | 内容描述 | 来源 | 许可协议 | 处理 |
|--------|----------|------|---------|------|
| alan-turing-portrait.jpg | 图灵肖像 | Wikimedia Commons | Public Domain | 压缩 4.1MB→1.1MB |
```

`news_editorial` 模式——图源署名表：

```markdown
| 文件名 | 内容描述 | 图源媒体 + 文章标题 | URL | 署名 | 处理 |
|--------|----------|---------------------|-----|------|------|
| openai-ceo-portrait.jpg | Sam Altman 演讲照片 | TechCrunch - "..." | https://... | 图源：TechCrunch | WebP→JPEG |
```

「处理」列填格式转换与压缩结果（`SVG→PNG`、`WebP→PNG`、`WebP→JPEG`、
`压缩 4.1MB→1.1MB`），未处理则留 `-`。

---

## 异常处理

| 场景 | 处理 |
|------|------|
| 搜索 10 次无果 | 停止搜索；若 `allow_ai_fallback=true` 生成 AI 提示词，否则标注缺失并告知用户 |
| 下载超时 / 连不上（退出码 6/7/28/35） | 先探测代理（环境变量 → `scutil --proxy` → 本地 7890 等端口，并实测能出网），有则带 `-x` 重试 |
| 代理重试后仍失败 | 走 Playwright 兜底（两种模式都可用）；仍失败则换图，记录 `failed: timeout` |
| 下载重试超预算 | 停止该图，不无限重试；整任务超 10 分钟直接输出已有结果 + 缺失清单 |
| 403 | 不走代理（换代理无用）；换 UA、走 Playwright 或换来源 |
| 404 | 不走代理；直接换来源或换文件名 |
| 429（限流） | 退避 5–10 秒重试 1 次；仍 429 则换来源，不连续重试 |
| 图片是 SVG / WebP | 按 `references/format-and-optimize.md` 归一化：SVG→PNG，WebP 按内容转 PNG 或 JPEG |
| 图片 > 3MB 或长边 > 2400px | 压到长边 ≤ 2000px，目标 ≤ 1.5MB；最多再压 1 轮（总计 2 轮），仍超则保留并记录原因 |
| 类型不在 `image_types` 白名单 | 丢弃换图，不将就使用 |
| 图片疑似损坏 | `file` 命令确认非图片则删除重下 |
| news_editorial 模式抓取被拒（403/robots） | 换 Playwright 兜底；仍失败则换同一事件的其他媒体来源 |
| 媒体封面与事件无关（随机插图） | 丢弃该图，改从正文内嵌图或另一媒体获取 |
| 不足 `image_list` 要求数量（未指定时按 3 张） | 记录缺失清单，提供手动下载 URL，不阻塞正文写作 |
