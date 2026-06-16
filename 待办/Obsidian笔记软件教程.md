# Obsidian 笔记软件完全教程

> Obsidian 是一款基于**本地 Markdown 文件**的现代化知识管理工具，对 AI Agent 极其友好。  
> 官网: [obsidian.md](https://obsidian.md) · 最新版本: v1.13.x (2026) · 个人使用免费

---

## 目录

1. [什么是 Obsidian](#1-什么是-obsidian)
2. [为什么对 Agent 友好](#2-为什么对-agent-友好)
3. [安装](#3-安装)
4. [快速上手](#4-快速上手)
5. [Markdown 语法指南](#5-markdown-语法指南)
6. [核心插件](#6-核心插件)
7. [必装社区插件](#7-必装社区插件)
8. [Agent 集成配置](#8-agent-集成配置)
9. [知识管理工作流](#9-知识管理工作流)
10. [从思源迁移](#10-从思源迁移)
11. [快捷键速查](#11-快捷键速查)
12. [最佳实践](#12-最佳实践)

---

## 1. 什么是 Obsidian

Obsidian 是一个**以本地 Markdown 文件为基础**的知识管理工具。它的核心理念：

- **数据完全归你**：所有笔记以 `.md` 纯文本文件存储在本地硬盘，零锁定
- **双向链接**：通过 `[[笔记名]]` 建立笔记间的关联，形成知识网络
- **图谱视图**：将笔记间的链接关系可视化为交互式图谱
- **无限可扩展**：4300+ 社区插件覆盖几乎所有知识管理场景

对比同类工具：

| 特性 | Obsidian | Notion | 思源 | Logseq |
|------|----------|--------|------|--------|
| 数据存储 | 本地 `.md` 文件 | 云端私有格式 | 本地 `.sy` 私有格式 | `.md` / SQLite |
| 离线可用 | ✅ 完全离线 | ❌ 需网络 | ✅ | ✅ |
| Agent 友好 | ★★★★★ | ★★★☆☆ | ★☆☆☆☆ | ★★★★☆ |
| MCP 支持 | ✅ 多个成熟方案 | ❌ | ❌ | ✅ 有限 |
| 开源 | ❌（数据格式开放） | ❌ | ✅ | ✅ |
| 插件生态 | 4300+ | 有限集成 | 有限 | 200+ |
| 双向链接 | ✅ | ✅ 数据库关联 | ✅ | ✅ |
| 移动端 | ✅ iOS/Android | ✅ | ✅ | ✅ |

---

## 2. 为什么对 Agent 友好

Obsidian 是**目前对 AI Agent（Claude Code、Cursor、Codex 等）最友好的笔记软件**，原因有三层：

### 2.1 数据层：纯 Markdown 文件

```
我的知识库/
├── 编程/
│   ├── Python 异步编程.md
│   ├── Rust 所有权机制.md
│   └── 设计模式笔记.md
├── 项目/
│   ├── 2026-Q2 项目计划.md
│   └── 架构评审记录.md
├── 日记/
│   ├── 2026-06-15.md
│   └── 2026-06-14.md
└── 索引-MOC.md
```

Agent 可以用任何文件工具直接读写，无需 API、无需认证、无频率限制。

### 2.2 工具层：Obsidian CLI（v1.12+）

Obsidian 1.12 起内置 CLI，提供 100+ 命令：

```bash
# 搜索（比 grep 快 70,000 倍 token 效率）
obsidian search "异步编程"

# 查找反向链接
obsidian backlinks "Python 异步编程.md"

# 检测孤立笔记
obsidian orphans

# 图谱遍历
obsidian graph "设计模式笔记.md" --depth 2

# 批量重命名（自动更新所有引用）
obsidian rename "旧标题" "新标题"
```

### 2.3 协议层：MCP Server

社区提供了多种 MCP Server，让 Agent 通过标准协议访问知识库：

| MCP Server | 工作方式 | 需要 Obsidian 运行 | 特点 |
|------------|----------|-------------------|------|
| `lstpsche/obsidian-mcp` | 直接读文件系统 | ❌ 不需要 | Rust 实现，最快 |
| `LWaetzig/obsidian-mcp` | 直接读文件系统 | ❌ 不需要 | 纯 Node，简洁 |
| `yanxue06/obsidian-mcp` | Local REST API | ✅ 需要 | 图谱感知，反向链接 |
| `fufic123/obsidian-mcp` | 文件系统 + 索引 | ❌ 不需要 | 多 vault，命名空间 |
| `Leonezz/obsidian-mcp-server` | Obsidian 内置插件 | ✅ 需要 | 直接嵌入 Obsidian |

---

## 3. 安装

### 3.1 Windows

```powershell
# 方式一：官网下载安装包（推荐）
# https://obsidian.md/download

# 方式二：Winget
winget install Obsidian.Obsidian

# 方式三：Scoop
scoop bucket add extras
scoop install obsidian
```

### 3.2 macOS

```bash
# Homebrew
brew install --cask obsidian
```

### 3.3 Linux

```bash
# AppImage（推荐）
# 从 obsidian.md/download 下载 .AppImage

# Flatpak
flatpak install flathub md.obsidian.Obsidian

# Snap
sudo snap install obsidian --classic
```

### 3.4 移动端

- iOS：App Store 搜索 "Obsidian"
- Android：Google Play 或 [GitHub Releases](https://github.com/obsidianmd/obsidian-releases/releases) 下载 APK

---

## 4. 快速上手

### 4.1 创建 Vault

Vault（知识库）就是一个本地文件夹：

1. 打开 Obsidian，点击「Create new vault」
2. 输入名称（如 `我的知识库`），选择存储位置
3. 点击「Create」——完成

> 也可以直接打开已有文件夹作为 Vault：点击「Open folder as vault」

### 4.2 写第一篇笔记

```
Ctrl+N  →  新建笔记
输入内容  →  Markdown 实时预览
Ctrl+S  →  保存
```

Obsidian 支持**实时预览**（Live Preview）——输入 Markdown 语法后即时渲染，所见即所得。

### 4.3 创建链接——Obsidian 的灵魂

在笔记中输入 `[[` 即可触发链接自动补全：

```markdown
# Python 学习笔记

[[异步编程]] 是 Python 并发的重要部分。
可以参考 [[Rust 所有权机制]] 对比理解内存管理。

## 相关概念
- [[协程]]
- [[事件循环]]
- [[GIL 全局解释器锁]]
```

### 4.4 查看反向链接

打开任意笔记，右侧面板显示**哪些笔记引用了当前笔记**——这是 Obsidian 最强大的发现机制：

```
# 在 "异步编程" 笔记中

反向链接：
← Python 学习笔记    (第 3 行提及)
← 面试准备            (第 12 行提及)
← 并发模型对比         (第 7 行提及)
```

### 4.5 图谱视图

```
Ctrl+G  →  打开图谱
```

图谱将笔记显示为节点，链接显示为连线。可以缩放、拖拽、点击节点直达笔记。随着笔记增多，图谱会成为你的**知识导航地图**。

### 4.6 命令面板

```
Ctrl+P  →  命令面板（类似 VS Code）
```

所有操作都可以通过命令面板完成——切换主题、打开插件、格式化、导出等。

---

## 5. Markdown 语法指南

### 5.1 基础语法

```markdown
# 一级标题
## 二级标题
### 三级标题

**粗体**  *斜体*  ~~删除线~~  ==高亮==

- 无序列表
- 无序列表

1. 有序列表
2. 有序列表

> 引用块

`行内代码`

​```python
# 代码块
def hello():
    print("Hello, Obsidian!")
​```
```

### 5.2 内部链接

```markdown
# 基本链接
[[笔记名]]

# 别名链接（显示文本与文件名不同）
[[笔记名|显示的文本]]

# 标题锚点
[[笔记名#章节标题]]

# 块引用（链接到笔记中的某一段落）
[[笔记名#^block-id]]
```

### 5.3 嵌入

在链接前加 `!` 即可**嵌入**内容：

```markdown
# 嵌入整篇笔记
![[其他笔记]]

# 嵌入图片
![[图片名.png]]

# 嵌入音频
![[录音.mp3]]

# 嵌入 PDF 的某一页
![[文档.pdf#page=3]]
```

### 5.4 Callout（标注块）

```markdown
> [!note] 标题
> 这是一个普通标注。

> [!warning] 注意
> 这是一个警告。

> [!tip] 技巧
> 这是一个提示。

> [!danger] 危险
> 请勿在生产环境执行此操作。

> [!question] 问题
> 为什么会这样？

> [!example] 示例
> 这是一个示例。
```

支持 12 种类型：`note`、`abstract`、`info`、`todo`、`tip`、`success`、`question`、`warning`、`failure`、`danger`、`bug`、`example`。

### 5.5 Frontmatter（前置元数据）

在笔记开头用 YAML 定义结构化属性：

```markdown
---
title: Python 异步编程
tags:
  - python
  - 异步
  - 并发
created: 2026-06-15
status: 进行中
rating: 4
---

# Python 异步编程

正文内容...
```

Frontmatter 是 Dataview 等查询插件的**数据基础**。

### 5.6 标签

```markdown
# 标签语法
#python
#编程/异步

# 嵌套标签（子标签）
#项目/年度/2026-Q2
```

标签面板（右侧）可浏览所有标签及其使用频率。

### 5.7 Mermaid 图表

```markdown
​```mermaid
graph TD
    A[开始] --> B{判断}
    B -->|是| C[执行A]
    B -->|否| D[执行B]
    C --> E[结束]
    D --> E
​```
```

支持流程图、时序图、甘特图、类图等全部 Mermaid 图表类型。

---

## 6. 核心插件

Obsidian 内置 30+ 核心插件，按需开启（设置 → 核心插件）。

### 6.1 日记 (Daily Notes)

每天自动创建一篇日记：

```
设置 → 核心插件 → 日记 → 开启
  - 日期格式：YYYY-MM-DD
  - 模板文件：templates/日记模板
  - 存储位置：日记/
```

点击日历图标或 `Ctrl+Shift+D` 即可打开今日日记。

### 6.2 模板 (Templates)

插入预定义的笔记骨架：

```markdown
---
title: "{{title}}"
tags: []
created: {{date:YYYY-MM-DD}}
---

# {{title}}

## 概述

## 笔记

## 参考资料
- 
```

`设置 → 核心插件 → 模板 → 指定模板文件夹`，新建笔记时 `Ctrl+Shift+T` 选择模板。

### 6.3 Bases（数据库视图）— v1.9 新功能

Bases 将笔记集合变成**可查询、可排列的数据库**：

- 基于 YAML Frontmatter 字段自动建表
- 支持表格、看板、日历、画廊四种视图
- 拖拽排序、筛选、分组
- 公式列计算动态值（如 `进度 % = 已完成 / 总任务数`）

> 比 Notion 数据库更灵活——数据仍然是纯 Markdown，只是多了视图层。

### 6.4 Canvas（画布）

无限画布，用于可视化组织：

- 拖入笔记、图片、PDF 形成卡片
- 连线表示关系
- 适合项目规划、头脑风暴、架构设计

```
设置 → 核心插件 → Canvas → 开启
Ctrl+Shift+C → 新建画布
```

### 6.5 大纲 (Outline)

左侧面板显示当前笔记的标题结构，点击即可跳转。长笔记导航必备。

### 6.6 书签 (Bookmarks)

收藏常用笔记、搜索、文件夹，快速访问。

---

## 7. 必装社区插件

```
设置 → 社区插件 → 浏览 → 搜索安装
```

### 7.1 Dataview（数据查询引擎）

**将你的知识库变成可查询数据库**：

```markdown
# 所有未完成的任务
​```dataview
TASK
FROM "项目"
WHERE !completed
GROUP BY file.link
​```

# 最近修改的 10 篇笔记
​```dataview
TABLE file.mtime AS "修改时间"
FROM ""
SORT file.mtime DESC
LIMIT 10
​```

# 包含特定标签的笔记列表
​```dataview
LIST
FROM #python
SORT file.ctime ASC
​```

# 按字段聚合（如全部书籍按评分排列）
​```dataview
TABLE author AS "作者", rating AS "评分", status AS "状态"
FROM "阅读"
SORT rating DESC
​```
```

> Dataview 是 Obsidian 最强大的插件，没有之一。

### 7.2 Templater（高级模板引擎）

比核心模板更强大——支持 JavaScript 脚本、日期计算、文件操作：

```markdown
# 日记模板示例
---
title: "<% tp.date.now("YYYY-MM-DD") %>"
tags: [日记]
---

# <% tp.date.now("YYYY年MM月DD日 dddd") %>

## 今日待办
- [ ] 

## 笔记

## 复盘
- 做得好：
- 可改进：

---
<< [[<% tp.date.yesterday("YYYY-MM-DD") %>]] | [[<% tp.date.tomorrow("YYYY-MM-DD") %>]] >>
```

结合 QuickAdd 插件可实现**一键创建结构化笔记**。

### 7.3 Tasks（任务管理）

将 `- [ ]` 任务语法升级为完整任务系统：

```markdown
# 在所有笔记中搜索到期任务
​```tasks
not done
due before tomorrow
sort by due
​```

# 按标签过滤
​```tasks
not done
tags include #重要
​```

# 创建任务时使用完整语法
- [ ] 完成接口文档 📅 2026-06-20 ⏫ #重要 #项目A
```

支持：截止日期、优先级、标签、重复任务、依赖关系。

### 7.4 Git（自动版本控制）

```markdown
设置 → Obsidian Git → 配置：
  - 自动提交间隔：30 分钟
  - 自动推送：开启
  - 启动时拉取：开启
```

每次修改自动备份到 Git，可随时回滚到任意历史版本。**这是 Agent 操作时的安全网**——Agent 改错了，`git checkout` 即可恢复。

### 7.5 其他推荐插件

| 插件 | 用途 | 必装程度 |
|------|------|---------|
| **Calendar** | 日历视图，直观导航日记 | ★★★★★ |
| **Kanban** | 看板视图，项目管理 | ★★★★☆ |
| **Excalidraw** | 手绘图表、架构图 | ★★★★☆ |
| **QuickAdd** | 快速创建笔记、执行脚本 | ★★★★☆ |
| **Style Settings** | 精细化调整主题外观 | ★★★☆☆ |
| **Zotero Integration** | 学术文献管理 | ★★★☆☆ |
| **Paste URL into selection** | 粘贴 URL 自动转为链接 | ★★★☆☆ |
| **Note Refactor** | 将长笔记拆分为多个子笔记 | ★★★☆☆ |
| **Tag Wrangler** | 批量重命名/管理标签 | ★★★☆☆ |

> 原则：**从少开始**。只用你真正需要的插件，避免把 Obsidian 变成万花筒。

---

## 8. Agent 集成配置

### 8.1 方案 A：文件系统直读（最简，推荐入门）

Agent 直接读写 `.md` 文件，无需任何配置：

```json
// Cursor: .cursor/mcp.json 或全局 ~/.cursor/mcp.json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-filesystem", "/path/to/vault"]
    }
  }
}
```

优点：零配置、零依赖、Obsidian 无需运行。  
缺点：无法感知反向链接和图谱关系。

### 8.2 方案 B：Obsidian CLI + MCP（平衡推荐）

Obsidian CLI 提供结构化访问（需要 Obsidian 运行）：

```bash
# 安装 Obsidian CLI（1.12+ 内置，只需确认可用）
obsidian --version

# Claude Code：注册为 MCP 工具
claude mcp add obsidian-cli \
  --command obsidian \
  --args search,backlinks,graph,list
```

### 8.3 方案 C：专用 MCP Server（功能最强）

#### Claude Code 配置

```bash
# 方案 1：yanxue06/obsidian-mcp（需要 Obsidian 运行 + Local REST API 插件）
# 1. 先在 Obsidian 装 Local REST API 插件，复制 API Key
# 2. 注册 MCP
claude mcp add obsidian \
  npx -y @yanxue06/obsidian-mcp \
  -e OBSIDIAN_API_KEY=你的API密钥

# 方案 2：LWaetzig/obsidian-mcp（不需要 Obsidian 运行）
git clone https://github.com/LWaetzig/obsidian-mcp
cd obsidian-mcp && npm install && npm run build
claude mcp add obsidian \
  node /path/to/obsidian-mcp/dist/index.js \
  -e OBSIDIAN_VAULT_PATH=/path/to/vault

# 方案 3：fufic123/obsidian-mcp（多 vault 支持）
uvx obsidian-mcp  # 自动读取 ~/.config/obsidian-mcp/config.toml
```

#### Cursor 配置

编辑 `.cursor/mcp.json`：

```json
{
  "mcpServers": {
    "obsidian": {
      "command": "npx",
      "args": ["-y", "@yanxue06/obsidian-mcp"],
      "env": {
        "OBSIDIAN_API_KEY": "你的API密钥"
      }
    }
  }
}
```

#### Claude Desktop 配置

```json
// Windows: %APPDATA%\Claude\claude_desktop_config.json
// macOS: ~/Library/Application Support/Claude/claude_desktop_config.json
{
  "mcpServers": {
    "obsidian": {
      "command": "npx",
      "args": ["-y", "@yanxue06/obsidian-mcp"],
      "env": {
        "OBSIDIAN_API_KEY": "你的API密钥"
      }
    }
  }
}
```

### 8.4 安全最佳实践

```
1. 从只读开始        → 先验证搜索和读取正常，再开写入权限
2. Git 自动备份       → Obsidian Git 插件 30 分钟自动提交
3. 分离 Sensitive 内容 → 给 Agent 单独的子目录，个人笔记不可访问
4. 避免在笔记里存密钥  → API Key 放系统环境变量，不放 frontmatter
5. 定期审查 Agent 改动  → git diff 检查 Agent 的修改
```

> 推荐架构：`个人知识库/Agent工作区/` 子目录放 Agent 可读写的笔记，其他目录只读。

---

## 9. 知识管理工作流

### 9.1 Zettelkasten（卡片盒笔记法）

核心理念：每篇笔记只记**一个概念**（原子化），通过链接建立关联。

```
1. 闪念笔记 → 快速捕获想法（收件箱文件夹）
2. 文献笔记 → 用自己的话重述（来源 + 摘录）
3. 永久笔记 → 一个想法一篇，互相链接
4. 索引笔记 → MOC (Map of Content) 导航入口
```

实践：

```markdown
# 闪念笔记
今天看到一个关于 Python asyncio 的有趣用法...

# 文献笔记
## [[Python 官方文档 - asyncio]]
> 原文摘要...
我的理解：asyncio 本质是一个单线程事件循环...

# 永久笔记
# [[协程的本质是协作式调度]]
协程不是并发，是协作——主动让出控制权给调度器...

# 索引笔记（MOC）
# [[Python 并发编程 MOC]]
- [[协程的本质是协作式调度]]
- [[GIL 与多线程的真相]]
- [[async/await 语法糖]]
```

### 9.2 PARA 方法

适合项目驱动型知识管理：

| 层级 | 文件夹 | 用途 |
|------|--------|------|
| **P**rojects | 1-Projects/ | 当前进行中的项目，有明确的截止时间 |
| **A**reas | 2-Areas/ | 持续负责的领域，无截止时间 |
| **R**esources | 3-Resources/ | 感兴趣的主题、参考资料 |
| **A**rchives | 4-Archives/ | 已完成或不再活跃的项目 |

```
我的知识库/
├── 1-Projects/
│   ├── 2026-Q2 系统重构/
│   └── 开源项目贡献/
├── 2-Areas/
│   ├── 技术写作/
│   └── 职业发展/
├── 3-Resources/
│   ├── Rust 学习/
│   └── 系统设计/
└── 4-Archives/
    └── 2025 年度项目/
```

### 9.3 日记驱动工作流

以**每日日记**为入口，逐步沉淀知识：

```
1. 每天 Ctrl+Shift+D 创建日记
2. 日记中写下想法、任务、记录
3. 周末回顾：有价值的内容拆成独立笔记 → 链接
4. 月度整理：归类到 PARA 文件夹，更新 MOC
```

### 9.4 MOC（内容地图）

手动维护的导航页——知识库的首页：

```markdown
# [[编程 MOC]]

## Python
- [[Python 异步编程]]
- [[Python 类型系统]]

## Rust
- [[Rust 所有权机制]]
- [[Rust 异步 vs Python 异步]]

## 系统设计
- [[分布式系统基础]]
- [[微服务 vs 单体]]
```

---

## 10. 从思源迁移

### 10.1 导出思源笔记

```
思源 → 设置 → 导出 → 
  格式：Markdown
  选项：导出子文档、包含资源文件
  → 导出到文件夹
```

### 10.2 导入 Obsidian

```
方式一：直接打开
  1. Obsidian → Open folder as vault
  2. 选择思源导出的文件夹
  
方式二：新建 Vault 后复制
  1. 创建新的 Obsidian Vault
  2. 将思源导出的 .md 文件复制到 Vault 文件夹
```

### 10.3 迁移注意事项

| 思源特性 | Obsidian 对应 | 说明 |
|----------|--------------|------|
| 块引用 `((id))` | `[[笔记^block-id]]` | 需手动转换 |
| 数据库 | Bases 核心插件 / Dataview | 学习成本低 |
| `{{{col 内容}}}` 分栏 | 不原生支持 | 可用 CSS 代码段实现 |
| 闪卡 | Spaced Repetition 插件 | 功能完整 |
| 属性面板 | Properties 核心插件 (Frontmatter) | 更灵活 |
| 集市/插件 | 社区插件 4300+ | 生态大得多 |

> 迁移后建议保留思源备份，花 1-2 周适应 Obsidian 的文档式组织方式。

---

## 11. 快捷键速查

### 11.1 笔记操作

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+N` | 新建笔记 |
| `Ctrl+O` | 快速打开（搜索文件名） |
| `Ctrl+S` | 保存 |
| `Ctrl+W` | 关闭当前标签 |
| `Ctrl+Shift+T` | 插入模板 |

### 11.2 视图切换

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+E` | 切换编辑/阅读视图 |
| `Ctrl+P` | 命令面板 |
| `Ctrl+G` | 图谱视图 |
| `Ctrl+Shift+D` | 今日日记 |
| `Ctrl+Shift+C` | 新建 Canvas |

### 11.3 编辑

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+B` | 粗体 |
| `Ctrl+I` | 斜体 |
| `Ctrl+K` | 插入链接 |
| `Ctrl+Shift+F` | 全局搜索 |
| `Ctrl+,` | 打开设置 |

### 11.4 面板

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+Shift+←/→` | 切换左右侧边栏 |
| `Ctrl+Shift+1/2/3...` | 切换标签页 |

> 所有快捷键可在 `设置 → 快捷键` 中自定义。

---

## 12. 最佳实践

### 12.1 文件夹要浅

```
❌ 过深
编程/Python/异步/协程/高级用法/2026笔记.md

✅ 推荐
编程/Python 异步编程.md
```

用**链接**代替深层文件夹。文件夹只做粗粒度分类（4-6 个顶层文件夹即可）。

### 12.2 链接优于标签

标签是平面的，链接是网状的。优先用 `[[链接]]` 建立关系，标签仅用于跨文件夹的分类（如 `#待整理`、`#重要`）。

### 12.3 原子化笔记

一篇笔记聚焦一个概念。拆分原则：**当你需要单独引用某个段落时，它就应该成为独立笔记**。

### 12.4 定期整理

```
每天：日记记录
每周：回顾本周日记，提炼出永久笔记
每月：整理文件夹，更新 MOC
每季度：归档已完成项目，清理无用笔记
```

### 12.5 同步方案

| 方案 | 适用场景 | 成本 |
|------|---------|------|
| Obsidian Sync | 官方方案，端到端加密 | $5/月 |
| iCloud | macOS + iOS 用户 | 免费 |
| Syncthing | 跨平台 P2P 同步 | 免费 |
| Git + Working Copy | 开发者，版本控制 | 免费 |
| OneDrive / Dropbox | Windows 用户 | 免费 |

### 12.6 不要把 Obsidian 变成"万能工具"

Obsidian 的灵活性容易导致"插件无限"的困境。记住：

> Obsidian 是一个**知识管理工具**，不是项目管理软件，不是数据库，不是一切。  
> 用 Obsidian 管理**思想**，用专用工具管理**事务**。

---

> 写于 2026-06-15 · Obsidian v1.13.x · 以官方文档和社区实践为准  
> GitHub: [obsidianmd/obsidian-releases](https://github.com/obsidianmd/obsidian-releases) · 社区插件: [obsidian.md/plugins](https://obsidian.md/plugins)
