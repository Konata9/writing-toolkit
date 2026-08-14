# Writing Toolkit

中文内容创作 SKILL 集合 — 为 Claude Code/OpenCode 提供的一系列写作工具，覆盖从调研验证、故事化写作、配图下载，到润色去 AI 味和平台合规检查的完整发布流程。

## 包含的 SKILL

### source-verify — 来源验证

调研阶段的事实交叉验证与来源可靠性评级：
- 每个关键事实至少 2 个独立来源
- 来源优先级判定（学术 > 一手 > 权威参考 > Wikipedia > 个人内容）
- 可靠性星级评级（1-5 星）
- 独立来源判断（识别互相抄袭的假"两个来源"）

### image-sourcing — 图片采购

为文章搜索、下载并记录公有领域/CC 许可的配图：
- 来源优先级（Wikimedia Commons → 博物馆馆藏 → 权威档案）
- 4 种下载方法（`Special:FilePath` → HTML 抓取 → REST API → 直接下载）
- 质量阈值验证（文件类型、大小、水印检测）
- AI 后备方案（参数化 `allow_ai_fallback`，必须标注）

### story-write — 故事化写作

把调研事实写成"有人、有冲突、有起伏"的文章：
- 标题设计（8 种公式 + 三要素 + 新鲜度测试）
- 导读块 + 开篇钩子
- 12 条叙事技法（阻力才是故事、人物锚点、情绪弧线、数字武器化等）
- AI 痕迹排查清单
- 小红书配文生成

### publish-guard — 发布守卫

发布前自动扫描小红书和微信公众号的合规问题：
- 禁用词扫描（极限词、营销词、政治敏感词）
- 灰色词汇计数（≤2 个限额）
- 最高级替换（"最大""最快"→ 具体描述）
- 外文专有名词首次出现格式检查
- 标题安全评估

### humanize-plus — 增强版去 AI 味

在 humanizer-zh 基础上增加结构性检查：
- 句长方差扫描（短句 ≤5 字 vs 长句 ≥35 字交替）
- 段高方差扫描（冲击式 / 沉浸式 / 喘息式三类交替）
- 过渡词多样性（机械过渡 vs 自然过渡）
- AI 套话扫描（结构套话、开头套话、结尾套话）
- 第二人称分布检查
- 口语插入检查
- 结尾质量评估

## 兼容性

本仓库同时兼容 **Claude Code** 和 **OpenCode**，SKILL 文件结构无需更改即可在两个平台使用。

## 安装

### General
```bash
# 通过 npx skills 一键安装全部 SKILL（推荐）
npx skills install Konata9/writing-toolkit

# 全局安装
npx skills install Konata9/writing-toolkit -g

# 或分别安装单个 SKILL
npx skills install Konata9/writing-toolkit/source-verify
npx skills install Konata9/writing-toolkit/image-sourcing
npx skills install Konata9/writing-toolkit/story-write
npx skills install Konata9/writing-toolkit/publish-guard
npx skills install Konata9/writing-toolkit/humanize-plus
```

### Claude Code/OpenCode
OpenCode 会自动扫描 `~/.claude/skills/` 目录，安装后重启即可加载：

```bash
# 克隆仓库手动安装
git clone https://github.com/Konata9/writing-toolkit.git
mkdir -p ~/.claude/skills
cp -r writing-toolkit/skills/source-verify ~/.claude/skills/
cp -r writing-toolkit/skills/image-sourcing ~/.claude/skills/
cp -r writing-toolkit/skills/story-write ~/.claude/skills/
cp -r writing-toolkit/skills/publish-guard ~/.claude/skills/
cp -r writing-toolkit/skills/humanize-plus ~/.claude/skills/
```

推荐使用软链接方式安装，确保 `git pull` 更新后 SKILL 自动同步，避免多份拷贝：

```bash
# 克隆仓库（如已克隆可跳过）
git clone https://github.com/Konata9/writing-toolkit.git

# 软链接安装（推荐）
mkdir -p ~/.claude/skills
ln -sfn "$(pwd)/writing-toolkit/skills/source-verify" ~/.claude/skills/source-verify
ln -sfn "$(pwd)/writing-toolkit/skills/image-sourcing" ~/.claude/skills/image-sourcing
ln -sfn "$(pwd)/writing-toolkit/skills/story-write" ~/.claude/skills/story-write
ln -sfn "$(pwd)/writing-toolkit/skills/publish-guard" ~/.claude/skills/publish-guard
ln -sfn "$(pwd)/writing-toolkit/skills/humanize-plus" ~/.claude/skills/humanize-plus
```

## 使用

### Claude Code

直接对话即可触发：

```
"对这篇调研做交叉验证"    → 自动调用 source-verify
"下载这几张配图"          → 自动调用 image-sourcing
"根据调研写一篇文章"      → 自动调用 story-write
"对这篇文章做深度润色"    → 自动调用 humanize-plus
"检查平台合规"            → 自动调用 publish-guard
```

也可以在项目 SKILL 工作流中显式调用：

```
Skill(source-verify)   # 交叉验证，输出验证报告
Skill(image-sourcing)  # 配图下载，参数 allow_ai_fallback / image_types
Skill(story-write)     # 故事化写作，参数 structure / length
Skill(humanize-plus)   # 深度润色，内容类型选择 narrative/casual/analysis
Skill(publish-guard)   # 平台合规扫描，平台选择 xiaohongshu/wechat/both
```

### OpenCode

OpenCode 会自动将已安装的 SKILL 注入 agent 上下文。在对话中描述需求即可触发，也可通过 `skill` 工具显式加载：

```
"对这篇调研做交叉验证"
"根据 research 写一篇故事化文章"
"检查这篇文章在小红书上能不能发"
```

## 完整创作流水线

```
确定主题（各项目主编排技能）
    ↓
调研 → source-verify（交叉验证）──→ story-write（故事化写作）
    │                                   ↓
    └── image-sourcing（配图，可并行）──┤
                                        ↓
                              humanize-plus（深度润色去 AI 味）
                                        ↓
                              publish-guard（平台合规检查）
                                        ↓
                                    发布
```

## 依赖关系

```
source-verify   ──独立运行──→  交叉验证 / 来源优先级 / 星级评级
image-sourcing  ──独立运行──→  图片搜索 / 下载 / 质量验证 / 版权记录
story-write     ──独立运行──→  标题 / 导读 / 叙事技法 / AI 痕迹排查 / 配文
humanize-plus   ──内部调用──→  humanizer-zh（表层去 AI 味）+ 结构性检查
publish-guard   ──独立运行──→  禁用词 / 灰色词汇 / 最高级 / 外文名词 / 标题安全
```

- `humanize-plus` 依赖 `humanizer-zh`（需预先安装）
- 其余 SKILL 无外部依赖

## 适用项目
欢迎关注公众号：此方的手账
![此方的手账](https://raw.githubusercontent.com/Konata9/pic-base/main/pics/此方的手账公众号二维码.png)

这些 SKILL 已在公众号长文合集中使用：
- [编程语言知多少？](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA5MjEzMjg2NA==&action=getalbum&album_id=4504434141060235268#wechat_redirect)
- [科技爆米花](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA5MjEzMjg2NA==&action=getalbum&album_id=4450861279548637185#wechat_redirect)
- [伟大的科技公司](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA5MjEzMjg2NA==&action=getalbum&album_id=4465296632921554946#wechat_redirect)

## License

MIT
