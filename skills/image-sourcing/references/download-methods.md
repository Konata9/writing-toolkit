# 图片下载方法

本文件包含 image-sourcing 的图片下载方法（按顺序尝试）、超时/代理/重试预算与质量阈值。

---

## 依赖校验

下载与验证依赖以下系统工具，执行前先确认存在，缺失则安装：

```bash
command -v curl || echo "缺少 curl"
command -v file || echo "缺少 file"
command -v python3 || echo "缺少 python3"   # 仅方法 3 需要
command -v grep || echo "缺少 grep"
command -v magick || echo "缺少 ImageMagick（brew install imagemagick）"  # 格式转换/压缩需要
command -v dwebp || echo "缺少 webp 工具（brew install webp）"
command -v sips || echo "缺少 sips"                    # macOS 自带
command -v rsvg-convert || echo "缺少 librsvg（brew install librsvg，可选）"
```

- macOS/Linux 默认自带 `curl`、`file`、`grep`；`python3` 通常在 macOS 预装（无则 `brew install python`）。
- macOS 的 `grep` 为 BSD 版本，**不支持 `-P`**，本文所有正则均用 `-E`（扩展正则），勿改用 `-P`。
- 格式转换与压缩的依赖降级规则见 `format-and-optimize.md`。

---

## 超时、代理与重试预算（所有方法通用）

### 统一 curl 参数

**本文所有 `curl` 命令都必须带这组参数**（示例里为可读性有时省略，实际执行时补上）：

```bash
# 用数组，不要用字符串：zsh 不会对未加引号的变量做空格拆分，
# 写成 CURL_OPTS="..." 会让整串被当成一个参数，curl 直接报 unknown option
CURL_OPTS=(--connect-timeout 8 --max-time 30 --retry 1 --retry-delay 1 --retry-connrefused)
```

完整写法：

```bash
curl -sL "${CURL_OPTS[@]}" -A "<UA>" "<URL>" -o assets/<target-filename>.jpg
```

- `--connect-timeout 8`：连不上 8 秒就放弃，别挂死
- `--max-time 30`：单次请求总时长上限（含传输）
- `--retry 1 --retry-delay 1 --retry-connrefused`：瞬时错误自动重试 1 次

### 重试预算（防任务无限拖长）

| 层级 | 上限 | 超限动作 |
|------|------|----------|
| 单次请求 | 连接 8s / 总 30s | 判为 timeout，进入下一步 |
| 单个 URL | **1 次 curl 调用**，内含 2 个请求（首次 + `--retry 1`），≤ 60s | 换下载方法 |
| 单张图片 | 2 种下载方法（≤ 120s，加代理探测约 2.5 分钟） | 换一张替代图 |
| 单张图片总耗时 | 3 分钟 | 记录 `failed: timeout`，换图 |
| 整个任务 | 10 分钟 | 停止下载，输出已有图片 + 缺失清单 |

**计数口径**：`--retry 1` 就是那 1 次重试，一次 curl 调用内最多 2 个请求；
**不要手动重跑同一条命令**，失败就换方法，否则实际请求数会翻倍。

**计时口径**：从第一次下载请求开始，到最后一张图验证完成；搜索、压缩、内容确认
不计入。Playwright 兜底**不计入单图 3 分钟**，但计入整任务 10 分钟。

**硬性要求**：任何情况下都不许"一直等下去"或无限重试。到达预算上限就停止该图，
把失败原因写进记录表，继续处理下一张。

### 代理探测（仅在直连失败后触发）

**触发条件**——curl 退出码为下列之一，说明是网络层问题，值得换代理再试：

| 退出码 | 含义 |
|--------|------|
| 6 | 无法解析域名（DNS） |
| 7 | 连接失败 |
| 28 | 超时 |
| 35 | SSL 握手失败 |

**不触发**：`403` / `404` / `429`——站点拒爬、资源不存在或限流，换代理没用：

- `403`：换 UA、走 Playwright，或换来源
- `404`：直接换来源 / 换文件名
- `429`：退避 5–10 秒重试 1 次；仍 429 则换来源，不连续重试

探测顺序（找到第一个可用代理即停）：

```bash
# 1) 环境变量
env | grep -iE '^(http|https|all)_proxy='

# 2) macOS 系统代理
scutil --proxy | grep -E 'HTTPSEnable|HTTPSProxy|HTTPSPort'

# 3) 本地常见代理端口
for p in 7890 7897 1087 1080 6152 8118; do
  nc -z -w 1 127.0.0.1 $p 2>/dev/null && echo "可用代理端口: $p"
done
```

**端口开着不等于能出网**，必须实测一次再依赖它：

```bash
curl -sI --max-time 10 -x http://127.0.0.1:7890 https://commons.wikimedia.org \
  -o /dev/null -w "%{http_code}\n"
```

返回 `200` / `301` / `302` 才算可用；`000`（连不上 / 超时）或 `407`（需要认证）
就换下一个端口。注意别用 `head -1` 判断——走代理时第一行是 `200 Connection established`
（CONNECT 隧道建立），不代表目标可达。

常见对应关系：7890（Clash）、7897（Clash Verge）、1087/1080（Shadowsocks）、
6152（Surge）、8118（Privoxy）。

### 代理使用规则

```bash
# 直连（默认）
curl -sL "${CURL_OPTS[@]}" --noproxy '*' "<URL>" -o <file>

# 走代理重试
curl -sL "${CURL_OPTS[@]}" -x http://127.0.0.1:7890 "<URL>" -o <file>
```

- **直连能通就不要走代理**：代理可能更慢，部分 CDN 还会封代理 IP
- 探测成功后记住该代理，但后续图片**仍先直连**；连续 2 张直连失败后，本任务剩余图片全部走代理
- `localhost` / 内网地址加 `--noproxy`，避免请求被绕进代理
- 代理重试仍失败 → 走 Playwright 兜底（方法 5c）→ 仍失败则换图并记录

---

## 下载方法（按顺序尝试）

### 方法 1：Special:FilePath 重定向（首选）

Wikimedia 内置重定向，将文件名解析为完整尺寸下载 URL：

```bash
# 找到重定向目标 URL
curl -sIL "${CURL_OPTS[@]}" "https://commons.wikimedia.org/wiki/Special:FilePath/<FILENAME>.jpg" \
  | grep -i "location:" | grep "upload.wikimedia" | tail -1

# 下载图片（跟随重定向）
curl -sL "${CURL_OPTS[@]}" "https://commons.wikimedia.org/wiki/Special:FilePath/<FILENAME>.jpg" \
  -o assets/<target-filename>.jpg
```

**若响应 HTTP 404**：文件名在 Commons 上不存在，转方法 2。

### 方法 2：抓取 Wikipedia 页面 HTML（最可靠的回退）

方法 1 失败时，抓取 Wikipedia 文章页，找到嵌入的图片 URL：

```bash
# 跨平台写法（macOS 的 BSD grep 不支持 -P，故用 grep -oE）
curl -s "${CURL_OPTS[@]}" "https://en.wikipedia.org/wiki/<ARTICLE_NAME>" \
  | grep -oE 'src="[^"]*upload\.wikimedia\.org[^"]*\.(jpg|jpeg)[^"]*"' \
  | head -5
```

> 若需 Perl 正则（`-P`），仅在确认 GNU grep 可用时使用；macOS 默认 grep 用 `-E` 替代。

**缩略图转原图**：Wikipedia 页面嵌入的是缩略图。缩略图路径形如：

```
//upload.wikimedia.org/wikipedia/en/thumb/d/d8/Manchester_Mark2.jpg/250px-Manchester_Mark2.jpg
```

去掉 `/thumb/` 前缀和 `/Npx-FILENAME` 后缀得到原图 URL：

```
https://upload.wikimedia.org/wikipedia/en/d/d8/Manchester_Mark2.jpg
```

**注意**：
- Wikipedia 本地图片用 `wikipedia/en/{hash}/` 而非 `wikipedia/commons/{hash}/`，hash 必须来自实际页面 HTML，不可猜。
- Wikipedia API（`action=query`）被 robots.txt 阻止，不要用它发现图片。

### 方法 3：Wikimedia REST API

REST API 在 robots.txt 白名单内：

```bash
curl -s "${CURL_OPTS[@]}" "https://en.wikipedia.org/api/rest_v1/page/media/<ARTICLE_NAME>" \
  | python3 -c "import json,sys; d=json.load(sys.stdin); \
     [print(i.get('srcset',[{}])[0].get('src','')) for i in d.get('items',[])]"
```

### 方法 4：直接下载

拿到正确的原图 URL 后：

```bash
curl -sL "${CURL_OPTS[@]}" "<FULL_UPLOAD_URL>" -o assets/<target-filename>.jpg
```

### 方法 5：新闻媒体图片直链下载（news_editorial 模式）

用于科技新闻 / 深度报道的新闻实拍图。**先定位图片 URL，再下载。**
其中 5c（Playwright）是通用兜底，public_domain 模式被拒爬时同样适用。

#### 5a. 提取文章头图（og:image）

从文章 HTML 中提取 og:image（头图）：

```bash
curl -sL "${CURL_OPTS[@]}" "<ARTICLE_URL>" | grep -oE '<meta[^>]*property="og:image"[^>]*content="[^"]*"' | head -3
```

或直接抓取页面中 `upload` / CDN 路径的图片 URL：

```bash
curl -sL "${CURL_OPTS[@]}" "<ARTICLE_URL>" | grep -oE 'src="[^"]*(image|wp-content/uploads|images2/news/bigimage)[^"]*\.(jpg|jpeg|png|webp)[^"]*"' | head -10
```

#### 5b. 下载 CDN 直链

拿到原图 URL 后直接下载：

```bash
curl -sL "${CURL_OPTS[@]}" "<FULL_IMAGE_URL>" -o images/<target-filename>.jpg
```

常见 CDN 路径特征：`hsossms`（36氪）、`resource/image`（InfoQ）、`i.insider.com`、
`GettyImages`（人像）。直接下载 CDN 图片，比手动截图清晰。

> 路径含 `GettyImages` 只说明是人像类素材，**不代表可用**：带水印一律丢弃；
> 无水印的编辑图需调用方确认授权后再用（见 `source-and-license.md` 的禁止来源）。

#### 5c. Playwright 兜底（应对拒爬站点）

某些站点（如 The Register）对 curl / webfetch 返回 403 或 robots 拦截。此时用
Playwright MCP 浏览器。**两种模式都适用**：news_editorial 抓媒体图，public_domain
抓被拦截的来源页。耗时不计入单图 3 分钟预算，但计入整任务 10 分钟。

操作步骤：

1. 导航到图片 URL（`mcpm_playwright_browser_navigate`）
2. 在 `mcpm_playwright_browser_network_requests` 中找到该图片请求
3. 用 `mcpm_playwright_browser_network_request` 获取 `response-body` 保存

> **不要用 `page.evaluate` 里的 `fetch`**：跨域图片会被浏览器拦截。正确做法是
> 导航到图片 URL → 从 network 请求中取 response-body。

---

## 通用技巧

- 先试方法 1（最简单），再试方法 2（最可靠回退）
- 所有 `curl` 都带 `CURL_OPTS` 数组（超时 + 有限重试），别裸跑
- 超时/连不上先探测代理再重试一次，仍不行就走 Playwright 或换图，不无限等
- 用 `curl -sIL`（HEAD 请求，跟随重定向）检查 URL 是否有效再下载
- `Special:FilePath` 返回 404 时，图片可能已被重命名或删除，回 Wikipedia 文章页找当前文件名
- Commons 图片在 `upload.wikimedia.org/wikipedia/commons/`；Wikipedia 本地图片在 `upload.wikimedia.org/wikipedia/en/`

---

## 下载后处理（两种模式通用）

按顺序执行，详细命令见 `format-and-optimize.md`：

### 1. 类型与内容验证

```bash
file --mime-type -b assets/<filename>   # 不是 image/* 直接删除重下
ls -lh assets/<filename>
```

**必须打开/检查图片内容**：封面图通常 1200-2500px 宽，截图通常 1080px 宽。

- 确认图片与事件相关（有些媒体封面是随机插图，如 InfoQ 的猫/建筑，不要想当然）
- 人物肖像必须确认是目标人物，不是同名者或无关配图
- 尺寸/分辨率过低（宽 < 600px）的缩略图丢弃，换其他媒体来源
- 类型不在 `image_types` 白名单内 → 丢弃换图

此步**不判断体积**，体积留到压缩后复验。

### 2. 格式归一化

- SVG → PNG（优先让 Wikimedia 用 `?width=` 直接返回 PNG）
- WebP：透明底 / 截图 / 图表 → PNG；连续色调照片 → JPEG
- 扩展名必须与真实格式一致，禁止 `.jpg` 里装 PNG

### 3. 压缩

> 3 MB 或长边 > 2400px 的图片，压到长边 ≤ 2000px，目标 ≤ 1.5MB。

### 4. 复验

重新确认尺寸、体积、宽度 ≥ 600px，并打开图片确认文字可读、无块状伪影。

---

## 质量阈值

| 大小 | 判定 | 处理 |
|------|------|------|
| < 10 KB | ⚠️ 需确认 | 用 `file --mime-type` 检查：非 `image/*` 删除；照片类高色数图丢弃；纯色图形 / 截图（`colors` < 50000）可保留 |
| 10–50 KB | ⚠️ 可疑 | 同上确认，排除缩略图或损坏 |
| 50–500 KB | ✅ 合格 | 适合网页 |
| 500 KB–3 MB | ✅ 优秀 | 高分辨率，适合各种用途 |
| > 3 MB | ⚠️ 需要压缩 | 压到长边 ≤ 2000px，目标 ≤ 1.5MB（见 `format-and-optimize.md`） |

**另外**：长边 > 2400px 的图片即使不到 3 MB 也压缩。2400px 只是触发阈值，
压缩目标统一为长边 ≤ 2000px，封面类（文章头图 / og:image）不再单独放宽。

**立即删除的情况**：
- `file` 报告为 "HTML document"、"ASCII text" 或 "data" 而非图片
- 图片带水印
- 明显是 AI 生成图（除非项目允许且已标注）

---

## 图片类型覆盖（防重复）

本表即 `image_types` 白名单的取值来源：调用方指定的类型必须能对应到下表某一类，
不在清单内的图直接丢弃换图。

每张图片须承担不同的视觉角色，禁止同一类型的重复：

| 类型 | 说明 | 示例 |
|------|------|------|
| 人物肖像 | 照片/画像/雕塑 | `alan-turing-portrait.jpg` |
| 机器/设备 | 实物照片 | `manchester-mark1.jpg` |
| 核心产物 | 关键输出/文档/界面 | 代码截图、手册封面 |
| 历史场景 | 当代插图/新闻照片 | 人们使用机器的场景 |
| 纪念碑/地标 | 雕像/铭牌/纪念地 | 博物馆复刻、纪念碑 |

**反模式**：同一人物的两张肖像。选最好的一张，删除重复。

### news_editorial 模式的图片类型

| 类型 | 说明 | 示例 |
|------|------|------|
| 封面级插图 | 媒体文章头图（og:image），有视觉冲击力 | 事件相关的高质量新闻图 |
| 人物肖像 | 事件核心人物的照片（专访/发布会图） | CEO 演讲照 |
| 截图证据 | 推文、公告、GitHub Issue、Code of Conduct | 事件当事人发言截图 |
| 概念插图 | 手绘/AI 生成（仅当 `allow_ai_fallback=true`） | 无法用实拍表达的概念 |

同一事件不要重复用同一媒体头图；截图优先于 AI 生图（读者对真实截图的信任度远高于 AI 插图）。
