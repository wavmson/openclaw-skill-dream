# 🌙 Dream — Memory Consolidation Skill for OpenClaw

> Like a human brain consolidating memories during sleep, Dream periodically reviews your agent's daily notes and merges durable knowledge into long-term memory.

**Inspired by Claude Code's `autoDream` module** — adapted for [OpenClaw](https://github.com/openclaw/openclaw) agents.

## The Problem

OpenClaw agents accumulate daily memory files (`memory/YYYY-MM-DD.md`) through conversations. Over time:

- 📈 Daily files pile up (easily 20+ files, thousands of lines)
- 🧊 `MEMORY.md` (long-term memory) becomes stale — it only gets updated during compaction flushes
- 🔄 Important facts get buried in old daily logs that the agent never re-reads
- 🤔 New sessions start with outdated context

## The Solution

Dream runs a structured consolidation process:

```
Daily journals ──▶ Dream Skill ──▶ Updated MEMORY.md
(raw, verbose)     (scan+merge)    (curated, compact)
```

It scans recent daily files, extracts what matters, and surgically updates `MEMORY.md` — adding new facts, correcting outdated ones, and removing obsolete entries.

## Installation

### Option A: ClawHub (Recommended)

```bash
clawhub install dream
```

### Option B: Manual Install

```bash
# Clone to your OpenClaw skills directory
git clone https://github.com/wavmson/openclaw-skill-dream.git ~/.openclaw/skills/dream

# Or just copy the SKILL.md file
mkdir -p ~/.openclaw/skills/dream
curl -o ~/.openclaw/skills/dream/SKILL.md \
  https://raw.githubusercontent.com/wavmson/openclaw-skill-dream/main/SKILL.md
```

### Option C: Copy-Paste

1. Create directory: `mkdir -p ~/.openclaw/skills/dream`
2. Copy the contents of `SKILL.md` from this repo into `~/.openclaw/skills/dream/SKILL.md`
3. Restart OpenClaw Gateway: `openclaw gateway restart`

## Usage

### Manual Trigger

Just tell your agent:

- "做梦" / "dream"
- "整理记忆" / "consolidate memory"
- "memory consolidation"

### Automatic (Cron)

Set up a nightly cron job so your agent dreams every night:

```
/cron add --schedule "0 3 * * *" --task "Execute dream skill: consolidate memory" --label dream-nightly
```

### Heartbeat Integration

Add to your `HEARTBEAT.md`:

```markdown
- [ ] If memory/ has >20 files or >3000 total lines, run dream skill
```

## What It Does

| Phase | Action | Details |
|-------|--------|---------|
| 1. Orient | Inventory | List all `memory/*.md` files, read current `MEMORY.md` |
| 2. Scan | Extract | Read last 7 days of daily files, identify new/changed/contradicting facts |
| 3. Merge | Update | Surgically edit `MEMORY.md` — add, correct, deduplicate, prune |
| 4. Mark | Archive | Tag journals older than 30 days as consolidated (never deletes) |
| 5. Report | Summary | Output what changed: `+N added / ~N corrected / -N pruned` |

## Example Output

```
🌙 Dream complete
- Scanned 19 daily files (2877 lines total)
- MEMORY.md: +8 sections added / ~0 corrected / -0 pruned
- New discoveries: user is a car designer, ESP32 device configs, 3 Notion databases
- Next suggestion: consider archiving files older than 2026-03-10
```

## Design Principles

- **Conservative** — when unsure, keeps information rather than removing it
- **Surgical** — uses precise edits, never overwrites entire files
- **Token-efficient** — only reads last 7 days, caps large files at 200 lines
- **Safe** — never deletes files, never exposes secrets in reports
- **Idempotent** — running twice produces the same result

## Requirements

- [OpenClaw](https://github.com/openclaw/openclaw) (any version with Skill support)
- A workspace with `MEMORY.md` and `memory/` directory (standard OpenClaw layout)
- Works with any model (no model-specific features)

## Background: Claude Code's autoDream

This skill is inspired by the `autoDream` module found in Claude Code's source:

```
src/services/autoDream/
├── autoDream.ts              # Main consolidation orchestrator
├── config.ts                 # Feature flag & settings
├── consolidationLock.ts      # Prevents concurrent runs
└── consolidationPrompt.ts    # The "dream" prompt template
```

Key ideas borrowed:
- **Time-gated execution** — only runs after enough time has passed
- **Session-count gate** — only runs after enough sessions have accumulated
- **Lock file** — prevents concurrent consolidation
- **Structured phases** — orient → gather → consolidate → prune → index

What we simplified for OpenClaw:
- No lock file needed (OpenClaw's Cron/Heartbeat handles scheduling)
- No transcript JSONL parsing (we use Markdown daily files instead)
- No forked agent pattern (direct execution in the main session)

## File Structure

```
dream/
├── SKILL.md          # Skill definition (required by OpenClaw)
└── README.md         # This file
```

## License

MIT — Use it, fork it, improve it.

## Contributing

PRs welcome! Ideas for improvement:

- [ ] Add a `scripts/dream-stats.sh` for quick memory health checks
- [ ] Support topic-based memory files (not just chronological)
- [ ] Add conflict resolution strategies for contradicting memories
- [ ] Integration with OpenClaw's upcoming persistent memory features

---

*"We are such stuff as dreams are made on, and our little life is rounded with a sleep."* — Shakespeare
