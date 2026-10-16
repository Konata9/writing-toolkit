---
name: imagetext-cards
description: >-
  把素材（博客文章、草稿、大纲）生成可直接发布的图文卡片（贴图）与配套笔记文案，适用于小红书、
  微信贴图号等图文平台：固定版式引擎 + 可切换风格预设，HTML/CSS 渲染、截图导出 1080×1440 PNG，
  并做反 AI 味自检。当用户要求"把文章做成小红书卡片/贴图""生成小红书图文""做小红书配图"
  "把博客改成小红书""做成公众号贴图/图文卡片"时使用。不用于纯文案润色（用 humanize-plus）
  或平台合规扫描（用 publish-guard）。
---

# 图文卡片生成器（小红书 / 微信贴图号）

根据用户提供的素材，生成一套风格稳定、可直接发布的图文卡片（贴图）与配套笔记文案。

**核心模型：引擎固定，风格可变。**
卡片版式、字号层级、安全区、间距是「引擎」（`assets/core.css`，不变量）；
配色、字体、背景、装饰是「风格预设」（`references/styles/`，变量）。
换风格只换皮，不改引擎。平台差异（画布比例、外链/引流规则、标签）见 `references/platforms.md`。

> 一句话流程：`素材 → 补充(+source-verify) → 风格(选/研) → 出图 → 反AI味自检(仅建议) → 确认后改`

---

## 何时使用 / 不使用

**使用**：把一篇文章/草稿/大纲做成**图文卡片（贴图）**；生成配图与笔记文案；把已有内容"贴图化"。适用于**小红书、微信贴图号等图文平台**。

**不使用**：
- 只改文案、去 AI 味 → `humanize-plus`
- 只做平台合规扫描 → `publish-guard`
- 只做事实核验 → `source-verify`

---

## 不可违反的硬规则

1. **素材必须由用户提供。** 没有源素材就停下索要，绝不凭空编内容。
2. **禁止编造数据。** 数字、用户量、百分比、评价必须有来源或来自用户素材；否则删掉或标注"待补"。
3. **外链 / 引流按平台规则处理。** 小红书禁止正文外链与站外引流，统一用"商店/主页搜名字"；微信贴图号按公众号规则（可留链接，但避免硬导流）。详见 `references/platforms.md`。
4. **事实校验出现 `[unverified]` 的内容不得进入卡片**（与规则 2 一致）。
5. **发布需勾选「AI 生成内容」声明**（贴图由工具渲染）。
6. **反 AI 味检查只给建议，不自动改**；必须经用户确认后才修改。
7. **不重画 OS/浏览器 chrome。** 界面展示用真实截图；确无截图时才用最小化的 `.mock` 概念示意。

---

## 依赖与降级

- **渲染依赖 Playwright**（MCP，或本地 `npx playwright`）。缺失时**停止导出**并提示 `npx playwright install chromium`；**不要伪造图片或结果**。
- 可选能力：事实校验 `source-verify`、配图 `image-sourcing`、文案润色 `humanize-plus`、合规 `publish-guard`。对应能力缺失时跳过该步并说明。
- 字体：默认用 Google Fonts（Noto Sans SC / Space Grotesk / JetBrains Mono）；无网环境回退系统 PingFang SC，版式基本一致。

---

## 六步流程

详细话术与判定见 `references/workflow.md`。

| 步 | 做什么 | 关键点 |
|----|--------|--------|
| **1 · 收素材** | 读取用户给的博客/草稿/大纲 | 没有 → 停止索要；存为输出目录的 `source.md` |
| **2 · 补充 + 风格** | **合并为一次提问**：补充项（官方文档？图片需求？事实校验？）+ 目标平台（默认小红书）+ 想要的风格 | 图片 → `Skill(image-sourcing)`；校验 → `Skill(source-verify)`；平台差异见 `references/platforms.md` |
| **3 · 定风格分支** | 命中 `references/styles/` → 列预设让用户选；未命中 → 检索该平台高分笔记 → 出预设草案 → 确认后登记 | 风格问题在 Step 2 已一并问出，此处不再重复问 |
| **4 · 生成卡片** | 选叙事骨架→拆 6–8 卡→（条件项）大纲确认→套预设+填模板→导出 PNG+写文案 | 见 `references/content-templates.md`、`references/export-and-qc.md` |
| **5 · 反 AI 味自检** | 按 `references/anti-slop.md` 逐条核对，输出**建议报告**（真该改 / 本风格豁免） | **只读，不改文件** |
| **6 · 确认后修改** | 用户勾选建议 → 只改这些项 → 只重导出受影响卡片 | 文案去 AI 味可调 `Skill(humanize-plus)`；合规可调 `Skill(publish-guard)` |

> 「大纲确认」为**条件项**：卡数 ≥6 / 新风格 / 涉及多个数字事实时**必做**；短内容且沿用既有系列风格时可跳过。确认内容为标题候选 + 每图要点。

---

## 输出目录

生成物建议放在一个**专门的图文卡片项目**下（与博客/代码仓库解耦），每个系列一个目录：

```
<cards-project>/<slug>/
├── README.md          # 发布说明：标题候选/正文/标签/发布时间
├── post-1-<name>.md   # 帖子一完整文案
├── post-2-<name>.md   # （可选）系列第二篇
├── src/
│   ├── core.css       # 从本 skill assets/ 复制
│   ├── post1.html     # 由 post-template.html 填充
│   ├── post2.html
│   └── assets/        # logo、截图等素材
└── cards/
    ├── post1/01-cover.png … 08-cta.png
    └── post2/…
```

例：`~/Workspace/imagetext-posts/<slug>/`。若用户没有专门的卡片项目，再回落到当前项目的 `docs/cards/<slug>/`。

---

## 跨 SKILL 复用（不重复造轮子）

- **事实校验** → `Skill(source-verify)`：对关键事实交叉验证，输出报告；`[unverified]` 不进卡片。
- **配图** → `Skill(image-sourcing)`：搜索/下载/记录许可图片。
- **文案去 AI 味** → `Skill(humanize-plus)`：润色笔记正文与卡片文案。
- **平台合规** → `Skill(publish-guard)`：禁用词/灰色词/绝对化/AI 声明检查。

---

## References 索引（按需加载，不要全读）

| 文件 | 何时读 |
|------|--------|
| `references/workflow.md` | 每次生成前：六步细则 + 向用户提的问题 + 自检报告格式 |
| `references/platforms.md` | 第 2 步起：平台差异（画布/外链/标签/AI 声明）+ 平台专属规则 |
| `references/design-system.md` | 改版式/组件，或新增风格预设前：引擎规范与主题变量契约 |
| `references/anti-slop.md` | 第 5 步自检：卡片向反 AI 味规则 + 豁免清单 + 清单 |
| `references/content-templates.md` | 第 4 步：叙事骨架 + 封面钩子 + 标题/标签公式 |
| `references/export-and-qc.md` | 第 4/6 步：Playwright 导出 + 卡片专属 QC |
| `references/styles/_index.md` | 第 3 步：风格登记表 + 新增风格方法 |
| `references/styles/<style>.md` | 选定风格后：取该风格的 token 与 do/don't |
| `assets/core.css` | 每次：复制到项目 `src/`，不修改引擎本身 |
| `assets/post-template.html` | 每次：复制并填充 |
