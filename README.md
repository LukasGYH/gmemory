# gmemory — Claude Code 知识管理技能包

一组可移植的 Claude Code Skill，将 `/remember`、Web Clipper、`/import` 捕获的碎片知识，通过 Ingest → Compile → Query → Lint 流水线加工成互联的知识图谱。所有 Skill 限定在 `gmemory/` 目录内工作，不污染项目代码。

```
Web Clipper ──→ gmemory/Clippings/
/remember ────→ gmemory/收件箱/    ──→ Ingest ──→ Compile ──→ Query
/import ──────→ gmemory/收件箱/          ↑                        |
                                         +──── Lint ←─────────────+
```

## 六个 Skill

| Skill | 触发 | 职责 |
|-------|------|------|
| **remember** | `/remember`、记住、记一下 | Agent 工作中随手记录碎片洞察（三行极简格式） |
| **import** | `/import`、导入、import chat | 批量导入外部知识：聊天记录、.md、PDF → 归一化 |
| **ingest** | `/ingest`、摄入、处理剪藏 | 清洗 Clippings + 收件箱碎片，标准化 frontmatter |
| **compile** | `/compile`、编译、提取概念 | 生成摘要、提取概念、建 MOC 索引、交叉引用 |
| **query** | `/query`、我的笔记里有没有 | 多策略搜索 + 综合回答（只读） |
| **lint** | `/lint`、检查笔记、vault health | 断链 / frontmatter / 孤立笔记 / 过期 / 标签审计 |

## 安装

```bash
# 1. 将 skills 目录复制到目标工作区的 .claude/skills/
cp -r skills/* /path/to/your/project/.claude/skills/

# 2. 在工作区根目录创建 gmemory 子目录结构
mkdir -p gmemory/{Clippings,收件箱,学习/concepts,索引,项目,附件}

# 3. 重新加载插件
# 在 Claude Code 中执行: /reload-plugins
```

## 快速开始

```bash
# 随手记一条碎片
/remember "K8s Job 的 backoffLimit 默认值是 6"

# 导入一段 ChatGPT 对话
/import "paste your chat here..."

# 用 Obsidian Web Clipper 剪藏网页 → Clippings/ 目录

# 清洗入库
/ingest

# 生成摘要、提取概念、建立链接
/compile

# 自然语言检索
/query "我有哪些关于 Rancher 的笔记？"

# 定期健康检查
/lint
```

## 目录结构 (gmemory/)

```
gmemory/
├── Clippings/          # Web Clipper 保存的原始剪藏
├── 收件箱/             # remember / import 写入的碎片
├── 学习/               # 知识笔记
│   └── concepts/       # Compile 自动创建的概念页
├── 索引/               # MOC (Map of Content) 主题索引
├── 项目/               # 按项目归类的笔记
└── 附件/               # 图片、PDF
```

## 原理

### 为什么分阶段？

原始知识和碎片直接丢进知识库会变成垃圾堆。分阶段加工让每步只做一件事：

- **capture**（remember/import）：轻量录入，零 friction
- **clean**（ingest）：清洗格式、标准化 frontmatter
- **connect**（compile）：LLM 生成摘要、提取概念、建立 MOC 和交叉引用
- **query**（query）：多策略搜索 + 综合回答
- **audit**（lint）：6 项健康检查

### 为什么限定 gmemory/？

- **可移植**：Skill 路径全部相对 `gmemory/`，复制到任何项目即用
- **隔离**：知识和项目代码物理分离
- **跨 Agent**：Claude Code、Trae SOLO 等不同 agent 共用同一套知识图谱

### 笔记规范

所有笔记遵循统一的 frontmatter 标准：

```yaml
---
tags:
  - 标签1
  - 标签2
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: 草稿 | 进行中 | 已完成 | 归档
---
```

## 示例：一条知识的旅程

```
/remember "ZGC 在 JDK 23 默认启用了分代模式"
    ↓
gmemory/收件箱/2026-05-15-jdk-zgc-generational.md
    ↓  /ingest
gmemory/学习/JDK ZGC 分代模式.md  (tags: java, jdk, 性能, status: 草稿)
    ↓  /compile
+ ## 摘要 (LLM 生成)
+ **关键概念:** [[Java]], [[JDK]], [[ZGC]]
+ 交叉引用: ↔ [[JDK 虚拟线程与GC选型]]
+ status → 已完成
    ↓  /query "JDK GC 演进"
→ 综合 [[JDK 虚拟线程与GC选型]] + [[JDK ZGC 分代模式]] 给出回答
```

## 适用场景

- **Obsidian 知识库**：配合 Web Clipper 自动化剪藏→笔记链路
- **项目文档积累**：开发过程中把碎片洞察沉淀为结构化知识
- **多 Agent 协作**：不同 AI 助手共享同一套记忆
- **个人第二大脑**：任何需要长期积累、定期检索的知识体系

## 依赖

- Claude Code（或任何支持 Skill 的 Claude 运行环境）
- 基础文件系统工具（Grep, Glob, Read, Edit, Write）

## License

MIT
