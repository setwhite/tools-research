# Lazygit 终端 Git 客户端完全教程

> Lazygit 是一个用 Go 编写的**终端 Git 客户端**，让你通过直观的 TUI（终端界面）操作 Git，告别记忆晦涩命令的痛苦。  
> 作者：Jesse Duffield · GitHub: [jesseduffield/lazygit](https://github.com/jesseduffield/lazygit) · Stars: 79k+ · License: MIT

---

## 目录

1. [什么是 Lazygit](#1-什么是-lazygit)
2. [核心特性](#2-核心特性)
3. [安装](#3-安装)
4. [快速上手：5 分钟入门](#4-快速上手5-分钟入门)
5. [完整快捷键参考](#5-完整快捷键参考)
6. [进阶功能详解](#6-进阶功能详解)
7. [配置指南](#7-配置指南)
8. [编辑器集成](#8-编辑器集成)
9. [竞品对比](#9-竞品对比)
10. [参考资源](#10-参考资源)

---

## 1. 什么是 Lazygit

Lazygit 是一个**终端界面的 Git 客户端**，让你通过按键操作来完成 Git 的各种操作——暂存、提交、分支、合并、变基（rebase）、挑拣（cherry-pick）、二分查找（bisect）等——而无需记忆繁琐的命令行参数。

作者 Jesse Duffield 的吐槽很有代表性：

> Git 很强大，但当你每天被它折磨时，这种强大有什么用？交互式 rebase 要编辑一个 TODO 文件？**开什么玩笑？** 要暂存部分文件得用命令行逐 hunk 操作，如果某个 hunk 不能再拆分但里面有你不想暂存的代码，还得手动编辑一个晦涩的 patch 文件？**你他妈在逗我？！**

Lazygit 的目标就是解决这些痛点。它不是一个 Git 的替代品，而是 Git 的一个**更友好的操作界面**——你仍然在使用 Git，只是不再需要记住命令。

---

## 2. 核心特性

| 特性 | 说明 |
|------|------|
| **逐行暂存** | 在文件内按行或范围暂存，可视化操作 |
| **交互式 Rebase** | 移动、压缩、删除、编辑 commit，无需编辑 TODO 文件 |
| **Cherry-pick** | 选中一个 commit，粘贴到当前分支 |
| **Bisect** | 可视化标记 good/bad commit 进行二分查找 |
| **Undo/Redo** | 通过 reflog 撤销/重做操作（commit 和分支级别） |
| **Commit Graph** | 可视化 commit 历史图，按作者着色 |
| **Worktree** | 创建工作树，同时处理多个分支 |
| **自定义 Patch** | 从旧 commit 提取变更，拆分或移除 |
| **自定义命令** | 扩展任意 Git 命令到按键绑定 |
| **过滤/搜索** | 在任何面板中按 `/` 过滤 |
| **GitHub PR 集成** | 显示分支关联的 PR 状态，一键在浏览器打开 |
| **Diff 对比** | 对比两个 commit 的差异 |
| **Stash 管理** | 可视化 apply、pop、drop stash |

---

## 3. 安装

### macOS

```bash
# Homebrew（推荐）
brew install lazygit

# MacPorts
sudo port install lazygit
```

### Windows

```powershell
# Scoop（推荐）
scoop bucket add extras
scoop install lazygit

# Chocolatey
choco install lazygit

# Winget
winget install lazygit
```

### Linux

```bash
# Arch Linux
sudo pacman -S lazygit

# Fedora / RHEL
sudo dnf install lazygit

# Ubuntu / Debian（从 GitHub Releases 下载）
# NixOS
nix-env -iA nixpkgs.lazygit

# 通用方法（Go）
go install github.com/jesseduffield/lazygit@latest
```

### 验证安装

```bash
lazygit --version
lazygit --help        # 查看所有选项
```

---

## 4. 快速上手：5 分钟入门

### 启动

进入任意 Git 仓库，直接运行：

```bash
cd your-project
lazygit
```

### 界面布局（5 个面板）

```
┌──────────────────────────────────────┐
│  Status（状态）   │   Files（文件）   │
│  当前分支/最近操作  │  暂存/未暂存变更  │
├──────────────────┼──────────────────┤
│  Branches（分支） │  Commits（提交）  │
│  本地/远端/标签   │  提交历史 + 图    │
├──────────────────┼──────────────────┤
│  Stash（暂存区）  │                  │
│  stash 列表       │                  │
└──────────────────┴──────────────────┘
```

- 按 `Tab` / `Shift+Tab` 在面板之间切换
- 按数字键 `1`–`5` 直接跳转到对应面板
- 按 `?` 随时查看当前面板的快捷键

### 第一个工作流

**Step 1：暂存文件**

进入 **Files** 面板（按 `2`），你会看到变更的文件列表。

| 按键 | 作用 |
|------|------|
| `Space` | 暂存/取消暂存文件 |
| `a` | 暂存/取消暂存全部 |
| `Enter` | 打开文件逐行查看 |

**Step 2：提交**

| 按键 | 作用 |
|------|------|
| `c` | 提交暂存的变更（输入提交信息） |
| `w` | 提交但不触发 pre-commit hooks |
| `A` | 将暂存变更追加到上次提交 |

**Step 3：推送**

| 按键 | 作用 |
|------|------|
| `P` | 推送到远端（Shift+P） |
| `p` | 拉取远端更新 |

### 逐行暂存

在 Files 面板选中文件按 `Enter`，进入 hunk 视图：

| 按键 | 作用 |
|------|------|
| `Space` | 暂存/取消暂存当前行 |
| `v` | 进入可视模式选择多行 |
| `a` | 暂存/取消暂存整个 hunk |
| `E` | 在编辑器中编辑 hunk |

---

## 5. 完整快捷键参考

### 全局快捷键

| 按键 | 作用 |
|------|------|
| `?` | 查看当前面板快捷键 |
| `q` | 退出 |
| `Tab` / `Shift+Tab` | 下一个/上一个面板 |
| `1`–`5` | 跳转到对应面板 |
| `Ctrl+z` | 撤销上次操作 |
| `Ctrl+y` | 重做 |
| `@` | 打开命令日志 |
| `+` | 放大 diff 视图 |
| `_` | 缩小 diff 视图 |
| `/` | 过滤当前列表 |

### Files 面板

| 按键 | 作用 |
|------|------|
| `Space` | 暂存/取消暂存文件 |
| `a` | 暂存/取消暂存全部 |
| `Enter` | 进入 hunk 视图 |
| `d` | 丢弃变更 |
| `e` | 在编辑器中打开 |
| `c` | 提交暂存变更 |
| `w` | 跳过 pre-commit hooks 提交 |
| `A` | 追加到上次提交 |
| `s` | 暂存（stash）变更 |
| `S` | Stash 选项菜单 |

### Branches 面板

| 按键 | 作用 |
|------|------|
| `Space` | 切换到选中分支 |
| `n` | 创建新分支 |
| `d` | 删除分支 |
| `D` | 强制删除 |
| `r` | 将当前分支 rebase 到选中分支 |
| `M` | 将选中分支合并到当前分支 |
| `f` | 强制切换（丢弃本地变更） |
| `u` | 设置 upstream |
| `p` | 拉取 |
| `P` | 推送 |
| `w` | 从选中分支创建工作树 |

### Commits 面板

| 按键 | 作用 |
|------|------|
| `s` | 合并（squash）到上一个 commit |
| `f` | Fixup（合并并丢弃消息） |
| `r` | 重命名 commit 消息 |
| `d` | 删除 commit |
| `e` | 编辑 commit（rebase 暂停于此） |
| `i` | 开始交互式 rebase |
| `Ctrl+k` / `Ctrl+j` | 上移/下移 commit |
| `Shift+c` | 标记 commit 用于 cherry-pick |
| `Shift+v` | 粘贴（cherry-pick）标记的 commits |
| `Shift+a` | 用暂存变更修改此 commit |
| `b` | Bisect 选项 |
| `c` | 复制 commit SHA |
| `Shift+w` | 对比两个 commits |

### Stash 面板

| 按键 | 作用 |
|------|------|
| `Space` | 应用 stash（保留） |
| `g` | Pop stash（应用并删除） |
| `d` | 删除 stash |
| `n` | 从 stash 创建新分支 |

---

## 6. 进阶功能详解

### 6.1 交互式 Rebase

这是 Lazygit 最强大的功能之一。选中一个 commit 按 `i` 开始交互式 rebase，然后可以对任意 commit：

- `s` — Squash（合并到上一个 commit）
- `f` — Fixup（合并，丢弃消息）
- `d` — Drop（删除）
- `e` — Edit（编辑，rebase 在此暂停）
- `Ctrl+k` / `Ctrl+j` — 上移/下移

你也可以在不显式启动 rebase 的情况下执行以上操作 —— 对某个 commit 按 `s` 就会自动 squash 并触发 rebase。

### 6.2 Cherry-pick

1. 在 Commits 面板选中一个 commit，按 `Shift+c` 复制
2. 切换到目标分支
3. 按 `Shift+v` 粘贴（cherry-pick）

### 6.3 Bisect（二分查找）

1. 在 Commits 面板按 `b` 进入 bisect 模式
2. 按 `b` 标记当前 commit 为 good
3. 按 `b` 标记另一个 commit 为 bad
4. Lazygit 自动二分，逐个标记 good/bad
5. 找到罪魁祸首的 commit

### 6.4 Undo / Redo

Lazygit 通过 reflog 跟踪每次操作。你可以撤销大多数操作而不会丢失工作。

- `Ctrl+z` — 撤销
- `Ctrl+y` — 重做

注意：Undo 基于 reflog，只对 commit 和分支级别的操作有效，不能撤销工作树或 stash 的变更。

### 6.5 自定义 Patch（Rebase Magic）

从旧 commit 中提取特定变更：

1. 在 Commits 面板中选中 commit，按 `Enter` 查看文件
2. 选中文件按 `Enter` 聚焦 patch
3. 按 `Space` 将某行添加到自定义 patch
4. 按 `Ctrl+p` 查看 patch 选项
5. 选择从当前 commit 中移除 patch，或拆分为新 commit

### 6.6 Worktree（工作树）

在 Branches 面板中选中分支按 `w`，可以为其创建 worktree。这让你无需 stash 或创建 WIP commit 就能同时在多个分支上工作。

### 6.7 Commit Graph

按 `+` 放大视图，commit graph 会显示出来。颜色对应 commit 作者，导航时父 commit 会被高亮。

### 6.8 对比两个 Commits

1. 在一个 commit 上按 `Shift+w`
2. 选择标记方式
3. 导航到另一个 commit
4. 主视图显示两者的 diff
5. 按 `Shift+w` 查看 diff 菜单（反转方向、退出对比等）

### 6.9 GitHub PR 集成

在 Branches 面板中，有 GitHub PR 的分支旁边会显示图标；颜色表示 PR 状态（开放、已合并等）。按 `Shift+G` 在浏览器中打开 PR。

需要安装 `gh` CLI 并执行 `gh auth login`。

---

## 7. 配置指南

配置文件位置：

- **Linux/macOS**: `~/.config/lazygit/config.yml`
- **Windows**: `%APPDATA%\lazygit\config.yml`

### 基础配置示例

```yaml
# ~/.config/lazygit/config.yml
gui:
  theme:
    lightTheme: false
    activeBorderColor:
      - green
      - bold
    inactiveBorderColor:
      - white
    searchingActiveBorderColor:
      - yellow
      - bold
    optionsTextColor: blue
    selectedLineBgColor:
      - default
    cherryPickedCommitFgColor: cyan
    cherryPickedCommitBgColor:
      - default
    markedBaseCommitFgColor: blue
  authorColors:
    '*': '#b8c0e0'
  nerdFontsVersion: "3"   # 启用 Nerd Font 图标
  showFileIcons: true
  showNumstat: false      # 显示 +/- 统计
  showCommitHash: true
  commandLogSize: 8

git:
  paging:
    colorArg: always
    pager: delta           # 使用 delta 作为 diff 分页器
  merging:
    args: "--no-ff"       # merge 默认参数
  commitPrefixes:
    main: "[main] "
    feat/*: "[feat] "
    fix/*: "[fix] "

os:
  editPreset: "nvim"      # 编辑器预设
  openDirLinkCmd: "code ." # 在编辑器中打开目录
```

### 关键配置项

| 配置 | 说明 |
|------|------|
| `gui.language` | 界面语言（支持中文等） |
| `gui.nerdFontsVersion` | Nerd Font 版本，设置为 "3" 获得更好图标 |
| `git.paging.pager` | 自定义 diff 分页器（推荐 `delta`） |
| `os.editPreset` | 外部编辑器预设（vim/neovim/emacs/sublime/vscode） |
| `gui.skipUnstageLineWarning` | 跳过取消暂存行警告 |
| `confirmOnQuit` | 退出时确认 |

### 自定义命令

Lazygit 支持通过配置扩展任意 Git 命令：

```yaml
customCommands:
  - key: "<c-r>"
    command: git reset --soft HEAD~1
    description: "Soft reset to previous commit"
    context: "commits"
  - key: "<c-f>"
    command: git fetch --prune
    description: "Fetch and prune"
    context: "global"
```

每个自定义命令有四个属性：`key`（快捷键）、`command`（执行的命令）、`description`（描述）、`context`（生效面板）。

### 自定义 Pager（推荐 delta）

安装 [delta](https://github.com/dandavison/delta) 后，配置：

```yaml
git:
  paging:
    colorArg: always
    pager: delta --dark
```

---

## 8. 编辑器集成

### Neovim / Vim

```lua
-- lazy.nvim
{ "kdheepak/lazygit.nvim" }
```

快捷键映射：

```lua
vim.keymap.set("n", "<leader>lg", ":LazyGit<CR>", { silent = true })
```

### VS Code

搜索并安装 **Lazygit** 插件（作者：maiastewart），然后 `Ctrl+Shift+P` → `Lazygit: Open`。

### Tmux

在 tmux 中打开 Lazygit 的推荐方式（带浮动窗口）：

```bash
# ~/.tmux.conf
bind-key g run-shell "tmux neww lazygit"
```

---

## 9. 竞品对比

| 特性 | Lazygit | GitUI | Tig | gitui |
|------|---------|-------|-----|-------|
| **语言** | Go | Rust | C | Rust |
| **Stars** | 79k+ | 18k+ | 12k+ | 2k+ |
| **交互式 Rebase** | ✅ 优秀 | ✅ 好 | ✅ 基础 | ❌ |
| **逐行暂存** | ✅ 可视化 | ✅ 可视化 | ❌ | ✅ |
| **Undo/Redo** | ✅ 基于 reflog | ❌ | ❌ | ❌ |
| **Cherry-pick** | ✅ 一键 | ✅ | ✅ | ❌ |
| **Commit Graph** | ✅ | ✅ | ✅ | ❌ |
| **Stash 管理** | ✅ 可视化 | ✅ | ✅ | ❌ |
| **自定义命令** | ✅ | ❌ | ❌ | ❌ |
| **Worktree** | ✅ | ❌ | ❌ | ❌ |
| **GitHub PR 集成** | ✅ | ❌ | ❌ | ❌ |
| **配置难度** | 低 | 中 | 高 | 低 |

**选择建议：**

- **Lazygit** — 功能最全面，社区最大，适合大多数开发者
- **Tig** — 如果你已经熟悉 Vim 风格的按键，想要一个纯粹的 Git 浏览器
- **GitUI** — Rust 实现，性能稍好，功能接近但不支持 undo/worktree
- **gitui** — 极简，适合只需要基本操作的用户

---

## 10. 参考资源

| 资源 | 链接 |
|------|------|
| 官方文档 | https://lazygit.dev/docs/ |
| 完整快捷键 | https://lazygit.dev/keybindings/ |
| GitHub 仓库 | https://github.com/jesseduffield/lazygit |
| 配置指南 | https://lazygit.dev/docs/configuration/ |
| 视频教程（15 个特性） | https://youtu.be/CPLdltN7wgE |
| 基础教程 | https://youtu.be/VDXvbHZYeKY |
| Rebase Magic 教程 | https://youtu.be/4XaToVut_hs |
| 下载页面 | https://lazygit.dev/download/ |

---

> Lazygit 不是 Git 的替代品，它是 Git 的一个**更友好的操作界面**。你仍然在使用 Git 的所有功能，只是不再需要在终端里背那些命令。
>
> 开始方式：进入一个 Git 仓库，键入 `lazygit`。按 `?` 查看所有快捷键。5 分钟后你就会发现——你再也回不去了。
