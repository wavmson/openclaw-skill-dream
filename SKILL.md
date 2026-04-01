---
name: memory-dream
description: "Memory consolidation for OpenClaw agents. Periodically reviews daily memory files (memory/*.md), extracts durable knowledge, and merges it into MEMORY.md — like a human brain consolidating memories during sleep. Triggers: dream, 做梦, consolidate memory, 整理记忆, memory consolidation. Can also run via Cron for automatic nightly consolidation."
---

# Dream — Memory Consolidation for OpenClaw

Inspired by Claude Code's `autoDream` module. Your agent accumulates daily notes in `memory/YYYY-MM-DD.md` files, but without periodic consolidation, long-term memory (`MEMORY.md`) becomes stale while daily files grow endlessly. Dream fixes this.

## When to Use

- User says "dream", "做梦", "consolidate memory", or "整理记忆"
- Cron trigger (recommended: nightly at 3:00 AM)
- Heartbeat detects memory files piling up (>20 files or >3000 total lines)

## Consolidation Flow

### Phase 1 — Orient

1. `ls memory/` to inventory daily files
2. Read `MEMORY.md` to understand current long-term memory structure
3. Count: total files, total lines, files from last 7 days

### Phase 2 — Scan Recent Journals

Read daily files from the **last 7 days** (`memory/YYYY-MM-DD.md`). Extract:
- New facts, preferences, or decisions not yet in MEMORY.md
- Information that **contradicts** MEMORY.md (needs correction)
- Recurring themes (indicates importance)

**Token budget rules:**
- Skip files older than 7 days (too many tokens)
- If a file exceeds 500 lines, read only the first 200 lines
- Focus on headings and key facts, not verbose logs

### Phase 3 — Merge into MEMORY.md

Apply these operations to MEMORY.md using **surgical edits** (not full rewrites):

1. **Add**: Append new facts to the appropriate section
2. **Correct**: Update outdated information with newer data from journals
3. **Deduplicate**: Merge entries that say the same thing
4. **Prune**: Remove clearly obsolete info (e.g., "temporary workaround for X" when X is resolved)
5. **Absolutize dates**: Convert "yesterday", "just now" → actual dates (e.g., "2026-03-27")

### Phase 4 — Mark Old Journals (Optional)

For daily files **older than 30 days**:
- If content is consolidated → prepend: `<!-- consolidated to MEMORY.md on YYYY-MM-DD -->`
- **Never delete** any file (user may want to look back)

### Phase 5 — Report

Output a brief consolidation report:

```
🌙 Dream complete
- Scanned N daily files (X lines total)
- MEMORY.md: +N added / ~N corrected / -N pruned
- Next suggestion: [if any]
```

## Critical Rules

- **MEMORY.md is the primary output** — all consolidated knowledge goes here
- **Surgical edits only** — use the `edit` tool for precise changes, never overwrite the whole file
- **Conservative by default** — when unsure whether to remove something, keep it
- **No secrets in logs** — don't expose API keys, passwords, or tokens in the dream report
- **Log each run** — append a `## Dream Log (HH:MM)` entry to today's `memory/YYYY-MM-DD.md`

## Cron Setup

```
/cron add --schedule "0 3 * * *" --task "Execute dream skill: consolidate memory" --label dream-nightly
```

## How It Works (Under the Hood)

This skill mimics the human sleep cycle's memory consolidation process:

```
Daily experiences          Nightly consolidation        Long-term memory
┌─────────────────┐       ┌─────────────────┐         ┌─────────────────┐
│ memory/03-26.md │──┐    │                 │         │                 │
│ memory/03-27.md │──┤    │    Dream Skill   │────────▶│   MEMORY.md     │
│ memory/03-28.md │──┤    │   (scan+merge)  │         │  (structured,   │
│ memory/03-29.md │──┘    │                 │         │   deduplicated) │
│       ...       │       └─────────────────┘         └─────────────────┘
└─────────────────┘
     Raw daily logs            Consolidation              Curated knowledge
```

Without Dream, MEMORY.md stays frozen at whatever was manually written, while daily files accumulate unbounded. With Dream, your agent's long-term memory stays fresh, accurate, and compact.
