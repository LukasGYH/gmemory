---
name: compile
description: >
  LLM-powered processing of vault notes: summarization, concept extraction,
  MOC (Map of Content) building, and cross-referencing. ALWAYS trigger this skill
  when the user says "compile", "编译", "summarize my notes", "summarize note",
  "extract concepts", "build MOC", "create index", "map of content", "link my notes",
  "find connections", "cross-reference", "connect my notes", or asks to "process" or
  "synthesize" their knowledge base. Also trigger after Ingest has run and the user
  wants next steps, or when the user points to specific notes and says "help me make
  sense of this" / "帮我整理一下".
---

# Compile（编译）

Transform raw and ingested notes into a connected knowledge graph through
summarization, concept extraction, and MOC building. All created content
follows [[笔记规范]].

## Directory language

This skill supports bilingual directory structures. On first access, detect
which language the vault uses by checking for existing directories:

| Chinese | English | Purpose |
|---------|---------|---------|
| `gmemory/收件箱/` | `gmemory/inbox/` | Incoming fragments |
| `gmemory/学习/` | `gmemory/notes/` | Notes and learning content |
| `gmemory/学习/concepts/` | `gmemory/notes/concepts/` | Concept notes |
| `gmemory/索引/` | `gmemory/index/` | MOCs and indexes |
| `gmemory/项目/` | `gmemory/projects/` | Project-specific notes |
| `gmemory/conflict/` | `gmemory/conflicts/` | Semantic conflicts |

Rule: check which variant exists and use it for all operations. If both exist,
prefer Chinese. If neither exists, create the Chinese variant (backward compatible).

## Before you begin

1. Read `笔记规范.md` for the current frontmatter and linking conventions.
2. DO NOT touch anything in `.obsidian/`.

## Workflow

### Step 1: Determine scope

Ask the user what to compile. Present these options clearly:

- **Recent**: Notes with `status: 草稿` or no status -- typically newly
  ingested content waiting for processing.
- **Directory**: All notes under a specific directory (e.g., everything in
  `gmemory/学习/` or `gmemory/项目/ProjectName/`).
- **Tag**: All notes sharing one or more tags (e.g., everything tagged
  `obsidian`).
- **Named**: A specific list of notes the user names.
- **All**: The entire vault (use sparingly -- confirm with the user).

If the user doesn't specify a scope, default to **Recent** (notes with
`status: 草稿` or `updated` within the last 7 days, or notes that lack
a `## 摘要` section entirely).

### Step 2: Read and analyze

For each note in scope:
1. Read the full content.
2. Identify:
   - **Core topic**: The one or two things this note is primarily about
   - **Key concepts**: Terms, ideas, frameworks, or entities the note
     discusses or depends on
   - **Named entities**: People, tools, projects, companies, or other
     proper nouns mentioned
   - **Existing links**: Any `[[wiki links]]` already in the note
   - **Current tags and status**: From frontmatter

Keep these in mind through the next steps -- you'll use them to build
summaries, concept notes, and cross-references.

### Step 3: Generate or update summaries

For each note, add (or update) a `## 摘要` section. Place it right after
the frontmatter and before the main content:

```markdown
## 摘要
<2-4 sentence summary capturing the main argument, key findings,
or purpose of the note. Write this in the note's primary language.>

**关键概念:** [[概念A]], [[概念B]]
**相关笔记:** [[笔记X]], [[笔记Y]]
```

The `[[concept links]]` and `[[note links]]` are initially placeholders.
You will resolve them in the next steps by creating actual concept notes
and finding real related notes.

### Step 4: Extract concepts and create concept notes

Concepts are the atomic units of knowledge: terms, ideas, frameworks,
or entities that appear across multiple notes or are significant enough
to deserve their own page.

For each key concept identified in Step 2:
1. **Check if a concept note already exists.** Search the vault for a note
   with the concept name as its filename (e.g., `概念名.md` or a similar
   variant).
2. **If it exists**, just make sure it's linked from the current note.
3. **If it doesn't exist**, create one in `gmemory/学习/concepts/` or `gmemory/学习/`
   (use `gmemory/学习/concepts/` if you anticipate many concept notes). Use this
   template:

```markdown
---
tags:
  - concept
  - <domain-tag>
created: <today>
updated: <today>
status: 草稿
---
# <概念名>

## 定义
<1-2 sentence definition of what this concept means>

## 出处
- [[来源笔记1]] -- <how this concept appeared in that note>
- [[来源笔记2]] -- <how this concept appeared in that note>

## 相关概念
- [[相关概念A]]
- [[相关概念B]]
```

4. **Update the original note's `**关键概念:**` line** to link to the
   now-existing concept note.

### Step 5: Build or update MOCs (索引 notes)

An MOC (Map of Content) is a navigational index note in `gmemory/索引/` that
aggregates `[[links]]` to related notes around a theme or topic.

Check `gmemory/索引/` for existing MOCs that relate to the current scope:

**If a relevant MOC already exists**: Add links to the newly processed
notes and any new concept notes, under the appropriate section heading.

**If no relevant MOC exists**, create one:

```markdown
---
tags:
  - moc
  - <topic-tag>
created: <today>
updated: <today>
status: 进行中
---
# <Topic> MOC

## 核心概念
- [[概念1]] -- <one-line description>
- [[概念2]] -- <one-line description>

## 笔记
- [[笔记1]] -- <one-line description of what this note covers>
- [[笔记2]] -- <one-line description>

## 外部资源
- [Resource Name](URL) -- <brief description of the external resource>
```

### Step 6: Cross-reference

Look for notes in the vault that discuss the same concepts but don't
yet link to each other. This is where you build the knowledge graph.

1. For each processed note, search the vault (using Grep) for mentions
   of its key concepts.
2. Where you find related notes that aren't already linked, add a
   `**另见:**` line at the end of each note's `## 摘要` section:

```markdown
**另见:** [[other-related-note]], [[another-related-note]]
```

3. Make cross-references bidirectional -- if A links to B, add a link
   back from B to A.

### Step 7: Detect semantic conflicts

When multiple notes discuss the same concept, tool, process, or factual
claim, check whether they **disagree**. This is especially important in
team workflows where different members contribute fragments via git --
two people may record conflicting observations about the same thing.

**What constitutes a conflict:**

| Type | Example |
|------|---------|
| Contradictory claims | "K8s Job backoffLimit 默认 6 次" vs "backoffLimit 默认值是 3" |
| Opposing recommendations | "用 Redis 做缓存" vs "不要用 Redis，用本地缓存" |
| Incompatible version/date info | "JDK 21 是当前 LTS" vs "JDK 23 是当前 LTS" |
| Same process, different steps | 两人描述同一个部署流程但步骤矛盾 |
| Terminology clash | 同一个东西叫不同的名字，或不同东西叫同一个名字 |

**How to detect:**

1. During Step 6 (cross-reference), when you find two notes that share
   a key concept, read the **specific claims** each note makes about that
   concept -- not just that they both mention it.
2. Compare claims pairwise. If they make assertions about the same
   property (default value, recommended approach, version, definition)
   that cannot all be true, flag it.
3. Use your own knowledge to judge whether the difference is a genuine
   contradiction or just different perspectives/contexts. For example:
   - "K8s 1.28 中 X 默认开启" vs "K8s 1.26 中 X 默认关闭" -> same fact,
     different versions, NOT a conflict
   - "backoffLimit 默认 6" vs "backoffLimit 默认 3" -> same property,
     same context -> IS a conflict

**When a conflict is found:**

1. **Create `gmemory/conflict/`** if it doesn't exist.
2. **Create a conflict note** at `gmemory/conflict/<topic>-语义冲突.md`:

```markdown
---
tags:
  - conflict
  - <domain-tag>
created: <today>
status: 待决策
---
# <Topic> -- 语义冲突

## 冲突摘要
<1-2 sentences describing the disagreement in plain language.>

## 冲突来源

### 观点 A
- **来源:** [[源笔记1]]
- **主张:** <what this note claims>
- **上下文:** <relevant quote or context from the note>

### 观点 B
- **来源:** [[源笔记2]]
- **主张:** <what this note claims>
- **上下文:** <relevant quote or context from the note>

## 待决策
- [ ] 确认哪个说法正确（或两者在不同条件下成立）
- [ ] 更新或合并冲突笔记
- [ ] 决策后删除本冲突文件或标记 status: 已解决
```

3. **Do NOT modify the conflicting source notes.** Leave them as-is --
   the lead must decide which is correct before changes are made.

4. **Accumulate all conflicts.** Continue processing all notes; don't stop
   at the first conflict. Collect them all for a unified report.

**Present conflicts to the user (the lead):**

After cross-referencing is complete, present ALL conflicts found before
proceeding to Step 8 (Update status):

```
### 语义冲突检测到 N 个冲突

| # | 主题 | 冲突来源 | 类型 |
|---|------|---------|------|
| 1 | K8s backoffLimit 默认值 | [[笔记A]] vs [[笔记B]] | 矛盾主张 |
| 2 | 缓存方案选择 | [[笔记C]] vs [[笔记D]] | 对立建议 |

冲突文件已创建在 gmemory/conflict/ 下。

请逐一决策后，我可以继续更新笔记状态并完成编译。
是否现在查看冲突详情并逐一处理？
```

Wait for the user's decision. Do NOT proceed to Step 8 until the user
has reviewed conflicts (they may choose to skip and resolve later, which
is fine -- the conflict files persist in `gmemory/conflict/` as a todo list).

If no conflicts are found, report this briefly and proceed directly to Step 8.

### Step 8: Update status

For each note you've processed, update its frontmatter:
- Set `status` to `已完成` (unless the content is clearly unfinished)
- Set `updated` to today's date

For concept notes you created, keep them as `status: 草稿` -- they will
grow as more notes reference them.

Important: If conflicts were detected, only update status for notes
NOT involved in any conflict. Leave conflicting notes at their current
status until the lead resolves the conflict.

### Step 9: Summarize what was done

Present a clear summary to the user:

```
## Compile 完成

### 笔记已处理 (N)
- gmemory/学习/Note1.md -- 已添加摘要, 状态 → 已完成
- gmemory/学习/Note2.md -- 已添加摘要, 状态 → 已完成

### 概念笔记已创建 (N)
- gmemory/学习/concepts/概念A.md
- gmemory/学习/concepts/概念B.md

### MOC 已更新 (N)
- gmemory/索引/Obsidian MOC.md -- 新增 3 个链接

### 新建交叉引用 (N)
- Note1 ↔ Note2 (共同概念: 概念A)
- Note3 ↔ Note1 (共同概念: 概念B)

### 语义冲突 (N)
- gmemory/conflict/缓存方案-语义冲突.md -- [[笔记C]] vs [[笔记D]]
```

### Step 10: Flag knowledge gaps

As you compile, you may notice concepts referenced by multiple notes
but lacking a dedicated note in the vault. Flag these as gaps:

```
### 知识空白
- **[[RAG]]** -- Mentioned in 3 notes but has no dedicated page
- **[[Markdown 渲染]]** -- Referenced but not explained anywhere yet
```

Offer to create stub notes for these gaps (marked `status: 草稿`).

## Guidelines

- **Language**: Write summaries and concept definitions in the same language
  as the source note (Chinese for Chinese notes, English for English notes).
- **Link granularity**: Don't link every word. Link concepts that are
  meaningful knowledge nodes -- things someone would want to navigate to.
- **MOC freshness**: An MOC is a living index. Update it when new notes
  arrive in its domain; don't create a new one for every compile run.
- **Avoid over-linking**: A note with 20 concept links is noisy. Be selective.
  Target 2-5 key concept links per note. 
