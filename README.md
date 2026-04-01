# motion-dna

**Reverse-engineer and replicate UI motion from any reference — adapted to your design system.**

An agent skill that extracts animation parameters from any website, adapts colors and shadows to your theme, and writes the animation code directly into your project.

> Companion to [design-dna](https://github.com/zanwei/design-dna). design-dna captures visual style; motion-dna captures how things move. Both output compatible JSON that can be merged.

## Installation

```bash
npx skills add ZKbuildingHyperSolid/motion-dna
```

Works with 40+ agents including Claude Code, Cursor, and GitHub Copilot.

## How It Works

### Capture + Apply (Default)

The primary workflow — point to an animation you like, and motion-dna applies it to your project:

```
1. You: "Copy this hover effect to my card component" + screenshot
   ↓
2. motion-dna: Loads page with Playwright, extracts exact parameters
   ↓
3. motion-dna: Adapts colors/shadows to your design tokens
   ↓
4. motion-dna: Writes animation code directly into your component file
```

**What to say:**
- "Copy this animation to my project" + screenshot/URL
- "Make my hero section animate like this site"
- "I want this hover effect on my cards"
- "How does this animation work?" + URL

**What you provide → What you get:**

| Your input | Result |
|------------|--------|
| Screenshot + "I want this fade-in" | Identifies animation, asks which component to apply it to |
| URL + "copy the hero animation" | Loads page, extracts hero animation, writes code into your file |
| Screenshot + URL + description | Highest accuracy — visual reference + source code extraction |

### Motion Audit

Full-page animation extraction for documentation or competitive analysis:

```
You: "Audit all motion on linear.app"
   ↓
motion-dna: Categorized summary of all 23 animations found
   ↓
You: "Apply the hero-title-enter to my project"
   ↓
motion-dna: Writes it into your component
```

**What to say:**
- "Audit all motion on this page"
- "Document the motion system"
- "Analyze design+motion" (combines with design-dna)

## Key Features

### Smart Extraction

Uses Playwright to load pages in a real browser — handles SPAs, client-rendered content, and dynamically loaded animations. Falls back to WebFetch when Playwright isn't available.

Captures everything down to parameter-level precision:
- **Easing / Timing** — cubic-bezier values, duration, delay
- **Transforms** — translateX/Y, scale, rotate, opacity, clip-path, blur
- **Spring Physics** — stiffness, damping, mass, velocity
- **Stagger / Choreography** — element orchestration, delay sequences
- **Scroll-driven** — ScrollTrigger, IntersectionObserver, parallax
- **Visual Effects** — shaders, WebGL, particles, cursor effects, text effects

### Design-Aware Adaptation

When applying animations to your project, motion-dna separates **motion parameters** (copy directly) from **design parameters** (adapt to your theme):

| Copied as-is | Adapted to your theme |
|--------------|----------------------|
| Duration, easing, delay | Colors, backgrounds |
| Transforms (translate, scale, rotate) | Box-shadow colors |
| Spring physics | Gradient colors |
| Stagger timing | Border colors |
| Opacity range | Text-shadow colors |

A dark-theme hover effect applied to your light-theme project will use your `--shadow-color` and `--color-surface` tokens instead of hardcoded dark values.

### Direct Code Writing

motion-dna writes animation code directly into your project files — no copy-paste needed. It:
- Auto-detects your tech stack from `package.json`
- Follows your existing code style
- Imports required libraries
- Adds `prefers-reduced-motion` accessibility support
- Includes `// motion-dna: {id}` traceability comments

### Target Stacks

| Stack ID | Libraries |
|----------|-----------|
| `react-framer` | React + Framer Motion |
| `react-gsap` | React + GSAP + ScrollTrigger |
| `html-gsap` | Vanilla HTML/CSS + GSAP |
| `html-css` | Vanilla HTML/CSS only |
| `vue-gsap` | Vue 3 + GSAP |

Stack is auto-detected from your project. If no animation library exists, motion-dna recommends one and asks before installing.

## Relationship to design-dna

| | design-dna | motion-dna |
|---|---|---|
| Focus | Visual design tokens | Animation parameters |
| Motion support | Minimal | Full, parameter-level |
| JSON compatible | — | ✅ Mergeable |

When used together (`analyze design+motion`):

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
- [Schema Reference](references/schema.md) — Complete Motion DNA JSON field documentation
- [Generation Guide](references/generation-guide.md) — Code generation specs per stack

## License

[MIT](LICENSE)
