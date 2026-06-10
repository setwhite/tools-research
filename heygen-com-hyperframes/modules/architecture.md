# Architecture: heygen-com/hyperframes

## Overview

HyperFrames is an open-source **HTML-to-video rendering framework** by HeyGen. It turns HTML, CSS, media, and seekable animations into deterministic MP4 videos using headless Chrome and FFmpeg. The primary differentiator from tools like Remotion is that compositions are authored as **plain HTML files** (not React components), making them agent-friendly and requiring no build step.

Published under **Apache 2.0** license. Node.js >=22 required. FFmpeg is a system dependency.

---

## Monorepo Structure

The repository uses **Bun Workspaces** (`"workspaces": ["packages/*"]`) with 9 packages:

| Package | Description |
|---------|-------------|
| `@hyperframes/core` | Types, parsers, linter, compiler, browser runtime (IIFE), GSAP parser, font handling, registry types |
| `@hyperframes/engine` | Seekable page-to-video capture engine using Puppeteer + FFmpeg |
| `@hyperframes/producer` | Full rendering pipeline: capture, encode, audio mix. Includes regression/parity/perf harnesses |
| `@hyperframes/cli` | User-facing CLI (`hyperframes` binary) — scaffold, preview, lint, render, deploy, transcribe, background removal |
| `@hyperframes/studio` | Browser-based composition editor UI (React 19 + Vite + CodeMirror 6) |
| `@hyperframes/player` | Embeddable `<hyperframes-player>` web component for playing compositions |
| `@hyperframes/shader-transitions` | WebGL shader-based scene transitions |
| `@hyperframes/aws-lambda` | AWS Lambda SDK and CDK deployment for distributed rendering |
| `@hyperframes/gcp-cloud-run` | GCP Cloud Run deployment surface |

Each package has its own `tsconfig.json`; there is **no root tsconfig.json**.

---

## Architecture Pattern: Layered Rendering Pipeline

```
HTML Composition
     │
     ▼
┌──────────────────┐
│  @hyperframes/   │  Parse HTML, extract compositions, validate data attributes,
│  core            │  compile runtime. Provides lint rules and frame-adapter protocol.
└────────┬─────────┘
     │
     ▼
┌──────────────────┐
│  @hyperframes/   │  Load page in headless Chrome, seek to each frame, capture
│  engine          │  screenshots via Puppeteer's BeginFrame API. Uses Hono for
│                  │  HTTP server mode. Outputs raw frame data.
└────────┬─────────┘
     │
     ▼
┌──────────────────┐
│  @hyperframes/   │  Orchestrates capture → encode → audio mix → output MP4.
│  producer        │  Supports local, Docker, and distributed (Lambda-simulated)
│                  │  rendering modes. Golden regression test suite.
└────────┬─────────┘
     │
     ▼
   MP4 Video
```

### Front-end Consumers

- **CLI** (`@hyperframes/cli`): wraps `core` + `engine` + `producer` behind a `citty`-based CLI. Exposed as the `hyperframes` binary. Offers commands like `init`, `preview`, `render`, `lint`, `validate`, `cloud render`, `lambda deploy`, `transcribe`, `remove-background`, `capture`, etc.

- **Studio** (`@hyperframes/studio`): React 19 SPA (Vite) providing a visual composition editor with CodeMirror 6 for HTML editing, timeline inspector, and preview integration. Uses Zustand for state management and Tailwind CSS for styling.

- **Player** (`@hyperframes/player`): standalone `<hyperframes-player>` web component for embedding rendered compositions in any webpage.

- **AWS Lambda / GCP Cloud Run**: serverless rendering surfaces for distributed/cloud-based video production.

### Animation Model: Adapter-Based

Compositions declare animation libraries via `<script>` tags (GSAP, CSS animations, WAAPI, Lottie, Anime.js, Three.js, or custom). The HyperFrames runtime (a browser IIFE built from `@hyperframes/core`) provides a deterministic frame-adapter protocol:

- Animations expose `window.__timelines` (e.g., a paused GSAP timeline)
- The runtime seeks the timeline to the requested frame
- The renderer captures the frame state deterministically

This means any animation library that supports seekable/paused timelines can be used as an adapter.

---

## Technology Stack

- **Language:** TypeScript 5.x throughout
- **Package Manager:** Bun (with `bun.lock`)
- **Runtime:** Node.js >=22
- **Bundlers:** esbuild, tsup, Vite (studio)
- **Rendering:** Puppeteer (headless Chrome), FFmpeg (video encode)
- **HTTP:** Hono, `@hono/node-server`
- **DOM (server-side):** linkedom
- **Studio UI:** React 19, Zustand, CodeMirror 6, Tailwind CSS 3, PostCSS
- **Testing:** Vitest (unit + integration), custom regression/parity/perf harnesses in producer
- **Lint/Format:** oxlint, oxfmt, Prettier, lefthook (git hooks), commitlint, fallow (dead-code/complexity), knip (unused deps)
- **AI/ML:** ONNX Runtime (background removal in CLI)

---

## Key Directories

```
.
├── packages/           # Monorepo packages (9 total)
├── docs/               # Mintlify documentation site
├── registry/           # Catalog: blocks, components, examples (HTML + registry-item.json)
├── skills/             # AI agent skill definitions (Claude Code, Cursor, etc.)
├── scripts/            # Build, changelog, release, and catalog generation scripts
├── examples/           # AWS Lambda / GCP Cloud Run deployment examples
├── releases/           # Per-version changelog entries
├── .github/workflows/  # CI pipelines
├── package.json        # Root workspace config
└── bun.lock            # Bun lockfile
```

---

## CI/CD

Comprehensive GitHub Actions CI (`.github/workflows/ci.yml`):

- **Change detection** (`dorny/paths-filter`) gates subsequent jobs on code changes
- **Build** — `bun run build` across all packages
- **Lint** — `oxlint .` + skills lint
- **Format** — `oxfmt --check`
- **Typecheck** — `tsc --noEmit` across all packages
- **Test** — `vitest run` across all except producer (which has its own harness)
- **Runtime contract tests** — dedicated job for core runtime contract/behavior/seek/duration/parity/security tests
- **Studio smoke** — starts studio dev server, loads in headless Chrome, asserts no console errors
- **CLI smoke** — packs CLI as tarball, installs globally outside monorepo, runs init → lint → validate → render
- **Global install smoke** — end-to-end test of `hyperframes` as a globally installed npm package
- **Fallow audit** — dead-code/complexity analysis gating new issues
- **File size check** — enforces ≤600 lines per file in `packages/studio`
- **Semantic PR title** — conventional commit format validation

Additional workflows: `publish.yml`, `docs.yml`, `codeql.yml`, `catalog-previews.yml`, `regression.yml`, `preview-regression.yml`, `player-perf.yml`, `windows-render.yml`.

---

## Design Decisions

1. **HTML-native, not React-native:** compositions are HTML files with data attributes. No build step. An `index.html` plays directly in a browser. Agents already write HTML — this lowers the barrier for AI-generated video.

2. **Deterministic rendering:** the engine seeks frames deterministically. Same input = same frames = same video. This enables CI regression testing and reproducible cloud renders.

3. **Adapter-based animation:** not tied to one animation library. Any library that supports pause + seek can be wired in via `window.__timelines`.

4. **Monorepo with workspace protocol:** internal packages reference each other via `workspace:*` / `workspace:^`, keeping the dependency graph explicit and versioning coherent.

5. **Server-side DOM via linkedom:** allows the linter, compiler, and runtime tests to parse and validate HTML compositions without a browser.

6. **Modular runtime:** the core runtime can be built as a monolithic IIFE or as modular ESM for different consumption patterns.
