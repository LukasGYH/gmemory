---
name: ingest
description: >
  Process raw web clippings from the gmemory/Clippings/ directory into standardized vault notes.
  ALWAYS use this skill whenever the user mentions "ingest", "摄入", "process clippings",
  "organize my clippings", "clean up web clips", "convert clippings", "整理剪藏",
  "处理剪藏", or asks to process anything in the gmemory/Clippings/ folder. Also trigger when
  the user says they just saved something from the web and want it organized into their
  vault, or when gmemory/Clippings/ has unprocessed files and the user asks "what should I do
  next?" after browsing.
---

# Ingest（摄入）

Convert raw Obsidian Web Clipper output into polished vault notes that follow
the conventions in [[笔记规范]].

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
`gmemory/Clippings/` is English-only — it matches the Obsidian Web Clipper default.

## Before you begin

1. Read `笔记规范.md` to refresh yourself on the vault's frontmatter and
   linking conventions.
2. List all `.md` files in `gmemory/Clippings/` AND `gmemory/收件箱/` to identify unprocessed content.
3. DO NOT touch anything in `.obsidian/`.

## Workflow

### Step 1: Scan and report

List every file in `gmemory/Clippings/` and present a table to the user:

| File | Has clipper frontmatter? | Likely topic | Suggested destination |
|------|--------------------------|--------------|----------------------|

A file has "clipper frontmatter" if its YAML frontmatter contains `source:`
(with a URL), or a `tags` field that includes `"clippings"`, or a `title`
field paired with a URL-based `source` -- these are the telltale signs
of Obsidian Web Clipper output.

Wait for the user to confirm which files to process and any corrections to
your suggested destinations. Do NOT edit files until confirmed.

### Step 2: Process each clipping

For each approved clipping:

**a) Read the clipping** to understand its full content.

**b) Convert frontmatter** from the clipper format to the vault standard.

The clipper format typically has:
```yaml
title: "..."
source: "https://..."
author: "..."
published: "..."
created: YYYY-MM-DD
description: "..."
tags:
  - clippings
```

Transform to vault standard:
```yaml
---
tags:
  - <inferred-tag-1>
  - <inferred-tag-2>
created: <original created date, or today if missing>
updated: <today>
status: 草稿
source: "<original URL if available>"
---
```

- The `source` field is preserved as extra context but is not part of the
  core vault schema.
- The `title` from clipper becomes the `# Title` heading in the note body.
- Original `author` and `published` are preserved as body content if they
  add value; otherwise omit them.

**c) Infer tags** by reading the content:

- Extract the primary topic (e.g., "obsidian", "插件", "机器学习")
- Add context/layer tags (e.g., "工具", "教程", "参考", "文献")
- Tags should be in the same language as the note's primary content
- Aim for 2-4 tags total -- specific enough to be useful, not so many they
  lose meaning
- Use existing vault tags when possible (check what tags are already in use)

**d) Clean web clipping artifacts.** These are common in clipper output:

- Remove "网上的精选摘要" / "Featured snippets from the web" headers
- Remove redundant source URLs that appear as inline text in the body
- Remove "显示更多图片" / "Show more images" snippets
- Remove "另外 N 行" / "N more rows" fragments
- Remove "N天前" / "N hours ago" / "N days ago" timestamp fragments
- Remove "更多结果" / "More results" and "相关搜索" sections
- Collapse multiple consecutive blank lines into one
- Remove navigation breadcrumbs and "..." placeholder lines
- Remove "您的搜索 -" prefixed headers
- Remove "People also ask" / "大家还搜了" sections
- Preserve the actual article links and their descriptions as structured content
- If the clip is a list of search results, format them as a clean bullet list
  with `[Title](URL)` links and one-line descriptions

**e) Move the file** to the target directory:

- `gmemory/学习/` for educational content, tutorials, reference material,
  theoretical articles
- `gmemory/项目/` for project-specific content (create a subdirectory like
  `gmemory/项目/ProjectName/` if the content belongs to a specific project)
- `gmemory/收件箱/` if the user is unsure or the content is temporary/transient

Create the target directory if it doesn't exist.

**f) Rename the file** if needed. Clipping filenames often include suffixes
like " - Google 搜索", " - Google Search", or similar search-engine
artifacts. Strip these and use a clean, descriptive Chinese title.
Also remove any trailing whitespace or odd characters.

### Step 3: Optional first-pass summary

Ask the user: "Would you like me to add a preliminary summary to each note?
(Compile will do a more thorough job later.)"

If yes, append a `## 摘要` section after the frontmatter:

```markdown
## 摘要
<2-3 sentence summary of the key points and why it matters>

**关键概念:** [[ConceptA]], [[ConceptB]]
**相关笔记:** (to be populated by Compile)
```

The `[[concept links]]` here are placeholders -- Compile will create the
actual concept notes and update these links.

### Step 4: Final report

After processing all approved files, present a summary:

```
## Ingest 完成

| 文件 | 原位置 | 新位置 | 标签 | 状态 |
|-------|--------|--------|------|------|
| xxx.md | gmemory/Clippings/ | gmemory/学习/ | obsidian, 插件 | 草稿 |
```

Also:
- Note any formatting issues that could not be automatically resolved
- Remind the user: "Ready for Compile -- run 'compile my new notes' when
  you're ready to build links and indexes."

## Important conventions

- All internal note links use `[[笔记名]]` syntax, never Markdown
  `[text](note.md)` for vault notes.
- External URLs use standard Markdown `[Title](URL)` syntax.
- New directories under `gmemory/学习/` or `gmemory/项目/` should be created as needed
  (use `mkdir` via Bash or just write the file to the new path).
- Never delete the original clipping until the user confirms the processed
  version looks correct.
- Status of all freshly ingested notes is `草稿` (draft) -- Compile will
  update them to `已完成` after processing.
- Update the `updated` field in frontmatter whenever you edit a note.
