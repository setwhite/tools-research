# Zed 编辑器完全教程

> **Zed** 是一款由 Rust 编写的高性能、开源代码编辑器，由前 Atom 核心团队（Nathan Sobo、Antonio Scandurra、Max Brunsfeld）打造。它集成了 AI 辅助编程、实时多人协作、远程开发等现代化特性，同时保持了极致的启动速度和响应性能。

---

## 目录

1. [Zed 是什么](#1-zed-是什么)
2. [安装与更新](#2-安装与更新)
3. [界面与布局](#3-界面与布局)
4. [核心快捷键速查](#4-核心快捷键速查)
5. [项目管理](#5-项目管理)
6. [代码编辑](#6-代码编辑)
7. [终端](#7-终端)
8. [Git 集成](#8-git-集成)
9. [AI 功能](#9-ai-功能)
10. [调试器](#10-调试器)
11. [Task 任务系统](#11-task-任务系统)
12. [Vim / Helix 模式](#12-vim--helix-模式)
13. [多人协作](#13-多人协作)
14. [远程开发](#14-远程开发)
15. [扩展生态](#15-扩展生态)
16. [配置与个性化](#16-配置与个性化)
17. [定价与许可](#17-定价与许可)
18. [常见问题与技巧](#18-常见问题与技巧)

---

## 1. Zed 是什么

Zed 是一个**为速度而生**的代码编辑器，核心特性包括：

| 特性 | 说明 |
| --- | --- |
| **高性能** | 基于 Rust 和自研 GPU 加速 UI 框架 GPUI，启动速度和编辑响应极快 |
| **AI 原生** | 内置 Agent Panel、Inline Assistant、Edit Prediction 等多个 AI 工作流 |
| **实时协作** | 支持多人同时编辑同一项目，可见彼此光标，支持语音通话 |
| **远程开发** | 通过 SSH 在远程服务器上编辑代码，本地 UI 保持流畅 |
| **Vim/Helix 模式** | 内置完整的 Vim 模拟层和 Helix 模式，模态编辑爱好者的首选 |
| **开源** | 代码托管在 [GitHub](https://github.com/zed-industries/zed)，社区驱动开发 |

**与 VS Code 的核心区别**：Zed 不是 Electron 应用，而是原生应用。它用 Rust 编写，直接调用系统图形 API，因此在性能上有数量级的优势——尤其是在大型项目中。

---

## 2. 安装与更新

### 2.1 macOS

```bash
# Homebrew 安装（稳定版）
brew install --cask zed

# Homebrew 安装（预览版，提前体验新功能）
brew install --cask zed@preview
```

也可以直接从 [zed.dev/download](https://zed.dev/download) 下载 `.dmg` 安装包。

**系统要求**：macOS 12 (Monterey) 及以上。Intel 和 Apple Silicon 均支持。

### 2.2 Windows

```bash
# winget 安装
winget install -e --id ZedIndustries.Zed
```

或从 [zed.dev/download](https://zed.dev/download) 下载安装程序。

**系统要求**：Windows 10 1903+ 或 Windows 11，需要 64 位系统、支持 DirectX 11 的 GPU。

### 2.3 Linux

```bash
# 一键安装脚本（推荐）
curl -f https://zed.dev/install.sh | sh

# 安装指定版本
curl -f https://zed.dev/install.sh | ZED_VERSION=0.216.0 sh

# 安装预览版
curl -f https://zed.dev/install.sh | ZED_CHANNEL=preview sh
```

**系统要求**：x86_64 或 aarch64 处理器，需要 Vulkan 1.3 驱动。支持 Ubuntu、Arch、Debian、Fedora、RedHat、CentOS 等主流发行版。

### 2.4 更新

Zed 安装后会**自动检查并下载更新**。安装完成后重启即可。你也可以手动检查更新：

- 打开命令面板（`Cmd+Shift+P` / `Ctrl+Shift+P`）
- 搜索 `auto update: check`

### 2.5 卸载

- **macOS**：将 Zed 拖入废纸篓，或 `brew uninstall --cask zed`
- **Windows**：通过系统设置 → 应用 → 卸载
- **Linux （脚本安装的）**：运行 `zed --uninstall`，然后按提示选择是否保留配置

---

## 3. 界面与布局

### 3.1 主要面板

| 面板 | 快捷键（macOS） | 快捷键（Linux/Windows） | 功能 |
| --- | --- | --- | --- |
| 项目面板 | `Cmd+Shift+E` | `Ctrl+Shift+E` | 浏览项目文件树 |
| 搜索面板 | `Cmd+Shift+F` | `Ctrl+Shift+F` | 全局搜索与替换 |
| Git 面板 | 点击状态栏 Git 图标 | 同左 | 暂存、提交、分支管理 |
| 终端面板 | `` Ctrl+` `` | `` Ctrl+` `` | 内置终端 |
| Agent 面板 | `Cmd+Shift+A` | `Ctrl+Shift+A` | AI 对话与代理 |
| 协作面板 | `Cmd+Shift+C` | `Ctrl+Shift+C` | 多人协作 |
| 扩展面板 | `Cmd+Shift+X` | `Ctrl+Shift+X` | 安装与管理扩展 |
| 调试面板 | 点击状态栏 Debug 图标 | 同左 | 设置断点与调试 |

### 3.2 面板布局切换

Zed 提供两种预设布局：

- **Agentic 布局**：Agent Panel 和 Threads Sidebar 并排在左侧，适合 AI 工作流
- **Classic 布局**：传统编辑器布局，最大化编辑区域

通过菜单 **Zed → Panel Layout** 或命令面板搜索 `panel layout` 切换。

### 3.3 分屏与标签页

| 操作 | macOS | Linux/Windows |
| --- | --- | --- |
| 垂直分屏 | `Cmd+K, Cmd+Down` | `Ctrl+K, Ctrl+Down` |
| 水平分屏 | `Cmd+K, Cmd+Right` | `Ctrl+K, Ctrl+Right` |
| 关闭分屏 | `Cmd+K, Cmd+W` | `Ctrl+K, Ctrl+W` |
| 切换标签页 | `Cmd+Shift+[` / `]` | `Ctrl+Shift+[` / `]` |

---

## 4. 核心快捷键速查

> 以下为 Zed **默认键位**。所有快捷键都可通过 `keymap.json` 自定义。

### 4.1 全局操作

| 操作 | macOS | Linux/Windows |
| --- | --- | --- |
| 命令面板 | `Cmd+Shift+P` | `Ctrl+Shift+P` |
| 快速打开文件 | `Cmd+P` | `Ctrl+P` |
| 跳转到符号 | `Cmd+Shift+O` | `Ctrl+Shift+O` |
| 全局搜索 | `Cmd+Shift+F` | `Ctrl+Shift+F` |
| 打开设置 | `Cmd+,` | `Ctrl+,` |
| 切换终端 | `` Ctrl+` `` | `` Ctrl+` `` |
| 打开主题选择器 | `Cmd+K, Cmd+T` | `Ctrl+K, Ctrl+T` |
| 切换禅模式 | `Cmd+Ctrl+Z` | `F11` |

### 4.2 编辑器操作

| 操作 | macOS | Linux/Windows |
| --- | --- | --- |
| 保存 | `Cmd+S` | `Ctrl+S` |
| 多光标（向上/下） | `Cmd+Alt+Up/Down` | `Ctrl+Alt+Up/Down` |
| 选中下一个相同词 | `Cmd+D` | `Ctrl+D` |
| 跳转到定义 | `F12` | `F12` |
| 重命名符号 | `F2` | `F2` |
| 格式化文档 | `Cmd+Shift+I` | `Ctrl+Shift+I` |
| 注释切换 | `Cmd+/` | `Ctrl+/` |
| 代码补全 | `Ctrl+Space` | `Ctrl+Space` |

### 4.3 自定义快捷键

快捷键配置文件位于：

- **macOS**：`~/.config/zed/keymap.json`
- **Linux**：`~/.config/zed/keymap.json`
- **Windows**：`%APPDATA%\Zed\keymap.json`

示例——将 `Ctrl+N` 绑定为新建文件：

```json
{
  "context": "Workspace",
  "bindings": {
    "ctrl-n": "workspace::NewFile"
  }
}
```

---

## 5. 项目管理

### 5.1 打开项目

```bash
# 命令行直接打开
zed ~/projects/my-app

# 在新窗口中打开
zed -n ~/projects/my-app

# 等待文件关闭后再返回（适合 Git 编辑器场景）
zed --wait ~/.gitconfig
```

或在编辑器内按 `Cmd+O` / `Ctrl+O` 打开文件夹。

### 5.2 多项目工作区

Zed 支持同时打开多个项目根目录，所有项目共用一个窗口。通过 `File → Add Folder to Workspace` 添加更多文件夹。

### 5.3 标签页切换器

按 `Cmd+Shift+[` / `Cmd+Shift+]` （macOS）或 `Ctrl+Tab` （Linux/Windows）在最近使用的标签页间切换。

---

## 6. 代码编辑

### 6.1 语言支持

Zed 内置对以下语言的一流支持：**Rust、TypeScript、JavaScript、Python、Go、C、C++、JSON、HTML、CSS、YAML、TOML、Markdown** 等。

对于其他语言，通过扩展面板安装对应扩展即可。目前已支持 50+ 种编程语言，包括：Java、C#、PHP、Ruby、Swift、Kotlin、Scala、Elixir、Haskell、Lua、Zig、Dart、Vue、Svelte、Astro 等。

### 6.2 LSP（语言服务器协议）

Zed 通过 LSP 与各语言的 Language Server 通信，提供：

- **自动补全**：输入时弹出智能建议
- **诊断信息**：实时显示错误和警告
- **代码跳转**：跳转到定义、查找引用
- **悬停提示**：鼠标悬停显示类型和文档
- **代码操作**：快速修复、自动导入等

首次打开某种语言的项目时，Zed 会**自动下载**对应的 Language Server（如 `rust-analyzer`、`pyright`、`typescript-language-server` 等）。

### 6.3 多光标编辑

Zed 天然支持多光标：

| 操作 | macOS | Linux/Windows |
| --- | --- | --- |
| 添加上方光标 | `Cmd+Alt+Up` | `Ctrl+Alt+Up` |
| 添加下方光标 | `Cmd+Alt+Down` | `Ctrl+Alt+Down` |
| 选中下一个相同词 | `Cmd+D` | `Ctrl+D` |
| 选中所有相同词 | `Cmd+Shift+L` | `Ctrl+Shift+L` |

### 6.4 多缓冲区（Multibuffers）

在搜索结果或 Git diff 视图中，Zed 将多个文件的摘录合并到一个可编辑的多缓冲区中。你可以：

- 像编辑普通文件一样编辑所有摘录
- 使用多光标跨文件修改
- 所有更改实时反映到原文件

### 6.5 代码片段（Snippets）

Zed 支持代码片段，按 `Ctrl+Space` 触发补全后选择相应片段。你可以通过扩展安装更多片段，或自定义：

```json
{
  "snippets": {
    "Rust": {
      "fn": {
        "prefix": "fn",
        "body": ["fn ${1:name}(${2:params}) -> ${3:ReturnType} {", "    ${4:todo!()}", "}"],
        "description": "插入 Rust 函数模板"
      }
    }
  }
}
```

---

## 7. 终端

### 7.1 基本操作

| 操作 | macOS | Linux/Windows |
| --- | --- | --- |
| 切换终端面板 | `` Ctrl+` `` | `` Ctrl+` `` |
| 新建终端标签 | `Ctrl+~` | `Ctrl+~` |
| 清屏 | `Cmd+K` | `Ctrl+Shift+L` |

### 7.2 终端位置

终端可以停靠在**底部**（默认）、**左侧**或**右侧**。也可以作为**中心标签页**打开（`workspace: new center terminal`）。

### 7.3 终端配置

```json
{
  "terminal": {
    "shell": { "program": "/bin/zsh" },
    "working_directory": "current_project_directory",
    "font_family": "JetBrains Mono",
    "font_size": 14,
    "dock": "bottom",
    "env": { "EDITOR": "zed --wait" },
    "detect_venv": {
      "on": {
        "directories": [".venv", "venv"],
        "activate_script": "default"
      }
    }
  }
}
```

### 7.4 Python 虚拟环境自动检测

Zed 默认扫描 `.env`、`env`、`.venv`、`venv` 目录，打开终端时自动激活虚拟环境。

### 7.5 路径超链接

终端输出中的文件路径（如 `src/main.rs:42`）会自动转换为可点击的超链接。`Cmd+Click`（macOS）或 `Ctrl+Click`（Linux/Windows）可直接跳转到对应位置。

---

## 8. Git 集成

### 8.1 Git 面板

打开方式：

- 点击状态栏的 Git 图标
- 或命令面板搜索 `git panel: toggle focus`

面板显示：当前分支、已更改文件、暂存区状态。

### 8.2 暂存与提交

**方式一：直接在 Git 面板中提交**

在面板底部的输入框写入提交信息，按 `Cmd+Enter` / `Ctrl+Enter` 提交。这会**自动暂存所有已跟踪文件的更改**。

**方式二：使用 Project Diff**

打开 Project Diff——命令面板 `git: diff` 或快捷键 `Ctrl+G, D`：

- `Cmd+Y` / `Alt+Y`：暂存当前 hunk 并跳到下一个
- `Cmd+Ctrl+Y` / `Ctrl+Space`：暂存全部
- `Cmd+Enter` / `Ctrl+Enter`：提交

### 8.3 分支管理

| 操作 | 说明 |
| --- | --- |
| 创建分支 | 命令面板 `git: branch` |
| 切换分支 | 命令面板 `git: switch` |
| 删除分支 | 在分支选择器中找到目标分支，选择删除选项 |

### 8.4 Git Worktrees

Zed 原生支持 Git Worktrees，可在**标题栏的项目选择器旁**打开 Worktree 选择器：

- 创建新的 linked worktree（从当前分支或默认分支）
- 切换到已有 worktree
- 在新窗口中打开 worktree
- 删除不再需要的 worktree

### 8.5 AI 生成提交信息

在 Git 面板的提交信息输入框内，点击左下角铅笔图标或使用 `Alt+Tab` / `Alt+L`，Zed 会自动分析暂存的更改并生成提交信息。

```json
{
  "agent": {
    "commit_message_model": { "provider": "anthropic", "model": "claude-4-5-haiku" },
    "commit_message_instructions": "使用 Conventional Commits 格式：<type>(<scope>): <description>。"
  }
}
```

### 8.6 撤销提交

提交后，Git 面板会显示最近的提交记录，点击 **Uncommit** 按钮即可撤销（执行 `git reset HEAD~ --soft`）。

---

## 9. AI 功能

Zed 的 AI 体系由三个层面构成：**Agent 路径**、**模型访问**、**AI 功能**。

### 9.1 Agent 路径

| 路径 | 说明 |
| --- | --- |
| **Zed Agent** | Zed 原生代理，可使用内置工具、Skills、Instructions 和 MCP 服务器 |
| **External Agents** | 通过 ACP 协议集成外部代理进程 |
| **Terminal Threads** | 在 Zed 侧栏中直接运行 Claude Code、Amp 等终端代理 |

### 9.2 模型访问方式

| 方式 | 说明 |
| --- | --- |
| **Zed 托管模型** | 直接订阅 Zed 的 AI 计划，无需自带 API Key |
| **自带 API Key** | 使用 Anthropic、OpenAI、Google 等提供商的 API Key |
| **已有订阅** | 使用 GitHub Copilot 等现有订阅 |
| **网关** | 通过自定义网关访问模型 |
| **本地模型** | 使用 Ollama 等本地运行的模型 |

### 9.3 核心 AI 功能

#### Agent Panel（代理面板）

按 `Cmd+Shift+A` / `Ctrl+Shift+A` 打开。你可以：

- 用自然语言描述需求，Agent 会查找相关文件、编写/修改代码
- 使用 `@` 符号添加上下文（文件、符号、终端输出等）
- 实时查看 Agent 的操作进度
- 审查每一步更改，选择接受或拒绝

#### Inline Assistant（内联助手）

选中代码后按 `Cmd+Enter` / `Ctrl+Enter`，输入指令即可用 AI 转换选中的代码。例如：

- "用 async/await 重写这段代码"
- "添加输入验证"
- "翻译注释为英文"

#### Edit Prediction（编辑预测）

在输入时，Zed 会自动预测你接下来要写的代码，以灰色虚影显示。按 `Tab` 接受，按 `Esc` 忽略。

这是一种比 Copilot 式补全更激进的预测——它可能预测多行更改，甚至是跨文件的修改。

#### Parallel Agents（并行代理）

你可以在 **Threads Sidebar** 中同时运行多个 Agent 线程，每个线程可以：

- 使用不同的 Agent/模型
- 处理不同的项目
- 并行推进多个任务

### 9.4 AI 配置示例

```json
{
  "agent": {
    "default_model": {
      "provider": "anthropic",
      "model": "claude-sonnet-4-20250514"
    },
    "commit_message_model": {
      "provider": "anthropic",
      "model": "claude-4-5-haiku"
    }
  }
}
```

### 9.5 Skills 和 Instructions

- **Skills**：为 Agent 定义特定领域的知识和操作流程，存放在 `~/.config/zed/skills/` 目录
- **Instructions**：在 `~/.config/zed/AGENTS.md` 或项目根目录的 `AGENTS.md` 中编写全局指令
- **MCP 服务器**：通过 Model Context Protocol 扩展 Agent 的工具能力

---

## 10. 调试器

Zed 通过 DAP（Debug Adapter Protocol）支持多种语言的调试。

### 10.1 支持的语言

| 语言 | 适配器 | 提供方式 |
| --- | --- | --- |
| Rust | CodeLLDB / cppvsdbg | 内置 |
| C/C++ | CodeLLDB / cppvsdbg | 内置 |
| Go | Delve | 内置 |
| Python | Debugpy | 内置 |
| JavaScript/TypeScript | Node.js / Chrome | 内置 |
| Java | 扩展提供 | 扩展 |
| Ruby | 扩展提供 | 扩展 |
| Swift | 扩展提供 | 扩展 |

### 10.2 快速开始

1. 按 `F4` 打开**新进程调试窗口**
2. Zed 会自动生成当前项目可用的调试配置（基于测试、入口函数等）
3. 选择配置，点击启动
4. 在行号左侧点击设置断点
5. 使用调试工具栏进行单步执行、查看变量等

### 10.3 自定义调试配置

在项目根目录创建 `.zed/debug.json`：

```json
[
  {
    "label": "Launch Program",
    "adapter": "CodeLLDB",
    "request": "launch",
    "program": "${workspaceRoot}/target/debug/my-app",
    "cwd": "$ZED_WORKTREE_ROOT",
    "build": {
      "command": "cargo",
      "args": ["build"]
    }
  }
]
```

### 10.4 断点类型

右键点击断点可以设置：

- **日志断点**：命中时输出自定义日志而不暂停
- **条件断点**：满足条件时才暂停
- **命中计数**：命中指定次数后才暂停
- **禁用断点**：保留标记但暂时不触发

---

## 11. Task 任务系统

Zed 的 Task 系统让你可以定义和运行 Shell 命令，与编辑器和终端深度集成。

### 11.1 定义任务

任务可以在三个地方定义：

| 位置 | 文件路径 | 作用范围 |
| --- | --- | --- |
| 全局 | `~/.config/zed/tasks.json` | 所有项目可用 |
| 项目本地 | `.zed/tasks.json` | 仅当前项目 |
| 即时任务 | 通过 `task: spawn` 临时输入 | 当前会话 |

示例任务：

```json
[
  {
    "label": "运行测试",
    "command": "cargo test",
    "args": ["--", "--nocapture"],
    "use_new_terminal": false,
    "allow_concurrent_runs": false,
    "reveal": "always",
    "hide": "on_success"
  }
]
```

### 11.2 任务变量

| 变量 | 说明 |
| --- | --- |
| `$ZED_FILE` | 当前文件的绝对路径 |
| `$ZED_FILENAME` | 当前文件名（如 `main.rs`） |
| `$ZED_RELATIVE_FILE` | 相对于项目根目录的路径 |
| `$ZED_DIRNAME` | 当前文件所在目录 |
| `$ZED_STEM` | 文件名去除扩展名 |
| `$ZED_SYMBOL` | 当前选中的符号 |
| `$ZED_SELECTED_TEXT` | 当前选中的文本 |
| `$ZED_WORKTREE_ROOT` | 项目根目录路径 |
| `$ZED_LANGUAGE` | 当前文件的语言 |

### 11.3 触发任务

- `task: spawn`：打开任务选择器，列出所有可用任务
- `task: rerun`：重新运行最近执行的任务
- 自定义快捷键绑定特定任务：

```json
{
  "context": "Workspace",
  "bindings": {
    "alt-t": ["task::Spawn", { "task_name": "运行测试" }]
  }
}
```

### 11.4 任务钩子

任务可以绑定到事件自动触发。当前支持的钩子：

- `create_worktree`：创建新的 Git Worktree 后自动运行（适合复制 `.env` 等设置文件）

---

## 12. Vim / Helix 模式

### 12.1 启用 Vim 模式

首次启动时，欢迎页面可直接勾选启用。也可随时通过命令面板：

```
workspace: toggle vim mode
```

或在 `settings.json` 中：

```json
{ "vim_mode": true }
```

### 12.2 Vim 模式设计理念

Zed 的 Vim 模式不是 Vim 的 1:1 复制，而是将 Vim 的模态设计与 Zed 的现代特性**深度融合**：

- **动作用 Tree-sitter 语义解析**：`%` 匹配括号时能正确理解 Rust 的 `|`、JavaScript 的 `$` 等
- **可视块选择用多光标实现**：插入内容在每一行实时更新
- **宏使用 Zed 录制系统**：可捕捉自动补全等复杂操作
- **搜索使用 Zed 的搜索系统**：正则语法略有差异

### 12.3 Zed 专属 Vim 快捷键

#### LSP 导航

| 快捷键 | 功能 |
| --- | --- |
| `g d` | 跳转到定义 |
| `g I` | 跳转到实现 |
| `g A` | 查找所有引用 |
| `c d` | 重命名符号 |
| `g s` | 文件内符号搜索 |
| `g S` | 项目内符号搜索 |
| `g .` | 打开代码操作菜单 |

#### Tree-sitter 文本对象

| 文本对象 | 说明 |
| --- | --- |
| `a c` / `i c` | 类/定义（外/内） |
| `a f` / `i f` | 函数/方法（外/内） |
| `a t` / `i t` | HTML 标签（外/内） |
| `i i` | 当前缩进层级 |

#### 多光标

| 快捷键 | 功能 |
| --- | --- |
| `g l` | 选中下一个相同词 |
| `g A` | 在选中行的末尾添加光标 |
| `g a` | 为每个相同词添加选区 |

#### Git

| 快捷键 | 功能 |
| --- | --- |
| `] c` / `[ c` | 跳转到下/上一个 Git 更改 |
| `d o` | 展开差异 hunk |
| `d u` | 暂存当前 hunk 并跳到下一个 |

### 12.4 Ex 命令

在 Normal 模式下按 `:` 打开命令面板，支持 Vim 风格的 Ex 命令：

- `:w` / `:write` → 保存
- `:q` / `:quit` → 关闭
- `:wq` → 保存并关闭
- `:E` / `:Explore` → 打开项目面板
- `:G` / `:Git` → 打开 Git 面板
- `:te` / `:term` → 打开终端
- `:A` / `:AI` → 打开 AI 面板

### 12.5 Helix 模式

如果你来自 Helix 编辑器，Zed 也提供了 Helix 模式：

```json
{ "helix_mode": true }
```

这会将键位映射为选择→动作（Select → Action）模式。

---

## 13. 多人协作

### 13.1 概述

Zed 支持**实时多人编辑**——多人同时编辑同一项目，彼此可见光标和修改。按 `Cmd+Shift+C` / `Ctrl+Shift+C` 打开协作面板。

### 13.2 协作方式

| 方式 | 说明 |
| --- | --- |
| **Channels（频道）** | 持久的项目房间，适合团队协作，支持语音通话 |
| **Contacts（联系人）** | 联系人列表，发起临时的私密会话 |

> **注意**：分享项目意味着协作者可以访问你本地文件系统内该项目的内容。只与你信任的人协作。

### 13.3 语音设置

Zed 支持在协作会话中进行语音通话，可在设置中选择输入/输出音频设备：

```json
{
  "audio": {
    "experimental.output_audio_device": "Device Name (device-id)",
    "experimental.input_audio_device": "Device Name (device-id)"
  }
}
```

---

## 14. 远程开发

### 14.1 原理

Zed 的远程开发通过 SSH 连接：本地机器运行 Zed UI，远程服务器运行 Zed Headless Server。源代码、Language Server、任务和终端都在远程服务器上运行，而 UI 保持本地流畅。

### 14.2 连接步骤

1. 确保本地和远程都已安装最新版 Zed（≥ v0.159）
2. 按 `Cmd+Ctrl+Shift+O` / `Alt+Ctrl+Shift+O` 打开 **Remote Projects** 对话框
3. 点击 **Connect New Server**，输入 SSH 连接命令（如 `user@192.168.1.10`）
4. Zed 通过 SSH 连接到服务器，自动下载并启动远程服务端
5. 选择要打开的远程项目路径

### 14.3 命令行方式

```bash
# 直接打开远程项目
zed ssh://user@host:port/path/to/project

# SCP 风格路径
zed ssh://user@host:~/project
zed ssh://user@host:/absolute/path
```

### 14.4 配置示例

```json
{
  "ssh_connections": [
    {
      "host": "192.168.1.10",
      "username": "dev",
      "port": 22,
      "args": ["-i", "~/.ssh/work_id_file"],
      "nickname": "开发服务器",
      "projects": [{ "paths": ["~/code/my-app"] }],
      "port_forwards": [
        { "local_port": 8080, "remote_port": 80 }
      ]
    }
  ]
}
```

### 14.5 WSL 支持（Windows）

Zed 原生支持在 WSL 中打开项目：

- `projects: open in wsl`：选择本地文件夹在 WSL 中打开
- `projects: open wsl`：打开 WSL 中已有的文件夹

---

## 15. 扩展生态

### 15.1 安装扩展

按 `Cmd+Shift+X` / `Ctrl+Shift+X` 打开扩展面板，搜索并安装。热门扩展：

| 扩展名 | 下载量 | 说明 |
| --- | --- | --- |
| HTML | 5.7M | HTML 语法支持 |
| Git Firefly | 1.2M | Git 语法高亮 |
| Java | 947k | Java 语言支持 |
| Catppuccin | 913k | 主题 |
| Dockerfile | 812k | Dockerfile 支持 |
| SQL | 603k | SQL 语言支持 |
| PHP | 593k | PHP 支持 |
| Vue | 526k | Vue 支持 |
| Ruby | 476k | Ruby 支持 |
| OpenCode | 427k | 开源编码代理 |

### 15.2 扩展类型

| 类型 | 说明 |
| --- | --- |
| **语言扩展** | 为特定语言提供语法高亮、LSP 集成、代码片段 |
| **主题扩展** | 自定义配色方案 |
| **图标主题** | 自定义文件图标 |
| **调试器扩展** | 为特定语言添加 DAP 调试适配器 |
| **MCP 服务器扩展** | 扩展 Agent 的工具能力 |
| **Agent 服务器扩展** | 集成外部 AI 代理 |

---

## 16. 配置与个性化

### 16.1 设置文件位置

| 平台 | 路径 |
| --- | --- |
| macOS | `~/.config/zed/settings.json` |
| Linux | `~/.config/zed/settings.json` |
| Windows | `%APPDATA%\Zed\settings.json` |

### 16.2 主题与外观

**切换主题**：`Cmd+K, Cmd+T` / `Ctrl+K, Ctrl+T`

常用配置：

```json
{
  "theme": "One Dark",
  "buffer_font_family": "JetBrains Mono",
  "buffer_font_size": 14,
  "ui_font_family": "Inter",
  "ui_font_size": 13,
  "format_on_save": "on",
  "tab_size": 4,
  "soft_wrap": "editor_width",
  "cursor_blink": true,
  "relative_line_numbers": true,
  "indent_guides": { "enabled": true }
}
```

### 16.3 语言特定配置

```json
{
  "languages": {
    "Rust": {
      "tab_size": 4,
      "format_on_save": "on",
      "formatter": { "external": { "command": "rustfmt" } }
    },
    "Python": {
      "tab_size": 4,
      "format_on_save": "on",
      "formatter": { "external": { "command": "black" } },
      "language_servers": ["pyright"]
    },
    "TypeScript": {
      "tab_size": 2,
      "format_on_save": "on",
      "formatter": { "external": { "command": "prettier" } }
    }
  }
}
```

### 16.4 终端配置

```json
{
  "terminal": {
    "font_family": "JetBrains Mono",
    "font_size": 14,
    "cursor_shape": "bar",
    "blinking": "on",
    "dock": "bottom",
    "copy_on_select": true,
    "working_directory": "current_project_directory",
    "env": { "EDITOR": "zed --wait" }
  }
}
```

### 16.5 导入 VS Code 设置

从命令面板运行 `migrate: import VS Code settings`，Zed 可自动导入 VS Code 的快捷键映射和常用设置。

---

## 17. 定价与许可

### 17.1 编辑器本身

Zed 编辑器**完全免费且开源**（GitHub 上以 GPL 协议发布）。

### 17.2 AI 功能

AI 功能分多个层级：

- **Free 层级**：有限的 AI 使用额度，适合轻度使用
- **Pro 层级**：更多 AI 额度，解锁高级模型访问
- **Business 层级**：团队管理功能、集中计费和管控

具体价格请访问 [zed.dev/pricing](https://zed.dev/pricing) 查看最新信息。

### 17.3 使用自有 API Key

你也可以**不购买 Zed 的 AI 计划**，直接配置自己的 API Key：

```json
{
  "agent": {
    "default_model": {
      "provider": "anthropic",
      "model": "claude-sonnet-4-20250514"
    }
  },
  "providers": {
    "anthropic": {
      "api_key": "sk-ant-..."
    }
  }
}
```

---

## 18. 常见问题与技巧

### 18.1 如何将 Zed 设为 Git 默认编辑器？

```bash
git config --global core.editor "zed --wait"
```

### 18.2 如何安装命令行工具 `zed`？

- **macOS**：在 Zed 中运行命令面板 `zed: install cli`
- **Linux**：安装脚本会自动配置
- **Windows**：安装程序会自动添加到 PATH

### 18.3 如何查看 Zed 日志？

命令面板搜索 `Open Log` 或 `Cmd+Shift+P, open log`。

日志文件位置：

- macOS：`~/Library/Logs/Zed/Zed.log`
- Linux：`~/.local/share/zed/logs/`
- Windows：`%LOCALAPPDATA%\Zed\logs\`

### 18.4 如何处理 Language Server 不工作？

1. 检查命令面板中 `language server: restart` 重启对应 Language Server
2. 确认系统已安装对应的 Language Server 工具（如 Node.js 环境）
3. 查看日志排查具体错误

### 18.5 如何贡献代码？

Zed 是开源项目，欢迎贡献：

- [GitHub 仓库](https://github.com/zed-industries/zed)
- [Discord 社区](https://discord.com/invite/zedindustries)
- [GitHub Discussions](https://github.com/zed-industries/zed/discussions)

### 18.6 提效小技巧

- **禅模式**：`Cmd+Ctrl+Z` (macOS) / `F11` (Linux/Windows) 进入全屏无干扰模式
- **即时任务**：`task: spawn` 输入任意 Shell 命令即时运行
- **命令面板即一切**：忘记快捷键时，`Cmd+Shift+P` 搜索即可
- **多光标 + Vim 模式**：在 Vim 模式下使用 `g l` / `g L` 快速添加多光标
- **保存时自动格式化**：设置 `"format_on_save": "on"`
- **路径跳转**：终端输出中的 `文件:行号` 可以 `Ctrl+Click` 直接跳转

---

## 参考资源

- [官方文档](https://zed.dev/docs)
- [GitHub 仓库](https://github.com/zed-industries/zed)
- [下载页面](https://zed.dev/download)
- [定价页面](https://zed.dev/pricing)
- [路线图](https://zed.dev/roadmap)
- [扩展市场](https://zed.dev/extensions)
- [Discord 社区](https://discord.com/invite/zedindustries)
- [Reddit 社区](https://www.reddit.com/r/ZedEditor)

---

> **最后更新：2026 年 6 月 13 日**
>
> 本教程基于 Zed 官方文档编写。Zed 更新频繁（每周发布），部分细节可能有所变化，建议以[官方文档](https://zed.dev/docs)为准。
