# Slide2Video / HyperFrames / Remotion 调研报告

> 调研日期：2026-06-02

---

## 1. Slide2Video — 幻灯片转 AI 视频

**官网**: https://slide2video.app/

Slide2Video 是一个面向非技术用户的 **AI SaaS 工具**，将 PDF 或 PPTX 演示文稿自动转换为带旁白的视频。

### 工作流程

1. 上传 PDF / PPTX（最大 50MB）
2. AI 分析每一页，抽取关键信息
3. 自动生成逐页旁白脚本（NLP）
4. 用户审核、修改脚本
5. 合成神经语音（TTS），按旁白时长自动匹配每页停留时间
6. 导出 MP4 视频

### 核心卖点

- **零手动操作**：不需要录制屏幕、不需要配音员、不需要时间轴编辑
- **多语音选择**：多种高保真 AI 声音可选
- **场景自动同步**：AI 计算每页旁白时长，自动匹配页面停留时间
- **人工审核环节**：可在渲染前修改 AI 生成的脚本

### 目标用户

- 创业者 — 快速制作融资 Pitch 视频
- 市场人员 — 将白皮书/案例转为社交媒体视频
- 教育/培训团队 — 将课件转为异步学习视频
- 产品经理 — 自动化产品 Demo 和功能演示

### 定价

免费套餐可用，付费解锁更多功能。

---

## 2. HyperFrames — HTML 写视频，面向 Agent

**官网**: https://hyperframes.heygen.com  
**GitHub**: https://github.com/heygen-com/hyperframes  
**许可证**: Apache 2.0（OSI 认证的真正开源）  
**开发商**: HeyGen

HyperFrames 是 **HeyGen 开源的 HTML-to-Video 渲染框架**。核心理念：用 HTML + CSS + JS（GSAP 动画）定义一个视频，然后用 headless Chrome + FFmpeg 逐帧渲染成 MP4。

### 核心渲染管线

```
HTML (composition) → Puppeteer/Chrome → 逐帧截图 → FFmpeg 编码 → MP4
```

每帧通过公式 `frame = floor(time * fps)` 计算，引擎在每帧暂停所有动画并 seek 到对应时间点，然后截图。同一输入永远产生完全相同的输出。

### 创作模型

每个元素通过 `data-*` 属性标记时序和轨道：

```html
<div id="root" data-composition-id="demo"
     data-start="0" data-width="1920" data-height="1080">

  <video data-start="0" data-duration="5"
         data-track-index="0" src="intro.mp4" muted playsinline />

  <h1 data-start="1" data-duration="4"
      data-track-index="1" style="font-size: 72px; color: white;">
    Welcome to HyperFrames
  </h1>

  <audio data-start="0" data-duration="5"
         data-track-index="2" data-volume="0.5" src="music.wav" />
</div>
```

### CLI 命令

```bash
npx hyperframes init my-video    # 脚手架
npx hyperframes preview           # 浏览器预览，热更新
npx hyperframes render            # 渲染为 MP4
npx hyperframes add data-chart    # 从 Catalog 安装组件
```

### 关键特征

| 维度 | 说明 |
|---|---|
| **创作方式** | 纯 HTML + CSS + GSAP（或其他可 seek 的动画库） |
| **构建步骤** | 无 — `index.html` 直接在浏览器播放 |
| **确定性渲染** | `frame = floor(time * fps)`，逐帧 seek，同输入 = 同输出 |
| **Agent 友好** | CLI 默认为非交互模式，纯文本输出；Agent（Claude Code / Cursor / Codex / Gemini CLI）可直接驱动 |
| **动画库兼容** | GSAP、Lottie、Three.js、Anime.js、CSS @keyframes、WAAPI — 通过 Frame Adapter 模式接入 |
| **两种捕获模式** | BeginFrame 模式（Linux，逐字节可复现）+ Screenshot 模式（macOS/Windows/自动降级） |
| **分布式渲染** | AWS Lambda + Step Functions，支持批量渲染和变量模板 |
| **HDR 输出** | 支持 HDR10（BT.2020 PQ/HLG，10-bit H.265） |
| **可视编辑器** | HyperFrames Studio — DOM 即编辑层，所见即所得 |
| **Catalog** | 50+ 可复用组件（图表、转场、字幕、地图、3D 设备展示、社交媒体卡片等） |
| **4K 渲染** | 通过 Chrome deviceScaleFactor 从 1080p 超采样到 4K |

### 技术栈

| 包名 | 功能 |
|---|---|
| `hyperframes` (CLI) | 脚手架、预览、lint、渲染 |
| `@hyperframes/core` | 类型、HTML 解析、运行时、linter |
| `@hyperframes/engine` | 基于 Puppeteer 的逐帧捕获引擎 |
| `@hyperframes/producer` | 完整渲染管线（捕获 + FFmpeg 编码 + 音频混音） |
| `@hyperframes/studio` | 浏览器端可视编辑器 |
| `@hyperframes/player` | 可嵌入的 `<hyperframes-player>` Web Component |

### 为什么叫 "Agent-first"

LLM 最擅长写 HTML — 这是互联网上训练数据最丰富的格式。Agent 不需要先学习 React 框架规则再发挥创意。CLI 全部 flag 驱动，非交互式，fail-fast，Agent 可直接操作。

---

## 3. Slide2Video vs HyperFrames 对比

| 维度 | Slide2Video | HyperFrames |
|---|---|---|
| **定位** | SaaS 产品，面向最终用户 | 开源框架，面向开发者 & AI Agent |
| **输入** | PDF / PPTX | HTML + CSS + JS |
| **输出** | AI 自动生成旁白 + 语音的 MP4 | 确定性逐帧渲染的 MP4 / MOV / WebM |
| **动画能力** | 无 — 静态幻灯片 + AI 语音 | 完整 Web 动画栈（GSAP / Three.js / Lottie / CSS / Shader） |
| **自定义程度** | 低 — 修改 AI 生成的脚本 | 极高 — 完全控制每个像素、每帧 |
| **技术门槛** | 零代码 | 需要前端 / HTML 基础 |
| **适合场景** | 培训视频、产品 Demo、营销内容快速产出 | 品牌视频、数据可视化、产品发布、自动化视频管线 |
| **定价** | 免费套餐 + 付费 | 完全免费（Apache 2.0），无渲染次数限制 |
| **Agent 能力** | 无 | 原生支持 |

**一句话总结**：

- Slide2Video 是「给我一个 PPT，给我一个视频」的成品工具
- HyperFrames 是「我用代码精确控制每一帧」的视频引擎

---

## 4. Remotion — React 写视频的先驱

**官网**: https://www.remotion.dev  
**GitHub**: https://github.com/remotion-dev/remotion  
**许可证**: 自定义商业许可证（源码可见但非开源，商业使用需付费）

Remotion 是 **用 React 组件程序化创建视频** 的框架，是这个赛道的先驱。同样走 headless Chrome + FFmpeg 渲染路线。

### 核心思路

```tsx
// 视频 = React 组件
export const MyVideo = () => {
  const frame = useCurrentFrame();
  return (
    <div style={{ opacity: frame / 30 }}>
      Hello World
    </div>
  );
};
```

### Remotion vs HyperFrames（详细对比）

| | HyperFrames | Remotion |
|---|---|---|
| **创作语言** | HTML + CSS + GSAP | React 组件 (TSX) |
| **运行时** | 浏览器 DOM，无框架开销 | React reconciliation 逐帧运行 |
| **构建步骤** | 无，`index.html` 直接播放 | 必须走 bundler（webpack 等） |
| **动画库时钟** | 可 seek，帧精确 — GSAP 暂停后 seek 到 `frame/fps` | 墙钟时间播放 — GSAP 等库在渲染时会"跑完"，导致空帧 |
| **任意 HTML/CSS** | 粘贴即用 | 必须改写为 JSX |
| **Agent 表现** | 更优 — LLM 训练数据中 HTML 远超 React | Agent 需先学习框架规则，输出创意度受限 |
| **React 组件复用** | 不支持 — 这不是 React | 强项 — 可直接用现有 React 设计系统 |
| **可视编辑器** | DOM 即编辑源，天然好做 | 源码 + 构建步骤，round-trip 编辑困难 |
| **分布式渲染** | AWS Lambda（较新） | Remotion Lambda（多年生产验证，极其成熟） |
| **HDR** | 支持 HDR10 | 官方标注不支持 |
| **许可证** | Apache 2.0（真正开源） | 商业许可证（商业使用需付费，有 per-render 费用） |
| **社区/成熟度** | 2026 年新项目，HeyGen 驱动 | 多年积累，大社区，文档完善 |

### 🔑 GSAP 动画的致命差异（来自 HyperFrames 官方对比）

同样的 GSAP 4 秒时间线动画（11 个字母逐个飞入 → 停留 1.5 秒 → 逐个旋转飞出）：

- **HyperFrames**：完整 4 秒全部正确渲染。引擎在每帧暂停 GSAP 并 seek 到精确时间点。
- **Remotion**：GSAP 在渲染的**第 1 秒内就跑完了整个 4 秒动画**（因为 `performance.now()` 按墙钟时间推进），剩余帧捕获到的都是空舞台。

**原因**：GSAP 内部通过 `performance.now()` 驱动自己的时间线。HyperFrames 在每帧暂停 GSAP 然后 seek 到 `frame/fps` 位置；Remotion 没有等效原语，GSAP 的 ticker 以实时速度跑完全程。

---

## 5. 选型建议

```
                    ┌──────────────────────────────────────────────┐
                    │           你要做什么？                        │
                    └──────────────────────────────────────────────┘
                                    │
            ┌───────────────────────┼───────────────────────┐
            ▼                       ▼                       ▼
    快速把 PPT 变成        精确控制视频每一帧       用 React 生态做视频
    带旁白的视频            （Agent 或开发者）       （已有 React 团队）
            │                       │                       │
            ▼                       ▼                       ▼
       Slide2Video            HyperFrames               Remotion
       (SaaS 工具)           (开源 HTML 引擎)          (React 框架)
```

### 选 Slide2Video，如果：

- 你只需要把 PPT/PDF 快速变成带 AI 旁白的培训/营销视频
- 不想写任何代码
- 不需要复杂动画或自定义视觉效果

### 选 HyperFrames，如果：

- 你需要 **精确控制每一帧**（品牌视频、数据可视化、产品发布视频）
- 你或你的团队会用 AI Agent 批量生产视频
- 你想用 GSAP / Three.js / Lottie / Shader 做高创意动画
- 你需要 **完全免费、真正开源**，无渲染次数上限
- 你要构建自动化视频管线（CI/CD、批量模板渲染）

### 选 Remotion，如果：

- 你的团队已经有 **React 组件设计系统**，想在视频中直接复用
- 你需要 React 生态的类型安全、IDE 智能提示、组件复用
- 你接受商业许可证
- 你需要极其成熟的 Remotion Lambda 分布式渲染（多年生产验证）

---

## 6. 关键洞察

1. **Slide2Video 和 HyperFrames 不是竞争关系**——一个面向最终用户（零代码 SaaS），一个面向开发者/Agent（开源框架）。它们在技术栈的完全不同的层次。

2. **HyperFrames 和 Remotion 是直接竞争者**——都是「代码写视频」的框架，核心技术差异在于 **HTML 还是 React 作为创作面**。这个选择决定了动画库兼容性、Agent 友好度、可视编辑器难度、以及许可证模式。

3. **HyperFrames 的 HTML-first 策略**是 2026 年 AI Agent 时代的有力赌注——Agent 不需要学框架，不需要 build step，不需要把 GSAP 代码翻译成 React。训练数据中 Web（HTML/CSS/JS）的内容量远超 React。

4. **许可证是关键决策因素**：HyperFrames 是 Apache 2.0 真正开源，商业使用零成本；Remotion 是商业许可，商业使用需付费。
