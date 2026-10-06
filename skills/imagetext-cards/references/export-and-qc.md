# 导出与 QC

把 `src/*.html` 渲染成 1080×1440 PNG，并做卡片专属质检。

---

## 1. 渲染方式

卡片是固定尺寸 HTML/CSS。用无头浏览器按**元素截图**导出，viewport 设为 `1080×1440`。

### 方式 A：Playwright MCP（推荐，零安装）
如果当前环境启动了 Playwright MCP：

1. `browser_navigate` 打开 `file://<abs>/src/postN.html`
2. 设置 viewport 1080×1440
3. 等待字体就绪：`await document.fonts.ready`
4. 对每个 `.card` 按 id（`#c-01`…）做元素截图，保存到 `cards/postN/01-cover.png` 等

页面级脚本（一次导出全部）示例：

```js
async (page) => {
  const base = '<abs-out>';
  await page.setViewportSize({ width: 1080, height: 1440 });
  await page.goto('file://' + base + '/src/post1.html', { waitUntil: 'networkidle' });
  await page.evaluate(() => document.fonts.ready);
  await page.waitForTimeout(400);
  const names = ['01-cover','02-point','03-list','04-shot','05-steps','06-grid','07-mock','08-cta'];
  const ids = ['c-01','c-02','c-03','c-04','c-05','c-06','c-07','c-08'];
  for (let i = 0; i < ids.length; i++) {
    await page.locator('#' + ids[i]).screenshot({ path: base + '/cards/post1/' + names[i] + '.png' });
  }
  return 'ok';
}
```

### 方式 B：本地 Playwright
```bash
npx playwright@latest install chromium
# 编写脚本：goto file:// + viewport 1080×1440 + document.fonts.ready
# + locator('#c-01').screenshot({ path }) …
```
> `page.locator().screenshot()` 默认 1×，输出正好 1080×1440（即平台规格）。

---

## 2. 命名与目录

```
cards/post1/01-cover.png  02-<role>.png … 08-cta.png
```
按顺序命名，避免手机相册按时间排序导致上传乱序。

---

## 3. 导出前 QC（渲染层）

- [ ] **字体已加载**：等待 `document.fonts.ready`；CJK 未回退成方块/错字。
- [ ] **尺寸**：每张 PNG 恰好 1080×1440。
- [ ] **无溢出**：列表/网格/时间线不压到页脚、不超出安全区（重点看内容最多的卡）。
- [ ] **对比度**：正文/次要文字在底色上可读（缩到 50% 再检查一次）。
- [ ] **图片**：截图/logo 路径正确、未拉伸变形。

---

## 4. 内容层 QC（内容层，配合 self-check）

- [ ] 一图一个观点；单卡 ≤60 字。
- [ ] 无编造数据 / 无绝对化用词 / 无外链。
- [ ] 系列一致：字体 / 配色 / 间距 / 封面模板。
- [ ] 封面 3 秒钩子。

> 内容层与 AI 生成感检查见 `references/anti-slop.md`（第 5 步，只给建议）。

---

## 5. 交付清单

1. 每张 PNG 的路径（`cards/postN/*.png`）。
2. 该帖的文案文件路径（`post-N-*.md`）。
3. 渲染 QC 结论：已确认 / 未确认（若无法查看图片，如实说明）。
4. 反 AI 味自检报告（建议）。
