# HyperFrames 教程：将 HTML 渲染为确定性 MP4 视频

> HyperFrames 是由 [HeyGen](https://github.com/heygen-com/hyperframes) 开源的视频渲染框架（Apache 2.0），将纯 HTML/CSS/JS 转化为**确定性、逐帧完美**的 MP4 视频。同一份输入在任何机器、任何时间运行都产生 SHA-256 完全一致的输出。

---

## 目录

1. [核心概念](#一核心概念)
2. [AI Agent 技能集成（推荐）](#二ai-agent-技能集成推荐用法)

**——— 以下为手动模式 ———**

3. [快速上手](#三快速上手)
4. [深入理解：`hf-seek` 驱动动画](#四深入理解用-hf-seek-驱动动画)
5. [GSAP 集成：声明式时间线](#五gsap-集成声明式时间线)
6. [嵌套合成与外部引用](#六嵌套合成与外部引用)
7. [变量与批量渲染](#七变量与批量渲染)
8. [CLI 命令速查](#八cli-命令速查)
9. [最佳实践与常见坑](#九最佳实践与常见坑)
10. [与 Remotion 的对比](#十与-remotion-的对比)

---

## 一、核心概念

HyperFrames 只有**三个核心思想**：

### 1.1 合成（Composition）

一份合成就是一个 HTML 文档。文档中**第一个带有 `data-width` + `data-height` + `data-duration` 的元素**即为合成根节点。根节点之外的所有内容在渲染时会被丢弃，根节点之内的一切都会被捕获为视频帧。

* 没有场景图、没有组件树、没有 DSL
* 浏览器能渲染什么，HyperFrames 就能捕获什么

### 1.2 时间（Timing）

渲染器从 `t=0` 逐帧步进到 `t=duration`。**每一帧**它会在 `window` 上派发 `hf-seek` 事件，携带当前时间（`event.detail.time`，单位秒）。你的工作是让 DOM 精确反映那一时刻的状态。

```js
// 这是 HyperFrames 的时间模型——纯函数：当前时间 → DOM 状态
window.addEventListener("hf-seek", (e) => {
  const t = e.detail.time;
  // 根据 t 更新 DOM
});
```

**关键约束**：绝不使用 `requestAnimationFrame`、`setTimeout`、`setInterval` 或 `Date.now()` —— 它们在确定性渲染中没有意义。帧 N 长什么样，是因为你写了一个从时间到像素的纯函数。

### 1.3 适配器（Adapters）

同一份合成可以适配不同平台的输出格式——只需修改根节点的 `data-width`/`data-height` 和对应的 CSS 类名，无需改动内部标记。

| 平台      | 宽高比 |
| --------- | ------ |
| YouTube   | 16:9   |
| TikTok    | 9:16   |
| Instagram | 1:1    |

---

## 二、AI Agent 技能集成（推荐用法）

HyperFrames 的核心设计目标之一就是**为 AI 编程 Agent 提供一等支持**。通过安装官方 Skills，Claude Code、Cursor、Gemini CLI、Codex、GitHub Copilot CLI 等 Agent 可以自动学会：

* 正确的 HTML 合成结构（`data-composition-id`、`class="clip"`、`window.__timelines`）
* GSAP 时间线约束（`paused: true`、同步创建、绝对时间位置参数）
* 确定性渲染规则（禁止 `Math.random()`、禁止 `Date.now()`、无异步时间线）
* 视频设计原则（品牌色、排版、动效、转场）
* CLI 操作（`init`、`lint`、`preview`、`render`）

**一句话概括**：你只需要用自然语言描述想要的视频，Agent 帮你写好 HTML，然后一条命令渲染出 MP4。

### 2.1 一键安装 Skills

```bash
# 安装到当前项目（推荐）
npx skills add heygen-com/hyperframes

# 全局安装（所有项目生效）
npx skills add heygen-com/hyperframes --global

# 针对特定 Agent 安装
npx skills add heygen-com/hyperframes --agent github-copilot --global
npx skills add heygen-com/hyperframes --agent claude-code --global
```

安装后，Skills 文件写入项目的 `.agents/skills/` 目录（或 Agent 的全局 skills 目录）。重启 Agent 会话即可生效。

### 2.2 支持哪些 Agent

| Agent              | 安装方式                                        | 备注                        |
| ------------------ | ----------------------------------------------- | --------------------------- |
| Claude Code        | `npx skills add heygen-com/hyperframes`         | 推荐；Slash Command 原生支持 |
| Cursor             | 同上                                            | 支持 `/hyperframes` 指令     |
| Gemini CLI         | 同上                                            | Google 官方 Agent            |
| Codex (OpenAI)     | 同上                                            | OpenAI 编程 Agent            |
| GitHub Copilot CLI | 加 `--agent github-copilot` 参数                | 支持自动检测 + 显式调用      |
| Google Antigravity | 同上                                            | 每工作区安装                 |

### 2.3 六大 Slash Command

安装后，在 Agent 对话中使用以下斜杠命令加载对应上下文：

| 命令                      | 加载的技能内容                                                |
| ------------------------- | ------------------------------------------------------------- |
| `/hyperframes`            | **合成创作**：HTML 结构、时间属性、字幕、TTS、转场规则        |
| `/hyperframes-cli`        | **CLI 开发循环**：`init`、`lint`、`preview`、`render`、`doctor` |
| `/hyperframes-media`      | **媒体预处理**：TTS 语音合成、音频转写、背景移除               |
| `/hyperframes-registry`   | **组件注册表**：通过 `hyperframes add` 安装可复用 block        |
| `/website-to-hyperframes` | **网站转视频**：输入 URL，自动走完 7 步流水线（详见 2.6）     |
| `/gsap`                   | **GSAP 动画**：时间线 API、缓动函数、ScrollTrigger、插件       |

> **关键规则**：每次 Prompt 必须以 `/hyperframes` 开头——这会加载 Skills 中的合成规则，防止 Agent 凭"通用 Web 经验"写出不符合 HyperFrames 规范的代码。

### 2.4 典型 Agent 工作流

```bash
# ===== 步骤 1：脚手架 =====
# 在终端执行（或让 Agent 执行）
npx hyperframes init my-video
cd my-video
# ↑ init 会自动安装 skills 到项目中

# ===== 步骤 2：在 Agent 中打开项目 =====
# Claude Code / Cursor / Codex 中打开 my-video/ 目录

# ===== 步骤 3：用自然语言下指令 =====
# 在 Agent 对话框中输入（以 /hyperframes 开头）：
```

```
/hyperframes 创建一个 10 秒的产品介绍视频：
* 1920x1080，深色背景
* Logo 从中心缩放入场（0 → 0.8s）
* 标题文字从下方淡入上移（0.5 → 2s）
* 一条橙色 Accent 线从左向右擦除（0.8 → 1.8s）
* 使用 GSAP 时间线，paused 模式
```

```bash
# ===== 步骤 4：预览迭代 =====
npx hyperframes preview
# → 浏览器打开，拖拽时间线逐帧检视
# → 不满意？在 Agent 中继续对话微调
# → 保存后浏览器自动热重载

# ===== 步骤 5：出片 =====
npx hyperframes render --output final.mp4 --quality high
```

### 2.5 Agent 学到的关键规则（Skills 注入内容）

Skills 安装后，Agent 会严格遵守以下规则——你不需要手动记忆：

| 类别       | Agent 强制遵守的规则                                                                      |
| ---------- | ----------------------------------------------------------------------------------------- |
| **时间线** | 必须 `paused: true` 创建；同步顶层注册到 `window.__timelines`；key 匹配 `data-composition-id` |
| **动画**   | 只能动画化 `opacity/x/y/scale/rotation/color/backgroundColor`；禁止动画化 `visibility/display` |
| **确定性** | 禁止 `Math.random()`（用 seeded PRNG）；禁止 `Date.now()`/`requestAnimationFrame`          |
| **媒体**   | `<video>` 必须 `muted`；音频独立 track；禁止直接动画化 `<video>` 尺寸                      |
| **结构**   | 所有 timed 元素必须 `class="clip"`；禁止无限循环（`repeat: -1`）                            |
| **异步**   | 时间线构建 100% 同步——禁止 `async/await`、`setTimeout`、`fetch`                             |
| **转场**   | 每个场景必须有入场动画和场景间转场，禁止生硬跳切                                            |

### 2.6 高级：网站转视频（`/website-to-hyperframes`）

这是 HyperFrames 最震撼的 Agent 功能——输入一个网址，Agent 自动执行 **7 步流水线**，输出渲染好的视频：

| 步骤 | 名称           | 产出物                                   | 说明                       |
| ---- | -------------- | ---------------------------------------- | -------------------------- |
| 0    | **Capture**    | `captures/` 截图、字体、设计 tokens      | 滚动截取整页，提取视觉资产 |
| 1    | **Design**     | `DESIGN.md` 品牌参考文档                 | 色彩方案、字体、排版建议   |
| 2    | **Script**     | `SCRIPT.md` 旁白脚本                     | Hook→Story→Proof→CTA 结构  |
| 3    | **Storyboard** | `STORYBOARD.md` 逐镜头分镜               | 创意方向、情绪、素材       |
| 4    | **VO+Timing**  | `narration.wav` + `transcript.json`      | TTS 语音 + 词级时间戳      |
| 5    | **Build**      | `compositions/*.html` 逐镜头合成         | 每个分镜一个 HTML          |
| 6    | **Validate**   | 截图快照 + lint 校验                     | 逐帧对比，确保视觉正确     |

使用方式：

```
/website-to-hyperframes https://你的产品网站.com
做一个 30 秒的产品介绍视频，风格现代简约，目标受众是开发者。
```

Agent 会自动完成全部 7 步，每一步产出物可单独编辑和重跑（比如修改 `STORYBOARD.md` 调整节奏后只重建对应步骤）。

> 这正是 HyperFrames 相比 Remotion 等框架的最大差异化能力——**视频生产流水线完全由 Agent 驱动**，你只需要确认关键决策点。

### 2.7 Agent 模式 vs 手动模式

| 维度         | Agent 模式（推荐）                                    | 手动模式                       |
| ------------ | ------------------------------------------------------ | ------------------------------- |
| **上手方式** | 自然语言描述 → Agent 生成 HTML                          | 手写 HTML + JS                  |
| **学习成本** | 几乎为零——规则由 Skills 注入                            | 需要记忆 data-* 属性和约束      |
| **出错率**   | 低——Agent 被 Skills 约束，首次输出通常正确               | 中等——容易遗漏 `class="clip"`    |
| **迭代速度** | 对话式调整，秒级                                        | 手动改代码，保存刷新             |
| **适合人群** | 所有人，尤其不想记 API 细节的用户                        | 想精确控制每个像素的开发者       |
| **批量生产** | "把这段 CSV 每人做一个视频"——Agent 自动写批量脚本        | 自己写 `--variants` 命令        |

> **以下第三章至第十章为手动模式**——不依赖 Agent，直接手写 HTML/JS、手动执行 CLI 命令。如果你使用 Agent 工作流，Agent 会自动处理这些细节，但你仍需了解底层 API 以便审查和微调 Agent 生成的代码。

---

## 三、快速上手

> 本章从零开始，手写一份合成并渲染为 MP4。

### 3.1 环境准备

```bash
# 必备依赖
# 1. Node.js >= 22
# 2. FFmpeg（加入系统 PATH）

# 全局安装 CLI
npm install -g hyperframes

# 验证安装
hyperframes --version
```

### 3.2 创建项目

```bash
# 交互式创建（会引导选择示例模板）
hyperframes init my-video

# 非交互式创建（适合 CI/脚本）
hyperframes init my-video --non-interactive --example blank

cd my-video
```

生成的项目结构：

```
my-video/
├── meta.json          # 项目元数据
├── package.json
├── comp.html          # 主合成文件（入口）
├── compositions/      # 子合成目录
└── assets/            # 媒体资源目录
```

### 3.3 编写第一个合成

打开 `comp.html`，写入最简合成：

```html
<!doctype html>
<div data-width="1920" data-height="1080" data-duration="5" data-fps="60">
  <div data-start="0" data-duration="5" class="bg"></div>
  <h1 data-start="1" data-duration="4" data-fade="in">
    Hello, HyperFrames!
  </h1>
</div>
```

* `data-width="1920"` / `data-height="1080"`：合成尺寸（像素）
* `data-duration="5"`：总时长（秒）
* `data-start="1"`：元素在第 1 秒开始显示
* `data-duration="4"`：元素持续显示 4 秒
* `data-fade="in"`：内置淡入效果

### 3.4 实时预览

```bash
hyperframes preview comp.html
# → 浏览器自动打开 http://localhost:5173
```

支持热重载：修改 `comp.html` 后保存，浏览器即时刷新。可以拖拽时间线逐帧检视。

### 3.5 渲染输出

```bash
# 基础渲染
hyperframes render comp.html --out video.mp4

# 验证确定性——运行两次，SHA-256 完全一致
sha256sum video.mp4
```

渲染选项速查：

```bash
# 快速草稿（迭代时用）
hyperframes render comp.html --out draft.mp4 --quality draft
# 高质量终稿（60fps）
hyperframes render comp.html --out final.mp4 --fps 60 --quality high
# 透明通道 WebM
hyperframes render comp.html --out output.webm --format webm
# Docker 内渲染（完全可复现）
hyperframes render comp.html --out video.mp4 --docker
# GPU 加速编码
hyperframes render comp.html --out video.mp4 --gpu
# 严格模式（lint 报错即失败）
hyperframes render comp.html --out video.mp4 --strict
```

> `data-fade` 等内置属性适合简单动画，但真正的威力在下一章——手动驱动每一帧。

---

## 四、深入理解：用 `hf-seek` 驱动动画

在第三章你用 `data-fade` 实现了淡入，但那只是 HyperFrames 能力的冰山一角。本章展示如何**直接用 JavaScript 响应 `hf-seek` 事件**，精确控制 DOM 在任意时刻的状态。

`hf-seek` 是底层 API，本书第五章的 GSAP 集成是它的上层封装——两者本质相同：从时间到像素的纯函数。选哪个取决于你对代码量 vs 声明式语法的偏好。

### 4.1 完整示例：个性化介绍卡片

```html
<!doctype html>
<div id="root" data-width="1920" data-height="1080" data-duration="5">

  <!-- 静态布局层 -->
  <div id="logo" style="
    width: 120px; height: 120px;
    background: var(--accent, #6366f1);
    border-radius: 24px;
    position: absolute; top: 50%; left: 50%;
    transform: translate(-50%, -50%);
  "></div>

  <h1 id="name" style="
    font: 800 64px/1 system-ui;
    color: white;
    position: absolute; top: calc(50% + 80px); left: 50%;
    transform: translateX(-50%);
  ">{{$NAME}}</h1>

  <div id="accent-bar" style="
    width: 200px; height: 4px;
    background: var(--accent, #6366f1);
    position: absolute; bottom: 120px; left: 50%;
    transform: translateX(-50%);
  "></div>

</div>

<script>
  const root   = document.getElementById("root");
  const logo   = document.getElementById("logo");
  const nameEl = document.getElementById("name");
  const bar    = document.getElementById("accent-bar");

  window.addEventListener("hf-seek", (e) => {
    const t = e.detail.time;        // 当前时间（秒）

    // Logo：0→0.8 秒从 scale(0) 弹性缩放到 scale(1)
    const logoProgress = Math.min(t / 0.8, 1);
    const logoScale    = logoProgress < 1
      ? 1 - Math.pow(1 - logoProgress, 3)   // ease-out cubic
      : 1;
    logo.style.transform = `translate(-50%, -50%) scale(${logoScale})`;

    // 名称：0.5→2 秒淡入 + 上移
    const nameProgress = Math.min(Math.max((t - 0.5) / 1.5, 0), 1);
    nameEl.style.opacity = nameProgress;
    nameEl.style.transform = `translateX(-50%) translateY(${(1 - nameProgress) * 30}px)`;

    // Accent 条：0.8→1.8 秒从左向右擦除
    const barProgress = Math.min(Math.max((t - 0.8) / 1.0, 0), 1);
    bar.style.clipPath = `inset(0 ${(1 - barProgress) * 100}% 0 0)`;
  });
</script>
```

### 4.2 核心要点

| 行为             | 正确做法                                         | 禁止做法                            |
| ---------------- | ------------------------------------------------ | ----------------------------------- |
| 获取当前时间     | `event.detail.time`                              | `Date.now()`, `performance.now()`   |
| 驱动动画         | 在 `hf-seek` 中根据 `t` 直接设置 DOM 属性         | `requestAnimationFrame`             |
| 延迟执行         | 用 `t` 做数学判断（如 `if (t >= 1.5)`）           | `setTimeout`, `setInterval`         |
| 时间线 scrubbing | 预览中拖拽时间线 = 逐帧派发 `hf-seek`             | 依赖真实时钟                        |

> **原则**：如果你能在预览中拖拽时间线看到正确的帧，渲染器就能生成相同的帧。预览和最终 MP4 **走同一条代码路径**。

> 手写 `hf-seek` 完全控制每一帧，但代码量不小。下一章介绍 GSAP 适配器——用声明式 API 达到相同效果。

---

## 五、GSAP 集成：声明式时间线

HyperFrames 对 GSAP 有一等支持。第四章用 `hf-seek` 手写了三个缓动动画，用 GSAP 只需几行声明式调用，且自动获得缓动编排、位置参数等能力。两者本质相同——`window.__timelines` 注册表只是 `hf-seek` 事件之上的一层薄封装。

### 5.1 基本用法

```html
<!doctype html>
<div id="main" data-composition-id="main"
     data-width="1920" data-height="1080" data-duration="5">

  <h1 class="title" style="
    font: 800 80px system-ui; color: white;
    position: absolute; top: 50%; left: 50%;
    transform: translate(-50%, -50%);
  ">Hello World</h1>

  <div class="accent" style="
    width: 300px; height: 4px; background: #6366f1;
    position: absolute; bottom: 200px; left: 50%;
    transform: translateX(-50%) scaleX(0);
  "></div>

</div>

<!-- 引入 GSAP CDN -->
<script src="https://cdn.jsdelivr.net/npm/gsap@3/dist/gsap.min.js"></script>

<script>
  // 初始化全局注册表
  window.__timelines = window.__timelines || {};

  // 创建暂停的时间线（必须设置 paused: true！）
  const tl = gsap.timeline({ paused: true });

  // 使用位置参数控制动画编排
  tl.from(".title", {
    y: 48, opacity: 0, duration: 0.6, ease: "power3.out"
  });
  tl.to(".accent", {
    scaleX: 1, duration: 0.5, ease: "power2.out"
  }, 0.3);  // 0.3 秒后开始

  // 注册时间线——key 必须与 data-composition-id 一致！
  window.__timelines["main"] = tl;
</script>
```

### 5.2 GSAP 时间线规则

| 规则                           | 说明                                                              |
| ------------------------------ | ----------------------------------------------------------------- |
| `paused: true` **必须**        | 由框架控制播放，绝不自驱                                          |
| 同步创建                       | 时间线必须在脚本顶层同步创建，禁止放在 `async`、`setTimeout`、事件回调中 |
| 注册 key 精确匹配              | `window.__timelines[key]` 的 `key` = 根元素的 `data-composition-id` |
| 不用绝对时间的位置参数         | 使用**秒数**，不要用 `"<"`、`">"` 等相对标记（它们依赖真实时钟）    |

### 5.3 支持的动画属性

```js
// ✅ 可安全动画化的属性
tl.to(el, { opacity: 0.5, x: 100, y: -50 });
tl.to(el, { scale: 1.2, scaleX: 0.8, rotation: 45 });
tl.to(el, { color: "#ff0000", backgroundColor: "#0000ff" });
tl.to(el, { width: 200, height: 100 }); // CSS 尺寸
tl.to(el, { visibility: "hidden" });

// ❌ 避免对原生媒体元素动画化布局属性
//    如需动画化视频尺寸，用包装 div 代替
```

### 5.4 GSAP 常用 API

```js
// 创建时间线（可设全局默认值）
const tl = gsap.timeline({
  paused: true,
  defaults: { duration: 0.5, ease: "power2.out" }
});

// 基础动画
tl.to(".box", { x: 200, duration: 1 });           // 动画到目标值
tl.from(".box", { opacity: 0, y: 50 });           // 从指定值动画到当前
tl.fromTo(".box", { x: 0 }, { x: 200 });          // 明确起点→终点
tl.set(".box", { opacity: 0 });                   // 立即设置（duration=0）

// 位置参数（第三个参数）
tl.to(".a", { x: 100 }, 0);      // 从 0 秒开始
tl.to(".b", { x: 100 }, 1.5);    // 从 1.5 秒开始
tl.to(".c", { x: 100 }, "+=0.5"); // 在上一个动画结束后 0.5 秒开始

// 获取时间线总时长
console.log(tl.duration()); // 5.2
```

> 掌握了 GSAP 基础后，下一章展示如何将多个合成嵌套组合——这是构建复杂视频的关键。

---

## 六、嵌套合成与外部引用

合成可以嵌套——一个合成的根节点内可以引用另一个 HTML 文件作为子合成。这让你能把视频拆成独立模块，分别开发和调试。

### 6.1 外部引用（`data-composition-src`）

```html
<!-- 主合成（index.html） -->
<div id="root" data-composition-id="root"
     data-width="1920" data-height="1080" data-duration="10">

  <!-- 引用外部合成文件 -->
  <div data-composition-id="intro-anim"
       data-composition-src="compositions/intro.html"
       data-start="0"
       data-track-index="1">
  </div>

  <div data-composition-id="outro-anim"
       data-composition-src="compositions/outro.html"
       data-start="6"
       data-track-index="2">
  </div>

</div>
```

被引用的 `compositions/intro.html` 自身也是一个完整合成：

```html
<!-- compositions/intro.html -->
<template id="intro-template">
  <div data-composition-id="intro-anim"
       data-width="1920" data-height="1080">
    <h1 class="intro-title">Welcome</h1>
  </div>
</template>

<script src="https://cdn.jsdelivr.net/npm/gsap@3/dist/gsap.min.js"></script>
<script>
  window.__timelines = window.__timelines || {};
  const tl = gsap.timeline({ paused: true });
  tl.from(".intro-title", { opacity: 0, y: -60, duration: 0.8, ease: "power3.out" });
  window.__timelines["intro-anim"] = tl;
</script>
```

### 6.2 内联嵌套合成

也可以直接在父合成中定义子合成，不引用外部文件：

```html
<div data-composition-id="inline-intro"
     data-start="0"
     data-track-index="3"
     data-width="1920" data-height="1080">
  <h2 class="welcome">内联定义的合成</h2>
</div>

<script>
  const innerTl = gsap.timeline({ paused: true });
  innerTl.from(".welcome", { opacity: 0, duration: 1 });
  window.__timelines["inline-intro"] = innerTl;
</script>
```

### 6.3 Time Track（`data-track-index`）

* `data-track-index` 控制元素的 Z 轴层级——数值越大越靠前
* **同一 track 上的 clip 不能时间重叠**
* 子合成自动嵌套父时间线，无需手动 `master.add(child)`

> 场景模块化之后，接下来让视频内容也模块化——用变量驱动不同版本。

---

## 七、变量与批量渲染

### 7.1 变量语法

在 HTML 中使用 `{{$VARIABLE_NAME}}` 占位符：

```html
<h1>{{$TITLE}}</h1>
<p style="color: {{$ACCENT}}">{{$SUBTITLE}}</p>
```

### 7.2 单次渲染传入变量

```bash
# 方式一：命令行直接传入
hyperframes render comp.html --out ada.mp4 \
  --variables '{"TITLE":"Ada Lovelace","ACCENT":"#f97316","SUBTITLE":"First Programmer"}'

# 方式二：从 JSON 文件读取
hyperframes render comp.html --out ada.mp4 \
  --variables-file vars.json

# 变量严格模式——未声明的变量或类型不匹配时失败
hyperframes render comp.html --out ada.mp4 \
  --variables '{"TITLE":"Ada"}' --strict-variables
```

### 7.3 批量渲染（CSV → 一人一视频）

```bash
# people.csv 内容：
# NAME,ACCENT,outputKey
# Ada Lovelace,#f97316,ada
# Alan Turing,#3b82f6,alan
# Grace Hopper,#10b981,grace

# 批量渲染——CSV 每行输出一个独立 MP4
hyperframes render comp.html --variants people.csv --out-dir renders/
```

渲染结果：

```
renders/
├── ada.mp4
├── alan.mp4
└── grace.mp4
```

> 适合场景：个性化营销视频、年会感谢卡、学员证书、KPI 报告视频化等。

### 7.4 编程式访问变量

在 JS 中通过 `getVariables()` 读取变量：

```js
// 在 hf-seek 或 GSAP 脚本中
const { title, color } = __hyperframes.getVariables();
root.querySelector(".title").textContent = title;
root.style.setProperty("--accent", color);
```

### 7.5 声明合成变量（`data-composition-variables`）

```html
<div data-composition-id="card"
     data-width="1920" data-height="1080"
     data-composition-variables='[
       {"id":"NAME","type":"string","label":"姓名","default":"朋友"},
       {"id":"ACCENT","type":"color","label":"主题色","default":"#6366f1"}
     ]'>
  <h1>{{$NAME}}</h1>
</div>
```

这样 HyperFrames Studio 的 UI 面板会自动生成对应的输入控件。

> 以上就是 HyperFrames 的核心编程模型。下面两张表供日常查阅。

---

## 八、CLI 命令速查

### 8.1 项目管理

| 想做什么                   | 命令                                          |
| -------------------------- | --------------------------------------------- |
| 交互式创建项目             | `hyperframes init <name>`                     |
| 非交互式空白项目           | `hyperframes init <name> --non-interactive --example blank` |
| 带源视频创建               | `hyperframes init <name> --example warm-grain --video clip.mp4` |

### 8.2 开发与调试

| 想做什么                   | 命令                                          |
| -------------------------- | --------------------------------------------- |
| 启动预览（默认端口）       | `hyperframes preview`                         |
| 指定端口预览               | `hyperframes preview --port 4567`             |
| 校验合成结构               | `hyperframes lint`                            |
| 可视化检视                 | `hyperframes inspect`                         |
| 诊断环境                   | `hyperframes doctor`                          |

### 8.3 渲染输出

| 想做什么                   | 命令                                          |
| -------------------------- | --------------------------------------------- |
| 基础 MP4 渲染              | `hyperframes render <file> --out out.mp4`     |
| 快速草稿                   | `hyperframes render <file> --quality draft`   |
| 60fps 高质量               | `hyperframes render <file> --fps 60 --quality high` |
| WebM（支持透明）           | `hyperframes render <file> --format webm`     |
| Docker 内渲染              | `hyperframes render <file> --docker`          |
| GPU 加速编码               | `hyperframes render <file> --gpu`             |
| Lint 错误时渲染失败        | `hyperframes render <file> --strict`          |
| 单变量渲染                 | `hyperframes render <file> --variables '{...}'` |
| CSV 批量渲染               | `hyperframes render <file> --variants data.csv --out-dir out/` |

---

## 九、最佳实践与常见坑

### 9.1 ✅ DO

* **始终用 `paused: true` 创建 GSAP 时间线**，由框架驱动
* **在脚本顶层同步注册时间线**，不放入 async 函数或事件回调
* **预览中 scrub 时间线**确认每一帧正确——预览即渲染路径
* **用 `--quality draft` 快速迭代**，最后 `--quality high` 出片
* **用 transform（`x`/`y`/`scale`）而非 `top`/`left`** 做动画——渲染性能更好
* **用包装 `<div>` 动画化视频元素尺寸**，不要直接动画化 `<video>` 的布局属性
* **提交前跑 `hyperframes lint`** 捕获属性缺失和引用断裂

### 9.2 ❌ DON'T

* **不要调用 `tl.play()`**——HyperFrames 通过 `totalTime()` seek 控制播放
* **不要使用 `requestAnimationFrame`、`setTimeout`、`setInterval`、`Date.now()`**
* **不要在异步代码中构建时间线**——时间线必须在渲染器开始捕获前就位
* **不要用无限循环**——合成是有限时长的视频
* **不要在同 track 上重叠 clip**——同一 `data-track-index` 的 clip 时间不能交叠
* **不要用 `"<"`、`">"` 等 GSAP 相对位置标记**——它们依赖真实时钟
* **不要混淆**：`window.__timelines` API 和纯 `hf-seek` API 可以共存，但同一合成选一种即可

### 9.3 调试技巧

```bash
# 检查合成结构是否合法
hyperframes lint

# 可视化检视（展示 track 时间分布）
hyperframes inspect

# 验证环境是否就绪
hyperframes doctor
```

---

## 十、与 Remotion 的对比

| 维度         | HyperFrames                              | Remotion                              |
| ------------ | ---------------------------------------- | ------------------------------------- |
| **技术栈**   | 纯 HTML/CSS/JS，无框架依赖                | React + JSX                           |
| **构建工具** | 无需构建——浏览器直接打开 HTML              | 需要 React 打包工具链                  |
| **确定性**   | 核心设计目标，SHA-256 字节级一致           | 基本一致但非核心卖点                   |
| **AI Agent** | 一等公民——Slash Command + Skills + 批量    | 无专门设计                             |
| **安装方式** | `npm install -g hyperframes` + FFmpeg     | `npx create-video` + FFmpeg           |
| **动画引擎** | `hf-seek` 事件 + GSAP 适配器              | React 组件 + `useCurrentFrame()`      |
| **许可证**   | Apache 2.0                                | 部分功能需商业许可                     |
| **批量渲染** | 原生 CSV → 多视频                          | 需自行实现                             |
| **学习曲线** | 会 HTML/CSS 即上手                         | 需要 React 基础                        |

**选 HyperFrames 的场景**：
* 你不想引入 React 全家桶
* 需要 AI Agent 自动生成视频（CI/CD 流水线）
* 批量为每个用户生成个性化视频
* 需要在不同机器上产出**字节级一致**的视频以做回归测试

**选 Remotion 的场景**：
* 团队已经是 React 技术栈
* 需要复杂的 React 组件生态（图表库、数据可视化组件等）
* 偏好 React 的声明式编程模型

---

## 附录

### 资源链接

| 资源       | 地址                                      |
| ---------- | ----------------------------------------- |
| 官网       | https://hyperframes.video                 |
| 文档       | https://hyperframes.video/docs            |
| Playground | https://hyperframes.video/playground      |
| GitHub     | https://github.com/heygen-com/hyperframes |
| npm        | https://www.npmjs.com/package/hyperframes |
| GSAP 文档  | https://gsap.com/docs/v3/                 |

---

> 写于 2026-06-14 · HyperFrames 当前为快速迭代期，API 以官方文档为准。
