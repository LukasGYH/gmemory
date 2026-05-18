---
name: remember
description: >
  Capture working memory fragments into gmemory/收件箱/ (or gmemory/inbox/). ALWAYS trigger when the user says
  "remember", "记住", "记一下", "note this", "make a note", "save this for later",
  "记录下来", "mark this", or wants to save a quick insight, tip, or lesson learned
  during a working session. Also trigger on casual mentions like "this is important,
  I want to remember this" or "add this to my notes". Fragments are later processed
  by Ingest → Compile into proper vault notes.
---

# Remember（记忆碎片）

Capture working insights as lightweight fragments in `gmemory/收件箱/` (or `gmemory/inbox/`).
These fragments will be processed later by Ingest → Compile.

## Directory language

This skill supports bilingual directories. On first use, check which exists:
`gmemory/收件箱/` (Chinese) or `gmemory/inbox/` (English). Use whichever exists.
If neither exists, create `gmemory/收件箱/` (backward compatible).

## Philosophy

When working with an agent, you learn things — a command flag you didn't
know, a bug pattern, a config trick. These insights are valuable but
don't warrant a full note right away. `remember` lets you save them
in seconds and move on. Later, Ingest + Compile will merge related
fragments into proper notes.

## Fragment format

Each fragment is a minimal `.md` file saved to the inbox directory (`gmemory/收件箱/` or `gmemory/inbox/` — see Directory language above). Only three parts:

```markdown
# <short title>
<one sentence describing the insight>

tags: <tag1>, <tag2>
```

No frontmatter, no dates, no status — Ingest adds all of that later.
Tags are comma-separated for easy typing (Ingest converts to list format).

## Filename convention

`gmemory/收件箱/YYYY-MM-DD-<short-title>.md`

## Workflow

1. Parse what the user wants to remember from context
2. Write a single-sentence summary — clear enough to understand weeks later
3. Assign 1-3 tags based on the topic
4. Save to the inbox directory (`gmemory/收件箱/` or `gmemory/inbox/`) with the date-title filename
5. Confirm: "已记住: <summary>" — one line, no ceremony
6. If 5+ fragments in the inbox, suggest: "收件箱有 N 条碎片，要跑 /ingest 吗？"

## Examples

User: "remember: K8s Job 的 backoffLimit 默认是 6 次"
→ `gmemory/收件箱/2026-05-15-k8s-backofflimit.md`:
```markdown
# K8s backoffLimit
K8s Job 的 backoffLimit 默认值是 6，设为 0 则失败一次就停止

tags: kubernetes, 运维
```

User: "记一下，ZGC 在 JDK 23 默认启用分代模式"
→ `gmemory/收件箱/2026-05-15-jdk-zgc-generational.md`:
```markdown
# ZGC 分代模式
JDK 23 默认启用分代 ZGC，JDK 24 移除非分代模式

tags: java, jdk, 性能
```

## Tag guidance

Use existing vault tags where possible. Current active tags include:
kubernetes, rancher, 运维, 故障排查, obsidian, claude-code, 教程, 插件,
工具, java, jdk, 并发, 性能, 经济学, AI, 半导体

## Cross-agent compatibility

The fragment format is intentionally minimal — no complex frontmatter,
no agent-specific syntax. Any agent (Claude Code, Trae SOLO, etc.) with
filesystem access to this vault can write fragments. Ingest and Compile
process them identically regardless of origin.
