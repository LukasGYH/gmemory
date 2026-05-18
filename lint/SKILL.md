---
name: lint
description: >
  Run health checks on the Obsidian vault: find broken links, inconsistent frontmatter,
  orphan notes, stale notes, and tag issues. ALWAYS trigger this skill when the user
  says "lint", "维护", "check my vault", "vault health", "health check", "audit my notes",
  "find broken links", "orphan notes", "fix frontmatter", "clean up tags",
  "note hygiene", "检查笔记", "整理标签", or any variation of "is my vault healthy?" /
  "check my knowledge base for issues". Also trigger proactively if the user has run
  Ingest and Compile and is asking "what next?" -- Lint is the natural maintenance step.
---

# Lint（维护）

Audit the `gmemory/` knowledge workspace for structural issues and produce an actionable health
report. Follows the standards defined in [[笔记规范]].

## Directory language

The vault may use Chinese or English directory names. Detect the language
by checking which variant exists, and use it consistently in all reports:

| Chinese | English | Purpose |
|---------|---------|---------|
| `gmemory/收件箱/` | `gmemory/inbox/` | Incoming fragments |
| `gmemory/学习/` | `gmemory/notes/` | Notes and learning content |
| `gmemory/学习/concepts/` | `gmemory/notes/concepts/` | Concept notes |
| `gmemory/索引/` | `gmemory/index/` | MOCs and indexes |
| `gmemory/项目/` | `gmemory/projects/` | Project-specific notes |
| `gmemory/conflict/` | `gmemory/conflicts/` | Semantic conflicts |

If both variants exist, report paths using whichever is primary (Chinese takes
precedence).

## Before you begin

1. Read `笔记规范.md` to confirm the current frontmatter and naming
   standards you're checking against.
2. DO NOT touch anything in `.obsidian/`.
3. **This skill is primarily DIAGNOSTIC (read-only).** It reports issues
   and proposes fixes. It must NOT apply fixes without the user's explicit
   approval. Show a before/after preview for each proposed fix and ask
   "Apply these fixes?" before making any changes.

## Workflow

### Step 1: Discover all notes

Use Glob to find every `.md` file in the vault:
```
Glob: **/*.md
```

Exclude these directories from the note list:
- `.obsidian/**` (Obsidian configuration, off-limits)
- `.claude/**` (Claude Code configuration)
- `.trash/**` (deleted files, if it exists)
- `附件/**` (attachments like images and PDFs, not markdown notes)

Build a master list of note file paths. This is the universe of notes
against which all checks run.

### Step 2: Run all six checks

Run the independent checks in parallel where possible. Each check
produces its own section of the final report.

---

#### Check 1: Broken `[[wiki links]]`

For every `[[link]]` in every note:

1. Extract the link target -- the part before any `|` (alias) or `#`
   (heading anchor). For `[[笔记名|显示文本]]`, the target is `笔记名`.
   For `[[笔记名#标题]]`, the target is `笔记名`.
2. Check if a file named `<target>.md` exists anywhere in the vault.
3. A link is BROKEN only if no matching `.md` file exists. Links to
   headings within existing notes are valid (Obsidian handles them).

Report format:
```
### Broken Links
| Source Note | Broken Link | Suggested Action |
|-------------|-------------|------------------|
| 学习/Obsidian.md | [[不存在的笔记]] | Create stub note or remove the link |
| 项目/Plan.md | [[OldProject]] | Remove or replace with current project note |
```

**Severity**: HIGH. Broken links degrade the knowledge graph.

---

#### Check 2: Frontmatter consistency

For every note, verify these requirements from [[笔记规范]]:

| Check | Rule |
|-------|------|
| `tags` present | Every note must have at least one tag |
| `tags` format | Must use list format (`- tag`), not inline (`[tag1, tag2]`) |
| `created` present | Must be in `YYYY-MM-DD` format |
| `updated` present | Not required but strongly recommended |
| `status` valid | If present, must be one of: `草稿`, `进行中`, `已完成`, `归档` |
| Valid YAML | The frontmatter must be parseable (no unbalanced quotes, no bad indentation) |

Report format:
```
### Frontmatter Issues
| Note | Issue | Detail |
|------|-------|--------|
| 学习/SomeNote.md | Missing `tags` | Has no tags frontmatter field |
| 学习/OtherNote.md | Invalid `status` | Value is "done" -- should be "已完成" |
| 项目/Note.md | Bad `tags` format | Uses `[tag1, tag2]` -- should be list format |
```

---

#### Check 3: Orphan notes

An orphan is a note with ZERO incoming `[[wiki links]]` -- no other note
in the vault references it.

1. Extract every `[[link]]` target from every note's body. This produces
   the set of notes that ARE linked to.
2. Compare against the master list of notes. Any note that exists but is
   never the target of a `[[link]]` is potentially orphaned.

**Important nuance**: Not all unlinked notes are problems:
- **Expected unlinked**: MOCs (`索引/`), templates (`模板/`), daily notes
  (`日记/`), and any note with `status: 草稿` that was recently created.
- **True orphans**: Notes in `学习/` or `项目/` that have been around for
  a while but nobody links to them.

Report format:
```
### Orphan Notes

#### Likely True Orphans (should be linked or reviewed)
- 学习/IsolatedConcept.md -- No incoming links, status: 草稿, created: 2026-03-01
  → Consider linking from related notes or creating a MOC entry
- 项目/OldProject/Notes.md -- No incoming links, status: 进行中
  → Is this project still active? Consider archiving if not.

#### Expected Unlinked (MOCs, templates, diaries -- typically OK)
- 日记/2026-05-10.md -- Daily note, naturally not linked from body notes
- 模板/ProjectTemplate.md -- Template file
```

---

#### Check 4: Stale notes

Find notes that may need attention based on their dates:

| Condition | Threshold | What it means |
|-----------|-----------|---------------|
| `updated` > 90 days ago | 90 days | May contain outdated information |
| `status: 草稿` + `created` > 30 days ago | 30 days | Possibly abandoned draft |
| `status: 进行中` + `updated` > 60 days ago | 60 days | Possibly stalled project |
| No `updated` field + old `created` date | 90 days | No update history -- hard to assess freshness |

Calculate "days stale" as the difference between today and the `updated`
date (or `created` date if no `updated` exists).

Report format:
```
### Stale Notes
| Note | Last Updated | Status | Days Stale | Suggested Action |
|------|-------------|--------|-----------|-----------------|
| 学习/OldTutorial.md | 2025-11-01 | 已完成 | 192 | Review for relevance, consider 归档 |
| 项目/Idea.md | 2026-03-01 | 草稿 | 72 | Either develop further or archive |
| 学习/ConceptNote.md | 2026-01-15 | 进行中 | 117 | Stalled project -- check if still relevant |
```

---

#### Check 5: Tag audit

Analyze all tags used across the vault:

1. Count tag frequency -- list all unique tags and how many notes use each.
2. Flag **single-use tags** -- tags that appear on exactly one note (may be
   typos or overly specific).
3. Flag **near-duplicates** -- tags that look like the same thing with
   slight variations (e.g., "obsidian" vs "Obsidian" vs "obisidian",
   "machine-learning" vs "机器学习").
4. Flag **redundant tags** -- tags that duplicate information already
   expressed by the note's directory location. For example, a tag "项目A"
   on a note already in `项目/项目A/` is redundant.
5. Flag **orphan tags** -- tags that appear in frontmatter but have no
   corresponding MOC or index note (low priority, informational only).

Report format:
```
### Tag Audit

#### Tags by frequency
- obsidian: 5 notes
- 插件: 3 notes
- clippings: 1 note
- 机器学习: 2 notes
- obisidian: 1 note ← TYPO

#### Recommended Renames
| From | To | Affected Notes | Reason |
|------|----|---------------|--------|
| obisidian | obsidian | 学习/SomeNote.md | Typo |
| machine-learning | 机器学习 | 2 notes | Inconsistent language (vault is primarily Chinese) |

#### Redundant Tags
- Tag "clippings" on notes already in `Clippings/` directory (consider removing after ingest)

#### Orphan Tags (no MOC or index note)
- 性能优化 (1 note)
```

---

#### Check 6: Clippings backlog

Check `Clippings/` for files that still have the raw clipper frontmatter
format -- these are unprocessed clips waiting for Ingest.

A file is "unprocessed" if its frontmatter contains:
- A `source:` field with a URL, AND/OR
- A `tags` field that includes `"clippings"` as the primary/only tag

Report format:
```
### Clippings Backlog
| File | Clipped Date | Likely Topic |
|------|-------------|--------------|
| Clippings/obsidian 插件推荐.md | 2026-05-12 | Obsidian plugins |
→ Run **Ingest** to process these clippings into proper vault notes.
```

---

### Step 3: Generate the health scorecard

Combine all checks into a single report with an overall scorecard:

```markdown
# Vault Health Report -- <today's date>

## Scorecard
| Check | Status | Count |
|-------|--------|-------|
| Broken Links | 🔴 N issues | N |
| Frontmatter | 🟡 N issues | N |
| Orphan Notes | 🟡 N notes | N |
| Stale Notes | 🟢 N notes | N |
| Tag Audit | 🔴 N issues | N |
| Clippings Backlog | 🟢 N files | N |

🔴 = Action needed (broken links, missing required frontmatter fields)
🟡 = Should review (orphans, stale notes, tag normalization)
🟢 = Looking good

## Top 3 Recommended Actions
1. <Most impactful fix -- broken links or missing required fields>
2. <Second most impactful -- tag cleanup or frontmatter normalization>
3. <Third most impactful -- stale note review or orphan linkage>
```

The scorecard uses these thresholds:
- 🔴: Any broken links, OR any note missing required frontmatter fields (`tags`, `created`)
- 🟡: Orphans > 10% of notes, OR stale notes > 20% of all notes, OR tag near-duplicates found
- 🟢: No issues found in this category

### Step 4: Propose fixes (NEVER auto-apply)

After presenting the full report, offer to fix specific issues:

```
Would you like me to fix any of these? Here are the most impactful:

1. Fix 3 broken links by creating stub notes (creates N new .md files)
2. Add missing `tags` frontmatter to 2 notes (edits N files)
3. Rename tag "obisidian" → "obsidian" in 1 note (edits 1 file)
4. Archive 2 stale draft notes by setting status → 归档

Which would you like me to address?
```

For each fix the user approves, show a before/after preview before editing.
Never batch-apply fixes; the user must confirm each category or the whole
set.

### Step 5: Schedule recommendation

Lint is designed to be run regularly. After the report, suggest a cadence:

- **Weekly**: Good for active vaults with frequent new notes
- **After every Ingest+Compile cycle**: Catch issues early
- **Monthly**: Sufficient for slower-moving vaults
- **Before major reorganizations**: Prevent propagating existing issues

The user can always just say "lint" for an ad-hoc check.

## Special cases

- **Vaults with few notes** (<10): Skip the orphan check entirely -- at
  this stage, everything looks like an orphan. Focus on frontmatter
  consistency and the clippings backlog.
- **Daily notes (日记/)**: These are expected to be unlinked from body
  notes. Don't flag them as orphans.
- **Template files (模板/)**: These are reference files, not content notes.
  Don't flag them as orphans or require status fields.
- **Notes with no frontmatter at all**: Treat as having ALL fields missing.
  Report the full list of missing fields. This is common for imported or
  legacy notes.
