# skill-hygiene

> **⚠️ This skill has moved to [bela-tools](https://github.com/belalee-ai/bela-tools).** Install from there for the latest version.

---

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

Install via [bela-tools](https://github.com/belalee-ai/bela-tools) (recommended):

```bash
git clone https://github.com/belalee-ai/bela-tools.git ~/.claude/plugins/bela-tools
```

### Legacy installation (this repo)

```bash
git clone https://github.com/belalee-ai/skill-hygiene.git
cp -r skill-hygiene/.claude ~/.claude
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

## License

MIT
