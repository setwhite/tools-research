# HyperFrames 使用流程

> 来源：[heygen-com/hyperframes](https://github.com/heygen-com/hyperframes) README
> "Write HTML. Render video. Built for agents."

---

## 环境要求

- **Node.js** >= 22
- **FFmpeg**（系统级依赖）
- 可选：Bun（monorepo 开发用）

---

## 方式一：AI Coding Agent 使用

### 1. 安装 HyperFrames Skills

```bash
npx skills add heygen-com/hyperframes
```

支持 Claude Code、Cursor、Gemini CLI、Codex 等。

### 2. 描述视频需求

使用 `/hyperframes` 指令描述想要的视频：

> Using `/hyperframes`, create a 10-second product intro with a fade-in title, a background video, and subtle background music.

### 3. Agent 自动执行制作流程

Skills 教会 Agent 完整的 HyperFrames 制作循环：

1. **Plan** — 规划视频结构
2. **Write HTML** — 编写合法的 HTML composition
3. **Wire Animations** — 接入可 seek 的动画
4. **Add Media** — 添加视频、音频、图片等素材
5. **Lint** — 检查 composition 合法性
6. **Preview** — 浏览器实时预览
7. **Render** — 渲染输出 MP4

---

## 方式二：手动 CLI 使用

### 1. 初始化项目

```bash
npx hyperframes init my-video
cd my-video
```

创建项目脚手架，生成 `index.html` 作为 composition 入口。

### 2. 编写 HTML Composition

在 `index.html` 中通过 **data 属性** 定义视频的各个元素：

```html
<div id="stage"
     data-composition-id="launch"
     data-start="0"
     data-width="1920"
     data-height="1080">

  <!-- 背景视频 -->
  <video class="clip"
        data-start="0"
        data-duration="6"
        data-track-index="0"
        src="intro.mp4"
        muted playsinline>
  </video>

  <!-- 标题文字 -->
  <h1 id="title" class="clip"
      data-start="1"
      data-duration="4"
      data-track-index="1">
    Launch day
  </h1>

  <!-- 背景音乐 -->
  <audio data-start="0"
         data-duration="6"
         data-track-index="2"
         data-volume="0.5"
         src="music.wav">
  </audio>

  <!-- 动画库 + 时间线注册 -->
  <script src="https://cdn.jsdelivr.net/npm/gsap@3/dist/gsap.min.js"></script>
  <script>
    const tl = gsap.timeline({ paused: true });
    tl.from("#title", { opacity: 0, y: 40, duration: 0.8 }, 1);
    window.__timelines = window.__timelines || {};
    window.__timelines.launch = tl;
  </script>
</div>
```

### 关键 data 属性

| 属性 | 说明 |
|------|------|
| `data-composition-id` | composition 唯一标识 |
| `data-start` | 开始时间（秒） |
| `data-duration` | 持续时间（秒） |
| `data-width` / `data-height` | 画布分辨率 |
| `data-track-index` | 轨道索引（决定图层顺序） |
| `data-volume` | 音频音量（0-1） |

### 动画接入

通过 `window.__timelines` 注册 GSAP 时间线（或其他动画库的可 seek 实例），引擎按帧 seek 并截图：

```js
window.__timelines = window.__timelines || {};
window.__timelines.launch = gsap.timeline({ paused: true });
```

支持的动画库：**GSAP、CSS Animations、WAAPI、Lottie、Anime.js、Three.js**，以及自定义帧适配器。

### 3. 实时预览

```bash
npx hyperframes preview
```

启动浏览器预览，支持 live reload。

### 4. 渲染输出

```bash
npx hyperframes render
```

渲染流程：headless Chrome 逐帧截图 → FFmpeg 编码 → 音频混流 → 输出 MP4。

---

## Catalog：使用预置组件

安装开箱即用的 blocks 和 components：

```bash
npx hyperframes add flash-through-white   # Shader 转场
npx hyperframes add instagram-follow      # 社交媒体叠层
npx hyperframes add data-chart            # 动画图表
```

浏览全部：[hyperframes.heygen.com/catalog](https://hyperframes.heygen.com/catalog/blocks/data-chart)

---

## Frame.md：视频设计系统

将 Web 设计规范转化为视频可用的 `DESIGN.md` 超集，Agent 可直接据此生成视频。

10 个内置模板：Biennale Yellow、BlockFrame、Blue Professional、Bold Poster、Broadside、Capsule、Cartesian、Cobalt Grid、Coral、Creative Mode。

在线体验：[hyperframes.dev/design](https://www.hyperframes.dev/design)

---

## 渲染 Pipeline 流程

```
HTML Composition (index.html + data 属性)
         │
         ▼
   @hyperframes/core         ← 解析 HTML、验证 data 属性、编译 runtime
         │
         ▼
   @hyperframes/engine        ← Puppeteer 加载页面，逐帧 seek + 截图
         │
         ▼
   @hyperframes/producer      ← 截图编码 + 音频混流 → MP4
         │
         ▼
      MP4 Video
```

---

## Monorepo 包概览

| Package | 职责 |
|---------|------|
| `@hyperframes/core` | 类型定义、parser、linter、compiler、浏览器 runtime |
| `@hyperframes/engine` | Puppeteer 逐帧截图引擎 |
| `@hyperframes/producer` | 完整渲染 pipeline：capture → encode → audio mix |
| `@hyperframes/cli` | 用户 CLI（`hyperframes` 命令） |
| `@hyperframes/studio` | 浏览器端可视化编辑器（React SPA） |
| `@hyperframes/player` | `<hyperframes-player>` Web Component |
| `@hyperframes/shader-transitions` | WebGL shader 场景转场 |
| `@hyperframes/aws-lambda` | AWS Lambda 分布式渲染 |

---

## 在线资源

| 资源 | 地址 |
|------|------|
| 快速开始 | https://hyperframes.heygen.com/quickstart |
| 作品展示 | https://hyperframes.heygen.com/showcase |
| 在线 Playground | https://www.hyperframes.dev/ |
| Catalog | https://hyperframes.heygen.com/catalog/blocks/data-chart |
| 完整文档 | https://hyperframes.heygen.com/introduction |
| Discord 社区 | https://discord.gg/EbK98HBPdk |
| frame.md 设计模板 | https://www.hyperframes.dev/design |
