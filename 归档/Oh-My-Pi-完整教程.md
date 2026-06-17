# Oh My Pi 从入门到精通 — 终端 AI 编程代理完全指南

> **版本**: 2026.06 | **仓库**: [can1357/oh-my-pi](https://github.com/can1357/oh-my-pi)  
> Oh My Pi（omp）是一个**终端优先的 AI 编程代理框架**，在 Pi（Mario Zechner 原作）之上构建，提供 40+ 模型供应商、32 个内置工具、LSP/DAP 深度集成，以及跨平台的 Rust 原生核心。

---

## 目录
1. [什么是 Oh My Pi](#1-什么是-oh-my-pi)
2. [安装](#2-安装)
3. [快速入门](#3-快速入门)
4. [核心概念](#4-核心概念)
5. [工具介绍](#5-工具介绍)
6. [上下文文件（AGENTS.md / RULES.md）](#6-上下文文件agents.md---rules.md)
7. [模型与供应商配置](#7-模型与供应商配置)
8. [Slash 命令](#8-slash-命令)
9. [TUI 交互界面](#9-tui-交互界面)
10. [会话管理](#10-会话管理)
11. [最佳实践](#11-最佳实践)
12. [Subagent 与任务并行](#12-subagent-与任务并行)
13. [进阶功能](#13-进阶功能)
14. [进阶主题](#14-进阶主题)
15. [疑难解答](#15-疑难解答)
---

## 1. 什么是 Oh My Pi

Oh My Pi 是一个终端 AI 编程代理——和 Claude Code、Codex 同类，但方向不同：

- **不锁供应商**：40+ 模型随便换，本地 Ollama 也行，不会被一家 API 涨价绑架
- **真 IDE 集成**：LSP 重构自动传播到 re-export 和 barrel 文件；DAP 调试器支持断点、单步、变量查看
- **hash 锚点编辑**：文件被外部改过就拒绝补丁，不会悄悄损坏代码

> 全文「代理」= agent，指 AI 编程助手本身。
---

## 2. 安装

### 前置条件

- **Bun** ≥ 1.3.14（推荐运行时）
- 或 Node.js ≥ 18
- macOS / Linux / Windows（原生支持，无需 WSL）

### 安装方式

**一键安装（推荐）**：
```bash
# macOS / Linux
curl -fsSL https://omp.sh/install | sh

# Windows (PowerShell)
irm https://omp.sh/install.ps1 | iex
```

**Homebrew**：
```bash
brew install can1357/tap/omp
```

**Bun（全局安装）**：
```bash
bun install -g @oh-my-pi/pi-coding-agent
```

**版本锁定（mise）**：

[mise](https://mise.jdx.dev) 是一个通用开发工具版本管理器（Rust 重写的 asdf 替代品），能锁定 omp 等任意工具的精确版本。适合团队统一开发环境，或需要固定版本的 CI 流程。

```bash
# 全局安装并锁定版本
mise use -g github:can1357/oh-my-pi

# 仅为当前项目安装（写入 .mise.toml，团队成员共享同一版本）
mise use github:can1357/oh-my-pi
```

### Shell 自动补全

```bash
# zsh
eval "$(omp completions zsh)"

# bash
eval "$(omp completions bash)"

# fish
omp completions fish > ~/.config/fish/completions/omp.fish
```

### 验证安装

```bash
omp --version
omp models          # 查看可用模型列表
```

---

## 3. 快速入门

### 3.1 首次启动

```bash
# 进入你的项目目录
cd ~/projects/my-app

# 启动交互式会话
omp
```

首次运行会：
1. 扫描可用模型和认证
2. 提示 `/login <provider>` 配置 API 密钥（如 `/login anthropic`）
3. 展示可用工具和模型

### 3.2 基本交互

```
>> 帮我找出项目中所有的 TODO 注释，并分类
```

代理会自动：
- 使用 `search` 搜索 TODO 模式
- 使用 `read` 查看匹配文件
- 生成分类报告

### 3.3 无头模式

不进入交互界面，直接在命令行执行一次提示后退出。适合脚本、CI 流水线、批量任务。

```bash
# 快速问答
omp -p "这个项目用了哪些依赖？"

# 指定模型
omp -p "解释 src/main.py" --model anthropic/claude-sonnet-4-5

# 批量提示
omp -p "审查最近的变更" && omp -p "生成 changelog"
```

### 3.4 配置 API 密钥

在交互会话中直接输入 `/login`，omp 会列出所有支持的供应商，**按上下键选择即可**——无需手动编辑配置文件或设置环境变量。

```
/login
```

如果需要通过环境变量配置（CI 等场景）：

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENAI_API_KEY="sk-..."
```

密钥也支持从 `.env` 文件加载（优先级：进程 env > `<cwd>/.env` > `~/.omp/agent/.env` > `~/.omp/.env` > `~/.env`）。

---

## 4. 核心概念

### 4.1 工具（Tools）

32 个内置工具，统一命名空间。代理自主选择调用哪个工具、传递什么参数。

最常用的几个：

| 工具 | 功能 |
|------|------|
| `read` | 读取文件、目录、URL、PDF、SQLite 数据库，支持行范围选择器 |
| `edit` | 基于 hash-anchor 的精确文本编辑 |
| `write` | 创建或覆写文件 |
| `search` | 正则搜索（Rust regex 引擎，进程内，极快） |
| `find` | glob 文件查找（进程内，尊重 `.gitignore`） |
| `bash` | 执行 shell 命令 |
| `lsp` | LSP 操作：定义跳转、引用、重命名、诊断、代码动作 |
| `debug` | DAP 调试：启动、断点、单步、变量查看 |
| `task` | 派生子代理并行工作 |
| `web_search` | 联网搜索（14 个搜索供应商链式回退） |
| `browser` | 无头 Chromium 操控 |
| `github` | GitHub CLI 集成：PR、Issue、搜索 |

### 4.2 会话（Session）

- **持久化**：会话自动保存为 JSONL 文件 (`~/.omp/agent/sessions/`)
- **追加式树结构**：每个条目有 `id` / `parentId`，支持回退和分支
- **支持恢复**：`omp --resume` 列出最近会话
- **分支导航**：`/tree` 查看会话树，可选择回退到历史节点

### 4.3 Subagent（子代理）

`task` 工具派生子代理处理子任务：
- 每个子代理独立运行，有自己的工具集
- 支持并行执行
- 类型化结果返回（通过 JSON Schema）
- 子代理间可通过 IRC 对等通信

### 4.4 知识记忆（Hindsight）

代理在会话间**记住你的代码库**：
- 使用 `retain` 写入事实
- 使用 `recall` 检索
- 项目级隔离，每个仓库独立记忆
- 会话压缩后持久化

### 4.5 TTSR（时间旅行流规则）

规则在模型偏离时**实时触发**：
- 正则匹配模型输出
- 匹配时中止流、注入规则提示、从断点重试
- 注入在 compaction 后仍然存活
- 零上下文开销——规则在不被触发时不消耗 token

### 4.6 Goal 模式

设置明确目标后，代理会持续朝该目标推进，直到完成或手动停止：
- 通过 `/goal` 命令或 `goal.enabled` 配置激活
- 目标激活后注入专用工具，代理自主分步执行
- 适合"实现用户登录模块"这类需要多轮操作的复杂任务

---

## 5. 工具介绍

### 5.1 `read` — 万能读取

```bash
# 基本用法（代理自动调用）
read path="src/main.ts"                    # 结构摘要模式
read path="src/main.ts:50-100"             # 指定行范围
read path="src/main.ts:raw"                # 原始模式
read path="https://arxiv.org/pdf/xxx.pdf"  # URL（包括 PDF）
read path="archive.zip:inner/file.ts"      # 压缩包内文件
read path="data.db:users"                  # SQLite 表
read path="skill://my-skill"               # 技能文件
```

`read` 的**结构摘要**模式会自动折叠函数体，只保留声明，节省大量上下文。

### 5.2 `edit` — 哈希锚点编辑

基于**内容哈希锚点**的编辑格式——比传统 diff 更可靠：

```
[src/main.ts#A1B2]          ← 哈希锚点头
SWAP 10.=10:                ← 替换第 10 行
+   const newLine = true;    ← 新内容
INS.POST 15:                ← 在第 15 行后插入
+   // 新增注释
DEL 20                      ← 删除第 20 行
```

优势：
- 文件被外部修改时，哈希不匹配 → **拒绝补丁**，防止数据损坏
- 模型不需要重复输出没改动的代码
- Grok 4 Fast 减少 61% 输出 token

### 5.3 `lsp` — 语言服务器集成

代理可以像 IDE 一样操作代码：

```bash
lsp action="definition" file="src/main.ts" line=42
lsp action="references" file="src/main.ts" line=42
lsp action="rename" file="src/main.ts" line=42 new_name="newFuncName"
lsp action="code_actions" file="src/main.ts" line=42
lsp action="diagnostics" file="src/**/*.ts"
```

重命名通过 `workspace/willRenameFiles` 传播，自动更新所有导入路径。

### 5.4 `debug` — 真实调试器

```bash
debug action="launch" program="./my_binary"
debug action="set_breakpoint" file="src/main.c" line=42
debug action="continue"
debug action="stack_trace"
debug action="variables" scope_id=1
```

支持的调试器：`lldb-dap`（C/C++/Rust）、`dlv`（Go）、`debugpy`（Python）、`gdb`。

### 5.5 `eval` — 持久化代码执行

Python (IPython) + JavaScript (Bun) 持久化内核：

```python
# Python cell
import pandas as pd
df = pd.read_csv("data.csv")
df.describe()
```

```javascript
// JS cell
const scores = df.toJSON(); // 共享 Python 变量
Math.max(...scores.map(r => r.score));
```

两个内核可以**回调代理工具**（read, search, task）。

### 5.6 `web_search` — 联网搜索

14 个搜索引擎供应商链式回退：Exa → Brave → Perplexity → Tavily → Kagi → Jina → SearXNG → Parallel → z.ai → Kimi → Moonshot → Anthropic → Codex → Parallel Web。自动选择第一个可用的。

**推荐配置 Exa**（质量最高），将 API 密钥写入 `~/.omp/agent/.env` 或项目 `.env`：

```bash
EXA_API_KEY="your-exa-key"
```

其他引擎同理，设置对应的环境变量即可。搜索引擎密钥**无法通过 `/login` 配置**，只能走环境变量或 `.env` 文件。

### 5.7 `search` 与 `find` — 文件搜索

两个进程内搜索工具，速度快且遵守 `.gitignore`：

- **`search`** — Rust regex 引擎，搜索文件**内容**。支持行锚点、多行模式、分页跳过。
- **`find`** — glob 模式匹配，搜索文件**名和目录**。支持 `.gitignore` 过滤、隐藏文件、超时限制。

对比传统 `grep`/`find` 命令，这两个工具输出结构化、带哈希锚点，代理可以直接用搜索结果定位编辑。

### 5.8 `bash` — 命令行执行

有状态的 shell 会话（基于 brush），支持构建、测试、git、包管理等所有 CLI 操作。

```bash
bash command="npm test -- --coverage" cwd="./packages/core"
bash command="git diff HEAD~3 --stat"
```

输出自动最小化（过滤 git/npm/cargo 的啰嗦日志），完整输出通过 artifact 引用恢复。

### 5.9 `browser` — 浏览器自动化

驱动无头 Chromium 标签页，支持导航、点击、填表、截图、JS 执行：

- `tab.observe()` — 获取页面可访问性快照（结构化元素列表）
- `tab.click()` / `tab.fill()` / `tab.type()` — 交互操作
- `tab.screenshot()` — 截图调试
- `tab.extract("markdown")` — 提取页面可读内容

默认无头模式，可通过 `browser.headless: false` 切换为可见窗口。

### 5.10 `github` — GitHub 集成

**前置条件**：安装 [GitHub CLI](https://cli.github.com)（`gh`），并完成 `gh auth login`。

支持的操作：

| 操作 | 说明 |
|------|------|
| PR 创建/检出 | `op: "pr_create"` / `op: "pr_checkout"` |
| PR 推送 | `op: "pr_push"`，将检出的 PR 分支推送回源 |
| Issue/PR/代码搜索 | `op: "search_issues"` / `op: "search_prs"` / `op: "search_code"` |
| Actions 监控 | `op: "run_watch"`，等待 CI 完成并返回失败日志 |

### 5.11 `ast_grep` 与 `ast_edit` — 结构化代码操作

基于 tree-sitter AST 的搜索和重写，比正则搜索更精确：

- **`ast_grep`** — 用 AST 模式匹配代码结构。例如找所有 `console.log($$$)` 调用，或找到某个函数的所有引用。
- **`ast_edit`** — 结构化批量替换。例如将 `oldApi($$$ARGS)` 重写为 `newApi($$$ARGS)`，跨文件生效。

两者的模式使用**元变量**（`$NAME`、`$$$ARGS`），匹配的是语法树节点而非文本，不受格式差异影响。

### 5.12 `inspect_image` — 图像分析

用视觉模型分析图片，支持 OCR 文字提取、UI 截图调试、场景识别。需要配置视觉模型（如 `modelRoles.vision`）。

### 5.13 `ask` / `todo` / `irc` — 交互与协作工具

| 工具 | 说明 |
|------|------|
| `ask` | 向用户发起选择题。当存在多套合理方案、用户需要决策时，代理调用此工具列出选项并等待选择。 |
| `todo` | 阶段性任务清单。代理自动创建、完成、放弃任务，用户可通过 `/todo` 查看进度。 |
| `irc` | **代理间实时通信**。子代理之间可直接互发消息（如协商文件编辑冲突），无需经过主代理中转。 |


### 5.14 `write` — 文件创建与覆写

与 `edit`（精确行级修改）互补，`write` 用于**创建新文件或整体覆写**已有文件。支持：

- 自动创建父目录
- 写入压缩包内文件（`archive.zip:inner/file.txt`）
- SQLite 行级操作（`db.sqlite:table:key`）

### 5.15 `task` — 子代理派发

将工作拆分为多个子代理并行执行，每个子代理独立运行、有自己的工具集。支持多种代理类型：

| 类型 | 用途 |
|------|------|
| `task` | 通用子代理，完整工具集 |
| `explore` | **只读**，代码搜索和分析 |
| `plan` | 架构规划，不写代码 |
| `oracle` | 高级工程师，调试/架构审查 |
| `designer` | UI/UX 设计审查 |
| `reviewer` | 代码审查 |

子代理通过结构化输出返回类型化结果，主代理直接读取字段而无需解析自然语言。详见 [第 12 章](#12-subagent-与任务并行)。

### 5.16 `job` — 异步任务管理

管理后台异步任务的生命周期：

- `list: true` — 查看所有运行中的任务
- `poll: [id, …]` — 等待指定任务完成
- `cancel: [id, …]` — 取消运行中的任务

配合 `async.enabled: true` 和 `bash.autoBackground.enabled`，可将长时间命令（构建、测试）放到后台执行，代理不用干等。

### 5.17 `checkpoint` 与 `rewind` — 会话检查点

**实验性探索的安全网**。`checkpoint` 标记当前会话状态，后续如果方向不对，`rewind` 一键回退到上一个检查点——探索性上下文被折叠为精简报告，而不是直接丢弃。

启用：`checkpoint.enabled: true`

### 5.18 `ssh` — 远程执行

通过 SSH 在远程主机上执行命令。需要预先配置 SSH 连接。适用于：

- 在远程服务器上运行构建/部署
- 操作无本地开发环境的嵌入式设备
- 跨机器文件操作

### 5.19 `render_mermaid` — 图表渲染

将 Mermaid 图表代码渲染为 ASCII 艺术，在终端内直接查看流程图、时序图、类图。启用：`renderMermaid.enabled: true`

---

## 6. 上下文文件（AGENTS.md / RULES.md）

### 6.1 自动加载机制

`omp` 启动时自动发现并注入上下文文件。**你不需要告诉代理去读它们**。

### 6.2 原生 `.omp` 约定（推荐）

```text
项目根目录/
  .omp/
    AGENTS.md     ← 项目级上下文（最近的祖先文件胜出）
    RULES.md      ← 粘性规则（始终生效，不随会话滚动而丢失）

用户目录/
  ~/.omp/agent/
    AGENTS.md     ← 用户级上下文（全局生效）
    RULES.md      ← 用户级粘性规则
```

- `AGENTS.md`：常规指导——代码风格、架构约束、测试规范、团队约定
- `RULES.md`：硬性规则——绝对禁止的操作、安全策略。**始终粘性**，在长会话中不会丢失

### 6.3 其他兼容格式

omp 自动读取其他工具的约定文件（**原生 > 其他**的优先级）：

| 文件 | 作用域 | 优先级 |
|------|--------|--------|
| `.omp/AGENTS.md` | 用户 + 项目 | 100（最高） |
| `.claude/CLAUDE.md` | 用户 + 项目 | 80 |
| `.agent/AGENTS.md` / `.agents/AGENTS.md` | 用户 + 项目 | 70 |
| `.codex/AGENTS.md` | 用户 | 70 |
| `.gemini/GEMINI.md` | 用户 + 项目 | 60 |
| `.config/opencode/AGENTS.md` | 用户 | 55 |
| `.github/copilot-instructions.md` | 项目 | 30 |
| `AGENTS.md`（独立文件） | 项目（祖先遍历） | 10 |

### 6.4 `@` 导入语法

在上下文文件中引用其他文件：

```markdown
# AGENTS.md
请先阅读 @docs/architecture.md 再修改存储代码。
共享发布步骤见 @../RELEASE.md
个人别名见 @~/.config/aliases.md
```

- 相对路径从导入文件所在目录解析
- `~/` 展开到家目录
- 代码块和行内代码中的 `@` 不展开
- 循环导入自动跳过
- 最多递归 5 层

### 6.5 文件注入行为

```xml
<context>
你必须遵循以下上下文文件：
<file path="/path/to/AGENTS.md">
...项目约定...
</file>
<file path="/path/to/package/.omp/AGENTS.md">
...子包约定...
</file>
</context>
```

- 文件按从远到近排序（祖先先、当前目录后）
- 同名文件深度优先（最近的赢）
- 字节完全相同的文件去重

---

## 7. 模型与供应商配置

> **绝大多数用户只需在交互会话中输入 `/login` 选择供应商即可**，omp 会自动配置 API 密钥、发现可用模型。本章后续内容（models.yml、本地模型、模型角色等）面向需要自定义端点、团队统一配置或高级调优的用户。

### 7.1 配置文件

核心配置文件：`~/.omp/agent/models.yml`

```yaml
# models.yml 示例
providers:
  # 自定义 OpenAI 兼容端点
  my-local:
    baseUrl: http://127.0.0.1:8000/v1
    api: openai-completions
    auth: none
    models:
      - id: local-model
        name: 本地模型
        contextWindow: 128000
        maxTokens: 8192

  # 带认证的代理
  my-gateway:
    baseUrl: https://gateway.example.com/v1
    api: openai-completions
    apiKey: MY_GATEWAY_API_KEY
    authHeader: true
    models:
      - id: claude-sonnet
        name: Claude Sonnet (via Gateway)
        contextWindow: 200000
        maxTokens: 8192

# 禁用不需要的供应商
disabledProviders:
  - google
  - groq
```

### 7.2 认证优先级

API 密钥解析顺序（先匹配先得）：

1. 运行时覆盖（`--api-key`）
2. `models.yml` 中的 `apiKey`
3. 存储的 API 密钥（`agent.db`）
4. 存储的 OAuth 凭证（含自动刷新）
5. 环境变量（`ANTHROPIC_API_KEY` 等）
6. 供应商自定义回退

### 7.3 本地模型

三个自动发现的本地引擎：

| 引擎 | 默认地址 | 环境变量覆盖 |
|------|----------|-------------|
| Ollama | `http://127.0.0.1:11434` | `OLLAMA_BASE_URL` / `OLLAMA_HOST` |
| llama.cpp | `http://127.0.0.1:8080` | `LLAMA_CPP_BASE_URL` |
| LM Studio | `http://127.0.0.1:1234/v1` | `LM_STUDIO_BASE_URL` |

无需配置——引擎运行时自动发现。

### 7.4 模型角色

```yaml
# ~/.omp/agent/config.yml
modelRoles:
  default: anthropic/claude-sonnet-4-5
  smol: anthropic/claude-haiku-4-5
  slow: anthropic/claude-opus-4-6
  plan: anthropic/claude-sonnet-4-5
  title: openai/gpt-4o-mini
```

启动时指定模型：
```bash
omp --model openai/gpt-5.3-codex
omp --smol anthropic/claude-haiku-4-5 --slow openai/gpt-4o
```

### 7.5 项目级模型限制

```yaml
# <project>/.omp/config.yml
enabledModels:
  - anthropic/claude-sonnet-4-5
  - anthropic/claude-haiku-4-5

# 或路径级范围
enabledModels:
  - path: ~/work/sensitive
    models:
      - anthropic/claude-sonnet-4-5
```

---

## 8. Slash 命令

### 8.1 内置命令

交互模式下可用：

| 命令 | 功能 |
|------|------|
| `/model` | 模型选择 |
| `/settings` | 设置管理 |
| `/mcp` | MCP 服务器管理 |
| `/move` | 切换工作目录 |
| `/login` / `/logout` | 认证管理 |
| `/plan` | 规划模式：先做只读探索、出方案，确认后再动手改代码 |
| `/review` | 代码审查 |
| `/commit` | AI 辅助提交 |
| `/compact` | 手动触发上下文压缩 |
| `/handoff` | 生成会话交接文档，保存到磁盘 |
| `/tree` | 会话树导航 |
| `/skill:<name>` | 调用技能 |
| `/exit` | 退出 |

### 8.2 自定义文件命令

```text
# ~/.omp/agent/commands/review-prs.md
---
description: 审查当前分支的所有 PR
---
请审查当前分支中所有未合并的 PR：
1. 阅读每个 PR 的变更
2. 检查代码质量、测试覆盖、安全性
3. 生成审查报告
```

参数支持：
- `$1`, `$2`, ... — 位置参数
- `$ARGUMENTS` — 所有参数
- `$@` — 所有参数作为数组

### 8.3 命令来源优先级

原生（native, 100） > omp 插件（90） > claude（80） > claude 插件/agents/codex（70） > opencode（55）

---

## 9. TUI 交互界面

### 9.1 布局

```
┌──────────────────────────────────────┐
│  状态栏: [模型] [会话名] [token] [成本] │
├──────────────────────────────────────┤
│                                      │
│  消息区域（代理输出 + 工具结果）         │
│                                      │
├──────────────────────────────────────┤
│  输入区域: >> 你的提示                 │
└──────────────────────────────────────┘
```

### 9.2 快捷键

| 按键 | 功能 |
|------|------|
| `Ctrl+C` | 中止当前操作 |
| `Ctrl+Enter` | 排队跟进消息（Follow-up） |
| `Enter` | 发送/转向消息 |
| `Ctrl+K` | 清除编辑器 |
| `Ctrl+L` | 清屏（滚动对话到顶部） |
| `Ctrl+O` | 切换工具面板展开 |
| `Ctrl+P` | 历史提示向上 |
| `Tab` | 自动补全（路径、命令） |

### 9.3 流式交互

代理流式输出时，你可以**中途转向**：
- 按 `Enter` → 打断当前输出，注入新指令（steer）
- 按 `Ctrl+Enter` → 排队，当前输出完成后再处理（follow-up）

### 9.4 主题

```yaml
# ~/.omp/agent/config.yml
theme:
  dark: catppuccin-mocha  # 暗色主题
  light: catppuccin-latte  # 亮色主题
```

---

## 10. 会话管理

### 10.1 会话存储

- **格式**：JSONL（每行一个 JSON 对象）
- **位置**：`~/.omp/agent/sessions/<project>/<timestamp>_<id>.jsonl`
- **结构**：追加式树，每个条目有 `id` / `parentId`

### 10.2 会话操作

```bash
# 恢复最近会话
omp --resume

# 列出项目会话
omp sessions list

# 列出所有会话
omp sessions list --all

# 恢复指定会话
omp --session <id-or-path>
```

### 10.3 会话树

会话支持**分支**和**回退**：

```
└── 初始分析
    ├── 方案 A（被放弃）
    ├── 方案 B
    │   ├── 实现
    │   └── 测试
    └── 方案 C（当前）
```

使用 `/tree` 命令查看和导航。

### 10.4 Compaction（上下文压缩）

当对话过长时，omp 自动压缩：
- 保留最近的上下文
- 摘要早期对话
- 规则注入在压缩后仍然存活

---

## 11. 最佳实践

### 11.1 项目配置

```text
my-project/
  .omp/
    AGENTS.md        ← 代码风格、架构约定
    RULES.md         ← 硬性规则（如"绝不提交未测试的代码"）
    config.yml       ← 项目级设置
    mcp.json         ← MCP 服务器
    commands/        ← 自定义斜杠命令
      deploy.md
    skills/          ← 项目专属技能
      database/
        SKILL.md
```

### 11.2 AGENTS.md 模板

```markdown
# 项目约定

## 代码风格
- TypeScript 严格模式，noUncheckedIndexedAccess
- 使用 Zod 进行运行时验证
- 异步优先，避免同步阻塞

## 测试
- 单元测试用 vitest
- 集成测试用 playwright
- 覆盖率目标 80%

## Git 约定
- 提交信息用中文
- 分支命名：`feat/`, `fix/`, `chore/`

## 架构
- 详见 @docs/architecture.md
```

### 11.3 RULES.md 模板

```markdown
# 硬性规则

- 绝对不要在没有明确要求时提交或推送代码。
- 绝对不要在未确认的情况下删除超过 50 行的代码。
- 不要编辑 `generated/` 目录下的文件。
- 数据库迁移必须包含回滚脚本。
- 所有 API 路由必须包含速率限制。
```

### 11.4 模型选择策略

| 场景 | 推荐模型 | 原因 |
|------|---------|------|
| 日常编码 | Claude Sonnet 4.5 | 性价比最优 |
| 复杂重构 | Claude Opus 4.6 / GPT-5.3 Codex | 深度推理 |
| 快速补全 | Claude Haiku 4.5 / Gemini Flash | 低延迟 |
| 代码审查 | Claude Sonnet 4.5 | 结构化输出好 |
| 设计/规划 | Claude Opus 4.6 | 创意和全局把握 |

```yaml
# config.yml
modelRoles:
  default: anthropic/claude-sonnet-4-5
  smol: anthropic/claude-haiku-4-5
  slow: anthropic/claude-opus-4-6
  plan: anthropic/claude-sonnet-4-5
```

### 11.5 成本控制

```yaml
# config.yml
compaction:
  enabled: true       # 自动压缩长对话
  threshold: 40000    # token 阈值

retry:
  enabled: true       # 自动重试失败
  maxRetries: 3

# 限制可用模型
enabledModels:
  - anthropic/claude-sonnet-4-5
  - anthropic/claude-haiku-4-5
```

### 11.6 常用设置速查

通过 `/settings` 命令交互式调整，或直接编辑 `~/.omp/agent/config.yml`。以下是最常修改的设置：

| 设置键 | 默认值 | `/settings` 位置 | 说明 |
|--------|--------|-------------------|------|
| `tools.approvalMode` | `"yolo"` | Interaction → 工具审批 | `always-ask`（每次确认）/ `write`（写操作确认）/ `yolo`（全部自动执行） |
| `task.maxConcurrency` | `32` | Tasks → 子代理行为 | 子代理最大并发数，低配机器可降到 4–8 |
| `browser.headless` | `true` | Tools → 获取与浏览 | `false` 时弹出可见浏览器窗口，方便调试 |
| `hideThinkingBlock` | `false` | Model → 推理与提示 | `true` 时隐藏模型的思考过程，输出更干净 |
| `theme.dark` | `"titanium"` | Appearance | 深色主题，可选 `catppuccin-mocha`、`dracula` 等 |
| `symbolPreset` | `"unicode"` | Appearance | 图标风格：`nerd`（Nerd Font）、`ascii`（Windows 终端兼容） |
| `compaction.strategy` | `"context-full"` | Context → 压缩 | `handoff` 时压缩并保存摘要到磁盘；`off` 关闭自动压缩 |
| `compaction.handoffSaveToDisk` | `false` | Context → 压缩 | handoff 模式下将摘要保存为 `.md` 文件 |
| `edit.mode` | `"hashline"` | Editing → 编辑工具 | 编辑模式：`replace` / `patch` / `hashline` / `apply_patch` |
| `github.enabled` | `false` | Tools → 获取与浏览 | 设为 `true` 启用 GitHub 集成（需安装 `gh` CLI） |
| `inspect_image.enabled` | `false` | Tools → 可选工具开关 | 设为 `true` 启用图像分析 |
| `async.enabled` | `false` | Tools → 异步任务 | 设为 `true` 启用异步后台任务 |
| `memory.backend` | `"off"` | Memory → 记忆后端 | `local` / `hindsight`（跨会话记住代码库） |

```yaml
# ~/.omp/agent/config.yml 示例
tools:
  approvalMode: yolo
task:
  maxConcurrency: 8
browser:
  headless: false
compaction:
  strategy: handoff
  handoffSaveToDisk: true
```


> 一份真实可用的完整配置示例见 [附录 D：个人配置示例](#附录-d个人配置示例)。
---

## 12. Subagent 与任务并行

### 12.1 基本用法

通过 `task` 工具派生子代理：

```typescript
// 代理内部调用 task 工具
{
  "tasks": [
    {
      "id": "ComponentExports",
      "assignment": "分析 src/components 的导出结构，返回所有公共 API",
      "role": "组件导出分析师"
    },
    {
      "id": "RoutesExports", 
      "assignment": "分析 src/routes 的路由结构，返回所有路由定义",
      "role": "路由结构分析师"
    }
  ],
  "context": "项目使用 React + TypeScript, 导出通过 index.ts barrel 文件"
}
```

### 12.2 子代理类型

| 类型 | 说明 | 工具权限 |
|------|------|----------|
| `task` | 通用子代理，完整工具集 | 全部 |
| `quick_task` | 轻量级，机械性更新 | 有限 |
| `explore` | **只读**代码搜索 | 只读 |
| `oracle` | 高级工程师，调试/架构 | 全部 |
| `designer` | UI/UX 设计审查 | 全部 |
| `reviewer` | 代码审查 | 全部 |
| `plan` | 架构规划 | 全部 |
| `librarian` | 外部库研究 | 只读 |

### 12.3 IRC 对等通信

子代理之间可以**直接通信**，无需经过主代理：

```
子代理 A → irc send to="子代理 B" message="你在处理 auth.ts 吗？我需要添加一个 401 路径。"
子代理 B → irc send to="子代理 A" message="是的，我刚完成。请使用我定义的类型。"
```

### 12.4 结构化输出

```typescript
// 子代理返回类型化结果
{
  "outputSchema": {
    "type": "object",
    "properties": {
      "totalFunctions": { "type": "number" },
      "totalClasses": { "type": "number" },
      "exportedFunctions": { "type": "array", "items": { "type": "string" } }
    }
  }
}
```

---

## 13. 进阶功能

### 13.1 技能系统（Skills）

#### 13.1.1 什么是技能

技能是**命名的知识/工作流包**，按需加载：
- 代理在系统提示词中看到技能列表（名称 + 描述）
- 通过 `skill://<name>` 读取完整内容
- 可通过 `/skill:<name>` 交互调用

#### 13.1.2 创建技能

```text
~/.omp/agent/skills/
  postgres/
    SKILL.md        ← 技能主文件
    references/
      schema.sql    ← 参考文件（通过 skill://postgres/references/schema.sql 访问）
```

```markdown
# SKILL.md
---
name: postgres
description: PostgreSQL 数据库设计、查询优化、迁移管理
globs:
  - "**/*.sql"
  - "**/migrations/**"
---

# PostgreSQL 最佳实践

## 查询优化
- 始终使用 EXPLAIN ANALYZE 验证查询计划
- 对频繁查询的列建立索引
- 避免 SELECT *，明确列出所需列

## 迁移
- 使用事务包裹迁移
- 先加列再删列
- 大表迁移使用批处理
```

#### 13.1.3 技能发现优先级

原生（100） > omp 插件（90） > claude（80） > claude 插件/agents/codex（70） > opencode（55） > github（30）

同名技能：优先级高的胜出。

#### 13.1.4 前端属性

```yaml
---
name: my-skill
description: 我的技能
hide: false       # true 时不在系统提示词中列出，但仍可通过 skill:// 访问
alwaysApply: true # 始终注入，不等待代理选择
globs:            # 关联文件模式（帮助代理判断何时使用）
  - "**/*.rs"
---
```

---

### 13.2 MCP 集成

#### 13.2.1 配置文件

**推荐位置**：
- 项目：`.omp/mcp.json`
- 用户：`~/.omp/agent/mcp.json`

#### 13.2.2 配置示例

```json
{
  "$schema": "https://raw.githubusercontent.com/can1357/oh-my-pi/main/packages/coding-agent/src/config/mcp-schema.json",
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed/dir"]
    },
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "slack": {
      "type": "http",
      "url": "https://mcp.slack.com/mcp",
      "oauth": {
        "clientId": "YOUR_CLIENT_ID",
        "clientSecret": "YOUR_CLIENT_SECRET"
      },
      "auth": {
        "type": "oauth",
        "tokenUrl": "https://slack.com/api/oauth.v2.user.access"
      }
    }
  },
  "disabledServers": []
}
```

#### 13.2.3 支持的传输方式

| 类型 | 字段 | 场景 |
|------|------|------|
| `stdio`（默认） | `command`, `args`, `env`, `cwd` | 本地进程 |
| `http` | `url`, `headers` | 远程 Streamable HTTP |
| `sse` | `url`, `headers` | 远程 SSE（兼容旧版） |

#### 13.2.4 环境变量和秘密

```json
{
  "env": {
    "GITHUB_PERSONAL_ACCESS_TOKEN": "GITHUB_TOKEN"
  },
  "headers": {
    "Authorization": "!printf 'Bearer %s' \"$MY_TOKEN\""
  }
}
```

- `"VAR_NAME": "ENV_VAR_NAME"` → 从当前环境复制
- `"VAR_NAME": "!command"` → 运行命令，使用 stdout（10s 超时）
- `"VAR_NAME": "literal"` → 字面值

#### 13.2.5 常用命令

```
/mcp add       # 向导式添加
/mcp list      # 列出所有服务器
/mcp reload    # 重新发现并连接
/mcp test <n>  # 测试单个服务器
/mcp reconnect <n>  # 重连单个服务器
```



### 13.3 系统提示词定制

#### 13.3.1 SYSTEM.md

创建 `~/.omp/agent/SYSTEM.md` 或 `<project>/.omp/SYSTEM.md` 来定制系统提示词。

```markdown
# SYSTEM.md
你是一个专注于后端开发的 Rust 专家。
始终优先使用 `anyhow::Result` 而不是手动错误处理。
```

#### 13.3.2 APPEND_SYSTEM.md

创建 `~/.omp/agent/APPEND_SYSTEM.md` 追加内容（不替换）：

```markdown
# APPEND_SYSTEM.md
在编写测试时，使用 `rstest` 框架而不是标准 `#[test]`。
```

#### 13.3.3 标题定制

`~/.omp/agent/TITLE_SYSTEM.md` 覆盖自动会话标题生成提示词：

```markdown
使用小写 `<type>:<primary-objective>` 格式生成会话名称。
```



### 13.4 扩展系统（Extensions）

#### 13.4.1 扩展能做什么

一个扩展可以同时包含：
- **事件处理器**：在会话/工具生命周期中挂载
- **自定义工具**：LLM 可调用的新工具
- **Slash 命令**：用户交互命令
- **渲染器**：自定义 TUI 显示
- **键盘快捷键**

#### 13.4.2 简单扩展示例

```typescript
// my-extension.ts
import type { ExtensionAPI } from "@oh-my-pi/pi-coding-agent";

export default function (pi: ExtensionAPI) {
  const { z } = pi.zod;

  // 阻止危险命令
  pi.on("tool_call", async (event) => {
    if (event.toolName === "bash" && event.input.command?.includes("rm -rf")) {
      return { block: true, reason: "扩展策略阻止" };
    }
  });

  // 注册自定义工具
  pi.registerTool({
    name: "hello",
    label: "问候",
    description: "返回问候信息",
    parameters: z.object({ name: z.string() }),
    async execute(_id, params, _signal, _onUpdate, _ctx) {
      return {
        content: [{ type: "text", text: `你好, ${params.name}` }],
        details: { greeted: params.name },
      };
    },
  });

  // 注册命令
  pi.registerCommand("status", {
    description: "显示当前状态",
    handler: async (_args, ctx) => {
      ctx.ui.notify(`工作目录: ${ctx.cwd}`, "info");
    },
  });
}
```

#### 13.4.3 事件生命周期

```
session_start → turn_start → tool_call → tool_result → turn_end → session_shutdown
```

关键事件钩子：

| 事件 | 说明 | 可阻止？ |
|------|------|----------|
| `tool_call` | 工具执行前 | 是（block） |
| `tool_result` | 工具执行后 | 可修改结果 |
| `before_agent_start` | 代理启动前 | 否 |
| `before_provider_request` | 请求发送前 | 可替换 payload |
| `session_before_compact` | 压缩前 | 是（cancel） |

#### 13.4.4 扩展加载

扩展通过以下方式发现：
- `~/.omp/agent/extensions/` 目录
- `<project>/.omp/extensions/` 目录
- CLI 参数 `omp --extension <path>`



### 13.5 SDK 嵌入模式

#### 13.5.1 快速开始

```typescript
import { createAgentSession } from "@oh-my-pi/pi-coding-agent";

const { session } = await createAgentSession();

session.subscribe((event) => {
  if (
    event.type === "message_update" &&
    event.assistantMessageEvent.type === "text_delta"
  ) {
    process.stdout.write(event.assistantMessageEvent.delta);
  }
});

await session.prompt("用三句话总结这个仓库。");
await session.dispose();
```

#### 13.5.2 受控嵌入

```typescript
import {
  createAgentSession,
  discoverAuthStorage,
  ModelRegistry,
  SessionManager,
  Settings,
} from "@oh-my-pi/pi-coding-agent";

const authStorage = await discoverAuthStorage();
const modelRegistry = new ModelRegistry(authStorage);
await modelRegistry.refresh();

const settings = Settings.isolated({
  "compaction.enabled": true,
  "retry.enabled": true,
});

const { session } = await createAgentSession({
  authStorage,
  modelRegistry,
  settings,
  sessionManager: SessionManager.inMemory(),
  toolNames: ["read", "search", "find", "edit", "write"],
  enableMCP: false,
  enableLsp: true,
  model: modelRegistry.getAvailable()[0],
});
```

#### 13.5.3 事件订阅

```typescript
session.subscribe((event) => {
  switch (event.type) {
    case "agent_start":      /* 代理启动 */
    case "turn_start":       /* 新一轮开始 */
    case "message_update":   /* 流式文本增量 */
    case "tool_execution_start": /* 工具执行开始 */
    case "auto_compaction_start": /* 自动压缩开始 */
    case "auto_retry_start": /* 自动重试开始 */
    case "agent_end":        /* 代理结束 */
  }
});
```

#### 13.5.4 会话管理 API

```typescript
// 文件持久化
SessionManager.create(process.cwd());

// 内存模式（测试/临时）
SessionManager.inMemory();

// 恢复/打开/列表
SessionManager.continueRecent(process.cwd());
SessionManager.list(process.cwd());
SessionManager.open(path);
```



### 13.6 ACP 编辑器驱动模式

在 Zed 等编辑器中运行 omp：

```bash
# 配置 Zed 使用 omp 作为 ACP 代理
omp acp
```

特点：
- 读取编辑器当前缓冲区
- 通过编辑器的保存路径写入
- 在编辑器的终端中运行 shell
- 破坏性操作弹出权限提示



## 14. 进阶主题

### 14.1 自定义工具

```typescript
// ~/.omp/agent/tools/my-tool.ts
import type { ToolFactory } from "@oh-my-pi/pi-coding-agent";

const tool: ToolFactory = (host) => ({
  name: "my_tool",
  description: "我的自定义工具",
  parameters: {
    type: "object",
    properties: {
      query: { type: "string", description: "查询字符串" }
    },
    required: ["query"]
  },
  async execute({ query }) {
    return {
      content: [{ type: "text", text: `处理查询: ${query}` }]
    };
  }
});

export default tool;
```

### 14.2 Hook 系统

```typescript
// ~/.omp/agent/hooks/pre/tool-call.ts
import type { HookHandler } from "@oh-my-pi/pi-coding-agent";

const handler: HookHandler = async (event, ctx) => {
  if (event.toolName === "bash") {
    ctx.logger.info(`即将执行命令: ${event.input.command}`);
  }
};

export default handler;
```

### 14.3 多 Profile

```bash
# 创建并使用命名 Profile
omp --profile work
omp --profile personal

# 或使用别名
omp --alias work
```

Profile 隔离：
- 独立的会话历史
- 独立的 MCP 配置
- 独立的 API 密钥和模型配置
- 共享项目级 `.omp/` 配置

### 14.4 Auth Broker（团队认证管理）

```bash
# 启动 Auth Broker 服务
omp auth-broker serve

# 客户端配置
export OMP_AUTH_BROKER_URL="https://broker.internal:8765"
export OMP_AUTH_BROKER_TOKEN="team-token"
```

- 集中管理 API 密钥和 OAuth 令牌
- 团队成员共享凭证而无需分发密钥
- 支持令牌刷新和轮换

### 14.5 RPC 模式

```bash
# 启动 RPC 服务器
omp rpc

# 客户端通过 stdio 发送 JSON-RPC 请求
echo '{"jsonrpc":"2.0","method":"prompt","params":{"text":"hello"}}' | omp rpc
```

### 14.6 Commit 助手

```bash
# AI 辅助提交
omp commit

# omp 会：
# 1. 分析工作树变更（git_overview, git_file_diff, git_hunk）
# 2. 按依赖关系自动拆分为原子提交
# 3. 源码优先于测试/文档/配置
# 4. 生成规范的提交信息
```

---

## 15. 疑难解答

### 15.1 常见问题

**Q: 模型不可用？**
```
omp models                    # 查看可用模型
/login <provider>             # 交互式登录
# 或检查环境变量是否设置
```

**Q: 工具执行失败？**
- 检查 LSP 服务器是否安装（如 `typescript-language-server`）
- 对于 bash 工具，检查命令是否在 PATH 中

**Q: 编辑结果不正确？**
- 哈希锚点编辑在文件被外部修改时会自动拒绝
- 使用 `read` 重新读取文件后再编辑

**Q: 会话太大？**
- 启用自动压缩：`config.yml` → `compaction.enabled: true`
- 手动压缩：在会话中发送 `/compact`

**Q: token 消耗过快？**
- 使用 `smol` 角色模型处理简单任务
- 启用压缩减少历史长度
- 使用 `RULES.md` 替代长 `AGENTS.md` 中的硬性规则

### 15.2 调试

```bash
# 查看启动耗时分解
PI_TIMING=1 omp

# 调试启动过程（显示每个阶段的开始/结束）
PI_DEBUG_STARTUP=1 omp

# Python 内核 IPC 追踪
PI_PYTHON_IPC_TRACE=1 omp
```

### 15.3 日志和状态

```
/status      # 查看会话状态（模型、工具、token 消耗）
/lsp status  # 查看 LSP 服务器状态
/mcp list    # 查看 MCP 服务器列表
```

---

## 附录 A: 配置文件速查

| 文件 | 用途 |
|------|------|
| `~/.omp/agent/config.yml` | 全局设置 |
| `~/.omp/agent/models.yml` | 模型/供应商配置 |
| `~/.omp/agent/mcp.json` | 用户级 MCP 服务器 |
| `~/.omp/agent/AGENTS.md` | 用户级上下文 |
| `~/.omp/agent/RULES.md` | 用户级粘性规则 |
| `~/.omp/agent/SYSTEM.md` | 系统提示词覆盖 |
| `<project>/.omp/config.yml` | 项目级设置 |
| `<project>/.omp/AGENTS.md` | 项目级上下文 |
| `<project>/.omp/RULES.md` | 项目级粘性规则 |
| `<project>/.omp/mcp.json` | 项目级 MCP 服务器 |

## 附录 B: 环境变量速查

| 变量 | 用途 |
|------|------|
| `ANTHROPIC_API_KEY` | Anthropic API 密钥 |
| `OPENAI_API_KEY` | OpenAI API 密钥 |
| `GEMINI_API_KEY` | Google Gemini API 密钥 |
| `DEEPSEEK_API_KEY` | DeepSeek API 密钥 |
| `OPENROUTER_API_KEY` | OpenRouter API 密钥 |
| `OLLAMA_BASE_URL` | Ollama 服务地址 |
| `LM_STUDIO_BASE_URL` | LM Studio 服务地址 |
| `PI_CODING_AGENT_DIR` | 代理数据目录（默认 `~/.omp/agent`） |
| `OMP_PROFILE` / `PI_PROFILE` | 当前 Profile 名称 |

## 附录 C: 官方资源

- 官网: [https://omp.sh](https://omp.sh)
- GitHub: [https://github.com/can1357/oh-my-pi](https://github.com/can1357/oh-my-pi)
- Discord: [https://discord.gg/4NMW9cdXZa](https://discord.gg/4NMW9cdXZa)
- NPM: [https://www.npmjs.com/package/@oh-my-pi/pi-coding-agent](https://www.npmjs.com/package/@oh-my-pi/pi-coding-agent)
- 博客: [https://blog.can.ac](https://blog.can.ac)


## 附录 D: 个人配置示例

以下是一份生产环境的 `~/.omp/agent/config.yml`，标注了关键决策：

```yaml
# === 模型路由 ===
modelRoles:
  default: opencode-go/deepseek-v4-pro:high      # 日常编码主力
  slow: opencode-go/deepseek-v4-pro:xhigh         # 深度推理（plan/task 复用）
  smol: opencode-go/deepseek-v4-flash:high        # 轻量/快速任务
  vision: opencode-go/kimi-k2.6:medium            # 图像分析
  designer: opencode-go/kimi-k2.6:xhigh           # UI/设计任务
  commit: opencode-go/deepseek-v4-pro:high        # 生成提交信息

# === 外观 ===
theme:
  dark: dark-monokai
symbolPreset: nerd                                # 需要 Nerd Font
hideThinkingBlock: true                           # 隐藏模型思考过程
collapseChangelog: true                           # 精简更新日志

# === 任务与隔离 ===
task:
  maxConcurrency: 32
  isolation:
    mode: rcopy                                   # 子代理文件系统隔离（Windows 友好）
    merge: patch                                  # 变更通过补丁合并

# === 工具 ===
tools:
  approvalMode: yolo                              # 全部自动执行，无需确认
browser:
  headless: true
  screenshotDir: ~/Desktop                        # 截图保存到桌面
inspect_image:
  enabled: true
github:
  enabled: true
web_search:
  enabled: true

# === 上下文与压缩 ===
compaction:
  strategy: handoff                               # 生成交接文档并保存到磁盘
  handoffSaveToDisk: true
  remoteEnabled: true
contextPromotion:
  enabled: false                                  # 关闭自动模型提升
autoResume: true                                  # 启动时自动恢复上次会话

# === 提供商 ===
providers:
  webSearch: exa                                  # 搜索引擎：Exa
  image: auto
  fetch: auto

# === 禁用 ===
disabledExtensions:
  - context-file:user:CLAUDE.md                   # 不加载 Claude 的上下文文件
commands:
  enableClaudeUser: false
  enableClaudeProject: false
  enableOpencodeUser: false
  enableOpencodeProject: false                    # 只使用 omp 原生命令

# === 其他 ===
edit:
  mode: hashline
lsp:
  enabled: true
async:
  enabled: true
memory:
  backend: "off"                                  # 暂不启用跨会话记忆
```

> **几点说明**：
> - `approvalMode: yolo` 适合个人开发，团队环境建议用 `write`（写操作需确认）
> - `compaction.strategy: handoff` 会在会话过长时生成 markdown 摘要文件，方便事后回顾
> - `task.isolation.mode: rcopy` 在 Windows 上表现最好，通过 robocopy 隔离子代理工作区
> - 禁用 `CLAUDE.md` 等外部命令加载，避免与 omp 原生约定冲突
