# skill-hygiene

让你的 Claude Code Skill 保持精简。检测重叠、审计膨胀、防止提示词污染。

## 问题背景

每安装一个 skill 或启用一个插件，它的描述都会被注入到系统提示中。当 skill 超过 50 个时，模型在选择调用哪个 skill 时会出现混乱 — 就像一个 20 页菜单的餐厅，点菜越来越难。

这个 skill 提供：

- **重复检测** — 安装新 skill 前，自动比对已有 skill 的语义重叠度
- **全量审计** — 扫描所有已安装 skill，输出重叠分析报告
- **健康告警** — 发现体积过大、文件过多、缺少描述的 skill
- **Hook 集成** — 安装 skill 时自动提醒先做检查

## 安装

**方式一 — 手动复制为个人 skill：**

```bash
git clone https://github.com/belalee-ai/skill-hygiene.git
cp -r skill-hygiene/.claude ~/.claude
```

**方式二 — 作为插件本地测试：**

```bash
git clone https://github.com/belalee-ai/skill-hygiene.git
claude --plugin-dir ./skill-hygiene
```

**方式三 — 第三方 CLI（如果你使用 `npx skills`）：**

```bash
npx skills add belalee-ai/skill-hygiene
```

## 使用方法

### 安装前查重

安装新 skill 之前，检查是否与已有 skill 重复：

```
> 我想装一个分镜 skill

Claude 会自动运行查重并展示结果：

=== Duplicate Check: new-storyboard ===

  HIGH overlap (71%) with [film-storyboard-skill]
    Shared keywords: crea, image, script, storyboard, visual
    Existing: Use when creating storyboards from scripts...

  建议：安装前先检查重叠的 skill，考虑替换而非共存。
```

### 全量审计

告诉 Claude "跑一下 skill 审计" 或输入 `/skill-hygiene audit`。输出示例：

```
========================================
  Skill Hygiene Report — 2026-04-16
========================================

[1] 已安装的自定义 Skill
  [animator-skill]    225L    52K   4 files
  [web-access]        246L    76K  10 files
  ...
  合计：12 个自定义 skill

[2] 重叠分析
  中等 (40%): [animator-skill] <-> [film-storyboard-skill]
    共享关键词: boards, convert, platform, prompt, sequence

[3] 已启用插件
  - superpowers@claude-plugins-official
  - vercel@claude-plugins-official
  ...
  预估提示词中的 skill 总量：~92
  > 50 个 skill — 选择混乱风险高，建议清理。

[4] 健康告警
  [follow-builders] 100 个文件 — 可能包含打包的 node_modules
  [ppt-generator] 缺少 description — 影响模型匹配
```

### 归档（软删除）

```bash
mv ~/.claude/skills/旧skill ~/.claude/skills/.archived-旧skill
```

归档的 skill 会显示在审计报告中，随时可以恢复。

## 阈值标准

| 指标 | 健康（绿） | 注意（黄） | 危险（红） |
|------|-----------|-----------|-----------|
| 提示词中 skill 总量 | < 30 | 30–50 | > 50 |
| 两个 skill 之间的关键词重叠率 | < 40% | 40–69% | >= 70% |
| Skill 目录大小 | < 5 MB | 5–10 MB | > 10 MB |
| Skill 目录文件数 | < 20 | 20–50 | > 50 |

## 工作原理

审计脚本从每个 skill 的 SKILL.md 中提取 `description` 字段的关键词，然后两两比对：

1. 提取英文单词（3 字符以上）
2. 去除停用词（常见英语虚词 + Claude 生态通用词如 skill、agent、tool）
3. 两遍词干归一：先去复数（-s/-ies/-ves），再去派生后缀（-ation/-ing/-ment/-ed）
4. 使用 `comm -12` 比对关键词集合
5. 输出重叠率 = 共享关键词数 / 较小集合大小

纯 bash 实现，兼容 macOS（bash 3）和 Linux（bash 4+），无外部依赖。

## Hook 设置（可选）

在 `~/.claude/settings.json` 中添加以下配置，安装 skill 时自动提醒：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -qE 'npx skills (add|remove|update)|skills add|skills remove'; then echo '[user-prompt-submit-hook] 请先运行 skill-hygiene 重复检测。'; fi"
          }
        ]
      }
    ]
  }
}
```

## 管理策略

- **全局插件**：只保留每个项目都用的 skill（工作流、调试类）
- **项目级插件**：领域插件（Vercel、Figma 等）放到具体项目的 `.claude/settings.json` 或 `.claude/settings.local.json` 里
- **流水线 skill**：形成流水线的 skill（编剧 → 分镜 → 动画）虽然关键词重叠，但职责不同 — 不要盲目合并

## 许可证

MIT
