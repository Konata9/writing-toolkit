# 科技风 · 电光钴蓝（tech-cobalt）

由本仓库首个实例（MiaoMint × Vibe Coding 卡片）沉淀而来。适用于编程 / 开发 / 工具 / 产品的图文卡片。

> 本预设**等于 `assets/core.css` 的默认主题**。使用时可省略覆盖块；但建议仍显式粘贴，便于日后切换或对比其它风格。

## 适用 / 不适用
- **适用**：软件工具、开发经验、技术复盘、AI/效率工具。
- **不适用**：美食、萌宠、母婴等需要暖/软调性的内容（另建风格）。

## 调性关键词
冷静 · 克制 · 精确 · 工程感（像被精心排版的工程文档 / 一个 IDE 面板）

## 色板（粘贴到卡片 HTML 的 `:root`）
```css
:root {
  --paper:#f6f7fa; --paper-2:#eef1f6; --paper-3:#e6eaf2; --surface:#ffffff;
  --ink:#171b24; --ink-2:#3c4353; --ink-3:#6b7280; --ink-4:#98a0ae;
  --rule:#e4e7ee; --rule-2:#d3d8e2;
  --accent:#2352e0; --accent-2:#1b41bc; --accent-soft:#e9eeff; --accent-line:#c3d0ff;
  --accent-ink:#ffffff; --accent-on-dark:#8fb0ff;
  --highlight:#ff7a3c; --highlight-soft:#ffede4; --highlight-ink:#d1541f;
  --dark:#14171f; --dark-2:#1d2230; --dark-3:#262c3b;
  --dark-ink:#eef1f8; --dark-ink-2:#a9b2c4; --dark-rule:#2b3140;
  --cover-paper:linear-gradient(180deg,#fbfcff 0%,var(--paper) 46%);
  --cover-tint:linear-gradient(180deg,#eef2ff 0%,#e5ebff 58%,#dde5ff 100%);
  --cover-tint-glow:#cdd9ff; --cover-paper-glow:var(--accent-soft);
  --font-display:"Space Grotesk","Noto Sans SC","PingFang SC",sans-serif;
  --font-body:"Noto Sans SC","PingFang SC","Hiragino Sans GB",sans-serif;
  --font-mono:"JetBrains Mono","SF Mono",Menlo,monospace;
}
```

- 冷白纸面 `#f6f7fa`（不用纯白，更工程感）；
- **accent 电光钴蓝** = 专业/信任/科技，用于结构件与链接；
- **highlight 暖橙** = 突出数字/关键/CTA，用量克制（≈10%）；
- 中性炭黑文字，发丝线做结构。

## 字体
- 中文 **Noto Sans SC（思源黑体）**，英文/数字 **Space Grotesk**，命令/键位/标签/页码 **JetBrains Mono**。
- Google Fonts：
  ```
  https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@500;600;700&family=Noto+Sans+SC:wght@400;500;700;900&family=JetBrains+Mono:wght@400;500;700&display=swap
  ```
- 无网环境回退 PingFang SC，版式基本一致。

## 背景与封面偏好
- 常规卡：纸面底色 + 顶部极轻点阵纹理（仅封面用）。
- 封面默认 **浅钴蓝着色**（`.card--cover-tint`）；系列第二篇可用 **纸面封面**（`.card--cover`）区分。
- **深色重音**（`.card--dark`）可选，**≤1 张卡**使用（如封面或结尾），不宜两类封面对撞。

## 装饰语言
- 结构靠 **1px 发丝线**，不靠色块/粗边框。
- 命令与键位用等宽字与键帽（`.cmd` / `.key`），这是本风格的标识。
- 阴影克制贴合；不使用光晕/玻璃/霓虹。
- 允许最多一处强调色高亮关键词。

## Do / Don't
| Do | Don't |
|----|-------|
| 近白底 + 钴蓝主 + 橙点缀 | 紫蓝渐变、霓虹、纯黑底大字 |
| 等宽字承载"命令/键位/标签" | 等宽字滥用为大段正文 |
| 发丝线与留白 | 卡中卡、色块堆叠 |
| 真实截图/产品界面 | 手绘浏览器/IDE chrome |
| 系列统一封面模板 | 每篇换封面风格 |

**本风格豁免项**（自检时保留、不报为问题）：emoji（克制）、大字号短标题、系列角标、等宽标签、功能型 pill、统一封面模板、一处高亮关键词。

## 参考来源
- 调性参考：MiaoMint 官网主题（Cobalt · 冷白纸面 + 电光钴蓝 + 石墨暗色重音）。
- 首实例产出：本仓库 `docs/xiaohongshu/miaomint-vibecoding/`。
