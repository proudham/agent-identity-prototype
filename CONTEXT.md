# Agent Identity Prototype — Handoff Context

## What This Is

A visual identity system for **AI agents** at Harvey (legal AI company). Each agent gets a unique typographic avatar: characters from the agent's name fill a circular form, with **animated font weight variation** (via variable font) creating the illusion of rolling light/shadow across a 3D surface.

The prototype is a single HTML file (`index.html`) with a centered canvas and a controls panel. It runs on a local HTTP server so the variable font loads correctly.

## Two Agent Types

- **Workflow (WF) agents** — single-purpose tasks (e.g. "Ingest data room", "Extract key terms"). Characters come from the agent's own name.
- **Long Horizon (LH) agents** — orchestrate multiple WF agents (e.g. "Diligence Agent"). Characters come from the **combined names of all constituent workflow agents**, creating a richer character set. LH agents should feel more unified/resolved than WF agents, not chaotic.

## How It Works

1. **Grid of characters** fills a circular region inside the canvas
2. **Characters scale down** toward the circle edge (cubic falloff) — no hard clipping
3. **Layered sine waves** (3-5 per agent, seeded from a name hash) determine brightness at each grid position
4. **Brightness maps to font weight** (200-1000 via variable font) — heavy = bright peaks, light = valleys
5. **Animation**: the sine wave phase advances over time, creating rolling waves of weight variation
6. **Name hash** makes each agent's wave pattern deterministic and unique — "Ingest data room" always looks the same

### Key Animation Math

```
phase = (elapsedMs / 1000) * (speed / 10)   // at speed=12 → 1.2 rad/s
brightness = layered_sines(x, y, phase)       // 0-1, linear
weight = minW + brightness * (maxW - minW)    // maps to font weight
```

Each sine layer has:
- `angle` — direction the wave rolls (seeded from name)
- `freqMul` — spatial frequency multiplier (0.6-1.8)
- `phase` — initial offset (seeded from name)
- `amp` — amplitude, decreasing for higher layers (1/1, 1/1.5, 1/2, ...)

`BASE_FREQ = 0.055` controls the global spatial frequency (~1.4 visible wave cycles across a 250px canvas).

## Files

```
/Users/ryan/Desktop/agent-identity-prototype/
  index.html                          # The prototype (single file, all JS inline)
  HarveySansDiatypeVariable.woff2     # Harvey's brand font (variable, weight 200-1000)
  .claude/launch.json                 # Dev server config (npx serve on port 8753)
```

### Running It

```bash
cd /Users/ryan/Desktop/agent-identity-prototype
npx serve -l 8753 --no-clipboard
# Open http://localhost:8753
```

The font **requires HTTP** — it won't load via `file://` protocol. The code has a 2-second timeout fallback so the animation starts even if the font hangs, falling back to Inter from Google Fonts.

## Current State & Known Issue

**The animation may still appear too slow or imperceptible.** This has been the main unresolved bug across two sessions. The most recent fix (this session) was:

1. Removed `n * n` contrast squaring that compressed brightness variation
2. Increased `BASE_FREQ` from 0.04 to 0.055
3. Simplified the phase formula from `(seconds * speedFactor / LOOP_SECONDS) * 2pi` to `(timeMs/1000) * (speed/10)` — at speed=12, phase advances 1.2 rad/s

**If the animation still isn't visible**, the next things to try:
- **Log values to console**: Add `if (row===10 && col===10) console.log(phase, n, weight)` inside the render loop to confirm phase is actually changing and weights are varying
- **Increase BASE_FREQ further** to 0.08-0.1 for tighter spatial waves
- **Increase speed default** from 12 to 20-30
- **Check the font is actually variable**: If the woff2 isn't loading or isn't truly variable, all weights render identically. Test by setting one character to weight 200 and another to 900 and see if they look different.
- **Consider using opacity instead of/in addition to weight** as a more visually obvious animation signal

## UI Layout

```
┌─────────────────────────────────┬──────────┐
│                                 │ Presets   │
│                                 │ ───────── │
│         [canvas]                │ Animation │
│     "Ingest data room"         │  Speed    │
│                                 │ ───────── │
│                                 │ Typography│
│                                 │  Font Size│
│                                 │  Min Wt   │
│                                 │  Max Wt   │
│                                 │  Letter Sp│
│                                 │  Chars    │
│                                 │ ───────── │
│                                 │ Container │
│                                 │  Size     │
│                                 │  Border R │
│                                 │ ───────── │
│                                 │ Display   │
│                                 │  Invert   │
│                                 │  Pause    │
└─────────────────────────────────┴──────────┘
```

## Preset Agents

| Label | Characters | Type |
|-------|-----------|------|
| Ingest data room | `Ingest data room` | WF |
| Extract key terms | `Extract key terms` | WF |
| Flag regulatory triggers | `Flag regulatory triggers` | WF |
| Cross-ref precedent | `Cross-reference precedent` | WF |
| Draft red flag report | `Draft red flag report` | WF |
| Diligence Agent (LH) | All 5 WF names concatenated | LH |

## Default Control Values

| Control | Value |
|---------|-------|
| Speed | 12 |
| Font Size | 10 |
| Min Weight | 200 |
| Max Weight | 1000 |
| Letter Spacing | 1 |
| Canvas Size | 250px |
| Border Radius | 20% |

## Design Principles

- **Monochromatic** — `#0F0E0D` background, `#FAFAF9` text (invertible)
- **Harvey brand font** (HarveySans Diatype Variable) is the primary typeface
- **Deterministic per agent** — same name always produces the same visual pattern
- **Compact/avatar-sized** — ultimately lives in a small card UI (~64-80px) alongside agent name and metadata
- **LH agents are calm, not chaotic** — they are the connective/orchestrating layer, should feel more resolved and unified than individual WF agents
- **WF vs LH differentiation** is still being explored — current approach uses the combined character set for LH agents, but visual differentiation (wave complexity, number of sine layers, etc.) is not yet fully implemented

## Design Exploration History

Concepts explored and abandoned (for context on what's been tried):
1. Gradient orbs / geometric shapes
2. Pixel grid squares
3. Dot matrix
4. Rotating ASCII sphere (WebGL — crashed due to shader NaN)
5. Simplex noise / Perlin noise breathing mesh
6. Radial ripple with interference patterns
7. Current: **Layered sine waves with font weight mapping**

## Reference Files

- Harvey glyph SVG: `/Users/ryan/Desktop/H Glyph.svg` (80x80 rounded rect with H lettermark)
- The logo shape is a squircle/rounded rect — the canvas border radius mimics this

## What To Work On Next

1. **Fix the animation** if it's still not visibly running — this is the blocking issue
2. **WF vs LH visual differentiation** — make Long Horizon agents visually distinct from Workflow agents in a meaningful way (more resolved/unified, not more chaotic)
3. **Card preview mode** — show the visualization at realistic product scale (64-80px) inside a dark card component with agent name and metadata
4. **Multiple agents side by side** — compare different agent identities to verify they look distinct from each other
