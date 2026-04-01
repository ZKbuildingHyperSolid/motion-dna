# motion-dna

| name | description |
|------|-------------|
| motion-dna | Extract, reverse-engineer, and replicate UI motion from any reference — URL, source code, or screenshots. Outputs a structured Motion DNA JSON spec covering easing, transforms, scroll systems, choreography, spring physics, and visual effects. Optionally combines with design-dna for full design+motion extraction. Use this skill when: (1) a user wants to analyze and capture motion/animation from a reference UI, (2) a user wants to replicate or "copy" the animation feel of a website or app, (3) a user has a Motion DNA JSON and wants working animation code generated from it. Triggers on: "copy animation", "replicate motion", "analyze animation", "reverse engineer animation", "motion DNA", "capture animation style", "extract animation", "how does this animation work", "make it animate like". |

---

## Overview

A three-phase workflow for reverse-engineering, structuring, and replicating UI motion across all dimensions:

1. **Analyze** — Extract motion properties from URL, source code, or screenshots into a structured Motion DNA JSON
2. **Generate** — Turn Motion DNA JSON into working animation code for any target stack
3. **Full** — Analyze design + motion together (extends design-dna output with a `motion_dna` block)

---

## Modes

Detect the user's intent and activate the correct mode:

| Mode | Trigger phrases | What it does |
|------|----------------|--------------|
| **`analyze motion`** | "analyze motion", "capture animation", "reverse engineer this", "copy animation from URL/code/screenshot" | Extracts Motion DNA JSON only. Skips design tokens. |
| **`analyze design+motion`** | "analyze design and motion", "full extract", "copy everything" | Produces complete design-dna JSON extended with a `motion_dna` block. Run design-dna phases first, then append motion_dna. |
| **`generate`** | "generate code", "implement this motion", "build animation from spec" | Takes existing Motion DNA JSON + target stack, outputs ready-to-run animation code. |

If mode is ambiguous, ask: `"Do you want motion only, or design + motion together?"`

---

## Phase 1 — Analyze

### Input Routing

**If URL provided:**
1. Fetch and load the page
2. Scan for animation libraries: GSAP, Framer Motion, Motion One, Anime.js, Lottie, Three.js, Pixi.js, Barba.js, Locomotive Scroll, Lenis
3. Extract CSS: all `@keyframes`, `transition`, `animation` declarations, `animation-timeline` (scroll-driven)
4. Extract JS: timeline configs, `gsap.to/from/fromTo`, `useAnimation`, `motion()` calls, `IntersectionObserver` callbacks, scroll event listeners
5. Note all interaction triggers: load, hover, scroll position, click, programmatic
6. Flag animations requiring user interaction: ⚠️ *"This animation triggers on [hover/click] — screenshot or DevTools capture recommended for full fidelity"*

**If source code provided (CSS/JS/TS):**
1. Parse directly — highest fidelity path, no browser step needed
2. Extract all animation declarations, library config objects, trigger conditions
3. Treat as ground truth

**If screenshots or video provided:**
1. Identify start state → mid state(s) → end state per animated element
2. Estimate duration by visual rhythm feel
3. Classify easing curve: linear / ease-out / ease-in-out / spring/bounce / custom
4. Identify animating properties: opacity, transform (translate/scale/rotate), clip-path, filter, color, blur
5. Note stagger relationships and choreography order
6. Set confidence: **Low** — always flag estimated values with `[estimated]`

**If both URL and source code provided:**
- Source code = ground truth for parameters
- URL = visual reference to verify behavior

---

### Confidence Levels

| Level | Meaning |
|-------|---------|
| **High** | Extracted directly from source code or computed styles |
| **Medium** | Inferred from DOM inspection or library detection |
| **Low** | Estimated from visual observation only |

Always mark individual estimated values with `[estimated]` inline.

---

### Motion DNA JSON Schema

Output one complete JSON. Every field must be populated — use `null` for genuinely absent values, never omit fields.

```json
{
  "motion_dna": {
    "meta": {
      "source": "URL or 'source-code' or 'screenshot'",
      "captured_at": "ISO timestamp",
      "overall_confidence": "high | medium | low",
      "libraries_detected": [],
      "motion_personality": "e.g. snappy and physical / slow and cinematic / playful with spring bounce"
    },

    "global_defaults": {
      "duration_base_ms": 300,
      "easing_default": "cubic-bezier(0.4, 0, 0.2, 1)",
      "distance_unit": "px | rem | %",
      "reduced_motion_strategy": "fade-only | instant | none"
    },

    "animations": [
      {
        "id": "unique-kebab-case-id",
        "label": "Human readable name",
        "trigger": "load | scroll-enter | scroll-exit | hover | click | programmatic",
        "target": "CSS selector or element description",
        "duration_ms": 600,
        "delay_ms": 0,
        "easing": "cubic-bezier(0.16, 1, 0.3, 1)",
        "properties": {
          "opacity": ["0", "1"],
          "translateY": ["24px", "0px"],
          "translateX": null,
          "scale": null,
          "rotate": null,
          "blur": null,
          "clip_path": null,
          "color": null
        },
        "keyframes": null,
        "spring": {
          "enabled": false,
          "stiffness": null,
          "damping": null,
          "mass": null,
          "velocity": null
        },
        "stagger": {
          "enabled": false,
          "delay_ms": null,
          "from": "start | center | end | random",
          "ease": null
        },
        "loop": "none | infinite | 3",
        "direction": "normal | reverse | alternate",
        "fill_mode": "forwards | backwards | both | none",
        "confidence": "high | medium | low",
        "notes": ""
      }
    ],

    "scroll_system": {
      "enabled": false,
      "library": "GSAP ScrollTrigger | Intersection Observer | CSS scroll-timeline | Lenis | Locomotive | none",
      "smooth_scroll": {
        "enabled": false,
        "library": null,
        "ease": null,
        "duration_ms": null
      },
      "parallax_layers": [
        {
          "target": null,
          "speed_factor": null,
          "direction": "vertical | horizontal",
          "confidence": "high | medium | low"
        }
      ],
      "scrub_animations": [
        {
          "target": null,
          "scrub": true,
          "start_trigger": null,
          "end_trigger": null,
          "pin": false,
          "confidence": "high | medium | low"
        }
      ]
    },

    "choreography": {
      "sequences": [
        {
          "id": "sequence-id",
          "label": "Human readable sequence name",
          "animation_ids": [],
          "timing": "concurrent | sequential | overlap",
          "overlap_ms": null,
          "notes": ""
        }
      ],
      "global_stagger_rhythm_ms": null,
      "entry_philosophy": "staggered | simultaneous | cascade | random"
    },

    "visual_effects": {
      "shaders": {
        "enabled": false,
        "type": null,
        "library": null,
        "notes": ""
      },
      "particles": {
        "enabled": false,
        "library": null,
        "count_estimate": null,
        "behavior": null,
        "notes": ""
      },
      "webgl": {
        "enabled": false,
        "library": null,
        "scene_description": null,
        "notes": ""
      },
      "cursor_effects": {
        "enabled": false,
        "type": null,
        "notes": ""
      },
      "svg_animations": {
        "enabled": false,
        "technique": "CSS | SMIL | JS",
        "notes": ""
      },
      "text_effects": {
        "enabled": false,
        "type": "split-chars | split-words | scramble | typewriter | gradient-sweep",
        "notes": ""
      }
    }
  }
}
```

---

### Analysis Rules

- **Never fabricate parameter values** — if uncertain, mark `[estimated]` and state why
- **Separate cosmetic animations** (pure aesthetics) from **functional ones** (state feedback, loading, error) — note in `label`
- **Detect the dominant motion personality** — note in `meta.motion_personality`
- **If a library is proprietary or paid**, flag it and suggest an open-source alternative in `notes`
- After outputting JSON, ask: `"Want to adjust any values before generating code?"`

---

## Phase 2 — Generate

When user provides Motion DNA JSON + target stack:

### Supported Stacks

| Stack | Libraries used |
|-------|---------------|
| `react-framer` | React + Framer Motion |
| `react-gsap` | React + GSAP + ScrollTrigger |
| `html-gsap` | Vanilla HTML/CSS + GSAP |
| `html-css` | Vanilla HTML/CSS only (no JS library) |
| `vue-gsap` | Vue 3 + GSAP |

If stack not specified, ask: `"Which stack? (react-framer / react-gsap / html-gsap / html-css / vue-gsap)"`

### Generation Rules

1. Parse all `animations` entries and map to target stack syntax
2. Respect `choreography.sequences` — wire up timelines in correct order
3. Implement `scroll_system` using the appropriate library for the target stack
4. For `spring` entries: use stack-native spring physics (Framer Motion `type: "spring"`, or GSAP elastic ease)
5. For `visual_effects` marked `enabled: true`: implement using appropriate library, or note if requires additional setup
6. Output clean, self-contained, copy-paste ready code
7. Every code block must reference its animation `id` as a comment: `// motion-dna: hero-enter`
8. Flag any dependency requiring installation:
   > `npm install framer-motion`
9. Implement `reduced_motion_strategy` via `@media (prefers-reduced-motion: reduce)`

---

## Phase 3 — Full (design + motion)

When user requests `analyze design+motion`:

1. Run design-dna Analyze phase first → produces `design_system`, `design_style`, `visual_effects` blocks
2. Run motion-dna Analyze phase → produces `motion_dna` block
3. Merge into single JSON output:

```json
{
  "design_system": { ... },
  "design_style": { ... },
  "visual_effects": { ... },
  "motion_dna": { ... }
}
```

4. The merged JSON is the complete portable spec — version-controllable, shareable, stack-agnostic

---

## Quality Checks

Before finalizing any output, verify:

- [ ] Every `animations` entry has a unique `id`
- [ ] No field is omitted — use `null` not empty string for absent values
- [ ] All `[estimated]` values are flagged with a reason
- [ ] `overall_confidence` reflects the weakest link in the analysis
- [ ] `meta.motion_personality` is filled
- [ ] `reduced_motion_strategy` is defined
- [ ] Generated code has `// motion-dna: {id}` comments on every animation block
