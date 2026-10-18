# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

A browser-based yoga sequence trainer (Surya Namaskar A). The player holds keyboard combos to flow a stick-figure body through 11 poses, breathes with SPACE, and holds Downward Dog for 5 breath cycles to complete the sequence. Hosted on itch.io with zero server dependencies.

Open `index.html` in a browser to play.

---

## Tech Stack

- **Phaser 3.80.1** (CDN) — game loop, scene, input, particles, tweens
- **No Matter.js** — physics was replaced by IK (inverse kinematics)
- **Single HTML file** — one `<script>` block, ~660 lines
- **No build tools**

---

## Architecture

### Single Scene (`Game extends Phaser.Scene`)

Everything lives in one scene with these Phaser Graphics layers (by depth):
- `gBg` (0) — static background drawn once in `create()`
- `gMentor` (1) — static seated teacher silhouette, drawn once
- `gGhost` (2) — target pose ghost, redrawn each frame
- `gBody` (5) — player body, redrawn each frame
- `gBreath` (7) — breath circle, redrawn each frame
- `gUI` (8) — key combo feedback

### Body Representation — IK, Not Physics

The body is **not** a physics simulation. It uses five anchor points:

```
this.cur = { hip, handL, handR, footL, footR }
```

The `skel(cp, spa)` function derives all other joints (chest, neck, head, shoulders, elbows, knees) via the `ik()` 2D inverse kinematics solver. `spa` = spine angle.

The `BONES` array defines 10 segments connecting skeleton joints, with per-bone widths (`LW`).

### Pose System

Each pose in `P` is a config of the 5 anchor positions + spine angle (`spa`). There are 7 distinct poses:
`sama` (standing), `urdhva` (arms up), `uttana` (fold), `ardha` (half lift), `chatur` (low plank), `updog`, `ddog` (downward dog).

### Sequence and Pose Blending

`SEQ` is an 11-step array. Each step has:
- `tgt` — target pose (`P.*`)
- `keys` — subset of `['Q','W','E','A','D']` the player must hold
- `breath` — `'in'` (hold SPACE) | `'out'` (release SPACE) | `null`
- `type` — `'start'` | `'flow'` | `'hold'` | `'end'`

Key/breath correctness drives `poseBlend` (0→1). The body lerps between `P.sama` (standing) and `tgt` based on `poseBlend`:

```
correct? → poseBlend blends toward 1 at speed 3.5
wrong?   → poseBlend blends back to 0 at speed 1.5
```

Ghost silhouette (`this.ghost`) tweens to target anchors via `this.tweens.add()` when advancing to a new step.

### Breath System

`this.breath` (0–1) grows when SPACE is held (~1.2s to fill), shrinks when released (~1.7s to empty). A breath circle is drawn at the right side of the screen (blue tint = inhale, amber = exhale).

In the hold state (Downward Dog), one full breath cycle = Space held past 0.5 then released past 0.3. 5 breaths complete the hold.

### State Machine

```
ST = { INTRO: 0, FLOW: 1, HOLD: 2, COMPLETE: 3, FAIL: 4 }
```

Flow steps advance automatically when `poseBlend > 0.92` with correct input. The hold step requires 5 full breath cycles. Strain accumulates when incorrect and triggers `onFail()` at 1.0.

### Audio

Single `AudioContext` (`actx`) created on first keypress. Functions: `sfxSnap()`, `sfxBreath()`, `sfxWin()`, `sfxFail()`, `sfxCount(n)`.

Ambient music (`mStart()`): 3 sine oscillators at 130/196/261 Hz with subtle LFO modulation + random crystal bowl tones every 3–9s. Toggle with M key. Music auto-starts on first keypress.

---

## Key Bindings

| Key | Meaning |
|---|---|
| Q | Left hand anchor |
| W | Hip anchor |
| E | Right hand anchor |
| A | Left foot anchor |
| D | Right foot anchor |
| SPACE | Inhale (hold) / Exhale (release) |
| SPACE (intro) | Start sequence |
| R | Retry (complete/fail screens) |
| M | Toggle ambient music |

---

## Adding New Poses

1. Add a pose config to `P` with `{ hip, handL, handR, footL, footR, spa }` — anchor positions in screen coords.
2. Add a step to `SEQ` with the matching `tgt`, correct `keys` subset, `breath`, and `type`.
3. Derive `spa` from the spine angle using `Math.atan2` against the PDF reference.

---

## Performance Notes

- Target 60fps on mid-range mobile
- Background drawn once in `create()` — do not redraw it in the update loop
- Particle effects capped at 50 max
- Canvas renderer fallback if WebGL unavailable

---

## Reference Material

- `NøsenYoga_Ashtanga_Sheet_v6.pdf` — pose angles, body proportions, sequence order
- Phaser 3 docs: https://phaser.io/docs/3.80

---

## Git Branches

- `main` — stable builds
- `development` — active work
