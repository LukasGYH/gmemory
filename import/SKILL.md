---
name: import
description: >
  Batch import external knowledge into gmemory/收件箱/ (or gmemory/inbox/) for the Ingest → Compile pipeline.
  ALWAYS trigger when the user says "import", "导入", "import my notes", "import chat",
  "import conversations", "导入记忆", "导入笔记", "import this", or mentions external
  files to import. Handles chat exports (ChatGPT, Claude, Gemini), unstructured .md
  files, PDFs with text, and any text-based knowledge dump. Normalizes everything into
  minimal gmemory/收件箱/ fragments.
---

# Import（导入）

Normalize external knowledge into `gmemory/收件箱/` (or `gmemory/inbox/`) fragments for the Ingest →
Compile pipeline. Handles any format.

## Directory language

This skill supports bilingual directories. On first use, check which exists:
`gmemory/收件箱/` (Chinese) or `gmemory/inbox/` (English). Use whichever exists.
If neither exists, create `gmemory/收件箱/` (backward compatible).

## Philosophy

Knowledge lives in many places — ChatGPT conversations, exported notes,
PDF articles, Slack threads. Import brings everything into one pipeline
so it all flows into the same knowledge graph, accessible by any agent
via Query.

## Workflow

### Step 1: Identify the input

| Input type | How to detect |
|------|------|
| Chat export | Consecutive `**User** / **Assistant**` or `You: / AI:` patterns |
| Unstructured .md | `.md` file with non-standard or missing frontmatter |
| PDF | `.pdf` file — use the pdf skill to extract text first |
| Plain text dump | Any text the user pastes or points to |

### Step 2: Extract fragments

**For chat exports:**
- Extract Q&A pairs where the answer contains actionable knowledge
- Skip: greetings, follow-ups, "thanks", "can you also..."
- Keep: explanations, tips, config examples, debug patterns, definitions

**For unstructured .md:**
- Split by section headings (`##`) — each section is a candidate
- Skip sections < 50 chars
- Preserve code blocks and tables

**For PDF:**
- Use the pdf skill to extract all text
- Split by chapters, headings, or topic boundaries

**For plain text:**
- Split by double newlines or topic boundaries

### Step 3: Deduplicate

1. Extract 2-3 keywords from each fragment
2. Search existing notes with Grep for those keywords
3. Skip fragments already covered in existing notes
4. Report what was skipped and why

### Step 4: Write to inbox

Same lightweight format as `remember`. Target the inbox directory (`gmemory/收件箱/` or `gmemory/inbox/`):

```markdown
# <short title>
<one clear sentence>

tags: <tag1>, <tag2>
```

Filename: `gmemory/收件箱/<today>-<short-title>.md`

### Step 5: Report

```
## Import 完成
### 已导入 (N)
- gmemory/收件箱/xxx.md — 一句话描述
### 已跳过 (N)
- "xxx" — 与已有笔记 [[xxx]] 重叠
### 下一步
收件箱现有 N 条碎片。Run /ingest to process them.
```

## Examples

**ChatGPT conversation about Docker:**
```
Input: 50-line Docker networking chat
→ 3 fragments: bridge network, overlay network, DNS resolution
→ Skip 1: "how to install Docker" (already in existing notes)
→ Write 3 files
```

**Unstructured .md file:**
```
Input: messy-notes.md, no frontmatter, random headings
→ Split by ## → 5 fragments
→ 3 substantial → write, 2 too short → skip
```

**PDF article:**
```
Input: technical-article.pdf
→ Extract text with pdf skill → split by sections
→ 4 fragments written to gmemory/收件箱/
```

## Tag guidance

Use comma-separated tags (Ingest converts to list format later).
Current vault domains: kubernetes, rancher, 运维, obsidian, claude-code, java, jdk, 性能
