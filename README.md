# skill-hygiene

> [中文版](README.zh-CN.md)

Keep your Claude Code skills lean. Detect overlaps, audit bloat, prevent prompt pollution.

## Why It Matters

Every installed skill injects its description into the system prompt. The model scans **all** of them before every response to decide which to invoke.

With 10 skills, this is fast and precise. With 60+, three things break:

1. **Selection confusion** — Similar descriptions force the model to guess. Sometimes it picks wrong, sometimes it picks neither.
2. **Context budget eaten** — 60 skills can consume 6,000+ tokens before you type your first message.
3. **Silent misfires** — The model invokes a tangentially related skill instead of answering directly. You just feel the response is "off".

This skill gives you the tools to keep things clean: **duplicate detection** before installs, **full audits** with overlap analysis, **health warnings** for bloated skills, and **soft archiving** instead of permanent deletion.

## Installation

### Method 1: Skills CLI (Recommended)

```bash
npx skills add belalee-ai/skill-hygiene
```

### Method 2: Claude Code Plugin

```bash
/plugin marketplace add belalee-ai/skill-hygiene
/plugin install skill-hygiene
```

### Method 3: Ask Your Agent

Just tell Claude Code:

> Install the skill-hygiene skill from https://github.com/belalee-ai/skill-hygiene

Claude will handle cloning and setup automatically.

### Method 4: Manual Clone

```bash
git clone https://github.com/belalee-ai/skill-hygiene.git
cp -r skill-hygiene/.claude ~/.claude
```

This copies the skill into `~/.claude/skills/skill-hygiene/` (standard personal skill location), available in all projects immediately.

<details>
<summary>Try before installing (local plugin mode)</summary>

```bash
git clone https://github.com/belalee-ai/skill-hygiene.git
claude --plugin-dir ./skill-hygiene
```

</details>

### Verify Installation

```
~/.claude/skills/skill-hygiene/
├── SKILL.md                    ← Core instruction file (must exist)
└── scripts/
    └── skill-audit.sh          ← Audit engine (bash 3+, no dependencies)
```

## Usage

### Pre-Install Duplicate Check

Before installing a new skill, check for overlaps:

```
> I want to install a storyboard skill

Claude automatically runs the check:

=== Duplicate Check: new-storyboard ===

  HIGH overlap (71%) with [film-storyboard-skill]
    Shared keywords: crea, image, script, storyboard, visual
    Existing: Use when creating storyboards from scripts...

  Review overlapping skills before installing.
```

### Full Audit

Say "run a skill audit" or invoke `/skill-hygiene audit`:

```
========================================
  Skill Hygiene Report — 2026-04-16
========================================

[1] Installed Custom Skills
  [animator-skill]    225L    52K   4 files
  [web-access]        246L    76K  10 files
  Total: 12 custom skills

[2] Overlap Analysis
  MED  (40%): [animator-skill] <-> [film-storyboard-skill]
    Shared: boards, convert, platform, prompt, sequence

[3] Enabled Plugins
  Estimated total skills in prompt: ~92
  > 50 skills — high risk of selection confusion. Audit and trim.

[4] Health Warnings
  [follow-builders] 100 files — may have bundled node_modules
  [ppt-generator] missing description — hurts tool selection
```

### Archive (Soft Delete)

```bash
mv ~/.claude/skills/old-skill ~/.claude/skills/.archived-old-skill
```

Archived skills appear in the audit report and can be restored anytime.

## Health Thresholds

| Metric | Green | Yellow | Red |
|--------|-------|--------|-----|
| Total skills in prompt | < 30 | 30–50 | > 50 |
| Keyword overlap between two skills | < 40% | 40–69% | >= 70% |
| Skill directory size | < 5 MB | 5–10 MB | > 10 MB |
| Files per skill | < 20 | 20–50 | > 50 |

## How It Works

The audit script extracts keywords from each skill's `description` field in SKILL.md, then compares pairwise:

1. Extract English words (3+ chars) from description
2. Remove stopwords (common English + Claude ecosystem terms like "skill", "agent", "tool")
3. Two-pass stemming: strip plurals (-s, -ies, -ves), then derivational suffixes (-ation, -ing, -ment, -ed)
4. Compare keyword sets using `comm -12`
5. Report overlap % = shared keywords / smaller set size

Pure bash, compatible with macOS (bash 3) and Linux (bash 4+). No external dependencies.

## Strategy Tips

- **Global plugins** — Only keep skills used in every project (workflow, debugging)
- **Project-level plugins** — Move domain plugins to `.claude/settings.json` in specific repos
- **Pipeline skills** — Skills forming a pipeline (scriptwriter → storyboard → animator) may share keywords but have different roles — don't merge blindly

## Hook Integration (Optional)

Auto-remind before skill installs. Add to `~/.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -qE 'npx skills (add|remove|update)|skills add|skills remove'; then echo '[user-prompt-submit-hook] Run skill-hygiene duplicate check before proceeding.'; fi"
          }
        ]
      }
    ]
  }
}
```

## License

MIT
