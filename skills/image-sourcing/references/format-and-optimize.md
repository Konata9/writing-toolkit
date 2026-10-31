# 图片格式归一化与压缩

下载完成后的后处理：把 SVG / WebP 统一成下游可用的格式，并把过大的图片压到
合适体积。顺序固定：**先验证 → 再归一化格式 → 最后压缩体积**，每步都要复查。

---

## 依赖校验

```bash
command -v magick || echo "缺少 ImageMagick（brew install imagemagick）"
command -v dwebp || echo "缺少 webp 工具（brew install webp）"
command -v sips || echo "缺少 sips"                    # macOS 自带
command -v rsvg-convert || echo "缺少 librsvg（brew install librsvg，可选）"
command -v file || echo "缺少 file"
```

- `magick`（ImageMagick 7）是首选，`dwebp` / `rsvg-convert` / `sips` 为兜底。
- 若 `magick`、`dwebp`、`sips`、`rsvg-convert` 全缺，SVG/WebP 无法转换：保留原文件，
  在记录表标注 `needs-conversion`，并把来源 URL 交给调用方手动处理，不要硬转。
- `sips` 不支持 SVG，只作为 WebP 的最后兜底；`rsvg-convert` 单独存在时即可处理 SVG。

---

## 一、识别真实格式

不要相信扩展名——新闻 CDN 常把 WebP 存成 `.jpg`。

```bash
file --mime-type -b assets/<filename>
```

| mime 类型 | 处理 |
|-----------|------|
| `image/jpeg` | 跳过转换，直接进压缩判断 |
| `image/png` | 有透明或颜色数低（截图/图表）→ 保留 PNG；无透明的连续色调照片 → 按第三节转 JPEG |
| `image/webp` | 按第三节规则分流 |
| `image/svg+xml` | 转 PNG（第二节） |
| 其他（`text/html`、`text/plain` 等） | 不是图片，删除重下 |

> 命名规范要求「扩展名跟随真实格式」，所以照片存成 PNG 也要转回 JPEG，
> 否则体积会大 10 倍以上。

---

## 二、SVG → PNG

**优先让来源直接渲染 PNG**，比本地栅格化更准（字体替换、字重偏差更少）：

```bash
# Wikimedia：加 ?width= 直接返回 PNG
curl -sL --connect-timeout 8 --max-time 30 \
  "https://commons.wikimedia.org/wiki/Special:FilePath/<NAME>.svg?width=2000" \
  -o assets/<name>.png
```

> `?width=` 是**就近取标准档位**（实测 800→960、1200→1280、2000→3840），不是精确
> 2000px。拿到后必须 `magick identify` 确认实际尺寸，超了再走第四节压缩。

本地栅格化兜底（2000px 宽 + 300 DPI，保证细节）：

```bash
magick -background none -density 300 assets/<name>.svg -resize 2000x -strip assets/<name>.png

# 兜底
rsvg-convert -w 2000 assets/<name>.svg -o assets/<name>.png
```

规则：

- `-background none` 保留透明底；需要白底时改 `-background white -alpha remove -alpha off`
- SVG 是矢量，放大不损失清晰度，但**不要超过 2000px**（与压缩目标一致，网页用不上更大）
- 转换成功后删掉原 `.svg`，下游不认这个格式
- 记录表备注 `SVG→PNG`

**含文字的 SVG 报 `unable to read font`**：`magick` 在没有 `rsvg-convert` 时会退回内置
MSVG 渲染器，字体和滤镜支持很差，复杂 SVG 会走样或直接失败。处理顺序：

1. 回到来源直出 PNG（首选）
2. `brew install librsvg` 后重试（`magick` 会自动调用 `rsvg-convert` 委托）
3. 用 Playwright 打开 SVG URL 截图（`deviceScaleFactor` 设为 2，宽度对齐目标尺寸）

---

## 三、WebP / GIF / 无透明 PNG → PNG 或 JPEG

按内容分流，避免照片转 PNG 后体积爆炸（PNG 对连续色调压缩极差，200 KB 能涨到 4 MB）。

| 条件 | 输出 | 理由 |
|------|------|------|
| 有透明通道 | PNG | 透明底只能 PNG |
| 颜色数少（截图、图表、UI） | PNG | 文字边缘不能有 JPEG 伪影 |
| 连续色调照片 | JPEG | 体积合理，肉眼看不出差别 |

判断透明与颜色数：

```bash
magick identify -format "opaque=%[opaque] colors=%k\n" assets/<filename>
```

- `opaque=False` → 有透明 → PNG
- `colors` < 50000 → 截图 / 图表 → PNG
- 其余 → 照片 → JPEG

转换命令：

```bash
# → PNG
magick assets/<filename> -strip assets/<name>.png
dwebp assets/<filename> -o assets/<name>.png                    # 兜底
sips -s format png assets/<filename> --out assets/<name>.png    # macOS 兜底

# → JPEG（照片）：按最终尺寸 + 最终质量一次编码，避免 q88→q82 二次有损
# 长边 > 2400px 或原文件 > 3MB：直接缩到 2000px + q82
magick assets/<filename> -resize '2000x2000>' -strip \
  -sampling-factor 4:2:0 -quality 82 assets/<name>.jpg
# 已在目标尺寸内：不缩放，用 q88 保留更多细节
magick assets/<filename> -strip -sampling-factor 4:2:0 -quality 88 assets/<name>.jpg
```

规则：

- 上下文已明确是截图（推文、公告、GitHub Issue、UI 界面）→ 直接按 PNG 处理，不必看颜色数
- 动画 WebP / GIF 取首帧：`magick "assets/<filename>[0]" assets/<name>.png`
- 调用方明确要求全 PNG 时按调用方要求覆盖分流（接受体积代价，靠第四节缩尺寸）
- 扩展名必须同步改成真实格式，**禁止 `.jpg` 里装 PNG**
- 照片按上表判断是否需要在归一化阶段直接缩到 2000px；已一次编码到位的，第四节不再重复压
- 记录表备注 `WebP→PNG` 或 `WebP→JPEG`

---

## 四、压缩过大图片

**触发条件**（满足任一即压）：

- 文件 > 3 MB
- 长边 > 2400px

**目标**：长边 ≤ 2000px，文件目标 ≤ 1.5 MB（封面类不单独放宽）。

```bash
# 照片 JPEG
magick assets/<name>.jpg -resize '2000x2000>' -strip -interlace Plane \
  -sampling-factor 4:2:0 -quality 82 assets/<name>.jpg

# 截图 / 图表 / 带文字 PNG：保留 PNG，不转 JPEG
magick assets/<name>.png -resize '2000x2000>' -strip \
  -define png:compression-level=9 assets/<name>.png

# 可选：装了 pngquant 时再压一档（有损但可控）
pngquant --quality=80-92 --force --output assets/<name>.png assets/<name>.png
```

规则：

- `'2000x2000>'` 里的 `>` 表示**只缩不放**，小图不会被插值拉大失真
- 截图 / 图表**禁止转 JPEG**：文字边缘会出现块状伪影，可读性明显下降
- 归一化阶段已按最终尺寸 + 最终质量一次编码的照片，跳过本节，避免二次有损压缩
- **未达 1.5 MB 时的降级阶梯**（最多再压 1 轮，总计 2 轮）：
  1. 长边降到 1600px 再压一次
  2. 仍是照片，可把 quality 降到 75
  3. 仍超目标：保留当前结果，在记录表写明「已压 2 轮，仍 X MB」及原因，不反复压
- 默认覆盖原文件，不保留 `-orig` 副本；记录表写清体积变化
- 压缩完必须做第五节的验证，不能只看体积变小

---

## 五、转换 / 压缩后验证

```bash
file --mime-type -b assets/<name>.png
magick identify -format "%f %wx%h %b\n" assets/<name>.png
```

逐项确认：

- [ ] mime 与扩展名一致（PNG 就是 `image/png`，JPEG 就是 `image/jpeg`）
- [ ] 宽度 ≥ 600px（低于此丢弃，换图）
- [ ] 文件 ≥ 10 KB，或 `colors` < 50000 的纯色图形 / 截图（可能只有几 KB，别误删）
- [ ] 打开图片肉眼确认：文字可读、无块状伪影、透明底正常、内容未被裁切
- [ ] 记录表「处理」列已填（如 `WebP→PNG`、`压缩 4.1MB→1.1MB`）

---

## 反模式

| 反模式 | 后果 |
|--------|------|
| 把 `.webp` 直接改名成 `.png` | 文件内容没变，下游照样打不开 |
| 照片转 PNG 后不压缩 | 体积翻 10–20 倍，拖慢页面 |
| 截图转 JPEG 求"小" | 文字糊掉，读者看不清 |
| 先 q88 转 JPEG 再 q82 压缩 | 两次有损编码，画质白白损失 |
| 无限压缩循环 | 压到 200px 宽还在压，浪费预算 |
| 放大低清图充数 | 只是变模糊的大图，不算高清 |
