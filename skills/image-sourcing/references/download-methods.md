# 图片下载方法

本文件包含 image-sourcing 的图片下载方法（按顺序尝试）与质量阈值。

---

## 依赖校验

下载与验证依赖以下系统工具，执行前先确认存在，缺失则安装：

```bash
command -v curl || echo "缺少 curl"
command -v file || echo "缺少 file"
command -v python3 || echo "缺少 python3"   # 仅方法 3 需要
command -v grep || echo "缺少 grep"
```

- macOS/Linux 默认自带 `curl`、`file`、`grep`；`python3` 通常在 macOS 预装（无则 `brew install python`）。
- macOS 的 `grep` 为 BSD 版本，**不支持 `-P`**，本文所有正则均用 `-E`（扩展正则），勿改用 `-P`。

---

## 下载方法（按顺序尝试）

### 方法 1：Special:FilePath 重定向（首选）

Wikimedia 内置重定向，将文件名解析为完整尺寸下载 URL：

```bash
# 找到重定向目标 URL
curl -sIL "https://commons.wikimedia.org/wiki/Special:FilePath/<FILENAME>.jpg" \
  | grep -i "location:" | grep "upload.wikimedia" | tail -1

# 下载图片（跟随重定向）
curl -sL "https://commons.wikimedia.org/wiki/Special:FilePath/<FILENAME>.jpg" \
  -o assets/<target-filename>.jpg
```

**若响应 HTTP 404**：文件名在 Commons 上不存在，转方法 2。

### 方法 2：抓取 Wikipedia 页面 HTML（最可靠的回退）

方法 1 失败时，抓取 Wikipedia 文章页，找到嵌入的图片 URL：

```bash
# 跨平台写法（macOS 的 BSD grep 不支持 -P，故用 grep -oE）
curl -s "https://en.wikipedia.org/wiki/<ARTICLE_NAME>" \
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
curl -s "https://en.wikipedia.org/api/rest_v1/page/media/<ARTICLE_NAME>" \
  | python3 -c "import json,sys; d=json.load(sys.stdin); \
     [print(i.get('srcset',[{}])[0].get('src','')) for i in d.get('items',[])]"
```

### 方法 4：直接下载

拿到正确的原图 URL 后：

```bash
curl -sL "<FULL_UPLOAD_URL>" -o assets/<target-filename>.jpg
```

---

## 通用技巧

- 先试方法 1（最简单），再试方法 2（最可靠回退）
- 用 `curl -sIL`（HEAD 请求，跟随重定向）检查 URL 是否有效再下载
- `Special:FilePath` 返回 404 时，图片可能已被重命名或删除，回 Wikipedia 文章页找当前文件名
- Commons 图片在 `upload.wikimedia.org/wikipedia/commons/`；Wikipedia 本地图片在 `upload.wikimedia.org/wikipedia/en/`

---

## 下载后验证

每张图片下载后验证是否为有效图片：

```bash
file assets/<filename>.jpg
ls -lh assets/<filename>.jpg
```

## 质量阈值

| 大小 | 判定 | 处理 |
|------|------|------|
| < 10 KB | ❌ 损坏/HTML | 立即删除，换替代图 |
| 10–50 KB | ⚠️ 可疑 | 用 `file` 检查，非 "JPEG image data" 则删除 |
| 50–500 KB | ✅ 合格 | 适合网页 |
| 500 KB–3 MB | ✅ 优秀 | 高分辨率，适合各种用途 |
| > 3 MB | ⚠️ 偏大 | 需要细节可保留，否则考虑优化 |

**立即删除的情况**：
- `file` 报告为 "HTML document"、"ASCII text" 或 "data" 而非图片
- 图片带水印
- 明显是 AI 生成图（除非项目允许且已标注）

---

## 图片类型覆盖（防重复）

每张图片须承担不同的视觉角色，禁止同一类型的重复：

| 类型 | 说明 | 示例 |
|------|------|------|
| 人物肖像 | 照片/画像/雕塑 | `alan-turing-portrait.jpg` |
| 机器/设备 | 实物照片 | `manchester-mark1.jpg` |
| 核心产物 | 关键输出/文档/界面 | 代码截图、手册封面 |
| 历史场景 | 当代插图/新闻照片 | 人们使用机器的场景 |
| 纪念碑/地标 | 雕像/铭牌/纪念地 | 博物馆复刻、纪念碑 |

**反模式**：同一人物的两张肖像。选最好的一张，删除重复。
