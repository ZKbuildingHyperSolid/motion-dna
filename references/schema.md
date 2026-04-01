# Motion DNA JSON Schema Reference

Complete field documentation for the `motion_dna` JSON structure.

---

## Top-Level Structure

```
motion_dna
├── meta
├── global_defaults
├── animations[]
├── scroll_system
├── choreography
└── visual_effects
```

---

## `meta`

Metadata about the extraction.

| Field | Type | Required | Values | Description |
|-------|------|----------|--------|-------------|
| `source` | string | Yes | URL, `"source-code"`, `"screenshot"` | Where the motion data was extracted from |
| `captured_at` | string | Yes | ISO 8601 timestamp | When the extraction was performed |
| `overall_confidence` | string | Yes | `"high"`, `"medium"`, `"low"` | Reflects the weakest confidence in the entire spec |
| `libraries_detected` | string[] | Yes | Library names (e.g. `["gsap", "ScrollTrigger"]`) | Animation libraries found on the page. Empty array if none |
| `motion_personality` | string | Yes | Free text | Dominant motion feel, e.g. `"snappy and physical"`, `"slow and cinematic"`, `"playful with spring bounce"` |

**Example:**
```json
{
  "source": "https://linear.app",
  "captured_at": "2026-04-01T12:00:00Z",
  "overall_confidence": "high",
  "libraries_detected": ["framer-motion"],
  "motion_personality": "precise and snappy with subtle spring physics"
}
```

---

## `global_defaults`

Default values applied to all animations unless overridden.

| Field | Type | Required | Values | Description |
|-------|------|----------|--------|-------------|
| `duration_base_ms` | number | Yes | Positive integer | Default animation duration in milliseconds |
| `easing_default` | string | Yes | CSS cubic-bezier or keyword | Default easing curve, e.g. `"cubic-bezier(0.4, 0, 0.2, 1)"` |
| `distance_unit` | string | Yes | `"px"`, `"rem"`, `"%"` | Default unit for transform distances |
| `reduced_motion_strategy` | string | Yes | `"fade-only"`, `"instant"`, `"none"` | How to handle `prefers-reduced-motion` |

**`reduced_motion_strategy` values:**
- `"fade-only"` — Replace all transforms with simple opacity fade
- `"instant"` — Skip to final state immediately (duration: 0)
- `"none"` — No reduced motion handling

**Example:**
```json
{
  "duration_base_ms": 300,
  "easing_default": "cubic-bezier(0.4, 0, 0.2, 1)",
  "distance_unit": "px",
  "reduced_motion_strategy": "fade-only"
}
```

---

## `animations[]`

Array of individual animation definitions. Each entry is one discrete animation.

| Field | Type | Required | Values | Description |
|-------|------|----------|--------|-------------|
| `id` | string | Yes | Unique kebab-case | Unique identifier, e.g. `"hero-title-enter"` |
| `label` | string | Yes | Free text | Human-readable name. Prefix with `[cosmetic]` or `[functional]` |
| `trigger` | string | Yes | `"load"`, `"scroll-enter"`, `"scroll-exit"`, `"hover"`, `"click"`, `"programmatic"` | What initiates the animation |
| `target` | string | Yes | CSS selector or description | Element(s) being animated |
| `duration_ms` | number | Yes | Positive integer | Duration in milliseconds |
| `delay_ms` | number | Yes | Non-negative integer | Delay before animation starts |
| `easing` | string | Yes | CSS cubic-bezier or keyword | Easing curve for this animation |
| `properties` | object | Yes | See below | Animated CSS properties |
| `keyframes` | array\|null | No | Keyframe objects | Multi-step keyframes, `null` if simple from→to |
| `spring` | object | Yes | See below | Spring physics configuration |
| `stagger` | object | Yes | See below | Stagger configuration for grouped elements |
| `loop` | string | Yes | `"none"`, `"infinite"`, or integer as string | Loop behavior |
| `direction` | string | Yes | `"normal"`, `"reverse"`, `"alternate"` | Animation direction |
| `fill_mode` | string | Yes | `"forwards"`, `"backwards"`, `"both"`, `"none"` | CSS animation-fill-mode equivalent |
| `confidence` | string | Yes | `"high"`, `"medium"`, `"low"` | Confidence level for this animation's values |
| `notes` | string | Yes | Free text | Additional context, warnings, or `[estimated]` reasons |

### `properties` Object

Each property is either a `[from, to]` array of strings, or `null` if not animated.

| Field | Type | Values | Description |
|-------|------|--------|-------------|
| `opacity` | string[2]\|null | `["0", "1"]` | Opacity transition |
| `translateY` | string[2]\|null | `["24px", "0px"]` | Vertical translation |
| `translateX` | string[2]\|null | `["-100%", "0%"]` | Horizontal translation |
| `scale` | string[2]\|null | `["0.95", "1"]` | Scale transform |
| `rotate` | string[2]\|null | `["5deg", "0deg"]` | Rotation |
| `blur` | string[2]\|null | `["4px", "0px"]` | CSS filter blur |
| `clip_path` | string[2]\|null | CSS clip-path values | Clip-path animation |
| `color` | string[2]\|null | CSS color values | Color transition |

### `spring` Object

| Field | Type | Required | Values | Description |
|-------|------|----------|--------|-------------|
| `enabled` | boolean | Yes | `true`, `false` | Whether spring physics are used |
| `stiffness` | number\|null | Yes | Positive number | Spring stiffness (higher = snappier). Typical: 100-1000 |
| `damping` | number\|null | Yes | Positive number | Damping ratio (higher = less bounce). Typical: 10-100 |
| `mass` | number\|null | Yes | Positive number | Mass of the animated element. Typical: 0.1-10 |
| `velocity` | number\|null | Yes | Number | Initial velocity. Default: 0 |

**Example (Framer Motion style):**
```json
{
  "enabled": true,
  "stiffness": 400,
  "damping": 30,
  "mass": 1,
  "velocity": 0
}
```

### `stagger` Object

| Field | Type | Required | Values | Description |
|-------|------|----------|--------|-------------|
| `enabled` | boolean | Yes | `true`, `false` | Whether elements are staggered |
| `delay_ms` | number\|null | Yes | Positive integer | Delay between each element |
| `from` | string\|null | Yes | `"start"`, `"center"`, `"end"`, `"random"` | Stagger origin |
| `ease` | string\|null | No | CSS easing | Easing applied to the stagger delay distribution |

### `keyframes` Array (when not null)

For multi-step animations. Each keyframe object:

| Field | Type | Description |
|-------|------|-------------|
| `offset` | number | Position in timeline (0-1) |
| `properties` | object | Same structure as `properties` above, values at this keyframe |
| `easing` | string\|null | Easing to this keyframe |

**Example:**
```json
[
  { "offset": 0, "properties": { "opacity": "0", "translateY": "30px" }, "easing": null },
  { "offset": 0.6, "properties": { "opacity": "1", "translateY": "-5px" }, "easing": "cubic-bezier(0.16, 1, 0.3, 1)" },
  { "offset": 1, "properties": { "opacity": "1", "translateY": "0px" }, "easing": "cubic-bezier(0.4, 0, 0.2, 1)" }
]
```

---

## `scroll_system`

Configuration for scroll-driven animations and smooth scrolling.

| Field | Type | Required | Values | Description |
|-------|------|----------|--------|-------------|
| `enabled` | boolean | Yes | `true`, `false` | Whether any scroll-driven behavior exists |
| `library` | string | Yes | `"GSAP ScrollTrigger"`, `"Intersection Observer"`, `"CSS scroll-timeline"`, `"Lenis"`, `"Locomotive"`, `"none"` | Primary scroll library detected |

### `smooth_scroll` Object

| Field | Type | Required | Values | Description |
|-------|------|----------|--------|-------------|
| `enabled` | boolean | Yes | `true`, `false` | Whether smooth scrolling is active |
| `library` | string\|null | Yes | `"Lenis"`, `"Locomotive Scroll"`, `"custom"`, `null` | Smooth scroll library |
| `ease` | string\|null | No | Easing value | Scroll interpolation easing |
| `duration_ms` | number\|null | No | Positive integer | Scroll interpolation duration |

### `parallax_layers[]`

| Field | Type | Required | Values | Description |
|-------|------|----------|--------|-------------|
| `target` | string\|null | Yes | CSS selector | Element being parallaxed |
| `speed_factor` | number\|null | Yes | Float | Speed relative to scroll (1 = normal, 0.5 = half speed, 2 = double) |
| `direction` | string | Yes | `"vertical"`, `"horizontal"` | Parallax direction |
| `confidence` | string | Yes | `"high"`, `"medium"`, `"low"` | Confidence level |

### `scrub_animations[]`

| Field | Type | Required | Values | Description |
|-------|------|----------|--------|-------------|
| `target` | string\|null | Yes | CSS selector | Element being scrubbed |
| `scrub` | boolean\|number | Yes | `true`, `false`, or smoothing value | Whether animation progress is tied to scroll position |
| `start_trigger` | string\|null | Yes | ScrollTrigger syntax, e.g. `"top center"` | When the scrub begins |
| `end_trigger` | string\|null | Yes | ScrollTrigger syntax, e.g. `"bottom top"` | When the scrub ends |
| `pin` | boolean | Yes | `true`, `false` | Whether the element is pinned during scrub |
| `confidence` | string | Yes | `"high"`, `"medium"`, `"low"` | Confidence level |

---

## `choreography`

How multiple animations are orchestrated together.

| Field | Type | Required | Values | Description |
|-------|------|----------|--------|-------------|
| `global_stagger_rhythm_ms` | number\|null | Yes | Positive integer | Base stagger delay across all sequences |
| `entry_philosophy` | string | Yes | `"staggered"`, `"simultaneous"`, `"cascade"`, `"random"` | Overall entry animation approach |

### `sequences[]`

| Field | Type | Required | Values | Description |
|-------|------|----------|--------|-------------|
| `id` | string | Yes | Unique kebab-case | Sequence identifier |
| `label` | string | Yes | Free text | Human-readable name |
| `animation_ids` | string[] | Yes | Array of animation `id` values | Animations in this sequence, in order |
| `timing` | string | Yes | `"concurrent"`, `"sequential"`, `"overlap"` | How animations relate temporally |
| `overlap_ms` | number\|null | No | Integer | Overlap between sequential animations (negative = gap) |
| `notes` | string | Yes | Free text | Additional context |

**Example:**
```json
{
  "id": "hero-entrance",
  "label": "Hero section entrance sequence",
  "animation_ids": ["hero-bg-fade", "hero-title-enter", "hero-subtitle-enter", "hero-cta-enter"],
  "timing": "overlap",
  "overlap_ms": 150,
  "notes": "Each element starts 150ms after the previous begins"
}
```

---

## `visual_effects`

Advanced visual effects beyond standard CSS animations.

### `shaders`

| Field | Type | Required | Values | Description |
|-------|------|----------|--------|-------------|
| `enabled` | boolean | Yes | `true`, `false` | Whether shader effects are present |
| `type` | string\|null | No | `"fragment"`, `"vertex"`, `"compute"`, `"post-processing"` | Shader type |
| `library` | string\|null | No | `"Three.js"`, `"Pixi.js"`, `"custom GLSL"`, etc. | Library used |
| `notes` | string | Yes | Free text | Description of the effect |

### `particles`

| Field | Type | Required | Values | Description |
|-------|------|----------|--------|-------------|
| `enabled` | boolean | Yes | `true`, `false` | Whether particle effects are present |
| `library` | string\|null | No | `"tsParticles"`, `"Three.js"`, `"Canvas 2D"`, etc. | Library used |
| `count_estimate` | number\|null | No | Positive integer | Approximate particle count |
| `behavior` | string\|null | No | Free text | Particle behavior description |
| `notes` | string | Yes | Free text | Additional context |

### `webgl`

| Field | Type | Required | Values | Description |
|-------|------|----------|--------|-------------|
| `enabled` | boolean | Yes | `true`, `false` | Whether WebGL is used |
| `library` | string\|null | No | `"Three.js"`, `"Babylon.js"`, `"raw WebGL"`, etc. | Library used |
| `scene_description` | string\|null | No | Free text | Description of the 3D scene |
| `notes` | string | Yes | Free text | Additional context |

### `cursor_effects`

| Field | Type | Required | Values | Description |
|-------|------|----------|--------|-------------|
| `enabled` | boolean | Yes | `true`, `false` | Whether custom cursor effects exist |
| `type` | string\|null | No | `"trail"`, `"magnetic"`, `"custom-cursor"`, `"spotlight"`, `"repel"` | Effect type |
| `notes` | string | Yes | Free text | Description |

### `svg_animations`

| Field | Type | Required | Values | Description |
|-------|------|----------|--------|-------------|
| `enabled` | boolean | Yes | `true`, `false` | Whether SVG animations are present |
| `technique` | string | Yes | `"CSS"`, `"SMIL"`, `"JS"` | Animation technique used |
| `notes` | string | Yes | Free text | Description of SVG animations |

### `text_effects`

| Field | Type | Required | Values | Description |
|-------|------|----------|--------|-------------|
| `enabled` | boolean | Yes | `true`, `false` | Whether text effects are present |
| `type` | string | Yes | `"split-chars"`, `"split-words"`, `"scramble"`, `"typewriter"`, `"gradient-sweep"` | Effect type |
| `notes` | string | Yes | Free text | Description |

---

## Field Completeness Rules

1. **Every field must appear** in the output JSON — never omit fields
2. Use `null` for genuinely absent values — never use empty string `""` for non-string fields
3. Use empty string `""` for `notes` when there's nothing to note
4. Use empty array `[]` for list fields when none exist (e.g. `libraries_detected`, `animation_ids`)
5. Boolean fields must be explicitly `true` or `false`, never `null`
6. All `[estimated]` values must include a reason in `notes`
