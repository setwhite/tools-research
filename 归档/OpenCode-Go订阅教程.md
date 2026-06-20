# OpenCode Go 订阅教程：一站式低成本编程模型指南

> **撰写日期**：2026年6月18日  
> **适用版本**：OpenCode Go 当前订阅方案

---

## 目录

1. [什么是 OpenCode Go](#一什么是-opencode-go)
2. [四大核心模型详解](#二四大核心模型详解)
   - [GLM-5.2 — 开源编码之王](#21-glm-52--开源编码之王)
   - [DeepSeek V4 Pro — 推理与复杂任务首选](#22-deepseek-v4-pro--推理与复杂任务首选)
   - [DeepSeek V4 Flash — 极致性价比高频利器](#23-deepseek-v4-flash--极致性价比高频利器)
   - [Kimi K2.7 Code — 多模态编程新势力](#24-kimi-k27-code--多模态编程新势力)
3. [定价与使用额度](#三定价与使用额度)
4. [快速上手](#四快速上手)
5. [OpenCode Go vs 官方 API 直调 —— 深度对比](#五opencode-go-vs-官方-api-直调--深度对比)
6. [总结](#六总结)

---

## 一、什么是 OpenCode Go？

OpenCode Go 是 OpenCode 推出的**低成本编程模型订阅服务**。

省流：**首月 $5、续费 $10/月，一个 API Key，接入十余款开源大模型。**

| 维度 | 说明 |
|------|------|
| **定价** | 首月 $5，之后 $10/月 |
| **核心卖点** | 统一 API Key、无厂商限定、全球低延迟部署 |
| **托管区域** | 美国、欧盟、新加坡 |
| **隐私政策** | 零保留（Zero Retention），不用于模型训练 |
| **模型数量** | 当前 13 款，覆盖 GLM / DeepSeek / Kimi / Qwen / MiniMax / MiMo |

与自己去每个模型厂商单独注册、充值、管理 API Key 相比，Go 订阅把「选模型—付费用—管额度」这些琐事全部打包，让你专注于编码本身。

---

## 二、四大核心模型详解

以下四款是 Go 订阅中定位最清晰、编程场景最常用的模型。数据来源于官方文档。

### 2.1 GLM-5.2 — 开源编码之王

**模型画像**

| 项目 | 内容 |
|------|------|
| **开发商** | 智谱 AI（Z.ai） |
| **参数规模** | 753B（MoE 架构） |
| **上下文长度** | 1,000,000 tokens |
| **多模态** | 不支持（纯文本） |


**核心优势**

- **长程编码任务**的稳定性是最大亮点——1M 上下文窗口让模型在跨文件重构、多轮调试中不丢失上下文。
- 成本仅为 GPT-5.5 的约 **1/6**，性能却与之旗鼓相当。


**月度预估请求数**：约 4,300 次（基于 Go 的 $60/月额度）

**适用场景**：复杂工程重构、长文档理解、多文件代码生成、需要持续上下文的 Agent 任务。

---

### 2.2 DeepSeek V4 Pro — 推理与复杂任务首选

| 项目 | 内容 |
|------|------|
| **开发商** | DeepSeek |
| **参数规模** | ~1.6T 总参数 / 49B 活跃参数（MoE） |
| **上下文长度** | 1,000,000 tokens |
| **多模态** | 不支持（纯文本） |

**核心优势**

- 在**数学推理、代码竞赛、复杂逻辑推理**场景中表现出色。
- 自研 DSA（DeepSeek Sparse Attention），超长上下文场景相应更快、效果更好。（比起短上下文，长上下文往往效果更好）


**月度预估请求数**：约 17,150 次

**适用场景**：算法竞赛级代码、复杂架构设计、数学证明、需要深度推理的 Agent 调度任务。

---

### 2.3 DeepSeek V4 Flash — 极致性价比高频利器

| 项目 | 内容 |
|------|------|
| **开发商** | DeepSeek |
| **参数规模** | ~284B 总参数 / 13B 活跃参数（MoE） |
| **上下文长度** | 1,000,000 tokens |
| **多模态** | 不支持（纯文本） |

**核心优势**

- **价格极低**：输入仅 $0.14/1M tokens，是 V4 Pro 的 1/12。
- 速度极快，适合高并发、高频率的编程子任务。
- 在简单的 Agent 任务上表现接近 V4 Pro。
- 缓存命中价格仅 $0.0028/1M tokens——命中率高时几乎免费。


**月度预估请求数**：约 158,150 次——Go 订阅中额度最充裕的模型。

**适用场景**：代码补全、简单重构、测试生成、文档生成、高频 Agent 子任务调度。


---

### 2.4 Kimi K2.7 Code — 多模态编程新势力

| 项目 | 内容 |
|------|------|
| **开发商** | 月之暗面（Moonshot AI） |
| **参数规模** | 1.1T 参数（MoE） |
| **上下文长度** | 256,000 tokens |
| **多模态** | 支持视觉输入（图片理解） |

**核心优势**

- **原生多模态**：不同于纯文本模型，K2.7 Code 可以直接理解截图、架构图、UI 设计稿并生成对应代码。
- Token 消耗优化：相比前代 K2.6，标准场景下 token 消耗平均下降约 **30%**。
- 长程编程中的「过度思考」问题得到明显改善——输出更精准简洁。


**月度预估请求数**：约 9,250 次

**适用场景**：UI 截图→代码、架构图理解、设计稿→前端实现、带图片的 Bug 报告分析。

> **为什么需要多模态？** 当你在 TUI/IDE 中截一张 UI 稿发给模型，K2.7 Code 能直接看懂并写出对应组件代码——纯文本模型（包括 GLM-5.2 和 DeepSeek V4）做不到这一点。

---

## 三、定价与使用额度

### 订阅费用

| 周期 | 价格 |
|------|------|
| 首月 | **$5**（约 ¥36） |
| 续费 | **$10/月**（约 ¥72） |

### 三层使用限制

OpenCode Go 采用「美元额度」而非「请求次数」来限制用量——你实际能发多少请求，取决于所选模型的单价。**便宜模型能发更多请求，贵模型则发少一些。**

| 限制层级 | 额度 | 相当于 |
|----------|------|--------|
| 每 5 小时 | $12 | 防止短时间内过度消耗 |
| 每周 | $30 | 平滑周消耗 |
| 每月 | **$60** | 总使用上限 |

**各模型在三层限制下大致可发请求数：**

| 模型 | 每 5 小时（$12） | 周均（$30） | 月（$60） |
|------|:--------------:|:---------:|:-------:|
| DeepSeek V4 Flash | ~31,600 | 79,050 | 158,150 |
| DeepSeek V4 Pro | ~3,420 | 8,550 | 17,150 |
| Kimi K2.7 Code | ~1,850 | 4,630 | 9,250 |
| GLM-5.2 | ~860 | 2,150 | 4,300 |

> **关键理解**：你每月只花 $10，但能用到价值 $60 的额度。

### 超出限制怎么办？

在控制台中开启「**使用余额（Use Balance）**」选项后，超限时会自动回退到 Zen 按量付费余额，不会拦截请求。

---

## 四、快速上手

### 4.1 获取 API Key

1. 访问 OpenCode Zen 注册/登录。  
   - 不介意的话，可以通过作者的邀请链接注册：**[opencode.ai/go?ref=4GAS0M8SN8](https://opencode.ai/go?ref=4GAS0M8SN8)**（订阅后双方各得 $5 额度，这个额度不能用来购买套餐，只能用来扩充额度）。
   - 介意的话，走官网链接：**[opencode.ai/auth](https://opencode.ai/auth)**。
2. 订阅 **Go 计划**（首月 $5）。
3. 复制生成的 **Go 订阅 API Key**（注意不是 Zen 按量付费的 Key）。

### 4.2 API 端点

Go 提供标准 OpenAI 兼容接口，可在任何支持 OpenAI Chat Completions 的工具中使用：

```
POST https://opencode.ai/zen/go/v1/chat/completions
Authorization: Bearer <你的Go API Key>
Content-Type: application/json

{
  "model": "opencode-go/deepseek-v4-flash",
  "messages": [{"role": "user", "content": "用 Python 写一个快速排序"}]
}
```

### 4.3 模型 ID

| 模型 | Model ID |
|------|----------|
| GLM-5.2 | `opencode-go/glm-5.2` |
| DeepSeek V4 Pro | `opencode-go/deepseek-v4-pro` |
| DeepSeek V4 Flash | `opencode-go/deepseek-v4-flash` |
| Kimi K2.7 Code | `opencode-go/kimi-k2.7-code` |

### 4.4 与其他工具集成

Go 的 API 兼容 OpenAI 格式，主流编程工具均可接入：

- **Oh My Pi** → 运行 `/login`，选择 OpenCode Go 即可配置。
- **OpenCode** → 运行 `/connect`，选择 OpenCode Go 即可配置。
- **Claude Code** → 通过 ccswitch 配置即可，无需转换协议格式。
- **Cursor / Windsurf** → 直接配置自定义 OpenAI Endpoint。

---

## 五、OpenCode Go vs 官方 API 直调 —— 深度对比

### 5.1 价格对比

OpenCode Go 的 token 单价与官方 API 基本持平，只有一个例外：

- **DeepSeek V4 Pro** 在 Go 中约为官方定价的 **4 倍**（Go 内 $1.74/3.48 vs 官方 $0.435/0.87 每 1M tokens）。
- **GLM-5.2、Kimi K2.7 Code、DeepSeek V4 Flash** 与官方 API 定价完全一致。


### 5.2 什么时候该用 Go？

**强烈推荐 Go 的场景**：

- 你不想在 4–5 个模型厂商之间来回注册、充值、管理 Key。
- 你每月编程 AI 用量**在中高水平**（日均几百次请求），Go 的 $60 额度能覆盖。
- 你需要**频繁切换模型**——Flash 跑简单任务、Pro 跑复杂推理、Kimi 看图片。
- 你是个人开发者或小团队，追求**费用可预测**——$10/月雷打不动。

**建议用官方 API 的场景**：

- 你用量极低（月均 $2–$3），$10 月费不划算。
- 你几乎只用 **DeepSeek V4 Pro**——官方 API 便宜 4 倍。
- 你需要极高并发或在生产环境中自定义调度策略。
- 你需要模型厂商的专属功能（如智谱的 Coding Plan、Kimi Code Plan 等）。


---


## 六、总结

首月 $5，续费 $10/月，月额度 $60，覆盖 GLM-5.2、DeepSeek V4 Pro、DeepSeek V4 Flash、Kimi K2.7 Code 四款模型。

---

> **参考链接**
> - [OpenCode Go 官方文档](https://opencode.ai/docs/zh-cn/go/)
> - [OpenCode Zen 控制台](https://opencode.ai/auth)
> - [DeepSeek API 定价](https://api-docs.deepseek.com/zh-cn/quick_start/pricing)
> - [Kimi API 平台](https://platform.kimi.com/docs/pricing/chat)
> - [GLM-5.2 HuggingFace Blog](https://huggingface.co/blog/zai-org/glm-52-blog)
