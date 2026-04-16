# skill-hygiene

[中文版 README](README.zh-CN.md)

Keep your Claude Code skills lean. Detect overlaps, audit bloat, prevent prompt pollution.

## Why does having too many skills hurt accuracy?

Every skill you install injects its name and description into the system prompt. Before responding to you, the model scans **all** skill descriptions to decide: *"Should I invoke one of these?"*

With 10 skills, this is fast and precise. With 60+, three things break down:

1. **Selection confusion** — Two skills with similar descriptions (e.g., `film-storyboard-skill` and `storyboard-creation` both mention "storyboard + visual prompts") force the model to guess which one you meant. Sometimes it picks the wrong one, sometimes it hesitates and picks neither.

2. **Context budget eaten** — Each skill description costs tokens. 60 skills can consume 6,000+ tokens of your context window before you even type your first message. That's space that could hold your actual code or conversation history.

3. **Invisible misfires** — The model might silently invoke a tangentially related skill instead of answering directly. You won't know why the response feels off.

**A real example from my own setup:**

I had 64 skills loaded. When I said "help me create a storyboard", Claude had to choose between `film-storyboard-skill`, `storyboard-creation`, and `storyboard-review-skill` — all with overlapping descriptions. After running an audit, I archived the redundant one and moved 33 domain-specific skills to project-level. Selection accuracy improved immediately, and my context window got 4,000+ tokens back.

## What this skill does

This skill gives you:

- **Duplicate detection** before installing new skills
- **Full audit** of all installed skills with overlap analysis
- **Health warnings** for oversized skills and missing descriptions
- **Hook integration** to auto-check before skill installs

## Install

**Recommended — Install as a personal skill:**

```bash
git clone https://github.com/belalee-ai/skill-hygiene.git
cp -r skill-hygiene/.claude ~/.claude
```

This copies the skill into `~/.claude/skills/skill-hygiene/`, which is the standard Claude Code personal skill location. It will be available in all your projects immediately.

<details>
<summary>Other install methods</summary>

**Test as a plugin (try before installing):**

```bash
git clone https://github.com/belalee-ai/skill-hygiene.git
claude --plugin-dir ./skill-hygiene
```

**Third-party CLI (if you use `npx skills`):**

```bash
npx skills add belalee-ai/skill-hygiene
```

</details>

## Usage

### Pre-install check

Before installing a new skill, check for duplicates:

```
> I want to install a storyboard skill

Claude will automatically run the duplicate check and show:

=== Duplicate Check: new-storyboard ===

  HIGH overlap (71%) with [film-storyboard-skill]
    Shared keywords: crea, image, script, storyboard, visual
    Existing: Use when creating storyboards from scripts...

  Review overlapping skills before installing.
```

### Full audit

Tell Claude "run a skill audit" or invoke `/skill-hygiene audit`. Output:

```
========================================
  Skill Hygiene Report — 2026-04-16
========================================

[1] Installed Custom Skills
  [animator-skill]    225L    52K   4 files
  [web-access]        246L    76K  10 files
  ...
  Total: 12 custom skills

[2] Overlap Analysis
  MED  (40%): [animator-skill] <-> [film-storyboard-skill]
    Shared: boards, convert, platform, prompt, sequence

[3] Enabled Plugins
  - superpowers@claude-plugins-official
  - vercel@claude-plugins-official
  ...
  Estimated total skills in prompt: ~92
  > 50 skills — high risk of selection confusion. Audit and trim.

[4] Health Warnings
  [follow-builders] 100 files — may have bundled node_modules
  [ppt-generator] missing description — hurts tool selection
```

### Archive (soft delete)

```bash
mv ~/.claude/skills/old-skill ~/.claude/skills/.archived-old-skill
```

Archived skills show up in the audit report and can be restored anytime.

## Thresholds

| Metric | Green | Yellow | Red |
|--------|-------|--------|-----|
| Total skills in prompt | < 30 | 30–50 | > 50 |
| Keyword overlap between two skills | < 40% | 40–69% | >= 70% |
| Skill directory size | < 5 MB | 5–10 MB | > 10 MB |
| Files per skill | < 20 | 20–50 | > 50 |

## How it works

The audit script extracts keywords from each skill's `description` field in SKILL.md, then compares them pairwise:

1. Extract English words (3+ chars) from description
2. Remove stopwords (common English + Claude ecosystem terms like "skill", "agent", "tool")
3. Two-pass stemming: strip plurals first (-s, -ies, -ves), then derivational suffixes (-ation, -ing, -ment, -ed)
4. Compare keyword sets between all skill pairs using `comm -12`
5. Report overlap percentage (shared keywords / smaller set size)

Pure bash, compatible with macOS (bash 3) and Linux (bash 4+). No external dependencies.

## Hook setup (optional)

Add to `~/.claude/settings.json` to auto-remind before skill installs:

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

## Strategy tips

- **Global plugins**: Only keep skills you use in every project (workflow, debugging)
- **Project-level plugins**: Move domain plugins (Vercel, Figma) to `.claude/settings.json` or `.claude/settings.local.json` in specific repos
- **Pipeline skills**: Skills that form a pipeline (scriptwriter → storyboard → animator) may share keywords but have different roles — don't merge them blindly

## License

MIT
