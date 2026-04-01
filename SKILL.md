# motion-dna

| name | description |
|------|-------------|
| motion-dna | Reverse-engineer and replicate UI motion from any reference. Captures animation parameters (easing, transforms, spring physics, scroll triggers, choreography) into structured Motion DNA JSON, adapts design parameters to the user's theme, and writes animation code directly into the user's project. Two modes: **capture+apply** (default — screenshot-driven, focused on one animation, writes code into user's files) and **motion audit** (full-page extraction for documentation). Triggers on: "copy this animation", "make it animate like", "replicate this motion", "capture animation", "steal this animation", "I want this hover effect", "how does this animation work", "motion audit", "analyze all motion". |

---

## Overview

Two modes for working with UI motion:

1. **Capture + Apply** (default) — User points to a specific animation → extract it → adapt design parameters to user's theme → write code directly into user's project
2. **Motion Audit** — Full-page animation extraction into Motion DNA JSON for documentation or competitive analysis

---

## Mode Detection

| Mode | Trigger phrases | What it does |
|------|----------------|--------------|
| **capture + apply** | "copy this animation", "make my hero animate like this", "I want this hover effect on my cards", "apply this motion", "steal this animation", "how does this animation work" | Extracts ONE focused animation, adapts to user's design system, writes code into user's file |
| **motion audit** | "audit all motion on this page", "analyze the motion system", "document all animations", "motion DNA audit", "analyze design+motion" | Full-page extraction → categorized Motion DNA JSON |

If ambiguous, ask: `"Do you want to capture a specific animation to apply to your project, or audit all motion on the page?"`

---

## Mode 1: Capture + Apply (Default)

### Step 1 — Receive Reference

Accept whatever the user provides and adapt:

| Input combination | Fidelity | Action |
|-------------------|----------|--------|
| Screenshot + description + URL | Highest | Playwright extracts from URL; screenshot + description narrow the target |
| Screenshot + description | High | Visual analysis identifies animation; estimate parameters |
| URL + description | High | Playwright extracts; description narrows target |
| URL only | Medium | Playwright extracts; ask user which animation they want |
| Screenshot only | Medium | Visual analysis; ask what the animation does |

**If the user provides a URL but no screenshot and no description:**
> "I can load this page and extract animations. To find the right one, could you:
> 1. Share a screenshot showing the element with the animation you want
> 2. Describe what the animation does (e.g., 'the hero text fades up on page load')
>
> Either one works — both together is ideal."

### Step 2 — Extract Animation Parameters

**Extraction Engine: Playwright (preferred)**

When a URL is provided, use Playwright via Bash to load the page in a real browser and extract animation data. This handles SPAs, client-rendered content, and dynamically loaded animations that WebFetch cannot see.

```bash
npx playwright cr --channel chrome "[URL]"
```

**Playwright extraction script** (run via `page.evaluate`):

1. **Detect animation libraries** — Check for globals:
   - `window.gsap` → GSAP
   - `window.__FRAMER_MOTION__` or `__framer_motion` in module scope → Framer Motion
   - `window.anime` → Anime.js
   - `window.Lenis` or `window.__lenis` → Lenis smooth scroll
   - `window.LocomotiveScroll` → Locomotive Scroll
   - Scan `<script>` src attributes for CDN references to known libraries

2. **Extract CSS animations** — Iterate `document.styleSheets`:
   - All `@keyframes` rules (name, keyframe stops, property values)
   - All rules with `transition`, `animation`, `animation-timeline` properties
   - All `:hover`, `:focus`, `:active` state transitions

3. **Extract computed styles** — For elements matching the user's description:
   - `window.getComputedStyle(el)` for animation, transition, transform properties
   - `el.getAnimations()` for active Web Animations API data

4. **Discover animated elements** — Query:
   - `[class*="animate"], [class*="motion"], [class*="fade"], [class*="slide"]`
   - `[data-animate], [data-scroll], [data-motion], [data-aos]`
   - Elements with non-empty `animation` or `transition` computed styles

5. **Extract JS animation configs** — Search page source for:
   - `gsap.to`, `gsap.from`, `gsap.fromTo`, `gsap.timeline` calls
   - `ScrollTrigger.create` configurations
   - `motion()` or `useAnimation` usage patterns
   - `IntersectionObserver` callback patterns

**Fallback: WebFetch**

If Playwright is unavailable, use WebFetch to fetch the page HTML/CSS. Note the limitation to the user:
> "I'm fetching the page HTML, but this site appears to be a single-page app. For higher accuracy, you can also paste the animation source code from DevTools (Elements > Styles, or Sources panel)."

**If screenshots or video provided (no URL/code):**
1. Identify the animated element from the screenshot
2. Estimate: duration, easing type, animating properties, trigger
3. Set confidence: **Low** — flag all values with `[estimated]`
4. Ask user to confirm the estimates before proceeding

**After extraction, present a natural-language summary (NOT raw JSON):**

> "I identified the animation on your target element:
> - **What moves:** Hero headline fades in from below with a blur
> - **Duration:** 800ms
> - **Easing:** cubic-bezier(0.16, 1, 0.3, 1)
> - **Properties:** opacity 0→1, translateY 40px→0, blur 4px→0
> - **Trigger:** Page load, 200ms delay
> - **Stagger:** 80ms per word, from start
> - **Confidence:** High (extracted from source code)
>
> Want me to apply this to your project? If so, point me to the component or section."

The Motion DNA JSON is generated internally as an intermediate artifact. Do not show it unless the user explicitly asks.

### Step 3 — Detect User's Project

Before writing code, understand the user's project:

1. **Auto-detect tech stack** from `package.json`:

| Signal | Stack |
|--------|-------|
| `framer-motion` in dependencies | `react-framer` |
| `gsap` + React detected | `react-gsap` |
| `gsap` + Vue detected | `vue-gsap` |
| `gsap` + no framework | `html-gsap` |
| No animation library + React | Recommend framer-motion, ask to confirm |
| No animation library + no framework | `html-css` |

2. **Read the target component/file** the user wants to animate
3. **Scan for design tokens** — Look for:
   - CSS custom properties in `:root` or theme files (`--color-*`, `--shadow-*`, `--spacing-*`)
   - Tailwind config (`tailwind.config.js/ts`) color and shadow definitions
   - Styled-components theme object
   - Any `tokens.css`, `variables.css`, `theme.ts` files

If no animation library exists:
> "Your project doesn't have an animation library yet. Based on your [React/Vue/vanilla] setup, I'd recommend [Framer Motion / GSAP / CSS-only]. Should I install it, or would you prefer a different approach?"

### Step 4 — Adapt Design Parameters

Animation code often carries design parameters (colors, shadows, gradients) from the reference site. These must be adapted to the user's design system — not blindly copied.

**Separate motion parameters from design parameters:**

| Parameter type | Examples | Action |
|---------------|----------|--------|
| **Motion** (copy directly) | duration, easing, delay, translateX/Y, scale, rotate, opacity range, spring physics, stagger timing | Use extracted values as-is |
| **Design** (adapt to user's theme) | color, background-color, box-shadow color, gradient colors, border-color, text-shadow color | Map to user's design tokens |

**Adaptation rules:**

1. **Shadow colors:** Preserve the shadow's structure (offset, blur-radius, spread-radius) but replace the color component:
   - Reference: `box-shadow: 0 12px 40px rgba(0,0,0,0.4)` (dark theme)
   - User has `--shadow-color: rgba(0,0,0,0.08)` (light theme)
   - Output: `box-shadow: 0 12px 40px var(--shadow-color)`

2. **Background/color transitions:** Map to the nearest semantic token:
   - Reference animates `background: #1a1a2e → #16213e` (dark)
   - User has `--color-surface` and `--color-surface-hover`
   - Output: animates `background: var(--color-surface) → var(--color-surface-hover)`

3. **Gradients:** Preserve direction and stop positions, replace colors:
   - Reference: `linear-gradient(135deg, #667eea 0%, #764ba2 100%)`
   - User has `--color-primary` and `--color-secondary`
   - Output: `linear-gradient(135deg, var(--color-primary) 0%, var(--color-secondary) 100%)`

4. **No design tokens found:** Use neutral, theme-agnostic values and add comments:
   ```css
   /* motion-dna: customize these colors to match your theme */
   box-shadow: 0 12px 40px rgba(0, 0, 0, 0.1);
   ```

5. **Opacity values in motion context** (e.g., fade 0→1) are motion parameters, NOT design parameters. Copy directly.

### Step 5 — Write Code Into User's Project

Write the animation code directly into the user's file using the Edit tool. Do NOT output a standalone code block for copy-paste.

**Writing rules:**
1. Import any needed libraries (if not already imported in the file)
2. Add animation logic to the specific component/element the user pointed to
3. Follow the user's existing code style (indentation, naming conventions, patterns)
4. Design parameters use the user's tokens (from Step 4)
5. Include `// motion-dna: {id}` comment for traceability
6. Include `@media (prefers-reduced-motion: reduce)` handling
7. Flag any dependency installation needed:
   > Run: `npm install framer-motion`

**After writing, provide a brief summary:**

> "Done. I added the fade-up animation to your `HeroSection` component:
> - Wrapped the `h1` with `motion.h1` and applied the entrance animation
> - Shadow colors adapted to your `--shadow-color` token
> - Added reduced-motion fallback
>
> Run `npm install framer-motion` if you haven't already.
>
> Want me to adjust the timing, easing, or any other parameters?"

---

## Mode 2: Motion Audit

Full-page animation extraction for documentation, competitive analysis, or design system reference.

### Trigger
User says: "audit all motion", "analyze the motion system", "document all animations"

### Process

1. **Extract** — Use Playwright (preferred) or WebFetch to capture ALL animations on the page
2. **Categorize** — Group animations by page section/region
3. **Present summary first** (before JSON):

> "I found 23 animations on this page:
>
> **Hero (4):** hero-title-enter, hero-subtitle-enter, hero-cta-enter, hero-bg-parallax
> **Features grid (6):** feature-card-hover, feature-icon-spin, feature-reveal, ...
> **Navigation (3):** nav-link-hover, nav-dropdown-enter, nav-scroll-hide
> **Footer (2):** footer-fade-in, footer-link-hover
>
> Want the full Motion DNA JSON, or should I apply any of these to your project?"

4. **Output Motion DNA JSON** — Full schema (see below)
5. **Escape hatch** — After showing audit results, offer to transition to capture+apply for any specific animation

### With design-dna integration

When user says "analyze design+motion":
1. Run design-dna Analyze phase → `design_system`, `design_style`, `visual_effects`
2. Run motion-dna audit → `motion_dna`
3. Merge into single JSON

---

## Motion DNA JSON Schema

Internal reference format. Used as intermediate artifact in capture+apply mode; primary output in audit mode.

Every field must be populated — use `null` for absent values, never omit fields.

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

## Analysis Rules

- **Never fabricate parameter values** — if uncertain, mark `[estimated]` and state why
- **Separate cosmetic animations** (pure aesthetics) from **functional ones** (state feedback, loading, error) — note in `label`
- **Detect the dominant motion personality** — note in `meta.motion_personality`
- **If a library is proprietary or paid**, flag it and suggest an open-source alternative in `notes`
- **Batch-similar animations** — When multiple @keyframes belong to the same pattern (e.g., 25 grid-dot variants), merge into one animation record. Use a pattern selector in `target` (e.g., `.grid-dot-*-agent`) and note the variant count and differences in `notes`
- **CSS variables** — When `color` or other property values use CSS variables (e.g., `var(--color-text)`), preserve the variable reference. Note the variable name in `notes`
- In capture+apply mode: ask `"Want me to apply this to your project? Point me to the component."` after showing the summary
- In audit mode: ask `"Want to apply any of these to your project?"` after showing results

---

## Supported Target Stacks

| Stack | Libraries used |
|-------|---------------|
| `react-framer` | React + Framer Motion |
| `react-gsap` | React + GSAP + ScrollTrigger |
| `html-gsap` | Vanilla HTML/CSS + GSAP |
| `html-css` | Vanilla HTML/CSS only (no JS library) |
| `vue-gsap` | Vue 3 + GSAP |

---

## Code Generation Rules

1. Parse animation entries and map to target stack syntax
2. Respect `choreography.sequences` — wire up timelines in correct order
3. Implement `scroll_system` using the appropriate library for the target stack
4. For `spring` entries: use stack-native spring physics (Framer Motion `type: "spring"`, GSAP elastic ease)
5. For `visual_effects` marked `enabled: true`: implement or note additional setup needed
6. **Adapt design parameters** to user's tokens (Step 4 rules)
7. Include `// motion-dna: {id}` comment on every animation block
8. Flag dependency installations needed
9. Implement `reduced_motion_strategy` via `@media (prefers-reduced-motion: reduce)`

---

## Quality Checks

Before finalizing any output, verify:

- [ ] Every `animations` entry has a unique `id`
- [ ] No field is omitted — use `null` not empty string for absent values
- [ ] All `[estimated]` values are flagged with a reason
- [ ] `overall_confidence` reflects the weakest link in the analysis
- [ ] `meta.motion_personality` is filled
- [ ] `reduced_motion_strategy` is defined
- [ ] Design parameters adapted to user's theme (not blindly copied)
- [ ] Generated code has `// motion-dna: {id}` comments on every animation block
- [ ] Code written directly into user's file (capture+apply mode), not as standalone block
