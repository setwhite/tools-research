# README Analysis: heygen-com/hyperframes

## Overview

- **Has README:** Yes (`README.md`, 15,968 bytes)
- **Docs directory:** Yes — comprehensive docs site (quickstart.mdx, introduction.mdx, contributing.mdx, examples.mdx, guides/, reference/, etc.)
- **Examples directory:** Yes — deployment examples (`aws-lambda`, `gcp-cloud-run`, `k8s-jobs`)
- **Demo directory:** No (404)

## Quality Score: 5/5 — Excellent

The README is a model of open-source project presentation. It is comprehensive, well-structured, and immediately actionable.

## Section-by-Section Analysis

### Quick Start (Excellent)
Two clear onboarding paths:
1. **AI coding agent** — `npx skills add heygen-com/hyperframes` + prompt example
2. **Manual CLI** — `npx hyperframes init`, `preview`, `render`

Requirements (`Node.js 22+, FFmpeg`) stated clearly. Lists compatible AI tools (Claude Code, Cursor, Gemini CLI, Codex).

### What You Can Build (Good)
Six concrete use cases with links to the Showcase for finished videos. Covers product launches, PR walkthroughs, data visualizations, social videos, docs-to-video, and reusable motion graphics.

### Frame.md (Good)
Highlights a unique feature — a design-system-to-video translation layer. Includes a gallery table of 10 design templates with images and links. Strong visual differentiation.

### How It Works (Excellent)
Full, compilable HTML code example with data attributes for timing, tracks, and GSAP animation wiring. Shows the core mental model: HTML + data attributes = video.

### HyperFrames Stack (Good)
Table covering all pieces: CLI, Core/Engine/Producer, Catalog, Agent skills, Studio, AWS Lambda, hyperframes.dev playground, frame.md. Each with status and description.

### Catalog (Good)
Concrete `npx hyperframes add` commands for shader transitions, social overlays, and animated charts. Links to full catalog.

### Why HyperFrames? (Good)
Six concise differentiators: HTML-native, Agent-friendly, Deterministic, No build step, Adapter-based animation, Open source (Apache 2.0).

### HyperFrames vs Remotion (Excellent)
Honest comparison table across 6 dimensions: Authoring, Build step, Agent handoff, Library-clock animations, Distributed rendering, License. Links to full comparison guide.

### Documentation (Good)
Seven links: Quickstart, Showcase, Guides, API Reference, Catalog, Examples, AWS Lambda rendering. All point to external docs site.

### Packages (Good)
Eight-package monorepo table with descriptions: `hyperframes` (CLI), `@hyperframes/core`, `@hyperframes/engine`, `@hyperframes/producer`, `@hyperframes/studio`, `@hyperframes/player`, `@hyperframes/shader-transitions`, `@hyperframes/aws-lambda`.

### Community (Good)
Links to Discord, GitHub Issues, SECURITY.md, CONTRIBUTING.md. Mentions production use at HeyGen and community adopters (tldraw, TanStack).

### Development Note (Good)
Git LFS setup instructions for macOS/Ubuntu/Windows + skip-smudge flag for source-only clones.

## Strengths
- **Badges** — npm version, downloads, license, Node.js version, Discord (all visible at top)
- **Navigation bar** — 6 CTA links (Quickstart, Showcase, Playground, Catalog, Docs, Discord)
- **Demo GIF** — visual proof of concept immediately visible
- **Dual audience** — caters to both AI-agent users and manual developers
- **Code examples** — full HTML composition, CLI commands, catalog installs
- **Comparison table** — honest positioning vs Remotion without disparagement
- **Monorepo transparency** — package list shows what's included
- **Live links** — everything links to working docs/playground/showcase

## Minor Observations
- No in-README API reference (delegated to external docs site — acceptable for project scope)
- `examples/` directory contains only deployment examples, not composition examples; compositions live on external showcase/playground
- No contributing steps in README (delegated to CONTRIBUTING.md — standard practice)
- No in-README changelog (available externally via docs site)
