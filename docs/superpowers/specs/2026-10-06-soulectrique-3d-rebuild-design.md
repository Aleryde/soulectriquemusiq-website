# Soulectrique Musi_q — 3D Scroll-Animated Rebuild (Test Project)

**Date:** 2026-10-06
**Status:** Approved for planning
**Classification:** Architectural (new project, new tech stack)

## Purpose

Evaluate a modernized visual/animation direction for the Soulectrique Musi_q label site by rebuilding it with the `3d-scroll-website-skill` pipeline (Next.js 16, React 19, Framer Motion, Lenis, canvas frame-sequence hero), while preserving the existing Soulectrique brand identity and real catalog data. This is a standalone test — it does not replace or touch the live site at `aleryde.github.io/soulectriquemusiq-website`.

Success criteria: a working, deployed site at a new GitHub Pages URL that (a) demonstrates the scroll-driven canvas hero mechanic, (b) carries the real 23-release catalog with working audio previews, and (c) reads unmistakably as Soulectrique, not as the skill's generic neumorphic agency template.

## Scope

**In scope:**
- New standalone project, new repo, new deploy target
- Full site rebuild: hero, manifesto/stats, catalog (23 releases, filter + search), release modal, audio player, DSP links section, demo-submission modal, footer
- Placeholder frame-sequence hero animation (no real 3D render assets exist yet)
- Soulectrique visual identity (palette, type) applied over the skill's component patterns

**Out of scope:**
- Any change to the live `soulectriquemusiq-website` repo or its GitHub Pages deployment
- Any change to `loa-sound-visions`
- Real 3D-rendered hero frames (placeholder only; swapping in real renders is a later, separate task)
- Backend/CMS for the catalog — data stays a static TypeScript file, same as the current site's inline data
- Netlify deployment for this test project (GitHub Pages only, to avoid the credit-limit issue already hit on the main project)

## Project setup

- Location: `~/soulectriquemusiq-3d-test` (sibling directory to the current project, fully independent git history)
- GitHub repo: `Aleryde/soulectriquemusiq-3d-test`, public (same visibility pattern as the other two label repos; GitHub Pages on the free plan requires a public repo)
- Stack: pinned versions from the skill (`next@16.2.2`, `react@19.2.4`, `framer-motion@12.38.0`, `lenis@1.3.21`, `@phosphor-icons/react@2.1.10`, `tailwindcss@4`)
- `next.config.ts`: `output: "export"` for a static build (no server features are needed — no API routes, no SSR-only data)

## Visual identity override

The skill's default design system (zinc neutrals, indigo/violet accents, Geist font, neumorphic shadow stack) is replaced with Soulectrique's existing tokens, carried over from the current site's `:root` CSS variables:

| Token | Value |
|---|---|
| `--bg` | `#ffffff` |
| `--ink` | `#0a0a0a` |
| `--ink-dim` | `#3c3e42` |
| `--cyan` (accent) | `#049c8a` |
| `--magenta` (accent) | `#d81b60` |
| Display font | Poppins (700/800) |
| Body font | Inter (300–600) |
| Mono (eyebrow/labels) | SFMono/Menlo stack |

The neumorphic shadow stack and glassmorphic nav from the skill's design-patterns reference are kept as *structural* patterns (they're just depth/blur techniques) but recolored to these tokens — e.g. card shadows stay multi-layer but tuned to sit on white, not `#f5f5f5`.

## Hero section

- Structure: skill's standard sticky-canvas pattern — `400vh` section, `sticky top-0 h-screen` inner wrapper, canvas drawn via scroll-progress → frame-index mapping, RAF + ticking-ref throttling, DPR-aware sizing, passive scroll listener.
- Frames: no real 3D renders exist. A small Node script (`scripts/generate-placeholder-frames.mjs`) procedurally renders ~60 frames to `public/frames/frame_0001.jpg`…`frame_0060.jpg` using `node-canvas`: the SLQ square mark rotating/scaling against a drifting cyan↔magenta radial gradient, matching the live site's hero glow. This validates the scroll-canvas mechanic honestly, without pretending real 3D assets exist. Swapping in real Blender/C4D renders later is a drop-in replacement (same filenames, same `FRAME_COUNT`).
- Overlay text: real Soulectrique copy ("Soul, meaning the human current. Électrique, meaning the machine that carries it." / "Soulectrique is the point where the two meet...") fading out over the first 8% of scroll, per the skill's hero-text-fade pattern.
- Mobile: 1.3× canvas zoom, `300vh` section height, per skill defaults.

## Data layer

`src/data/releases.ts` — the real catalog, ported verbatim from the current site's `data-tracks` attributes and `discology.agency` MP3 URLs (23 releases, 110 tracks total):

```ts
export type Track = { title: string; mp3: string };
export type Release = {
  code: string; title: string; artist: string;
  genre: "techno" | "tech-house" | "experimental" | "compilation";
  genreLabel: string; year: string; cover: string; tracks: Track[];
};
export const releases: Release[] = [ /* SLQ001–SLQ023 */ ];
```

Cover images: reused as external URLs/base64 from the current site rather than re-encoded, to avoid bloating the new repo.

## Catalog section

- Grid of release cards (`AnimatedSection` / `AnimatedItem` stagger-reveal per skill pattern), genre filter pills + live text search — same filtering logic as the current site, ported to a `useMemo`-derived filtered list instead of direct DOM query/hide.
- Card click opens the release modal (React state: `selectedRelease`), no URL-hash deep-linking in the first pass (current site's `#SLQ00X` hash support is a nice-to-have, not required for the test).

## Release modal + audio player

Same mechanism already validated on the live site, ported to React:
- A single `<audio>` element owned by a small `useAudioPlayer()` hook (src, play/pause, currentTime/duration via `timeupdate`/`loadedmetadata` listeners).
- Modal lists real tracks; clicking a track sets the hook's source and plays it.
- A persistent bottom player bar (code/title/artist/cover, play-pause, seek, close) mirrors the current site's bar, driven by the same hook.
- No synthesized/fake audio — real MP3 preview files only, same as the live site today.

## Supporting sections

- **Manifesto/stats** ("Signal over noise", release/track/artist/since-2014 counters) — Framer Motion `AnimatedSection` port of the current copy.
- **Stream Everywhere** — the DSP links already gathered for the live site (Spotify, Apple Music, Beatport, SoundCloud, YouTube); no Bandcamp, per the earlier decision on the live site.
- **Demo submission** — modal with the same submission guidelines copy as the current site (no backend form processing exists on the current site either; parity, not a new feature).
- **Footer** — nav + LOA Sound Visions network links, same as current.

## Deployment

- `next build` with static export → `out/`
- GitHub Actions workflow (`.github/workflows/deploy.yml`) building and publishing `out/` to GitHub Pages, OR GitHub Pages configured to serve from a `gh-pages` branch populated by the export — decide exact mechanism during planning (both are standard, zero-cost options; avoids the Netlify credit-limit problem hit on the main project).
- Resulting URL: `https://aleryde.github.io/soulectriquemusiq-3d-test/`

## Testing approach

Given this is an exploratory visual/animation test, testing stays lightweight and matches the skill's own stated workflow rather than the project's full 80%-coverage automated suite:
- Each section checked manually in a real browser (real scroll, both desktop and a narrow mobile viewport) before moving to the next, per the skill's build order
- Performance hardening checklist from the skill followed as a manual verification pass before calling the hero "done" (RAF throttling, DPR scaling, preload bar, passive listeners, no dropped frames in devtools)
- Audio playback spot-checked the same way the live-site audio wiring was verified (real track plays, correct duration, per-track routing)
- No automated test suite for this pass — flagged here explicitly as a deliberate scope decision for a test/sandbox project, not an oversight

## Risks / open items

- Placeholder hero frames are a stand-in; the visual "modern 3D" impression is necessarily weaker than with real renders. This is expected and acceptable for evaluating the *mechanic*, not the final art direction.
- Static export (`output: "export"`) means no Next.js Image optimization API and no server actions — acceptable here since the catalog is static data and images are already plain URLs/base64.
- Bundle size: Framer Motion + Lenis + 60 placeholder JPGs will be heavier than the current single-file site. Not a concern for a test deploy, but worth noting if this ever becomes the production site.
