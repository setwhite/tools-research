# Oh My Pi Skill 编写完全指南

本文是 Oh My Pi 的 Skill 编写指南，同样适用于 Claude Code、Codex 等兼容平台。Skill 的目录结构和 `SKILL.md` 写法完全通用，仅调用方式（如斜杠命令）可能存在细微差异。

---

## 目录

1. [Skill 是什么](#1-skill-是什么)
2. [核心概念：渐进式信息披露](#2-核心概念渐进式信息披露)
3. [目录结构](#3-目录结构)
4. [SKILL.md 编写详解](#4-skillmd-编写详解)
5. [触发机制：写好 Description](#5-触发机制写好-description)
6. [编写流程](#6-编写流程)
7. [常见陷阱与最佳实践](#7-常见陷阱与最佳实践)
8. [完整实例：API 对接 Skill](#8-完整实例api-对接-skill)

---

## 1. Skill 是什么

Skill 是对**知识、工作流程、工具集**的封装。
形式为一个包含 `SKILL.md` 的文件夹，可以让 AI Agent 在需要时自动加载。

**核心原则**：

- **解决重复性工作**。只跑一次的任务没必要封装成 Skill；你手动执行超过三次的流程，才值得写。
- **优先考虑自己写，而不是盲目用别人的**。Skill 本质是纯文本指令，没有沙箱保护。如果你用第三方 Skill，先打开 `SKILL.md` 看看它到底做了什么——否则就是在承受黑箱风险。
---

## 2. 核心概念：渐进式信息披露

Skill 使用**三级加载系统**来管理上下文长度，这是其设计的核心：

| 层级 | 内容 | 何时加载 | 大小建议 |
|------|------|---------|---------|
| **Level 1** | 元数据（name + description） | 会话启动时，加载到上下文里 | ~100 词 |
| **Level 2** | SKILL.md 正文 | 触发时加载 | < 5,000 词（理想 1,500-2,000） |
| **Level 3** | 引用文件（references/、templates/ 等） | 按需加载 | 不限 |

**设计原则**：从轻到重，逐步展开。Agent 先看到摘要，确认匹配后加载正文，遇到细节再拉取引用文件。这样在不牺牲专业深度的前提下，最大限度节省 Token 消耗。

---

## 3. 目录结构

### 3.1 最小可工作 Skill（约 30 行）

```
skill-name/
└── SKILL.md          ← 唯一的必需文件
```

### 3.2 完整 Skill 结构

```
skill-name/
├── SKILL.md           ← 必需：核心指令文件
├── references/         ← 可选：Agnet 读取但不复制的参考文档
│   ├── api-docs.md
│   └── patterns.md
├── templates/          ← 可选：Agent 复制到项目中的模板文件
│   └── config.yaml
├── scripts/            ← 可选：Agent 执行的可执行脚本
│   └── validate.sh
└── assets/             ← 可选：用于最终输出的静态资源
    └── logo.png
```

### 3.3 OMP 中的存放位置

Skill 可以放在以下几个位置，按优先级从高到低：

1. **用户全局**：`~/.omp/agent/skills/<name>/SKILL.md`——对所有项目生效
2. **项目级**：`<project>/.omp/skills/<name>/SKILL.md`——仅当前项目生效
3. **插件级**：插件目录下的 `skills/<name>/SKILL.md`
4. **Claude 兼容**：`~/.claude/skills/<name>/SKILL.md`——与 Claude Code 共享
5. **自定义目录**：在 `config.yml` 中通过 `skills.customDirectories` 指定

> 同名 Skill 以高优先级为准。例如 `.omp/skills/` 中的 Skill 会覆盖 `.claude/skills/` 中的同名 Skill。

Skill 目录内的资源可通过 `skill://<name>/<path>` 协议访问，例如：
- `skill://my-api/references/api-docs.md`
- `skill://my-api/scripts/validate.sh`

---

## 4. SKILL.md 编写详解

`SKILL.md` 由两部分组成：**YAML 前置元数据** + **Markdown 正文**。

### 4.1 前置元数据（Frontmatter）

**必需字段**：

```yaml
---
name: skill-name          # 必需：≤64 字符，小写字母+数字+连字符
description: 一句话描述何时做什么事    # 必需：非空，≤1024 字符，写明触发时机和具体任务
---
```

**可选字段**：

```yaml
---
name: skill-name
description: 一句话描述何时做什么事
globs:                     # OMP 特有：关联的文件模式，辅助触发匹配
  - "**/*.sql"
  - "db/**"
alwaysApply: false         # OMP 特有：是否始终注入上下文（慎用）
hide: false                # OMP 特有：是否在 skill 列表中隐藏
disableModelInvocation: false  # 通用：禁止自动触发，仅手动调用
version: 0.1.0             # 通用：版本号
---
```

> `globs`、`alwaysApply`、`hide` 为 OMP 特有字段；`disableModelInvocation`、`version` 在 Claude Code、Codex 中同样适用。

**命名规范**：

- ✅ `api-integration`、`pdf-editor`、`code-review`
- ❌ `-bad-start`、`bad-end-`、`Under_score`、`HAS_CAPS`

### 4.2 正文（Body）写作规范

#### 写作风格：祈使句/不定式

使用**客观指令语言**，而非第二人称。

| ✅ 正确（祈使/不定式） | ❌ 错误（第二人称） |
|-----------------------|-------------------|
| 读取 `config.yaml`，提取数据库连接信息 | 你应该读取 config.yaml 并提取数据库连接 |
| 执行以下步骤完成部署 | 如果你需要部署，按以下步骤操作 |
| 遇到错误时重试最多 3 次 | 当你遇到错误时，你可以重试 |

#### 推荐章节结构

```markdown
# Skill 名称

一句话说明目的。

## 触发条件

描述哪些用户请求应触发此 Skill。

## 核心工作流

1. 第一步：做什么
2. 第二步：做什么
3. 第三步：做什么

## 硬规则

- NEVER 做 X（破坏性操作）
- MUST 做 Y（强制性步骤）
- AVOID 做 Z（不推荐做法）

## 输出格式

指定输出结构、文件命名等。

## 错误处理

| 错误场景 | 处理策略 |
|---------|---------|
| 场景 1 | 重试？降级？报错？ |
| 场景 2 | 具体处理步骤 |

## 引用资源

- `references/api-docs.md`：API 完整文档，需要查询参数时读取
- `scripts/validate.sh`：验证脚本，用于检查输出合规性

#### 长度控制

- **正文**：控制在 1,500-2,000 词（约 300-500 行）
- **超过 500 行**：将详细内容拆分到 `references/` 中
- **原则**：正文 = 核心流程 + 硬规则 + 资源索引；细节 → references
```

---

## 5. 触发机制：写好 Description

### 5.1 Description 的双重职责

Description 同时承担两个角色：

1. **给人看**：说明这个 skill 做什么
2. **给匹配器看**：决定是否触发

**绝大多数"skill 不触发"的问题，根源都在 description。**

### 5.2 写法对比

| 写法 | 示例 | 效果 |
|------|------|------|
| **抽象概括** | "管理我的项目操作" | ❌ 永远不会触发 |
| **动词抽象** | "帮助用户进行部署" | ❌ 很少触发 |
| **具体触发词** | "当用户说'部署到生产'、'上线'、'发布'时使用" | ✅ 精确触发 |

### 5.3 模板

```yaml
description: >
  当用户说"[具体短语1]"、"[具体短语2]"、"[具体短语3]"
  或提及[关键词A]、[关键词B]时使用此 Skill。
  用于[一句话说明核心目的]。
```

**示例**——一个 PDF 编辑 Skill 的 description：

```yaml
description: >
  当用户说"旋转 PDF"、"合并 PDF"、"提取 PDF 页面"、
  "给 PDF 加水印"或提及 PDF 编辑、PDF 处理时使用此 Skill。
  提供 PDF 文件的旋转、合并、拆分、水印等常用操作。
```

### 5.4 触发短语设计原则

1. **列出字面短语**：用户会说的原话，不要改写
2. **覆盖同义词**：同样的意思可能有多种说法（"部署" = "上线" = "发版" = "deploy"）
3. **包含领域关键词**：项目名、工具名、技术栈关键词
4. **避免泛化**：太宽泛的 description 会导致误触发

### 5.5 调用方式

Skill 有两种调用方式：

**自动触发**：Agent 根据语义判断用户请求是否匹配某个 Skill 的 description，匹配则自动加载执行。你正常说话即可，无需手动指定。

**手动调用**：当自动触发未命中，或你想明确指定某个 Skill 时，使用斜杠命令：

| 平台 | 手动调用方式 |
|------|-------------|
| Oh My Pi | `/skill:skill-name` |
| Claude Code | `/skill-name` |

> 如果 Skill 设置了 `disableModelInvocation: true`，则**只能**手动调用，永远不会自动触发。

---

## 6. 编写流程

以下流程参考了 Anthropic 官方的 Skill 编写指南。具体问题具体分析，切忌盲目套用——简单任务不需要走完所有步骤。

### Step 1：收集具体用例

**目标**：明确 skill 要解决什么问题。

向自己（或用户）提问：

- 这个 skill 应该支持哪些功能？
- 举几个具体的使用示例。
- 用户会说什么话来触发这个 skill？
- 除了常见场景，还有哪些边界情况？

**输出**：几个具体的用户请求示例。

### Step 2：规划可复用资源

对每个用例，分析执行它需要什么：

```
用例："帮我旋转这个 PDF"
├── 每次都需要重写旋转代码 → scripts/rotate_pdf.py
├── 需要知道 PDF 格式规范 → references/pdf-spec.md
└── 无需模板或静态资源

用例："帮我生成季度财报"
├── 需要查询数据库表结构 → references/schema.md
├── 需要统一报表模板 → templates/report.xlsx
└── 需要公司 logo → assets/logo.png
```

**输出**：资源清单（scripts × N, references × N, templates × N, assets × N）。

### Step 3：创建目录结构

```bash
mkdir -p skill-name/{references,templates,scripts,assets}
touch skill-name/SKILL.md
```

**注意**：只创建你实际需要的子目录。不需要的目录不要创建。

### Step 4：编写内容

**顺序**：先写辅助资源，再写 SKILL.md。

1. 先实现 `scripts/`、填充 `references/`、放入 `templates/` 和 `assets/`
2. 删除不需要的空目录和示例文件
3. 最后编写 `SKILL.md`，在正文中引用辅助资源

**SKILL.md 正文清单**：

- [ ] 一句话说明 skill 的用途
- [ ] 触发条件（含具体短语）
- [ ] 核心工作流（步骤清晰）
- [ ] 硬规则（NEVER/MUST/AVOID）
- [ ] 输出格式约定
- [ ] 引用文件索引（告诉 Agent 何时读哪个文件）
- [ ] 正文控制在 1,500-2,000 词

### Step 5：验证与测试

验证清单：

- [ ] 目录结构正确
- [ ] SKILL.md 有完整的 YAML 前置元数据
- [ ] Description 包含具体触发短语
- [ ] 正文使用祈使/不定式风格
- [ ] 渐进披露：正文精简，细节在 references
- [ ] 所有引用的文件实际存在
- [ ] 示例完整且可运行
- [ ] 脚本可执行且正确

**测试方法**：

1. 重启 OMP 会话（skill 不会自动热重载）
2. 说出 description 中的触发短语
3. 观察 Agent 是否正确触发了你的 skill
4. 测试边界情况：用不同措辞表达相同需求

### Step 6：迭代改进

使用后根据实际表现改进：

| 常见问题 | 修复方法 |
|---------|---------|
| Skill 不触发 | 加强 description 中的触发短语 |
| Skill 误触发 | 缩窄 description，增加区分条件 |
| 正文太长 | 拆分到 references/ |
| Agent 执行有误 | 明确硬规则，增加错误处理 |
| 缺少关键步骤 | 补充工作流步骤 |


## 7. 常见陷阱与最佳实践

### 7.1 🔴 致命错误

| 陷阱 | 症状 | 解决 |
|------|------|------|
| **Description 太抽象** | Skill 从不触发 | 列出字面触发短语 |
| **Description 太宽泛** | 误触发 | 增加限定词，缩窄范围 |
| **正文没人看得懂** | Agent 执行混乱 | 用祈使句，步骤化 |
| **正文太长** | Token 消耗过大 | 拆分到 references |
| **引用文件不存在** | Agent 读文件失败 | 确保所有引用都对应实际文件 |

### 7.2 🟡 常见失误

| 陷阱 | 说明 |
|------|------|
| **混淆 references 和 templates** | references 是读的（不复制），templates 是复制的。弄反会导致 Agent 把参考文档写入项目或把模板当作只读 |
| **一个 skill 做太多事** | 如果你发现自己写了三个互不共享状态的流程 → 那是三个 skill |
| **修改后不重启** | Skill 只在会话启动时加载。修改后必须重启会话才生效 |
| **引用深度过深** | 保持一级引用（SKILL.md → references/xxx.md），不要 A → B → C |
| **信息重复** | 信息应只存在于一处。正文 or references，选一个 |
| **忘记版本控制** | Skill 是代码化的知识。不用 Git 管理会导致多机器间漂移 |

### 7.3 🟢 黄金法则

1. **80% 的精力花在 description 上**。正文写得好但 description 差 = skill 从不触发 = 白写。
2. **一个 skill，一个目的**。犹豫要不要拆分时，问：这两个流程共享底层知识吗？不共享 = 拆。
3. **先写 Hello World**。第一个 skill 应该只有 ~30 行。跑通流程后再加 references/templates/scripts。
4. **列出具体短语，而非抽象概括**。用户说"部署到测试服"，不会说"进行部署管理操作"。
5. **正文长度：1,500-2,000 词**。超过就拆。
6. **SKILL.md 正文 = 核心流程 + 硬规则 + 资源索引**。不是百科全书。
7. **Skill 会腐化**。定期审计：规则是否还有效、引用文件是否过时、触发短语是否需要更新。

---

## 8. 完整实例：API 对接 Skill

以下是一个真实可用的 Skill 完整示例——对接一个虚构的支付网关 API `PayPlus`。

### 8.1 目录结构

```
payplus-integration/
├── SKILL.md
├── references/
│   ├── api-reference.md
│   └── error-codes.md
├── templates/
│   └── payplus.config.yaml
└── scripts/
    └── validate-signature.py
```

### 8.2 SKILL.md

```markdown
---
name: payplus-integration
description: >
  当用户提到"对接 PayPlus"、"接入支付网关"、"PayPlus 支付"、
  "微信支付接入"、"支付宝支付接入"、"生成支付签名"、
  "验证支付回调"时使用此 Skill。
  用于 PayPlus 支付网关的标准对接流程。
version: 0.1.0
---

# PayPlus 支付网关对接

提供 PayPlus 支付网关的标准对接流程：配置初始化 → 创建订单 → 处理回调 → 验证签名。

## 核心工作流

### 1. 初始化项目配置

从 `templates/payplus.config.yaml` 复制模板到项目根目录，
将 `{{MERCHANT_ID}}` 和 `{{API_KEY}}` 替换为实际值。

MUST 确认用户已提供商户号和 API 密钥后再继续。

### 2. 创建支付订单

1. 读取 `references/api-reference.md` 获取创建订单接口的完整参数
2. 使用 `scripts/validate-signature.py` 生成请求签名
3. 发送 POST 请求到 `POST /v1/orders`
4. 返回支付链接（`pay_url`）给用户

**硬规则**：
- MUST 使用 HMAC-SHA256 签名，签名算法见 `references/api-reference.md`
- NEVER 在代码中硬编码 API 密钥
- 金额单位 MUST 为「分」（整数）

### 3. 处理支付回调

1. 解析回调 POST 请求体
2. 读取 `scripts/validate-signature.py` 验证回调签名
3. 读取 `references/error-codes.md` 获取回调状态码映射
4. 根据 `status` 字段更新订单状态：
   - `SUCCESS` → 支付成功
   - `FAILED` → 支付失败
   - `REFUND` → 已退款

### 4. 查询订单状态

主动查询订单状态时，调用 `GET /v1/orders/{order_id}`。

## 错误处理

读取 `references/error-codes.md` 获取完整的错误码列表。

遇到以下情况时的处理策略：
- **网络超时**：重试最多 3 次，间隔递增（1s、2s、4s）
- **签名验证失败**：检查 API 密钥是否正确，密钥是否过期
- **订单不存在**：提示用户确认订单号
- **重复通知**：使用订单号做幂等处理，不重复更新业务状态

## 参考资源

### 文档
- `references/api-reference.md`：完整的 API 端点文档，包含所有参数和响应格式
- `references/error-codes.md`：错误码和回调状态码的完整映射表

### 脚本
- `scripts/validate-signature.py`：签名生成和验证工具
  用法：`python scripts/validate-signature.py generate --data '{"amount":100}' --key YOUR_KEY`

### 模板
- `templates/payplus.config.yaml`：配置文件模板，初始化时复制到项目中
```

### 8.3 references/api-reference.md（片段）

```markdown
# PayPlus API 参考文档

## 创建订单

**端点**：`POST https://api.payplus.example.com/v1/orders`

**请求参数**：

| 参数 | 类型 | 必需 | 说明 |
|------|------|------|------|
| amount | int | 是 | 金额（分） |
| currency | str | 是 | 货币代码，固定 `CNY` |
| out_trade_no | str | 是 | 商户订单号，≤32 位 |
| notify_url | str | 否 | 异步通知地址 |
| sign | str | 是 | HMAC-SHA256 签名 |

**签名算法**：

1. 将除 `sign` 外的所有参数按 key 字典序排序
2. 拼接为 `key1=value1&key2=value2` 格式
3. 在末尾追加 `&key=API_KEY`
4. 计算 HMAC-SHA256，转为十六进制小写字符串

**响应示例**：

```json
{
  "code": 0,
  "data": {
    "order_id": "PP2024061912345678",
    "pay_url": "https://pay.payplus.example.com/pay/xxx"
  }
}
```

### 8.4 scripts/validate-signature.py（片段）

```python
#!/usr/bin/env python3
"""PayPlus 签名验证脚本。"""

import hmac
import hashlib
import json
import sys

def generate_signature(data: dict, api_key: str) -> str:
    """生成 HMAC-SHA256 签名。"""
    sorted_keys = sorted(data.keys())
    sign_str = "&".join(f"{k}={data[k]}" for k in sorted_keys)
    sign_str += f"&key={api_key}"
    return hmac.new(
        api_key.encode(),
        sign_str.encode(),
        hashlib.sha256
    ).hexdigest()

def main():
    if len(sys.argv) < 4:
        print("用法: validate-signature.py generate --data '{}' --key KEY")
        sys.exit(1)
    # ... 省略参数解析
```

### 8.5 对这个 Skill 的点评

| 要素 | 评价 |
|------|------|
| Description | ✅ 列出具体触发短语（"对接 PayPlus"、"接入支付网关"等） |
| 正文长度 | ✅ ~350 词，控制在合理范围 |
| 硬规则 | ✅ 用 MUST/NEVER 明确约束 |
| 渐进披露 | ✅ API 细节在 references，不在正文 |
| 错误处理 | ✅ 覆盖网络超时、签名失败、幂等等场景 |
| 脚本 | ✅ 签名验证是确定性操作，适合做脚本 |
| 资源引用 | ✅ 正文中指明了何时读哪个文件 |

