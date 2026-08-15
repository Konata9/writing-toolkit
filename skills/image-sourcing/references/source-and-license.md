# 图片来源、许可与命名规范

本文件包含 image-sourcing 的来源优先级、许可协议判断、AI 后备方案与命名规范。

---

## 一、来源优先级（public_domain 模式）

| 优先级 | 来源 | 许可 | 搜索方式 |
|--------|------|------|---------|
| 1 | Wikimedia Commons | Public Domain / CC BY / CC BY-SA | 直接搜索 |
| 2 | 博物馆数字馆藏（大都会 Open Access、大英博物馆、CHM） | CC0 / CC BY-NC-SA 4.0 | 官网搜索 |
| 3 | 权威开放档案（NASA、Library of Congress） | 政府作品通常 Public Domain | 直接搜索 |
| 4 | World History Encyclopedia / Google Arts & Culture | 多数 CC BY-NC-SA | 网站搜索 |
| 5 | CC 许可的 Flickr | 需验证许可有效性 | 搜索后核对许可 |

### 禁止使用的来源

- 未经授权的个人博客图片
- 带水印的商业图库（Getty、Shutterstock 等）
- 来源不明的 Pinterest 图片
- 百度图片、Bing 图片（无许可信息）
- AI 生成的"历史复原图"冒充实物照片（AI 图仅可作氛围图且必须标注）

---

## 一·B、新闻/编辑图来源（source_mode=news_editorial）

**适用场景**：科技新闻、深度报道类文章需要新闻实拍图（封面头图、核心人物肖像、
事件截图、产品图）。新闻图受版权保护，按「图源署名」合理使用（`图源：媒体名`），
**不是**公有领域/CC 许可——不适用上表的许可协议判断，改记录图源署名。

### 来源优先级

| 优先级 | 来源 | 说明 |
|--------|------|------|
| 1 | 调用方 research.md / 事件来源文章中已引用的图片 | 文章头图（og:image）、正文配图——与内容最相关，优先 |
| 2 | 权威科技媒体的文章头图 | TechSpot、Ars Technica、New Scientist、WIRED、The Verge；中文：36氪、InfoQ、虎嗅（题图质量高） |
| 3 | 媒体正文内嵌实拍图 / 截图 | 人物肖像（专访照片）、推文/公告/GitHub Issue 截图、产品图 |
| 4 | 编辑插画 / 图表 | 仅在前三者缺时 |
| 5 | AI 生成 | 仅当 `allow_ai_fallback=true`，且必须标注"（AI生成示意图）" |

### 识别技巧（判断图片 URL 路径）

- URL 含 `image`、`wp-content/uploads`、`images2/news/bigimage` → 通常是高质量头图
- URL 含 `i.insider.com`、`GettyImages` → 通常是人像
- URL 含 `hsossms`（36氪 CDN）、`resource/image`（InfoQ CDN）→ 多是正文内嵌截图
- 中文媒体（36氪、InfoQ）文章往往内嵌大量截图，是最方便的截图来源

### 禁止使用的来源（news_editorial 模式）

- 带水印的商业图库图（Getty、Shutterstock 水印版）
- 低清缩略图（宽 < 600px）
- 无授权的社交媒体搬运图
- **内容与事件无关的媒体封面**：有些网站（如 InfoQ）封面是随机插图（猫、建筑），
  必须打开图片确认内容后再使用

### 图源署名记录格式

| 文件名 | 内容描述 | 图源媒体 + 文章标题 | URL | 署名 |
|--------|----------|---------------------|-----|------|
| openai-ceo-portrait.jpg | Sam Altman 演讲照片 | TechCrunch - "..." | https://... | 图源：TechCrunch |

正文插入格式统一为：

```markdown
![图片描述](图片URL)
*▲ 中文说明文字（图源：来源名）*
```

---

## 二、许可协议判断

在文件页面确认许可，优先顺序：

1. **Public Domain / CC0**（最优先，可自由使用）
2. **CC BY / CC BY-SA**（需署名，可自由使用）
3. **CC BY-NC-SA**（非商业使用，需署名，注意公众号/小红书是否算商业用途）

**记录**：每张图片在版权信息表记录完整许可类型。

---

## 三、AI 图片后备方案

当 `allow_ai_fallback=true` 且以下情况发生时，改用 AI 生成：

- Wikimedia Commons 搜索 10 次无合适结果
- 该时期无现存遗址照片（如古代夯土宫殿仅有地基）
- 需要氛围图而非实物照片

### 流程

1. 停止搜索，在版权信息表标注哪些图片无法找到
2. 为每张缺失图片生成中文提示词，写入 `ai-prompts.md`：

```markdown
# AI 图片生成提示词

## 图片 1：[用途]
**提示词**：[中文提示词，30-80 字]
**风格参考**：[如"仿古地图风格""羊皮纸暖色调""写实考古复原图"]
**建议尺寸**：16:9 横版（公众号）/ 3:4 竖版（小红书）
```

### 提示词编写原则

- 用中文，描述具体场景而非抽象概念——"夕阳下的金字塔，沙漠，暖金色光线，广角仰拍"而非"埃及文明"
- 包含：主体 + 环境 + 光线/氛围 + 构图角度
- 风格指定为"写实考古复原图"或"仿古地图风格"，避免"动漫""概念艺术"

### 强制标注

- 图片文件名末尾加 `-ai`
- alt text / 记录中标注"（AI生成示意图）"
- `ai-prompts.md` 末尾注明：**「以上图片由 AI 生成，仅作氛围配图，非历史事实影像」**

---

## 四、命名规范

- 小写英文 + 连字符
- `.jpg` 优先；`.png` 仅透明度需要时用
- 描述内容：`{subject}-{descriptor}.jpg`
- 示例：`alan-turing-portrait.jpg`、`05-greek-plato-academy.jpg`
- 若项目有前缀约定（如章节号 `05-`），遵循项目约定

---

## 五、版权记录格式

在 research.md 的图片来源部分或独立文件记录：

```markdown
| 文件名 | 内容描述 | 来源 | 许可协议 |
|--------|----------|------|---------|
| 05-greek-socrates-statue.jpg | 苏格拉底大理石雕像，罗马复制品，约公元1世纪 | Wikimedia Commons | Public Domain |
```
