# Image Sourcing：图片搜索、下载与版权记录

为文章搜索、下载并记录配图。支持两种模式：

- **`public_domain`（默认）**：从 Wikimedia Commons 等权威来源获取公有领域/CC 许可图片
- **`news_editorial`（新闻/编辑图）**：从科技媒体文章中提取新闻实拍图（og:image、正文图、截图），按「图源署名」记录

## 使用场景

- 写作流程中需要配图时
- 用户要求搜索图片 / 下载图片 / 找配图 / Wikimedia Commons 配图时
- 为正文寻找"文字先讲 → 图片验证"的视觉素材
- 科技新闻 / 深度报道类文章需要新闻实拍图（封面头图、人物肖像、事件截图）时

## 能力

- 按来源优先级搜索（public_domain：Wikimedia Commons → 博物馆馆藏 → 权威档案；news_editorial：来源文章图片 → 权威科技媒体头图 → 正文截图）
- 5 种下载方法（`Special:FilePath` → HTML 抓取 → REST API → 直接下载 → 媒体 CDN 直链 + Playwright 兜底）
- 下载韧性：统一超时 + 有限重试 + 代理探测（直连失败才走代理），整任务 10 分钟预算，绝不无限等待
- 格式归一化：SVG → PNG；WebP 按内容转 PNG（透明/截图/图表）或 JPEG（照片）
- 大图压缩：> 3MB 或长边 > 2400px 压到长边 ≤ 2000px，目标 ≤ 1.5MB，截图保留 PNG 保文字清晰
- 质量阈值验证（文件类型、大小、水印检测、内容与事件相关）
- 版权信息表（public_domain）或图源署名表（news_editorial）生成，统一含「处理」列

## 参数

| 参数 | 说明 | 默认 |
|------|------|------|
| `source_mode` | `public_domain` 或 `news_editorial` | `public_domain` |
| `image_list` | 图片清单（类型 + 关键词） | 必填 |
| `output_dir` | 输出目录 | 必填 |
| `allow_ai_fallback` | 是否允许 AI 生成后备 | `false` |
| `image_types` | 允许的图片类型（硬白名单，不在清单内的图丢弃换图） | 调用方指定 |

## 边界与风险

- 搜索上限 10 个 query，不无限搜索
- 下载不无限重试：单 URL ≤60 秒（1 次 curl 调用含 `--retry 1`）、单图 2 种方法、整任务 10 分钟；超时先探代理，代理也不行就换图
- AI 图仅在 `allow_ai_fallback=true` 时允许，且必须标注"（AI生成示意图）"
- 禁止使用水印图、来源不明的 Pinterest 图、无许可信息的百度/Bing 图
- news_editorial 模式禁止使用通用图库水印图和低清缩略图；媒体封面必须确认内容后再用（可能是随机插图）
- 图片不是正文的阻断条件——不足 `image_list` 要求数量（未指定时按 3 张）时提供手动下载 URL，不阻塞写作

## 参考文件

- `references/source-and-license.md`：来源优先级与许可判定
- `references/download-methods.md`：5 种下载方法、超时/代理/重试预算、质量阈值
- `references/format-and-optimize.md`：SVG/WebP 格式归一化与大图压缩

## 与 story-write 的关系

image-sourcing 与 [story-write](../story-write/) 并行：

- story-write 负责正文，image-sourcing 负责配图
- 两者可并行执行，最后由主编排技能把图片插入正文
