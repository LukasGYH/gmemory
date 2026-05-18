---
name: query
description: >
  Natural language search and Q&A against the Obsidian vault. ALWAYS trigger this
  skill when the user asks questions about their vault content like "what do I have
  on X?", "find notes about Y", "search my vault for Z", "what did I write about...",
  "do I have any notes on...", "summarize what I know about...", "based on my notes,
  what is...", or any question that starts with "in my vault" / "in my notes" /
  "across my knowledge base". Also trigger for "query", "查询", "search my notes",
  "问答", "ask my vault", "我的笔记里有没有", "帮我找一下笔记".
---

# Query（查询）

Answer natural language questions by searching, reading, and synthesizing
content from across the `gmemory/` knowledge workspace. Treat the vault as the user's
personal knowledge base to query against.

## Directory language

The vault may use Chinese or English directory names. When the user specifies
a scope (e.g., "in my learning notes"), resolve it against whichever variant
exists:

| Chinese | English |
|---------|---------|
| 学习/ | notes/ |
| 索引/ | index/ |
| 项目/ | projects/ |
| 收件箱/ | inbox/ |

Search both variants when the scope is ambiguous.

## Before you begin

1. Read `笔记规范.md` to understand the vault's structure and conventions.
2. DO NOT touch anything in `.obsidian/`.
3. **This skill is READ-ONLY.** Do not modify any notes unless the user
   explicitly asks you to create something new based on your findings --
   and even then, ask for confirmation first.

## Workflow

### Step 1: Parse the question

Analyze the user's question to identify:

- **Topic**: What subject or keyword are they asking about? Extract both
  the explicit terms and any implicit concepts.
- **Scope**: Where should you search?
  - All notes (default)
  - A specific directory (e.g., `学习/`, `项目/`)
  - Notes with a specific tag
  - Notes from a time period
- **Intent**: What kind of answer do they want?
  - A specific fact or definition
  - A summary or overview of a topic
  - A list of all notes on a topic
  - Connections or relationships between topics
  - A recommendation based on their notes
- **Language**: Note the language of the question -- search in that
  language first, then cross-reference in other languages if relevant.

### Step 2: Multi-strategy search

Run ALL of these search strategies, not just one. Execute independent
searches in parallel where possible.

**a) Filename search (Glob):**
Look for notes whose filenames match the topic keywords.
```
Glob: **/*<keyword>*.md
```

**b) Tag search (Grep):**
Search frontmatter for relevant tags.
```
Grep: "tags:[\s\S]*?- <keyword>" in *.md
```

**c) Content search (Grep):**
Search note bodies for the topic keyword and natural synonyms.
```
Grep -i: "<keyword>" in *.md
```
Search for at least 2-3 related terms, not just the exact keyword.

**d) Link graph search (Grep):**
Find notes that link to or are linked from relevant notes.
```
Grep: "\[\[<relevant note name>" in *.md
```

**e) Frontmatter field search (Grep):**
Check for mentions in title, source, and description fields.
```
Grep: "^(title|source|description):.*<keyword>" in *.md
```

### Step 3: Read and evaluate

From your search results, identify the most promising candidate notes.
Read at minimum the top 3-5 most relevant ones (more if the topic spans
many notes). For each candidate, evaluate:

- **Relevance**: Does it actually address the user's question, or is it
  just a keyword match?
- **Recency**: Check `created` and `updated` dates -- is the information
  still current?
- **Depth**: Is it a substantive note with real content, or just a stub?
- **Authority**: Does it cite sources? Is it a personal note or a clipping?

### Step 4: Synthesize the answer

Structure your response clearly:

```markdown
## 查询结果：<restate the question concisely>

### 直接回答
<A concise 1-3 paragraph synthesis drawn from the most relevant notes.
Use [[wiki links]] to cite the source notes so the user can navigate
to them directly.>

### 相关笔记
- [[笔记1]] -- <one line explaining why it's relevant>
- [[笔记2]] -- <one line explaining why it's relevant>
- [[笔记3]] -- <one line explaining why it's relevant>

### 相关概念
- [[概念A]] -- [[概念B]]

### 知识空白
<If the vault doesn't have good coverage, honestly note what's missing.
For example: "你的 vault 目前没有关于 X 的笔记。你可以考虑..." >
```

### Step 5: Offer follow-ups

Based on what you found (or didn't find), suggest 2-3 concrete next steps:

- "Would you like me to create a MOC for this topic? You have N notes
  scattered across different directories."
- "I found 2 notes that discuss the same concept but aren't linked.
  Want me to add cross-references?"
- "You have 5 scattered notes on this topic. Should I compile them into
  a single overview?"
- "Your vault doesn't have much on X. Should I help you create a stub
  note to collect future thoughts on this?"

## Search tips

- Search in Chinese AND English when the topic might cross languages
  (e.g., search for both "机器学习" and "machine learning").
- When a note filename is "Obsidian 使用指南与 Claude Code 联动", the
  link target is `[[Obsidian 使用指南与 Claude Code 联动]]` -- no `.md`.
- Use `Glob` with `**/*keyword*.md` to catch notes in subdirectories.
- When reading notes, pay attention to `**关键概念:**` and `**相关笔记:**`
  lines in `## 摘要` sections -- Compile puts cross-reference breadcrumbs
  there that are excellent for graph traversal.
- For time-based queries ("what did I write last month?"), filter by
  the `created` or `updated` frontmatter fields.

## Scope defaults

If the user doesn't specify scope:
- Search the ENTIRE vault EXCEPT `.obsidian/` and `.claude/` directories
- Include `Clippings/` in searches (unprocessed clips are still notes)
- Sort results by relevance to the question, not by date
- Prioritize notes with `status: 已完成` over `草稿` (completed notes are
  more reliable)

## When NOT to trigger

If the user is asking a factual question about the world (not about their
vault), answer normally without triggering Query. Example:
- "What is the capital of France?" → normal answer
- "What do I have in my vault about France?" → trigger Query
- "What is machine learning?" → normal answer (or suggest Compile if they
  want to create a note about it)
- "Based on my notes, how would you explain machine learning?" → trigger Query
