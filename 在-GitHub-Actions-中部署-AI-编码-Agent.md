# gh-agent：在 GitHub Issue 里召唤 AI 编码 Agent

> **撰写日期**：2026 年 7 月 12 日

## 省流

gh-agent 是一个 GitHub Action。放入仓库后，在 Issue 里留言就能召唤 AI Agent 来读代码、改文件、提 PR。无需服务器，所有 commit 以 bot 身份提交。

## 它是什么

你在 Issue 或 PR 里留言 → GitHub Actions 自动启动 Agent → Agent 读代码、分析问题、回复或改代码。

| 特性 | 说明 |
|------|------|
| **零服务器** | 跑在 GitHub Actions Runner 上 |
| **Bot 身份提交** | commit 归属 `github-actions[bot]`，和个人提交分开 |
| **模型无关** | 支持 OpenAI 兼容和 Anthropic 协议 |
| **私有仓库** | 能用，bot 无需被邀请 |
| **Skills** | 自动加载 `.agents/skills/` |
| **上下文文件** | 启动时读取 `AGENTS.md` |

类似项目：[Claude Code Action](https://github.com/anthropics/claude-code-action)（绑定 Anthropic）、[Codex Action](https://github.com/openai/codex-action)（绑定 OpenAI）。gh-agent 是社区项目，不绑定任何服务商。

## 部署

### 1. 创建 workflow 文件

`.github/workflows/gh-agent.yml`：

```yaml
name: gh-agent

# ===== 触发条件 =====
on:
  issues:                          { types: [opened] }      # 新建 Issue
  issue_comment:                   { types: [created] }     # Issue/PR 新评论
  pull_request_review_comment:     { types: [created] }     # PR diff 行评论

jobs:
  agent:
    if: github.event.sender.login == 'your-username'   # 仅允许你本人触发
    runs-on: ubuntu-latest

    permissions:
      contents: write             # 创建分支、提交、推送
      issues: write               # 回复 Issue 评论
      pull-requests: write        # 创建/操作 PR

    steps:
      - uses: linhan-dev/gh-agent@v0.1.2
        with:
          # ===== 模型连接 =====
          # base_url 不含端点路径，SDK 根据 provider_api 自动拼接
          base_url:              https://opencode.ai/zen/go/v1
          # API Key 存在仓库 Secret 中，不要硬编码
          # Settings → Secrets and variables → Actions → New secret
          llm_key:               ${{ secrets.GH_AGENT_LLM_KEY }}
          # openai-completions  → Chat Completions (/v1/chat/completions)
          # openai-responses    → Responses API (/v1/responses)
          # anthropic-messages  → Anthropic Messages API
          provider_api:          openai-completions
          # 是否在请求头带 Authorization: Bearer <key>
          provider_auth_header:  "true"
          model:                 deepseek-v4-pro

          # ===== 模型能力参数 =====
          model_context_window:  "128000"        # 上下文窗口 token 上限
          model_input:           text            # 输入模态：text / text,image
          model_max_tokens:      "16384"         # 单次输出 token 上限
          model_reasoning:       "true"          # 是否启用深度思考
          thinking_level:        "high"          # 思考深度：low / medium / high

          # ===== 会话行为 =====
          compaction_enabled:    "true"          # 上下文超窗口时自动压缩历史

          # ===== 工具权限 =====
          tools:                 "read,edit,write,bash"
          # read  → 读文件    edit → 精确行替换
          # write → 写文件    bash → shell（含 gh CLI、git）

          # ===== 上下文文件与技能 =====
          # ⚠️ 双重否定：false = 不禁用 = 生效
          no_context_files:      "false"
          # false → Agent 读取仓库根目录的 AGENTS.md / CLAUDE.md
          no_skills:             "false"
          # false → Agent 加载 .agents/skills/ 下的技能文件

          # ===== 容错与限制 =====
          retry_enabled:         "true"          # 模型请求失败自动重试
          retry_max_retries:     "2"             # 额外重试次数（共 1+2=3 次）
          timeout:               "1800"          # 单次最长运行 30 分钟
```

### 2. 配 Secret

Settings → Secrets and variables → Actions → New repository secret：

- Name：`AGENT_LLM_KEY`
- Value：你的 API Key

### 3. （推荐）分支保护

Settings → Rules → Rulesets，Default branch 开启 Restrict updates + Require a pull request before merging。Agent 不会直接 push main，加一层更安全。

## 参数说明

**模型连接**

| 参数 | 说明 |
|------|------|
| `base_url` | API 基础地址，不含端点路径 |
| `llm_key` | API Key，通过 Secret 传入 |
| `provider_api` | `openai-completions` / `openai-responses` / `anthropic-messages` |
| `provider_auth_header` | 是否带 `Authorization` 头，绝大多数 API 设 `"true"` |
| `model` | 模型名称 |

**模型能力**

| 参数 | 说明 |
|------|------|
| `model_context_window` | 上下文窗口 token 上限 |
| `model_input` | `text` 或 `text,image` |
| `model_max_tokens` | 单次输出上限 |
| `model_reasoning` | `"true"` 启用深度思考 |
| `thinking_level` | `low` / `medium` / `high` |

**行为**

| 参数 | 说明 |
|------|------|
| `compaction_enabled` | 上下文超窗口时自动压缩 |
| `tools` | `read,edit,write,bash` |
| `no_context_files` | ⚠️ 双重否定，`"false"` = 加载 `AGENTS.md` |
| `no_skills` | ⚠️ 同上，`"false"` = 加载 `.agents/skills/` |
| `retry_enabled` | 请求失败自动重试 |
| `retry_max_retries` | 额外重试次数 |
| `timeout` | 最长运行秒数，默认 1800 |

## 使用方式

**两种模式，自动切换：**

| 你说 | Agent 做 |
|------|---------|
| "帮我看看"、"分析一下"、"有什么建议" | 只读代码，回复分析 |
| "帮我改"、"实现这个"、"修复 bug" | 建分支 → 改代码 → 提 PR |

**触发方式：**
- 新建 Issue
- 在 Issue / PR 下评论
- 在 PR diff 某一行写评论

去 `Actions` 标签页可以看到 Agent 的实时日志。

## Skills 与上下文文件

**AGENTS.md**：放仓库根目录，Agent 启动时自动读。写项目规范：

```markdown
## 项目规范
- 文档用中文
- 提交信息遵循 conventional commits
```

**.agents/skills/**：放场景规则，Agent 按任务自动匹配：

```
.agents/skills/
  code-review/skill.md   ← 代码审查规则
  research/skill.md      ← 调研任务规则
```

写法和 Oh My Pi 本地 skill 完全一样。

| 机制 | 位置 | 何时加载 |
|------|------|---------|
| `AGENTS.md` | 仓库根目录 | 每次启动 |
| `.agents/skills/` | 仓库根目录 | 按任务匹配 |

## 总结

- gh-agent 零服务器、bot 身份提交，支持 OpenAI 兼容和 Anthropic 协议
- 部署就是一个 workflow 文件 + 一个 Secret
- 配合 `AGENTS.md` 和 `.agents/skills/`，Agent 自动遵守你的规范

## 参考链接

- [linhan-dev/gh-agent](https://github.com/linhan-dev/gh-agent)
- [Pi 编码 Agent SDK](https://www.npmjs.com/package/@mariozechner/pi-coding-agent)
- [Pi 自定义 Provider 文档](https://pi.dev/docs/latest/custom-provider)
