# gmemory — Claude Code Knowledge Management Skill Pack

A portable set of Claude Code Skills that transform fragmented knowledge from `/remember`, Web Clipper, and `/import` into an interconnected knowledge graph through an Ingest → Compile → Query → Lint pipeline. All Skills operate within the `gmemory/` directory — zero pollution to your project code.

> [中文🇨🇳](doc/README_CN.md)

```
Web Clipper ──→ gmemory/Clippings/
/remember ────→ gmemory/Inbox/    ──→ Ingest ──→ Compile ──→ Query
/import ──────→ gmemory/Inbox/          ↑                        |
                                        +──── Lint ←─────────────+
```

## Six Skills

| Skill | Trigger | Responsibility |
|-------|---------|---------------|
| **remember** | `/remember`, "note this", "save this" | Capture fleeting insights during agent work (3-line minimal format) |
| **import** | `/import`, "import chat" | Batch import external knowledge: chat logs, .md, PDF → normalize |
| **ingest** | `/ingest`, "process clippings" | Clean Clippings + Inbox fragments, standardize frontmatter |
| **compile** | `/compile`, "extract concepts" | Generate summaries, extract concepts, build MOC indexes, cross-reference |
| **query** | `/query`, "what do I have on..." | Multi-strategy search + synthesized answers (read-only) |
| **lint** | `/lint`, "check vault health" | Broken links / frontmatter / orphans / staleness / tag audit |

## Installation

```bash
# 1. Copy skills to your workspace's .claude/skills/
cp -r skills/* /path/to/your/project/.claude/skills/

# 2. Create the gmemory directory structure at workspace root
mkdir -p gmemory/{Clippings,Inbox,Learning/concepts,Indexes,Projects,Attachments}

# 3. Reload plugins
# In Claude Code: /reload-plugins
```

## Quick Start

```bash
# Capture a quick insight
/remember "K8s Job backoffLimit defaults to 6"

# Import a ChatGPT conversation
/import "paste your chat here..."

# Clip web pages with Obsidian Web Clipper → Clippings/ directory

# Clean and ingest
/ingest

# Generate summaries, extract concepts, build links
/compile

# Natural language retrieval
/query "what notes do I have about Rancher?"

# Periodic health check
/lint
```

## Directory Structure (gmemory/)

```
gmemory/
├── Clippings/          # Raw clips saved by Web Clipper
├── Inbox/              # Fragments written by remember / import
├── Learning/           # Knowledge notes
│   └── concepts/       # Auto-generated concept pages by Compile
├── Indexes/            # MOC (Map of Content) topic indexes
├── Projects/           # Notes organized by project
└── Attachments/        # Images, PDFs
```

## Design

### Why phased?

Dumping raw knowledge and fragments straight into a knowledge base creates a landfill. A phased pipeline ensures each step does one thing well:

- **capture** (remember/import): lightweight entry, zero friction
- **clean** (ingest): sanitize format, standardize frontmatter
- **connect** (compile): LLM generates summaries, extracts concepts, builds MOCs and cross-references
- **query** (query): multi-strategy search + synthesized answers
- **audit** (lint): 6 health checks

### Why scoped to gmemory/?

- **Portable**: all Skill paths are relative to `gmemory/` — copy to any project and it just works
- **Isolated**: knowledge and project code are physically separated
- **Cross-agent**: Claude Code, Trae SOLO, and other agents share the same knowledge graph

### Note Conventions

All notes follow a unified frontmatter standard:

```yaml
---
tags:
  - tag1
  - tag2
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: draft | in-progress | done | archived
---
```

## Example: A Knowledge Fragment's Journey

```
/remember "ZGC enables generational mode by default in JDK 23"
    ↓
gmemory/Inbox/2026-05-15-jdk-zgc-generational.md
    ↓  /ingest
gmemory/Learning/JDK ZGC Generational Mode.md  (tags: java, jdk, performance, status: draft)
    ↓  /compile
+ ## Summary (LLM-generated)
+ **Key concepts:** [[Java]], [[JDK]], [[ZGC]]
+ Cross-reference: ↔ [[JDK Virtual Threads and GC Selection]]
+ status → done
    ↓  /query "JDK GC evolution"
→ Synthesizes [[JDK Virtual Threads and GC Selection]] + [[JDK ZGC Generational Mode]] into an answer
```

## Use Cases

- **Obsidian vault**: automate clip → note pipeline with Web Clipper
- **Project documentation**: crystallize fleeting insights into structured knowledge during development
- **Multi-agent collaboration**: different AI assistants share the same memory
- **Personal second brain**: any knowledge system that requires long-term accumulation and periodic retrieval

## Dependencies

- Claude Code (or any Claude runtime with Skill support)
- Basic filesystem tools (Grep, Glob, Read, Edit, Write)

## License

MIT

