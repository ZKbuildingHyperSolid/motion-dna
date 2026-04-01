# motion-dna

**Reverse-engineer, structure, and replicate UI motion from any reference.**

An agent skill that extracts animation parameters from URLs, source code, or screenshots into a structured Motion DNA JSON spec — then generates working animation code for any target stack.

> Companion to [design-dna](https://github.com/zanwei/design-dna). design-dna captures visual style; motion-dna captures how things move. Both output compatible JSON that can be merged into a single portable spec.

## Installation

```bash
npx skills add <your-username>/motion-dna
```

Works with 40+ agents including Claude Code, Cursor, and GitHub Copilot.

## How It Works

```
Reference Source              Motion DNA JSON              Target Code
───────────────              ─────────────────            ───────────
URL / Code / Screenshot  →   Structured motion spec   →   Any stack
```

### Three Modes

| Mode | What to say | What it does |
|------|-------------|--------------|
| **Analyze motion** | "analyze motion [URL]" | Extracts Motion DNA JSON only |
| **Analyze design+motion** | "analyze design+motion [URL]" | Full design-dna + motion-dna merged output |
| **Generate** | "generate react-framer" | Converts existing JSON to runnable code |

### Input Sources

| Source | Precision | Notes |
|--------|-----------|-------|
| Source code (CSS/JS/TS) | **Highest** | Direct parse, parameter-level accuracy |
| Live URL | High | Fetches DOM, computed styles, JS libraries |
| Screenshots / video | Medium-Low | Visual estimation, marked `[estimated]` |

**Best practice:** URL + DevTools source code snippets together for maximum accuracy.

### Target Stacks

| Stack ID | Libraries |
|----------|-----------|
| `react-framer` | React + Framer Motion |
| `react-gsap` | React + GSAP + ScrollTrigger |
| `html-gsap` | Vanilla HTML/CSS + GSAP |
| `html-css` | Vanilla HTML/CSS only |
| `vue-gsap` | Vue 3 + GSAP |

## What It Captures

Every animation is extracted down to parameter-level precision:

- **Easing / Timing** — cubic-bezier values, duration, delay
- **Transforms** — translateX/Y, scale, rotate, opacity, clip-path, blur
- **Spring Physics** — stiffness, damping, mass, velocity
- **Stagger / Choreography** — element orchestration, delay sequences, concurrent/sequential logic
- **Scroll-driven** — ScrollTrigger, IntersectionObserver, scroll-timeline, parallax
- **Visual Effects** — shaders, WebGL, particles, cursor effects, SVG animation, text effects

## Example Output

```json
{
  "motion_dna": {
    "meta": {
      "source": "https://example.com",
      "captured_at": "2026-04-01T12:00:00Z",
      "overall_confidence": "high",
      "libraries_detected": ["gsap", "ScrollTrigger", "Lenis"],
      "motion_personality": "snappy and physical with scroll-driven depth"
    },
    "global_defaults": {
      "duration_base_ms": 400,
      "easing_default": "cubic-bezier(0.16, 1, 0.3, 1)",
      "distance_unit": "px",
      "reduced_motion_strategy": "fade-only"
    },
    "animations": [
      {
        "id": "hero-title-enter",
        "label": "Hero title fade-up on load",
        "trigger": "load",
        "target": ".hero h1",
        "duration_ms": 800,
        "delay_ms": 200,
        "easing": "cubic-bezier(0.16, 1, 0.3, 1)",
        "properties": {
          "opacity": ["0", "1"],
          "translateY": ["40px", "0px"],
          "translateX": null,
          "scale": null,
          "rotate": null,
          "blur": ["4px", "0px"],
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
          "from": null,
          "ease": null
        },
        "loop": "none",
        "direction": "normal",
        "fill_mode": "forwards",
        "confidence": "high",
        "notes": ""
      }
    ],
    "scroll_system": { "enabled": true, "..." : "..." },
    "choreography": { "..." : "..." },
    "visual_effects": { "..." : "..." }
  }
}
```

## Design Principles

- **Spec stays pure JSON** — no code mixed in, ensuring cross-stack portability
- **Confidence is transparent** — every value is tagged High/Medium/Low; estimates marked `[estimated]`
- **Never fabricate** — unknown values are `null`, never guessed
- **Accessibility built-in** — all generated code includes `@media (prefers-reduced-motion)` support
- **Traceable** — generated code includes `// motion-dna: {id}` comments linking back to the spec

## Relationship to design-dna

| | design-dna | motion-dna |
|---|---|---|
| Focus | Visual design tokens | Animation parameters |
| Motion support | Minimal, low precision | Full, parameter-level |
| JSON compatible | — | ✅ Mergeable |

When used together:

```json
{
  "design_system": { "..." },
  "design_style": { "..." },
  "visual_effects": { "..." },
  "motion_dna": { "..." }
}
```

## Documentation

- [Project Brief](docs/01-brief.md) — Background and positioning
- [Project Introduction](docs/02-project-intro.md) — Full feature overview
- [Goals](docs/03-goals.md) — Technical and product objectives
- [Schema Reference](references/schema.md) — Complete field documentation
- [Generation Guide](references/generation-guide.md) — Code generation specs per stack

## License

[MIT](LICENSE)
