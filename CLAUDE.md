# YOGA Ashtanga Simulator — Project Context for Claude

## What This Project Is

A fun, tactile ragdoll-physics yoga trainer built as a browser game. The player drags a human ragdoll into exact Ashtanga yoga poses on a yoga mat. Visual style is modeled after the male figure in `NøsenYoga_Ashtanga_Sheet_v6.pdf` — clean, minimalist illustrations of Surya Namaskar A/B sequences.

Target platform: browser (desktop + mobile), hosted on itch.io with zero server dependencies.

---

## Tech Stack

- **Phaser 3** (latest, loaded from CDN) — game framework
- **Matter.js** (Phaser built-in) — physics engine for ragdoll joints
- **Single HTML file** — MVP is one self-contained `index.html`
- **No build tools** — runs directly in browser, no bundler/transpiler
- **Assets:** inline or CDN-loaded; body parts are colored rectangles with rounded joints (sprite-ready later)

---

## Repository Structure

```
Yoga Simulator/
├── CLAUDE.md                              ← this file
├── NøsenYoga_Ashtanga_Sheet_v6.pdf        ← reference: pose angles & body proportions
├── index.html                             ← the game (single-file MVP)
└── assets/                                ← future: sprite sheets, sounds, mat texture
```

---

## Game Architecture

### Core Loop (MVP — Downward Dog only)

1. Scene loads: yoga mat background + ragdoll standing in **Samasthiti** (neutral upright)
2. Semi-transparent **ghost silhouette** of target pose (Adho Mukha Svanasana) fades in
3. Player **drags any body part** (mouse/touch) to shape ragdoll into the pose
4. When alignment is close enough → **LOCK** button appears (or auto-detect triggers)
5. **5-breath hold timer** starts — player must keep pose stable
6. **Strain system:** misalignment + time = trembling, red tint, sweat particles, eventual collapse
7. 5 clean breaths = success (score, confetti, "Namaste!")
8. Collapse = funny fail animation + retry with helpful tip

### Ragdoll System

- **9–11 rectangular body parts:** head, torso, upper arms (2), lower arms (2), upper legs (2), lower legs (2), feet (2)
- Connected with **Matter.js revolute (pin) joints** with realistic angle limits
- Side-view 2D perspective matching the PDF illustrations
- Body proportions taken from the PDF male model

### Pose Detection

- Compare **key joint angles** against ideal values for each pose:
  - Shoulder-hip angle
  - Hip-knee angle
  - Knee-ankle angle
  - Wrist-to-floor distance
  - Spine straightness
- **Downward Dog ideal:** inverted V — hands and feet on ground, hips high, arms and legs straight, ~60° hip angle
- Alignment scored as percentage; threshold triggers lock/hold phase

### Breath System

- Visual chest expansion/contraction cycle
- Inhale/exhale sound cues (placeholder beeps initially)
- Particle "air" puffs on exhale
- 5 full breath cycles = ~25 seconds hold

### Strain System

- Real-time angle deviation from ideal → increasing torque jitter on joints
- Visual feedback: body part trembling, red tint overlay, sweat particle emitter
- Strain meter UI element fills over time
- Exceeding strain threshold → collapse animation

### Scoring

- **Alignment %** — how close joint angles are to ideal
- **Strain control** — less deviation during hold = higher score
- **Time bonus** — faster setup = more points

---

## Pose Data Format

Each pose is defined as a config object:

```javascript
const POSES = {
  downward_dog: {
    name: "Adho Mukha Svanasana",
    displayName: "Downward Facing Dog",
    sequence: "Surya Namaskar B",
    sequencePosition: 6,
    jointAngles: {
      leftShoulder: { min: 160, ideal: 180, max: 200 },
      leftHip: { min: 50, ideal: 60, max: 75 },
      leftKnee: { min: 165, ideal: 180, max: 190 },
      // ... mirrored for right side
    },
    groundContacts: ['leftHand', 'rightHand', 'leftFoot', 'rightFoot'],
    breathCount: 5,
    tips: [
      "Press your heels toward the floor",
      "Keep your arms straight and ears between biceps",
      "Push your hips up and back"
    ]
  }
};
```

To add new poses: copy the structure, set joint angles from the PDF reference, add to POSES object.

---

## Visual Style

- **Color palette:** soft oranges, greens, blues — calming, matching PDF aesthetic
- **Background:** simple yoga mat rectangle with subtle wood-floor texture
- **Body parts:** colored rectangles (warm skin tone) with rounded joint circles
- **Ghost pose:** same shape but semi-transparent green/blue
- **Effects:** sweat particles, gentle screen shake on high strain, confetti on success
- **UI:** minimal — pose name, breath counter, strain meter, score

---

## Poses Roadmap (from NøsenYoga PDF)

### Surya Namaskar A (page 1 of PDF)
1. Samasthiti (neutral standing) — starting position, no scoring
2. Urdhva Hastasana (arms up)
3. Uttanasana (forward fold)
4. Ardha Uttanasana (half lift)
5. Chaturanga Dandasana (low plank)
6. Urdhva Mukha Svanasana (upward dog)
7. **Adho Mukha Svanasana (downward dog)** ← MVP pose

### Surya Namaskar B (page 1–2 of PDF)
8. Utkatasana (chair pose)
9. Virabhadrasana I (warrior I)
10. Virabhadrasana II (warrior II)

### Standing Sequence (pages 2–3)
11. Trikonasana (triangle)
12. Parivrtta Trikonasana (revolved triangle)
13. And more...

---

## Input Handling

- **Desktop:** Mouse drag on any body part; Matter.js mouse constraint
- **Mobile:** Touch drag with same constraint; prevent default scroll/zoom on canvas
- **Multi-touch:** future — drag two limbs simultaneously

---

## Development Conventions

- All game code in a single `index.html` for MVP; modularize into separate JS files only when complexity demands it
- Phaser scenes: `BootScene` (load assets) → `GameScene` (main gameplay) → `UIScene` (overlay HUD)
- Matter.js bodies use descriptive labels: `body_torso`, `body_upperArmL`, `body_lowerLegR`, etc.
- Joint angle calculations use `Phaser.Math.Angle.Between()` or `Math.atan2()`
- All magic numbers (angles, thresholds, timings) go in a `CONFIG` object at the top of the file
- Comments explain the "why" for physics tuning values

---

## Performance Notes

- Target: 60fps on mid-range mobile (iPhone SE 2022, Galaxy A54)
- Ragdoll = ~11 physics bodies + ~10 joints — lightweight for Matter.js
- Particle effects capped at 50 particles max
- Canvas renderer fallback if WebGL unavailable
- Touch events use passive listeners where possible

---

## Reference Material

- `NøsenYoga_Ashtanga_Sheet_v6.pdf` — pose angles, body proportions, sequence order
- Phaser 3 docs: https://phaser.io/docs/3.80
- Matter.js docs: https://brm.io/matter-js/docs/

---

## Git Branches

- `main` — stable, playable builds
- `development` — active work
