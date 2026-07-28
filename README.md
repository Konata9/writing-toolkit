# Writing Toolkit

中文内容创作 SKILL 集合 — 为 Claude Code/OpenCode 提供的一系列写作工具，覆盖从润色去 AI 味到平台合规检查的完整发布流程。

## 包含的 SKILL

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
npx skills install Konata9/writing-toolkit/publish-guard
npx skills install Konata9/writing-toolkit/humanize-plus
```

### Claude Code/OpenCode
OpenCode 会自动扫描 `~/.claude/skills/` 目录，安装后重启即可加载：

```bash
# 克隆仓库手动安装
git clone https://github.com/Konata9/writing-toolkit.git
mkdir -p ~/.claude/skills
cp -r writing-toolkit/skills/publish-guard ~/.claude/skills/
cp -r writing-toolkit/skills/humanize-plus ~/.claude/skills/
```

推荐使用软链接方式安装，确保 `git pull` 更新后 SKILL 自动同步，避免多份拷贝：

```bash
# 克隆仓库（如已克隆可跳过）
git clone https://github.com/Konata9/writing-toolkit.git

# 软链接安装（推荐）
mkdir -p ~/.claude/skills
ln -sfn "$(pwd)/writing-toolkit/skills/publish-guard" ~/.claude/skills/publish-guard
ln -sfn "$(pwd)/writing-toolkit/skills/humanize-plus" ~/.claude/skills/humanize-plus
```

## 使用

### Claude Code

直接对话即可触发：

```
"对这篇文章做深度润色"  → 自动调用 humanize-plus
"检查平台合规"           → 自动调用 publish-guard
```

也可以在项目 SKILL 工作流中显式调用：

```
Skill(humanize-plus)     # 深度润色，内容类型选择 narrative/casual/analysis
Skill(publish-guard)     # 平台合规扫描，平台选择 xiaohongshu/wechat/both
```

### OpenCode

OpenCode 会自动将已安装的 SKILL 注入 agent 上下文。在对话中描述需求即可触发，也可通过 `skill` 工具显式加载：

```
"对这篇文章做深度润色，去掉 AI 味"
"检查这篇文章在小红书上能不能发"
```

## 依赖关系

```
humanize-plus  ──内部调用──→  humanizer-zh（表层去 AI 味）
                              ↓
                         结构性检查（句长、段高、过渡、套话等 9 维度）

publish-guard  ──独立运行──→  禁用词 / 灰色词汇 / 最高级 / 外文名词 / 标题安全
```

- `humanize-plus` 依赖 `humanizer-zh`（需预先安装）
- `publish-guard` 无外部依赖

## SKILL 关系图

```
创作完成后:

  humanize-plus               publish-guard
  (深度润色去AI味)     →      (平台合规检查)
  ┌──────────────┐           ┌──────────────┐
  │ humanizer-zh  │           │ 禁用词扫描    │
  │ 句长方差      │           │ 灰色词汇计数  │
  │ 段高方差      │           │ 最高级替换    │
  │ 过渡词多样性  │           │ 外文名词规范  │
  │ AI套话扫描    │           │ 标题安全评估  │
  │ 第二人称分布  │           └──────────────┘
  │ 口语插入      │
  │ 结尾质量      │
  └──────────────┘
```

## 适用项目
欢迎关注公众号：此方的手账
![此方的手账](https://raw.githubusercontent.com/Konata9/pic-base/main/pics/此方的手账公众号二维码.png)

这些 SKILL 已在公众号长文合集中使用：
- [编程语言知多少？](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA5MjEzMjg2NA==&action=getalbum&album_id=4504434141060235268#wechat_redirect)
- [科技爆米花](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA5MjEzMjg2NA==&action=getalbum&album_id=4450861279548637185#wechat_redirect)
- [伟大的科技公司](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA5MjEzMjg2NA==&action=getalbum&album_id=4465296632921554946#wechat_redirect)

## License

MIT
