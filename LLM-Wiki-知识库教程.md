# LLM Wiki 完全教程：用 AI 构建持久化知识库

> 作者：Andrej Karpathy（OpenAI 联合创始人、前 Tesla AI 总监）于 2026 年 4 月提出此概念。  
> 原文：[LLM Wiki — A pattern for building personal knowledge bases using LLMs](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)  
> 本教程综合原始理念、社区实践、工具生态和批评视角，提供完整的上手指南。

---

## 目录

1. [什么是 LLM Wiki](#1-什么是-llm-wiki)
2. [解决的问题](#2-解决的问题)
3. [核心架构](#3-核心架构)
4. [LLM Wiki vs RAG](#4-llm-wiki-vs-rag)
5. [三大核心操作](#5-三大核心操作)
6. [快速上手：30 分钟搭建第一个 Wiki](#6-快速上手30-分钟搭建第一个-wiki)
7. [进阶：Obsidian 插件方案](#7-进阶obsidian-插件方案)
8. [Schema 文件：让 LLM 成为守纪律的图书馆员](#8-schema-文件让-llm-成为守纪律的图书馆员)
9. [批评与局限](#9-批评与局限)
10. [最佳实践](#10-最佳实践)
11. [参考资源](#11-参考资源)

---

## 1. 什么是 LLM Wiki

**LLM Wiki** 是一种个人知识库构建模式。它的核心思路是：

> 把你收集的原始资料（论文、文章、笔记）交给 LLM，由 LLM **增量式地构建和维护一个结构化的 Wiki**——一组相互链接的 Markdown 文件，作为你和原始资料之间的持久化知识层。

每次你添加新资料，LLM 不只是索引它以供后续检索，而是：

- **阅读**新资料，提取关键信息
- **整合**进已有 Wiki：更新概念页面，修订主题总结
- **链接**新概念与已有页面（使用 `[[wiki-links]]`）
- **标记矛盾**：新数据和旧说法冲突时自动标明

关键区别：**Wiki 是一个持久的、复合增长的知识制品。** 交叉引用已经存在，矛盾已经标注，合成结果已经反映了你读过的所有内容。你每次提问时 LLM 不需要从头重新发现知识。

> **你读，LLM 写，你在 Obsidian 中浏览结果。**  
> Obsidian 是 IDE，LLM 是程序员，Wiki 是代码库。

---

## 2. 解决的问题

### 现状：Stateless RAG

大多数人与 LLM + 文档的交互方式是这样的：

1. 上传一堆 PDF 到 ChatGPT / NotebookLM
2. 问一个问题，LLM 从文档中检索相关片段，生成答案
3. 第二天再问一个跟进问题，**一切从头开始**

**没有积累。没有构建。没有复合增长。**

每个会话从零开始。问同一个问题两次，LLM 每次都重新构建答案。问一个需要综合五篇文档的微妙问题，LLM 每次都要找到并拼接相关片段。

### 理想的分解

Karpathy 的观点是：**维护知识库最无聊的部分不是阅读或思考，而是图书管理——更新交叉引用、保持摘要最新、标注新老矛盾、维护数十页的一致性。**

人类放弃 Wiki 是因为维护成本的增速超过了价值增速。而 LLM **不会厌倦**，不会忘记更新交叉引用，可以一次修改 15 个文件。

人类的工作是：策划资料源、引导分析、提出好问题、思考这一切意味着什么。  
LLM 的工作是：其他所有事情。

---

## 3. 核心架构

LLM Wiki 有三层，每层有明确的职责边界：

```
┌─────────────────────────────────────┐
│          schema / AGENTS.md          │ ← 告诉 LLM Wiki 结构和约定
├─────────────────────────────────────┤
│             wiki/                    │ ← LLM 生成和维护的 Markdown 文件
│   ├── index.md       (目录索引)      │
│   ├── log.md         (操作日志)      │
│   ├── entities/      (实体页面)      │
│   ├── concepts/      (概念页面)      │
│   └── summaries/     (摘要页面)      │
├─────────────────────────────────────┤
│             raw/                     │ ← 原始资料（只读，LLM 永不修改）
│   ├── paper1.pdf                    │
│   ├── article2.md                   │
│   └── notes3.md                     │
└─────────────────────────────────────┘
```

### 3.1 Raw Sources（原始资料层）

你策划的源文档集合——文章、论文、图片、数据文件。**这些是不可变的**——LLM 从中读取但从不修改。这是你的事实来源（source of truth）。

### 3.2 The Wiki（知识层）

一组 LLM 生成的 Markdown 文件。总结、实体页面、概念页面、对比、概览、综合报告。**LLM 完全拥有这一层**——创建页面、在新源到达时更新、维护交叉引用、保持一致性。你阅读它；LLM 编写它。

### 3.3 The Schema（规范层）

一个文档（如 `AGENTS.md` 或 `CLAUDE.md`），告诉 LLM Wiki 如何结构化、遵循什么约定、在接入源、回答问题或维护 Wiki 时执行什么工作流。这是关键的配置文件——它使 LLM 成为一个守纪律的 Wiki 维护者，而非通用聊天机器人。你与 LLM 共同进化这个文件。

---

## 4. LLM Wiki vs RAG

| 维度 | RAG | LLM Wiki |
|------|-----|----------|
| **知识持久化** | 无状态，每次重来 | 完全持久化，随时间积累 |
| **多文档综合** | 每次查询实时综合 | 预编译到页面中 |
| **矛盾检测** | ❌ 无 | ✅ 编译时自动标记 |
| **源可追溯性** | 高（逐 chunk） | 中（页面级） |
| **设置复杂度** | 低 | 低-中 |
| **适合场景** | 快速问答、变化频繁的数据 | 深度研究、长期构建专业知识 |
| **知识复合** | 不复合 | 源越多，知识越丰富 |

### 何时选择哪个

- **数据每天变化** / **每个声明都需要精确来源追溯** → RAG
- **在数周或数月内深入构建某领域的专业知识** → LLM Wiki
- **希望 LLM 在知识库上进行推理，而不仅仅是检索** → LLM Wiki

---

## 5. 三大核心操作

### 5.1 Ingest（接入）

把新资料丢进 `raw/`，然后让 LLM 处理它。一个典型的流程：

1. LLM 读取源文件
2. 与你讨论关键 takeaways
3. 在 Wiki 中写入总结页面
4. 更新 `index.md`
5. 更新相关实体和概念页面（一个源可能涉及 10-15 个页面）
6. 追加日志条目到 `log.md`

> Karpathy 建议**逐个接入**并保持参与——你阅读总结、检查更新、引导 LLM 的侧重点。批处理也可以，但监督程度较低。

### 5.2 Query（查询）

你向 Wiki 提问。LLM 搜索相关页面、阅读它们、合成带引用的答案。

**重要洞察：好的答案可以被归档回 Wiki 作为新页面。** 你问出的比较、分析、发现——这些是有价值的，不应消失在聊天历史中。这样你的探索也会像接入的源一样在知识库中复合增长。

### 5.3 Lint（审计）

定期健康检查。LLM 检查：

- 页面间的矛盾
- 已被新源取代的过时声明
- 没有入链的孤立页面
- 被提及但缺少独立页面的重要概念
- 缺失的交叉引用
- 可以通过网络搜索填补的数据空白

> 建议每增加 20 个新页面运行一次 Lint 审计。

---

## 6. 快速上手：30 分钟搭建第一个 Wiki

### 前置条件

- 一台电脑（Mac / Windows / Linux）
- Claude.ai 账号（免费版即可）或 Claude Code
- Obsidian（免费，可选但推荐）

### Step 1：下载 5 篇论文

选择五篇同一主题的论文（主题越集中，Wiki 的图越丰富）：

| 论文 | arXiv 链接 |
|------|-----------|
| Attention Is All You Need (2017) | https://arxiv.org/pdf/1706.03762 |
| BERT (2018) | https://arxiv.org/pdf/1810.04805 |
| GPT-3 (2020) | https://arxiv.org/pdf/2005.14165 |
| Foundation Models (2021) | https://arxiv.org/pdf/2108.07258 |
| RLHF (2022) | https://arxiv.org/pdf/2203.02155 |

### Step 2：创建文件夹结构

```
my-wiki/
├── raw/          ← 放原始资料（PDF / 文章 / 笔记）
└── wiki/         ← LLM 写的实体页面（你只读）
```

把五篇 PDF 放进 `raw/`。

### Step 3：运行接入 Prompt

**方案 A：Claude.ai（无需终端）**

打开 Claude.ai，上传所有五篇 PDF，发送以下 prompt：

> 你正在构建一个 LLM Wiki。请执行以下操作：
>
> 1. 阅读 `raw/` 文件夹中的所有源文件
> 2. 为每个关键概念创建实体页面（每页一个概念），使用 markdown 格式
> 3. 每个页面包含：概念名称、简要总结、详细解释、与其他概念的 `[[wiki-links]]`
> 4. 创建 `index.md` 列出所有页面
> 5. 创建 `log.md` 记录本次接入
> 6. 如果发现概念之间有矛盾，在相关页面中标记
>
> 将每个页面保存为 `wiki/` 文件夹中的 `.md` 文件。

Claude 会生成 10-20 个 markdown 实体页面。把每个页面复制到 `wiki/` 中对应的 `.md` 文件。

**方案 B：Claude Code（终端）**

```bash
cd my-wiki
claude
```

粘贴同样的 prompt。Claude Code 会直接读取文件并将页面写入 `wiki/`——无需复制粘贴。

### Step 4：在 Obsidian 中打开

1. 安装 [Obsidian](https://obsidian.md/)
2. 点击 **Open folder as vault**，选择 `my-wiki/`
3. 按 `Ctrl+G`（Mac：`Cmd+G`）打开 Graph View

你会看到实体页面作为节点，`[[wiki-links]]` 作为边。5 篇论文应该已经产生一个小但有意义的图——transformer 链接到 attention，BERT 链接到 fine-tuning，RLHF 链接到 alignment 和 GPT。

### Step 5：复合增长

再丢一篇论文进 `raw/`，然后运行：

> 请将 `raw/` 中的新源接入现有 Wiki。更新相关实体页面，添加新页面，更新 `index.md` 和 `log.md`。标记发现的任何矛盾。

这次的效果会不同：新论文不仅创建新页面，还**丰富已有页面**。"Attention" 页面原来有 2 条出链，现在可能是 5 条。一个没有被质疑过的说法现在可能被标记了矛盾。

### Step 6：运行 Lint

每新增约 20 个页面后：

> 请对 Wiki 进行全面健康检查：
> - 检查页面之间的矛盾
> - 查找过时的声明
> - 标记孤立页面（没有入链）
> - 检查被遗漏的重要概念
> - 修复断裂的链接

---

## 7. 进阶：Obsidian 插件方案

社区已经有了成熟的 Obsidian 插件实现：[green-dalii/obsidian-llm-wiki](https://github.com/green-dalii/obsidian-llm-wiki)（Obsidian 官方评分 95/100）。

### 功能特性

| 功能 | 说明 |
|------|------|
| 单文件接入 | 对当前文件提取实体和概念 |
| 批量接入 | 从文件夹批量处理、增量更新 |
| 智能跳过 | 已处理过的文件自动跳过 |
| 对话查询 | 流式回答 + `[[wiki-links]]` 溯源 |
| Wiki Lint | 检测重复、死链、空页面、孤立页面 |
| 索引重建 | 重新生成 `index.md` |
| Schema 建议 | LLM 分析 Wiki 并提出结构改进 |
| 并行生成 | 支持并发页面生成（建议设 3） |
| 多语言 | 内置 8 种语言 |

### 支持的 LLM Provider

Anthropic、OpenAI、Google Gemini、DeepSeek、Kimi、GLM、MiniMax、LM Studio（本地）、Ollama（本地）、OpenRouter 等 12+ 种。

### 安装

**推荐：Obsidian 社区插件市场**

1. 设置 → 社区插件 → 浏览
2. 搜索 "Karpathy LLM Wiki"
3. 安装并启用

**手动安装**

从 [Releases](https://github.com/green-dalii/obsidian-llm-wiki/releases) 下载 `main.js`、`manifest.json`、`styles.css`，放入 `.obsidian/plugins/karpathywiki/`。

---

## 8. Schema 文件：让 LLM 成为守纪律的图书馆员

Schema 是整个系统中被严重低估但最关键的部分。一个好的 schema 文件（`AGENTS.md` / `CLAUDE.md`）应该包括：

### 目录结构约定

```
wiki/
├── index.md             # 目录索引，按类别组织
├── log.md               # 时间顺序的操作日志
├── entities/            # 实体页面（人物、组织、产品）
├── concepts/            # 概念页面（理论、方法、术语）
└── summaries/           # 原始资料的总结
```

### 页面格式约定

```markdown
---
tags: [transformer, attention, 2017]
sources: [paper1.pdf]
created: 2026-04-15
updated: 2026-04-20
---

# Transformer

## Summary
简短总结（2-3 句）

## Explanation
详细说明...

## Related
- [[Attention Mechanism]]
- [[Self-Attention]]
- [[Multi-Head Attention]]

## Contradictions
- [[Attention Is All You Need]] 使用 fixed attention；后续 [[Reformer]] 提出可学习的稀疏注意力模式
```

### 工作流约定

在 schema 中明确定义 LLM 必须遵循的流程：

```
## Workflows

### Ingest
1. 读取 raw/ 中的新源
2. 与我讨论关键发现
3. 在 wiki/summaries/ 写入总结
4. 更新或创建相关实体/概念页面
5. 更新 index.md
6. 追加 log.md
7. 搜索并标记矛盾

### Query
1. 阅读 index.md 找出相关页面
2. 深入阅读这些页面
3. 综合答案，标注 `[[wiki-links]]`
4. 如果答案有持久价值，问是否要写为新页面

### Lint
1. 扫描所有页面，检查矛盾
2. 查找过时声明
3. 标记孤立页面
4. 建议缺失的重要概念
5. 修复断链
```

---

## 9. 批评与局限

> 对于任何寻求在生产环境中直接采用 Karpathy 的 LLM Wiki 模式的人，有必要强调一个关键区别：你的 LLM 与你的开发环境绑定在一起。Karpathy 的 gist 是一个概念验证，而非生产标准。然而，标准可以在可靠的设计模式中通过合适的工具优雅地制定和坚持。

——来自社区讨论

### Chris Lettieri 的批评

Chris Lettieri 在 [An LLM Wiki Won't Compound Your Knowledge](https://bitsofchris.com/p/an-llm-wiki-wont-compound-your-knowledge) 中提出了一个重要观点：

**问题：谁在学习？**

- 在 LLM Wiki 模式中，**LLM** 读论文、写总结、维护交叉引用、回答问题
- **你的唯一贡献是选择放什么东西进去**
- 让你变聪明的连接，是你大脑自己建立的那些连接
- LLM 编译的 Wiki 给你一种知识积累的错觉，而实际上你的大脑并没有变得更好

**他的替代方案：**

1. **记录你自己的思考**——即使是混乱的、深夜的半成品想法。多年后这成为你最宝贵的数据集
2. **不要过度整理**——好的检索可以处理混乱
3. **提供一个导航层**——将笔记转化为块、链接和上下文摘要，让 agent 可以导航你的思维

> "AI 的工作不是替你思考，而是帮你看到你一直在思考什么。"

### 实际局限性

| 局限 | 说明 |
|------|------|
| 索引膨胀 | 超过数百页面后，`index.md` 会超出 LLM 上下文窗口 |
| 无状态幻觉 | LLM 每次会话都会"重新发现"Wiki 结构，除非有严格的 schema 约束 |
| 知识复合上限 | 纯文本的 Wiki 缺乏可编程的推理能力 |
| 源可追溯性 | 低于 RAG 的 chunk 级追溯 |
| 人工参与度 | 容易陷入"让 LLM 读一切"的被动模式 |

---

## 10. 最佳实践

### 选对场景

LLM Wiki 最适合：**数周至数月内深入钻研一个领域**。如果你在研究 transformer、RLHF、或者学习一个新语言——这正是它的强项。如果数据每天变化——用 RAG。

### 控制主题范围

Wiki 在主题相关的资料上复合效果最好。五篇同一子领域的论文产生的图比五篇无关文章丰富得多。

### 每个页面一个概念

如果一个页面开始覆盖两个想法，就拆分它。密度适中的单概念页面创建更好的链接和更好的答案。

### 保持 YAML frontmatter 完整

tags、dates、source 计数等元数据让 LLM 和 Dataview 插件都能高效查询。

### 定期 Lint

小错误在 Wiki 中传播很快。一个页面上有错误的主张，被另外三页链接引用，你就有了组织化的错误信息。

### 你的笔记 + LLM Wiki 结合

最好的方法可能是混合的：

- 用 LLM Wiki 处理**外部资料**（论文、文章、报告）
- 与此同时**用自己的话记录你对这些资料的思考**
- 六年后，LLM 的总结一文不值（你可以随时重新生成），但你在特定时刻、特定上下文中的思考——那是不可替代的

### 把答案写回 Wiki

好的答案不应该消失在聊天历史中。当 LLM 给出一个特别有洞察力的比较或分析时，把它作为新页面保存到 Wiki。你的探索也像接入的源一样复合增长。

### 用 git 做版本控制

Wiki 只是一堆 markdown 文件，天然适合 git。你免费获得版本历史、分支和协作能力。

---

## 11. 参考资源

### 原始资料

| 资源 | 链接 |
|------|------|
| Karpathy 原始 Gist（必读） | https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f |
| 讨论 / 评论 | https://news.ycombinator.com/item?id=42591457 |

### 教程

| 资源 | 链接 |
|------|------|
| Data Science Dojo 教程 | https://datasciencedojo.com/blog/llm-wiki-tutorial/ |
| MindStudio 指南（英文） | https://www.mindstudio.ai/blog/andrej-karpathy-llm-wiki-knowledge-base-claude-code/ |
| YouTube：Karpathy's LLM Wiki - Full Beginner Setup Guide | https://www.youtube.com/watch?v=WbPJ6Yo6s_A |
| YouTube：Build your own AI Knowledge Base | https://www.youtube.com/watch?v=3R5g5rPmN5g |

### 工具

| 工具 | 用途 | 链接 |
|------|------|------|
| Obsidian | Markdown 编辑器 + Graph View | https://obsidian.md |
| Obsidian Web Clipper | 网页转 Markdown | Obsidian 内置插件 |
| green-dalii/obsidian-llm-wiki | Obsidian LLM Wiki 插件（95/100 评分） | https://github.com/green-dalii/obsidian-llm-wiki |
| qmd | 本地 Markdown 搜索引擎（BM25/向量混合） | https://github.com/tobi/qmd |
| Marp | Markdown 转幻灯片 | Obsidian 插件 |
| Dataview | 对 YAML frontmatter 做查询 | Obsidian 插件 |

### 批评与深度阅读

| 资源 | 链接 |
|------|------|
| An LLM Wiki Won't Compound Your Knowledge | https://bitsofchris.com/p/an-llm-wiki-wont-compound-your-knowledge |
| Context Engineering is Index Design for Agents | https://bitsofchris.com/p/context-engineering-is-index-design |
| Zettelkasten 方法 | https://bitsofchris.com/p/bits-of-books-how-to-take-smart-notes |

### 学术渊源

- **Vannevar Bush, "As We May Think" (1945)** — Memex 概念，个人知识的关联存储。Bush 的愿景比 Web 更接近这里描述的模式：私有的、主动策划的、文档之间的连接与文档本身同样有价值。他没解决的正是"谁来维护"——现在 LLM 可以负责。

---

> **最后的话：** LLM Wiki 是一个模式，不是一个产品。它的核心洞察——把知识编译成持久化的、相互链接的结构，而不是在每次查询时重新发现——对于深度研究型工作流来说是一个真正的进步。但不要让它替你思考。用它来处理书籍保管工作，把真正的思考留给自己。
