# 🌙 Dream — Memory Consolidation Skill for OpenClaw

> Like a human brain consolidating memories during sleep, Dream periodically reviews your agent's daily notes and merges durable knowledge into long-term memory.

**Inspired by Claude Code's `autoDream` module** — adapted for [OpenClaw](https://github.com/openclaw/openclaw) agents.

---

## 🇨🇳 中文说明

### 这是什么？

你的 OpenClaw Agent 每天对话会产生日记文件（`memory/YYYY-MM-DD.md`），时间一长：

- 📈 日记越堆越多（轻松 20+ 个文件，几千行）
- 🧊 长期记忆（`MEMORY.md`）越来越过时
- 🔄 重要信息被埋在旧日记里，Agent 再也不会回头看
- 🤔 新对话开始时，Agent 带着过时的记忆上岗

**Dream Skill 就像人类的"做梦"** —— 定期扫描近期日记，把重要的新信息合并到长期记忆里，过时的信息修正掉。

### 安装方法

**方式 A：ClawHub（推荐）**

```bash
clawhub install memory-dream
```

**方式 B：从 GitHub 安装**

```bash
git clone https://github.com/wavmson/openclaw-skill-memory-dream.git ~/.openclaw/skills/dream
```

**方式 C：只复制核心文件**

```bash
mkdir -p ~/.openclaw/skills/dream
curl -o ~/.openclaw/skills/dream/SKILL.md \
  https://raw.githubusercontent.com/wavmson/openclaw-skill-memory-dream/main/SKILL.md
```

安装后重启 Gateway：`openclaw gateway restart`

### 怎么用？

**手动触发** — 直接跟 Agent 说：
- "做梦" / "dream"
- "整理记忆" / "记忆整合"

**自动执行（推荐）** — 设定每天凌晨 3 点自动做梦：
```
/cron add --schedule "0 3 * * *" --task "Execute dream skill: consolidate memory" --label dream-nightly
```

### 执行流程

| 阶段 | 动作 | 说明 |
|------|------|------|
| 1. 盘点 | 清点家底 | 列出所有 `memory/*.md` 文件，读取当前 `MEMORY.md` |
| 2. 扫描 | 提取新知 | 读取最近 7 天的日记，找出新事实、矛盾信息、重复主题 |
| 3. 合并 | 精准更新 | 编辑 `MEMORY.md` —— 新增、修正、去重、清理 |
| 4. 标记 | 归档旧记 | 给 30 天以上的日记打上"已整合"标签（绝不删除） |
| 5. 报告 | 输出结果 | `+N 新增 / ~N 修正 / -N 清理` |

### 设计原则

- **保守策略** — 拿不准就保留，宁多勿删
- **精准编辑** — 用 edit 而非 overwrite，绝不整体覆盖
- **省 token** — 只读最近 7 天，大文件截取前 200 行
- **安全** — 绝不删除文件，绝不在报告里暴露密钥
- **幂等** — 跑两次结果一样

---

## 🇬🇧 English

### The Problem

OpenClaw agents accumulate daily memory files (`memory/YYYY-MM-DD.md`) through conversations. Over time:

- 📈 Daily files pile up (easily 20+ files, thousands of lines)
- 🧊 `MEMORY.md` (long-term memory) becomes stale — it only gets updated during compaction flushes
- 🔄 Important facts get buried in old daily logs that the agent never re-reads
- 🤔 New sessions start with outdated context

### The Solution

Dream runs a structured consolidation process:

```
Daily journals ──▶ Dream Skill ──▶ Updated MEMORY.md
(raw, verbose)     (scan+merge)    (curated, compact)
```

It scans recent daily files, extracts what matters, and surgically updates `MEMORY.md` — adding new facts, correcting outdated ones, and removing obsolete entries.

### Installation

**Option A: ClawHub (Recommended)**

```bash
clawhub install memory-dream
```

**Option B: Manual Install**

```bash
# Clone to your OpenClaw skills directory
git clone https://github.com/wavmson/openclaw-skill-memory-dream.git ~/.openclaw/skills/dream

# Or just copy the SKILL.md file
mkdir -p ~/.openclaw/skills/dream
curl -o ~/.openclaw/skills/dream/SKILL.md \
  https://raw.githubusercontent.com/wavmson/openclaw-skill-memory-dream/main/SKILL.md
```

**Option C: Copy-Paste**

1. Create directory: `mkdir -p ~/.openclaw/skills/dream`
2. Copy the contents of `SKILL.md` from this repo into `~/.openclaw/skills/dream/SKILL.md`
3. Restart OpenClaw Gateway: `openclaw gateway restart`

### Usage

**Manual Trigger** — Just tell your agent:

- "做梦" / "dream"
- "整理记忆" / "consolidate memory"
- "memory consolidation"

**Automatic (Cron)** — Set up a nightly cron job:

```
/cron add --schedule "0 3 * * *" --task "Execute dream skill: consolidate memory" --label dream-nightly
```

**Heartbeat Integration** — Add to your `HEARTBEAT.md`:

```markdown
- [ ] If memory/ has >20 files or >3000 total lines, run dream skill
```

### What It Does

| Phase | Action | Details |
|-------|--------|---------|
| 1. Orient | Inventory | List all `memory/*.md` files, read current `MEMORY.md` |
| 2. Scan | Extract | Read last 7 days of daily files, identify new/changed/contradicting facts |
| 3. Merge | Update | Surgically edit `MEMORY.md` — add, correct, deduplicate, prune |
| 4. Mark | Archive | Tag journals older than 30 days as consolidated (never deletes) |
| 5. Report | Summary | Output what changed: `+N added / ~N corrected / -N pruned` |

### Example Output

```
🌙 Dream complete
- Scanned 19 daily files (2877 lines total)
- MEMORY.md: +8 sections added / ~0 corrected / -0 pruned
- New discoveries: user is a car designer, ESP32 device configs, 3 Notion databases
- Next suggestion: consider archiving files older than 2026-03-10
```

### Design Principles

- **Conservative** — when unsure, keeps information rather than removing it
- **Surgical** — uses precise edits, never overwrites entire files
- **Token-efficient** — only reads last 7 days, caps large files at 200 lines
- **Safe** — never deletes files, never exposes secrets in reports
- **Idempotent** — running twice produces the same result

---

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

## Requirements

- [OpenClaw](https://github.com/openclaw/openclaw) (any version with Skill support)
- A workspace with `MEMORY.md` and `memory/` directory (standard OpenClaw layout)
- Works with any model (no model-specific features)

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
