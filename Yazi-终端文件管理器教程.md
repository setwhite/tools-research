# Yazi 终端文件管理器完全教程

> Yazi（发音 /jɑːzi/，意为"鸭子"）是一款用 Rust 编写的、基于非阻塞异步 I/O 的**超快终端文件管理器**。  
> GitHub: [sxyazi/yazi](https://github.com/sxyazi/yazi) · Stars: 39k+ · License: MIT

---

## 目录

1. [什么是 Yazi](#1-什么是-yazi)
2. [核心特性](#2-核心特性)
3. [安装](#3-安装)
4. [快速上手](#4-快速上手)
5. [配置指南](#5-配置指南)
6. [图片预览](#6-图片预览)
7. [插件系统](#7-插件系统)
8. [实用技巧](#8-实用技巧)
9. [性能架构](#9-性能架构)
10. [常用插件与资源](#10-常用插件与资源)

---

## 1. 什么是 Yazi

Yazi 是一个**终端文件管理器**，定位类似于 `ranger`、`lf`、`nnn`，但具有压倒性的性能优势和现代化的交互体验。

对比竞品：

| 特性 | Yazi | ranger | lf | nnn |
|------|------|--------|-----|-----|
| 语言 | Rust | Python | Go | C |
| 异步 I/O | ✅ Tokio | ❌ | ❌ | ❌ |
| 图片预览 | ✅ 内置多种协议 | ✅ 需依赖 | ❌ | ✅ 有限 |
| 插件系统 | ✅ Lua 并发 | ✅ Python | ❌ | ❌ |
| 包管理器 | ✅ 内置 | ❌ | ❌ | ❌ |
| 多标签 | ✅ | ✅ | ❌ | ❌ |
| 任务调度 | ✅ 优先级 & 并发 | ❌ | ❌ | ❌ |

---

## 2. 核心特性

- **全异步支持**：所有 I/O 操作异步化，CPU 任务分散到多线程
- **强大的异步任务调度**：实时进度、取消、优先级分配
- **图片预览**：内置 Kitty、iTerm2、Sixel、Überzug++、Chafa 等多种协议
- **内置代码高亮和图片解码**：配合预加载机制大幅加速加载
- **并发插件系统**：UI 插件、功能插件、自定义预览器/预加载器，全部用 Lua 编写
- **虚拟文件系统**：远程文件管理、自定义搜索引擎
- **数据分发服务 (DDS)**：客户端-服务器架构，跨实例通信和状态持久化
- **包管理器**：一键安装/更新/锁定插件和主题
- 集成 ripgrep、fd、fzf、zoxide
- Vim 风格输入/选择/确认/帮助/通知组件，路径自动补全
- 多标签、跨目录选择、可滚动预览（视频/PDF/压缩包/代码/目录等）
- 批量重命名/创建、压缩包提取、可视模式、文件选择器、Git 集成
- 主题系统、鼠标支持、拖放、回收站、自定义布局、CSI u、OSC 52

---

## 3. 安装

### 前置依赖

- [`file`](https://github.com/file/file) — 文件类型检测（必需）
- [nerd-fonts](https://www.nerdfonts.com/)（推荐，用于图标显示）

### 可选增强工具

| 工具 | 用途 |
|------|------|
| `ffmpeg` | 视频缩略图 |
| `7-Zip` | 压缩包预览和提取（非 standalone 版本） |
| `jq` | JSON 预览 |
| `poppler` | PDF 预览 |
| `fd` | 文件搜索 |
| `rg` (ripgrep) | 文件内容搜索 |
| `fzf` (>= 0.53.0) | 快速文件子树导航 |
| `zoxide` | 历史目录导航（需 fzf） |
| `resvg` | SVG 预览 |
| ImageMagick (>= 7.1.1) | 字体、HEIC、JPEG XL 预览 |
| `xclip` / `wl-clipboard` / `xsel` | Linux 剪贴板支持 |

### Windows

**方案一：直接从 GitHub Releases 下载**

1. 从 [github.com/sxyazi/yazi/releases](https://github.com/sxyazi/yazi/releases) 下载最新 Windows 版本
2. 将 `yazi.exe` 所在目录加入 `PATH`
3. 设置 `YAZI_FILE_ONE` 环境变量指向 `file.exe` 路径
4. 推荐搭配 WezTerm 或 Windows Terminal (>= v1.22.10352.0) 使用以获得图片预览支持

**方案二：使用包管理器**

```powershell
# Scoop
scoop install yazi

# Winget
winget install yazi

# Chocolatey
choco install yazi
```

### macOS

```bash
# Homebrew
brew install yazi

# 可选增强工具
brew install ffmpeg sevenzip jq poppler fd ripgrep fzf zoxide resvg imagemagick
```

### Linux

```bash
# Arch Linux
sudo pacman -S yazi ffmpeg 7zip jq poppler fd ripgrep fzf zoxide resvg imagemagick

# Ubuntu / Debian（从源码或下载预编译包）
# Nix
nix-env -iA nixpkgs.yazi

# 通用方法：下载预编译二进制
```

### 验证安装

```bash
yazi --version
yazi --help        # 查看所有选项
yazi --debug       # 查看调试信息（终端类型、图片适配器等）
```

---

## 4. 快速上手

### 启动

```bash
yazi                    # 从当前目录启动
yazi /path/to/dir       # 从指定目录启动
yazi --cwd-file=/tmp/cwd  # 退出时将当前路径写入文件
```

### 界面布局

Yazi 默认三栏布局：

```
┌──────────────┬──────────────┬──────────────┐
│   父目录     │   当前目录    │    预览      │
│  (parent)   │  (current)   │  (preview)   │
│              │              │              │
│  ..          │  Documents/  │  文件预览     │
│  Downloads/  │  images/     │  图片/代码/   │
│  Music/      │  main.rs     │  PDF/视频     │
│              │  config.toml │              │
└──────────────┴──────────────┴──────────────┘
```

布局比例可通过配置调整，甚至可以隐藏某个面板（将其比例设为 0）。

### 最重要的快捷键（类 Vim）

#### 基本导航

| 键 | 作用 |
|----|------|
| `j` / `k` | 下/上移动 |
| `h` | 返回父目录 |
| `l` / `Enter` | 进入子目录 / 打开文件 |
| `g` 然后 `g` | 跳到顶部 |
| `G` | 跳到底部 |
| `Ctrl+d` / `Ctrl+u` | 下半页/上半页 |
| `[` / `]` | 上/下一个父目录兄弟 |

#### 文件操作

| 键 | 作用 |
|----|------|
| `y` | 复制（yank）文件 |
| `x` | 剪切文件 |
| `p` | 粘贴 |
| `d` | 删除（移入回收站） |
| `a` | 重命名 |
| `c` | 创建文件或目录（以 `/` 结尾创建目录） |
| `Space` | 切换选中状态 |
| `v` | 进入可视模式（批量选中） |
| `V` | 切换选择所有 |
| `o` | 打开文件（使用默认规则） |
| `O` | 交互式选择打开方式 |

#### 标签页

| 键 | 作用 |
|----|------|
| `t` | 创建新标签 |
| `1`~`9` | 切换到第 N 个标签 |
| `Tab` | 下一个标签 |
| `Shift+Tab` | 上一个标签 |
| `q` | 关闭当前标签（最后一标签则退出） |
| `w` | 关闭当前标签 |

#### 搜索和过滤

| 键 | 作用 |
|----|------|
| `/` | 搜索文件（支持 fd/rg 引擎） |
| `f` | 实时过滤文件名 |
| `n` / `N` | 下一个/上一个搜索结果 |
| `s` | 按某列排序 |
| `S` | 切换隐藏文件显示 |

#### 预览操作

| 键 | 作用 |
|----|------|
| `Tab` | 查看文件详细信息（spot） |
| `~` | 切换预览面板 |
| `Ctrl+r` | 刷新预览 |
| `Ctrl+p` | 全屏预览 |

#### 系统操作

| 键 | 作用 |
|----|------|
| `:` | 执行 shell 命令 |
| `!` | 打开 shell（需配置） |
| `Z` | 暂停 Yazi 回到父 shell（`fg` 恢复） |
| `Q` | 退出 Yazi |
| `?` / `~` | 查看帮助菜单 |
| `Esc` | 取消搜索/过滤/可视模式 |

按下 `?` 可在 Yazi 内查看完整按键映射。

---

## 5. 配置指南

Yazi 有三个核心配置文件，位于 `~/.config/yazi/`（Linux/macOS）或 `%AppData%\yazi\config\`（Windows）：

```
~/.config/yazi/
├── yazi.toml      # 通用配置
├── keymap.toml    # 按键映射
├── theme.toml     # 主题配色
└── init.lua       # 初始化脚本（Lua）
```

> 默认配置可从 [github.com/sxyazi/yazi/tree/shipped/yazi-config/preset](https://github.com/sxyazi/yazi/tree/shipped/yazi-config/preset) 查看。

### 5.1 yazi.toml — 通用配置

```toml
# ~/.config/yazi/yazi.toml

[mgr]
ratio = [1, 4, 3]       # 三栏比例：父目录:当前目录:预览
show_hidden = false      # 是否显示隐藏文件
show_symlink = true      # 显示符号链接指向
sort_by = "natural"      # 排序方式：natural/alphabetical/mtime/size/extension/random
sort_sensitive = false   # 是否区分大小写排序
sort_reverse = false     # 是否逆序
sort_dir_first = true    # 目录优先
linemode = "none"        # 右侧信息：none/size/mtime/btime/permissions/owner
scrolloff = 0            # 滚动时上下保留的文件数

[preview]
wrap = "no"              # 代码预览是否自动换行
tab_size = 4             # Tab 宽度
max_width = 600          # 图片预览最大宽度
max_height = 900         # 图片预览最大高度
image_filter = "lanczos3" # 图片缩放滤镜
image_quality = 75       # 预缓存图片质量 50-90
cache_dir = ""           # 缓存目录（默认系统缓存）

[opener]
# 定义打开器
edit = [
  { run = "$EDITOR %s", block = true, for = "unix" },
  { run = "notepad %s", block = true, for = "windows" },
]
play = [
  { run = "mpv %s", orphan = true, for = "unix" },
]

[open]
# 根据规则匹配打开器
prepend_rules = [
  { mime = "text/*", use = "edit" },
  { mime = "video/*", use = "play" },
  { url = "*.json", use = "edit" },
  { url = "*.html", use = ["open", "edit"] },  # 可配多个
]

[tasks]
file_workers = 10        # 最大并发文件操作数
plugin_workers = 5       # 最大并发插件任务数
preload_workers = 5      # 最大并发预加载任务数
```

### 5.2 keymap.toml — 按键映射

Yazi 使用分层按键映射，支持**前置追加**（`prepend_keymap`）、**后置追加**（`append_keymap`）和**完全覆盖**（`keymap`）。

```toml
# ~/.config/yazi/keymap.toml

# 前置追加（优先级高于默认键）
[mgr]
prepend_keymap = [
  # 单一按键
  { on = "<C-a>", run = "toggle_all", desc = "Select all files" },
  # 组合按键（g 系列快捷键做书签）
  { on = ["g", "d"], run = "cd ~/Downloads", desc = "Go to Downloads" },
  { on = ["g", "p"], run = "cd ~/Projects", desc = "Go to Projects" },
  { on = ["g", "h"], run = "cd ~", desc = "Go to home" },
  # 禁用默认键（将默认行为替换为 noop）
  { on = ["g", "c"], run = "noop", desc = "" },
]

[input]
prepend_keymap = [
  # 按一次 Esc 就直接退出输入（不进入 Vim 模式）
  { on = "<Esc>", run = "close", desc = "Cancel input" },
]
```

**按键表示法：**

| 写法 | 含义 |
|------|------|
| `a`-`z` | 小写字母 |
| `A`-`Z` | 大写字母 |
| `<Enter>` | 回车 |
| `<Space>` | 空格 |
| `<Tab>` / `<BackTab>` | Tab / Shift+Tab |
| `<Esc>` | 退出 |
| `<Left>` / `<Right>` / `<Up>` / `<Down>` | 方向键 |
| `<C-a>` | Ctrl + a |
| `<A-x>` | Alt + x（macOS 可能需额外配置） |
| `<S-x>` | Shift + x |
| `<D-x>` | Super/Win/Command（需终端支持 CSI u） |

**注意：** `<Tab>` 和 `<C-i>`、`<Enter>` 和 `<C-m>` 在传统终端协议中等价，如需区分需启用 CSI u。

> 所有内置动作见 [Keymap 官方文档](https://yazi-rs.github.io/docs/configuration/keymap)。

### 5.3 theme.toml — 主题配色

```toml
# ~/.config/yazi/theme.toml

[flavor]
dark = "catppuccin-mocha"   # 暗色主题 flavor 名称
light = "catppuccin-latte"  # 亮色主题 flavor 名称

[app]
overall = { bg = "#1e1e2e" }  # 终端背景色（需 OSC 11 支持）

[mgr]
cwd = { fg = "#89b4fa", bold = true }      # 当前路径样式
find_keyword = { fg = "#f5c2e7", italic = true }  # 搜索高亮
marker_selected = { fg = "green" }         # 选中标记
border_symbol = "│"
border_style = { fg = "#45475a" }
syntect_theme = "~/.config/yazi/themes/Dracula.tmTheme"  # 代码高亮主题

[tabs]
active = { fg = "#cdd6f4", bg = "#313244" }
inactive = { fg = "#6c7086" }

[status]
sep_left = { open = "", close = "" }   # 状态栏装饰

[icon]
# 自定义图标映射
dirs = { ".git" = "" }
files = { "Cargo.toml" = "" }
```

> 现成主题可从 [yazi-rs/flavors](https://github.com/yazi-rs/flavors) 获取。

### 5.4 配置混写规则

Yazi 支持部分覆盖而不必复制整个文件——你只需要在配置文件中写入**想覆盖的部分**。Yazi 会将你的配置与内置默认值合并。

对于 `keymap.toml`，三种模式：

| 模式 | 效果 |
|------|------|
| `prepend_keymap` | 在默认键之前插入，优先级最高 |
| `append_keymap` | 在默认键之后追加，优先级最低 |
| `keymap` | **完全覆盖**该层级所有默认键 |

同样的 `prepend_*` 和 `append_*` 机制也适用于 `[open]` 规则、图标、预览器和预加载器。

### 5.5 init.lua — 初始化脚本

```lua
-- ~/.config/yazi/init.lua

-- 加载插件并初始化
require("full-border"):setup()
require("starship"):setup()

-- 自定义 linemode
function Linemode:size_and_mtime()
  local time = math.floor(self._file.cha.mtime or 0)
  if time == 0 then
    time = ""
  elseif os.date("%Y", time) == os.date("%Y") then
    time = os.date("%b %d %H:%M", time)
  else
    time = os.date("%b %d  %Y", time)
  end
  local size = self._file:size()
  return string.format("%s %s", size and ya.readable_size(size) or "-", time)
end
```

### 5.6 自定义配置目录

```bash
YAZI_CONFIG_HOME=~/.config/yazi-alt yazi
```

---

## 6. 图片预览

Yazi 在图片预览方面做了大量工作，覆盖几乎所有常见终端。

### 支持列表

| 终端/环境 | 协议 | 支持方式 |
|-----------|------|----------|
| kitty (>= 0.28.0) | Kitty Unicode Placeholders | ✅ 内置 |
| iTerm2 | Inline Images | ✅ 内置 |
| WezTerm | Inline Images | ✅ 内置 |
| Konsole | Kitty Old Protocol | ✅ 内置 |
| foot | Sixel | ✅ 内置 |
| Ghostty | Kitty Unicode Placeholders | ✅ 内置 |
| Windows Terminal (>= v1.22) | Sixel | ✅ 内置 |
| VSCode 终端 | Inline Images | ✅ 内置 |
| Warp (macOS/Linux) | Inline Images | ✅ 内置 |
| Tabby | Inline Images | ✅ 内置 |
| X11/Wayland | X11/Wayland 窗口协议 | ☑️ 需 Überzug++ |
| 所有终端兜底 | ASCII Art (Unicode) | ☑️ 需 Chafa (>= 1.16) |

### tmux 用户

在 `tmux.conf` 中添加：

```tmux
set -g allow-passthrough on
set -ga update-environment TERM
set -ga update-environment TERM_PROGRAM
```

然后重启 tmux：

```bash
tmux kill-server && tmux || tmux
```

### 查看当前使用的图片协议

```bash
yazi --debug
```

输出中 `Adapter.matches` 字段指示当前协议。

### 图片预览配置

```toml
[preview]
max_width = 600
max_height = 900
image_filter = "lanczos3"   # 缩放滤镜：nearest/triangle/catmull-rom/lanczos3
image_quality = 75          # 预缓存质量 50-90
image_delay = 0             # 延迟毫秒数，缓解快速滚动时的终端渲染延迟
```

---

## 7. 插件系统

Yazi 的插件系统基于 Lua，所有插件放在 `~/.config/yazi/plugins/` 目录下。每个插件是一个以 `.yazi` 结尾的目录。

### 插件目录结构

```
~/.config/yazi/plugins/
├── my-plugin.yazi/
│   ├── main.lua      # 入口
│   ├── README.md     # 文档
│   └── LICENSE       # 许可证
```

### 两种插件类型

#### 功能插件（Functional Plugin）

通过按键绑定触发：

```toml
# keymap.toml
[[mgr.prepend_keymap]]
on  = ["t", "t"]
run = "plugin smart-tab"
desc = "Create a tab and enter the hovered directory"
```

插件代码：

```lua
-- ~/.config/yazi/plugins/smart-tab.yazi/main.lua
--- @sync entry
return {
  entry = function()
    local h = cx.active.current.hovered
    ya.emit("tab_create", h and h.cha.is_dir and { h.url } or { current = true })
  end,
}
```

#### 预览器/预加载器插件

在 `yazi.toml` 中配置：

```toml
[plugin]
prepend_previewers = [
  { mime = "image/heic", run = "heic" },
]
prepend_preloaders = [
  { mime = "image/heic", run = "heic" },
]
```

### 同步 vs 异步

- **同步插件**（标注 `--- @sync entry`）：共享单一状态，适合 UI 插件，生命周期与应用相同
- **异步插件**（默认）：每次执行创建独立上下文，可并发执行不阻塞主线程

```lua
--- @sync entry
return {
  entry = function(state)
    state.i = state.i or 0
    ya.dbg("i = " .. state.i)
    state.i = state.i + 1
  end,
}
```

### 跨线程通信

```lua
local set_state = ya.sync(function(state, a)
  state.a = a
end)

local get_state = ya.sync(function(state, b)
  local h = cx.active.current.hovered
  return h and state.a .. tostring(h.url) or b
end)
```

### 包管理器

```bash
# 安装插件
ya pack -a yazi-rs/plugins:full-border

# 安装主题
ya pack -a yazi-rs/flavors:catppuccin-mocha

# 更新所有包
ya pack -u

# 列出已安装
ya pack -l
```

---

## 8. 实用技巧

### 开箱即用：完整边框

在 `init.lua` 中添加：

```lua
require("full-border"):setup()
```

### 书签功能（无需插件）

在 `keymap.toml` 中直接定义：

```toml
[[mgr.prepend_keymap]]
on  = ["g", "d"]
run = "cd ~/Downloads"
desc = "Go to Downloads"

[[mgr.prepend_keymap]]
on  = ["g", "h"]
run = "cd ~"
desc = "Go to home"
```

Windows 切换驱动器：

```toml
[[mgr.prepend_keymap]]
on  = ["g", "d"]
run = "cd D:"
desc = "Switch to D drive"
```

### 快捷键打开 Shell

```toml
# keymap.toml
[[mgr.prepend_keymap]]
on  = "!"
for = "unix"
run = 'shell "$SHELL" --block'
desc = "Open $SHELL here"

[[mgr.prepend_keymap]]
on  = "!"
for = "windows"
run = 'shell "powershell.exe" --block'
desc = "Open PowerShell here"
```

### 智能标签页——把标签创建在悬停目录上

`~/.config/yazi/plugins/smart-tab.yazi/main.lua`：

```lua
--- @sync entry
return {
  entry = function()
    local h = cx.active.current.hovered
    ya.emit("tab_create", h and h.cha.is_dir and { h.url } or { current = true })
  end,
}
```

绑定到 `t t`：

```toml
[[mgr.prepend_keymap]]
on  = ["t", "t"]
run = "plugin smart-tab"
desc = "Create a tab and enter the hovered directory"
```

### 目录特定排序规则

`~/.config/yazi/plugins/folder-rules.yazi/main.lua`：

```lua
local function setup()
  ps.sub("ind-sort", function(opt)
    local cwd = cx.active.current.cwd
    if cwd:ends_with("Downloads") then
      opt.by, opt.reverse, opt.dir_first = "mtime", true, false
    else
      opt.by, opt.reverse, opt.dir_first = "natural", false, true
    end
    return opt
  end)
end
return { setup = setup }
```

### 批量重命名

在文件列表中选择多个文件（`Space` 或 `v` 可视模式），按 `a` 进入批量重命名 —— Yazi 会用 `$EDITOR` 打开一个文件列表供你编辑，保存即执行重命名。

### 屏蔽 Nerd Font 图标

如果不喜欢 Nerd Font 图标，在 `theme.toml` 中：

```toml
[status]
sep_left = { open = "", close = "" }
sep_right = { open = "", close = "" }

[icon]
globs = []
dirs  = []
files = []
exts  = []
conds = []
```

### 一键打开 shell 或运行命令

- 按 `:` 进入命令模式
- `:!` 打开交互式 shell
- 支持 `%h`（悬停文件）、`%s`（选中文件）、`%d`（所在目录）等模板变量

例如绑定一键压缩：

```toml
[[mgr.prepend_keymap]]
on  = ["c", "z"]
run = "shell 'zip -r %h.zip %h' --block"
desc = "Zip hovered file"
```

---

## 9. 性能架构

### Yazi 为什么这么快？

Yazi 的设计哲学是**纯异步、事件驱动、可丢弃任务**。

#### 异步运行时（Tokio）

所有 I/O 和 CPU 消耗型任务作为异步任务运行在 Tokio 上：

- 大目录（如 10 万文件）使用**分块加载**，不像 `ls`/`eza` 需要一次性加载全部
- 后台**预加载**目录列表

#### 可丢弃任务

快速滚动文件时，正在进行的预览任务会被**直接丢弃**：

- I/O 任务：使用 Tokio 的 `abort`
- CPU 任务（代码高亮）：使用 `Atomic` ticket 检查上下文变化
- Lua 插件：Yazi 可中断 Lua 脚本的执行

#### 代码高亮

Yazi 内置代码高亮，**只读取终端高度的行数**（如 10 行），而非像 `bat` 高亮整个文件再截取。

#### 分层任务调度

- **宏任务**：大而重的操作（如文件复制）
- **微任务**：小而紧急的操作（如 mime 类型获取、图片预加载）
- 三个优先级：低/普通/高，高优先级可抢占低优先级
- 默认 5 个微任务工作者 + 10 个宏任务工作者，可配置

#### 其他优化

- 内置自然排序算法比 `natord`（`eza` 使用）快约 6 倍
- 缓存已读取的目录状态，避免重复 I/O
- 只更新目录中变化的文件，而非重新读取整个目录
- 合并多次触发的渲染为一次
- 插件系统异步优先，不阻塞主线程

---

## 10. 常用插件与资源

### 必备插件

| 插件 | 用途 |
|------|------|
| [full-border.yazi](https://github.com/yazi-rs/plugins/tree/main/full-border.yazi) | 完整边框，美化界面 |
| [git.yazi](https://github.com/yazi-rs/plugins/tree/main/git.yazi) | 显示 Git 文件状态 |
| [smart-enter.yazi](https://github.com/yazi-rs/plugins/tree/main/smart-enter.yazi) | 一键进入目录或打开文件 |
| [smart-filter.yazi](https://github.com/yazi-rs/plugins/tree/main/smart-filter.yazi) | 智能过滤 |
| [toggle-pane.yazi](https://github.com/yazi-rs/plugins/tree/main/toggle-pane.yazi) | 切换面板显示/隐藏/最大化 |
| [starship.yazi](https://github.com/Rolv-Apneseth/starship.yazi) | Starship 提示符 |
| [bookmarks.yazi](https://github.com/dedukun/bookmarks.yazi) | Vi 风格书签 |
| [chmod.yazi](https://github.com/yazi-rs/plugins/tree/main/chmod.yazi) | 可视化修改文件权限 |

### 社区 Flavor（主题）

- [yazi-rs/flavors](https://github.com/yazi-rs/flavors) — 官方主题合集
- 热门主题：catppuccin, dracula, gruvbox, tokyonight, nord

### 编辑器集成

| 平台 | 插件 |
|------|------|
| Neovim | [yazi.nvim](https://github.com/mikavilpas/yazi.nvim) |
| Vim | [yazi.vim](https://github.com/chriszarate/yazi.vim) |
| Vim | [vim-yazi](https://github.com/yukimura1227/vim-yazi) |
| Helix | [Yazelix](https://github.com/luccahuguet/yazelix) |

### 官方资源

- 文档: <https://yazi-rs.github.io/>
- GitHub: <https://github.com/sxyazi/yazi>
- Discord（英文）: <https://discord.gg/qfADduSdJu>
- Telegram（中文）: <https://t.me/yazi_rs>
- 插件合集: <https://github.com/yazi-rs/plugins>
- 博客 — Why is Yazi Fast: <https://yazi-rs.github.io/blog/why-is-yazi-fast>

---

> **Yazi 目前仍处于公开测试阶段，高频迭代中，预期会有 Breaking Changes。**
> 如果你从 `ranger`/`lf`/`nnn` 迁移，建议逐步引入插件和配置，体验 Yazi 的异步架构带来的流畅感。
