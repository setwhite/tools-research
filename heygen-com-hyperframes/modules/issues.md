# Issues — heygen-com/hyperframes

## Summary

- **Open issues:** 4
- **Closed issues:** 124
- **Issue health:** Excellent — rapid triage, high maintainer engagement, strong closure discipline
- **Data collected:** 2026-06-08 via `gh api search/issues`

---

## Open Issues

| # | Title | Labels | Created | Comments | Type |
|---|-------|--------|---------|----------|------|
| 1260 | Three.js/WebGL content missing in rendered video (v0.6.x only, v0.5.5 works) | `bug` | 2026-06-07 | 2 | Bug |
| 1252 | Explore gesture-to-keyframes recording in Studio | — | 2026-06-07 | 2 | Feature request |
| 418 | Dependency Dashboard | — | 2026-04-22 | 0 | Bot (Renovate) |
| 337 | How do we connect ElevenLabs for TTS. | `enhancement` | 2026-04-19 | 6 | Enhancement |

**Notable:** Only 2 human-filed open issues. The Renovate Dependency Dashboard (#418) is intentionally pinned open for automated dependency tracking. The ElevenLabs TTS question (#337) has 6 comments from 4+ contributors but no definitive resolution yet.

---

## Serious Open Bugs

### [#1260 — Three.js/WebGL content missing in rendered video](https://github.com/heygen-com/hyperframes/issues/1260)

- **Severity:** High — any composition using Three.js/WebGL backgrounds produces broken (black/transparent) MP4 output in v0.6.x
- **Workaround:** Downgrade to `hyperframes@0.5.5`
- **Status:** Actively investigated. Maintainer `miguel-heygen` assigned within 1.5h, attempted reproduction on macOS arm64, requested `hyperframes doctor` output. Reporter provided full diagnostics (M1, 16GB, Node v26.0.0, v0.6.81). Last activity: 2026-06-08 04:07 UTC.
- **Impact:** Blocks upgrade to v0.6.x for users with Three.js compositions.

---

## Closed Issues (Sample of 15)

Bug resolution speed is the standout metric — most rendering bugs are closed same-day or within hours:

| # | Title | Type | Created → Closed | Comments |
|---|-------|------|-------------------|----------|
| 1231 | v0.6.74 regression: CLI render hangs (macOS M4) | Bug | 1.5 days | 33 |
| 1236 | Renderer freezes on low-memory hardware | Bug | ~40 min | 0 |
| 1218 | No progress/logs for 10+ min when stuck | Bug | ~2 hours | 1 |
| 1219 | Simple video timeline probe takes 10+ min | Bug | ~1 hour | 1 |
| 1199 | EISDIR regression + navigation timeout | Bug | ~10 hours | 1 |
| 1195 | Docker render: Navigation timeout at 25% | Bug | ~1 hour | 5 |
| 1193 | Docker render times out on macOS 26.5.1 | Bug | ~10 min | 2 |
| 1194 | Docker render: Navigation timeout (duplicate) | Bug (dup) | ~5 min | 0 |
| 1176 | Headless Chrome deadlock (GSAP audio tween) | Bug (self-fix) | ~4 min | 0 |
| 932 | Support Google Cloud Run for distributed rendering | Feature | 20 days | 2 |
| 1179 | Allow editing motion graphics via inspector | Feature | 3 days | 3 |
| 340 | Reusable Components and Global Design Systems | Feature | 47 days | 4 |
| 749 | Timeline comments with agent resolution | Feature | 25 days | 3 |
| 1029 | Support Clipping and content repurpose | Feature | 15 days | 1 |
| 1105 | Consider listing in awesome-codex-plugins | Meta | 9 days | 0 |

---

## Response Quality Assessment

### Speed
- **Median first response time:** < 2 hours for bugs; often minutes
- **Bug resolution:** Same-day closure is typical for rendering/platform issues
- **Example:** #1193 (Docker timeout) filed at 05:41, closed at 05:51 — **10 minutes**

### Maintainer Engagement
- Primary maintainer `miguel-heygen` is highly active: assigns issues, attempts reproduction, cross-references PRs, and communicates clearly
- Pattern seen repeatedly: assign → request diagnostics → reproduce → fix → close with PR link
- Community contributor `kiyeonjeon21` also offers diagnostic help, deferring to assigned maintainer

### Community Health
- Reporters provide high-quality bug reports: `hyperframes doctor` output, version details, hardware specs, reproduction URLs via `hyperframes.dev`
- Collaborative debugging culture: #1231 had **33 comments** of back-and-forth between reporter (`sarichan777`) and maintainer, with multiple diagnostic rounds
- Feature requests draw engagement from multiple users (#337 has 4+ distinct participants)

### Closure Discipline
- **Clean:** Bugs closed with PR cross-references or commit links
- **Duplicate handling:** Fast — #1194 identified as duplicate of #1195 and closed within 5 minutes
- **Feature requests:** Batch-closed when features ship (e.g., June 6 saw a wave of feature closures: #340, #749, #1029, #1105)
- **No abandoned closures:** Every sampled closed issue had a clear reason for closure

### Label Usage
- `bug` and `enhancement` used consistently for their categories
- Some issues lack labels (#1252, #1231, #1193, #1105) — room for improvement
- Only one `good first issue` tag observed (#932)

### Staleness
- **No staleness problem.** Only 4 open issues; oldest human issue is 7 weeks old and still receiving comments
- The Renovate Dependency Dashboard (#418) is intentionally perpetual (auto-updated by bot)

---

## Bug vs. Feature Breakdown

| Category | Open | Closed (sampled) | Notes |
|----------|------|------------------|-------|
| Bugs | 1 | 9 of 15 | Rendering/regression bugs dominate; platform-specific edge cases |
| Feature requests | 2 | 4 of 15 | Longer lifecycle (days to weeks); batch-closed on implementation |
| Bot/automation | 1 | 0 | Renovate dashboard (#418) |
| Meta/other | 0 | 2 of 15 | Community listing, duplicate |

**Pattern:** The repo sees a high volume of rendering bugs tied to specific hardware/OS/driver combinations (macOS M4, low-memory Linux, Docker environments). The team handles these with impressive turnaround — most critical rendering bugs are resolved same-day. Feature requests have a longer lifecycle (2–7 weeks) and appear to be triaged into development cycles.

---

## Verdict

**Issue management is a strength of this project.** The maintainer(s) are responsive, closure discipline is strong, and the community contributes high-quality diagnostic information. The only concern is the #1260 Three.js regression (the sole serious open bug), which is receiving active attention. Label coverage could be slightly more consistent, but this is minor.
