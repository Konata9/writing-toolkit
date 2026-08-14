# Image Sourcing：图片搜索、下载与版权记录

为文章搜索、下载并记录公有领域/CC 许可的配图，从 Wikimedia Commons 等权威来源获取。

## 使用场景

- 写作流程中需要配图时
- 用户要求搜索图片 / 下载图片 / 找配图 / Wikimedia Commons 配图时
- 为正文寻找"文字先讲 → 图片验证"的视觉素材

## 能力

- 按来源优先级搜索（Wikimedia Commons → 博物馆馆藏 → 权威档案）
- 4 种下载方法（`Special:FilePath` → HTML 抓取 → REST API → 直接下载）
- 质量阈值验证（文件类型、大小、水印检测）
- 版权信息表生成（文件名 / 描述 / 来源 / 许可）

## 参数

| 参数 | 说明 | 默认 |
|------|------|------|
| `image_list` | 图片清单（类型 + 关键词） | 必填 |
| `output_dir` | 输出目录 | 必填 |
| `allow_ai_fallback` | 是否允许 AI 生成后备 | `false` |
| `image_types` | 允许的图片类型 | 调用方指定 |

## 边界与风险

- 搜索上限 10 个 query，不无限搜索
- AI 图仅在 `allow_ai_fallback=true` 时允许，且必须标注"（AI生成示意图）"
- 禁止使用水印图、来源不明的 Pinterest 图、无许可信息的百度/Bing 图
- 图片不是正文的阻断条件——不足 3 张时提供手动下载 URL，不阻塞写作

## 与 story-write 的关系

image-sourcing 与 [story-write](../story-write/) 并行：

- story-write 负责正文，image-sourcing 负责配图
- 两者可并行执行，最后由主编排技能把图片插入正文
