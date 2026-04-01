# Motion DNA — Code Generation Guide

How to convert Motion DNA JSON into working animation code for each supported stack.

---

## Priority Sequence

When generating code from a Motion DNA JSON:

1. **Animations** — Core transforms and transitions (80% of the motion feel)
2. **Choreography** — Wire up sequences and stagger timing
3. **Scroll System** — Scroll triggers, scrub, parallax, smooth scroll
4. **Spring Physics** — Replace easing with spring where specified
5. **Visual Effects** — Shaders, particles, cursor effects, etc.
6. **Reduced Motion** — `@media (prefers-reduced-motion)` handling

---

## Stack Templates

### `react-framer` — React + Framer Motion

**Dependencies:**
```bash
npm install framer-motion
```

**Basic animation:**
```tsx
import { motion } from "framer-motion";

// motion-dna: hero-title-enter
export function HeroTitle() {
  return (
    <motion.h1
      initial={{ opacity: 0, y: 40 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{
        duration: 0.8,
        delay: 0.2,
        ease: [0.16, 1, 0.3, 1],
      }}
    >
      Title
    </motion.h1>
  );
}
```

**Spring physics:**
```tsx
// motion-dna: card-hover
<motion.div
  whileHover={{ scale: 1.05 }}
  transition={{
    type: "spring",
    stiffness: 400,
    damping: 30,
    mass: 1,
  }}
/>
```

**Stagger (parent + children):**
```tsx
// motion-dna: list-stagger
const container = {
  hidden: {},
  show: {
    transition: {
      staggerChildren: 0.08,
      staggerDirection: 1, // "start"
    },
  },
};

const item = {
  hidden: { opacity: 0, y: 24 },
  show: {
    opacity: 1,
    y: 0,
    transition: { duration: 0.6, ease: [0.16, 1, 0.3, 1] },
  },
};

<motion.ul variants={container} initial="hidden" animate="show">
  {items.map((i) => (
    <motion.li key={i} variants={item} />
  ))}
</motion.ul>
```

**Scroll-triggered (IntersectionObserver via `whileInView`):**
```tsx
// motion-dna: section-reveal
<motion.section
  initial={{ opacity: 0, y: 60 }}
  whileInView={{ opacity: 1, y: 0 }}
  viewport={{ once: true, margin: "-100px" }}
  transition={{ duration: 0.8, ease: [0.16, 1, 0.3, 1] }}
/>
```

---

### `react-gsap` — React + GSAP + ScrollTrigger

**Dependencies:**
```bash
npm install gsap
```

**Basic animation:**
```tsx
import { useRef, useEffect } from "react";
import gsap from "gsap";

// motion-dna: hero-title-enter
export function HeroTitle() {
  const ref = useRef<HTMLHeadingElement>(null);

  useEffect(() => {
    gsap.from(ref.current, {
      opacity: 0,
      y: 40,
      duration: 0.8,
      delay: 0.2,
      ease: "cubic-bezier(0.16, 1, 0.3, 1)",
    });
  }, []);

  return <h1 ref={ref}>Title</h1>;
}
```

**ScrollTrigger:**
```tsx
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);

// motion-dna: section-reveal
useEffect(() => {
  gsap.from(ref.current, {
    opacity: 0,
    y: 60,
    duration: 0.8,
    ease: "cubic-bezier(0.16, 1, 0.3, 1)",
    scrollTrigger: {
      trigger: ref.current,
      start: "top 80%",
      toggleActions: "play none none none",
    },
  });
}, []);
```

**Scrub animation:**
```tsx
// motion-dna: parallax-hero
useEffect(() => {
  gsap.to(ref.current, {
    y: -200,
    ease: "none",
    scrollTrigger: {
      trigger: ref.current,
      start: "top bottom",
      end: "bottom top",
      scrub: true,
    },
  });
}, []);
```

**Timeline (choreography):**
```tsx
// motion-dna: sequence: hero-entrance
useEffect(() => {
  const tl = gsap.timeline({ delay: 0.1 });

  tl.from(".hero-bg", { opacity: 0, duration: 0.6 })
    .from(".hero-title", { opacity: 0, y: 40, duration: 0.8 }, "-=0.45")   // overlap 150ms
    .from(".hero-subtitle", { opacity: 0, y: 24, duration: 0.6 }, "-=0.45")
    .from(".hero-cta", { opacity: 0, y: 16, duration: 0.5 }, "-=0.45");
}, []);
```

**Stagger:**
```tsx
// motion-dna: list-stagger
gsap.from(".list-item", {
  opacity: 0,
  y: 24,
  duration: 0.6,
  ease: "cubic-bezier(0.16, 1, 0.3, 1)",
  stagger: {
    each: 0.08,
    from: "start",
  },
});
```

---

### `html-gsap` — Vanilla HTML/CSS + GSAP

**Dependencies:**
```html
<script src="https://cdn.jsdelivr.net/npm/gsap@3/dist/gsap.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3/dist/ScrollTrigger.min.js"></script>
```

Same GSAP API as `react-gsap`, but without React refs. Use CSS selectors directly:

```js
// motion-dna: hero-title-enter
gsap.from(".hero h1", {
  opacity: 0,
  y: 40,
  duration: 0.8,
  delay: 0.2,
  ease: "cubic-bezier(0.16, 1, 0.3, 1)",
});
```

---

### `html-css` — Vanilla HTML/CSS Only

**No JS dependencies.** All animations via CSS.

**Basic animation:**
```css
/* motion-dna: hero-title-enter */
@keyframes hero-title-enter {
  from {
    opacity: 0;
    transform: translateY(40px);
    filter: blur(4px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
    filter: blur(0);
  }
}

.hero h1 {
  animation: hero-title-enter 800ms cubic-bezier(0.16, 1, 0.3, 1) 200ms both;
}
```

**Stagger via delay:**
```css
/* motion-dna: list-stagger */
.list-item {
  animation: fade-up 600ms cubic-bezier(0.16, 1, 0.3, 1) both;
}
.list-item:nth-child(1) { animation-delay: 0ms; }
.list-item:nth-child(2) { animation-delay: 80ms; }
.list-item:nth-child(3) { animation-delay: 160ms; }
.list-item:nth-child(4) { animation-delay: 240ms; }
```

**Scroll-driven (CSS scroll-timeline):**
```css
/* motion-dna: scroll-progress */
@keyframes scroll-progress {
  from { transform: scaleX(0); }
  to { transform: scaleX(1); }
}

.progress-bar {
  animation: scroll-progress linear;
  animation-timeline: scroll();
}
```

**Hover:**
```css
/* motion-dna: card-hover */
.card {
  transition: transform 300ms cubic-bezier(0.4, 0, 0.2, 1),
              box-shadow 300ms cubic-bezier(0.4, 0, 0.2, 1);
}
.card:hover {
  transform: translateY(-4px) scale(1.02);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.12);
}
```

---

### `vue-gsap` — Vue 3 + GSAP

**Dependencies:**
```bash
npm install gsap
```

**Basic animation:**
```vue
<script setup>
import { ref, onMounted } from "vue";
import gsap from "gsap";

// motion-dna: hero-title-enter
const titleRef = ref(null);

onMounted(() => {
  gsap.from(titleRef.value, {
    opacity: 0,
    y: 40,
    duration: 0.8,
    delay: 0.2,
    ease: "cubic-bezier(0.16, 1, 0.3, 1)",
  });
});
</script>

<template>
  <h1 ref="titleRef">Title</h1>
</template>
```

**ScrollTrigger:**
```vue
<script setup>
import { ref, onMounted } from "vue";
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";

gsap.registerPlugin(ScrollTrigger);

// motion-dna: section-reveal
const sectionRef = ref(null);

onMounted(() => {
  gsap.from(sectionRef.value, {
    opacity: 0,
    y: 60,
    duration: 0.8,
    ease: "cubic-bezier(0.16, 1, 0.3, 1)",
    scrollTrigger: {
      trigger: sectionRef.value,
      start: "top 80%",
    },
  });
});
</script>
```

---

## Field-to-API Mapping

### Properties → Transform Mapping

| JSON field | CSS | GSAP | Framer Motion |
|------------|-----|------|---------------|
| `opacity` | `opacity` | `opacity` | `opacity` |
| `translateY` | `transform: translateY()` | `y` | `y` |
| `translateX` | `transform: translateX()` | `x` | `x` |
| `scale` | `transform: scale()` | `scale` | `scale` |
| `rotate` | `transform: rotate()` | `rotation` | `rotate` |
| `blur` | `filter: blur()` | `filter: "blur(Xpx)"` | `filter: "blur(Xpx)"` |
| `clip_path` | `clip-path` | `clipPath` | `clipPath` |
| `color` | `color` | `color` | `color` |

### Easing → Stack Mapping

| JSON easing | CSS | GSAP | Framer Motion |
|-------------|-----|------|---------------|
| `cubic-bezier(a,b,c,d)` | Direct use | `"cubic-bezier(a,b,c,d)"` | `[a, b, c, d]` |
| `linear` | `linear` | `"none"` | `"linear"` |
| `ease-in-out` | `ease-in-out` | `"power2.inOut"` | `"easeInOut"` |
| `ease-out` | `ease-out` | `"power2.out"` | `"easeOut"` |

### Trigger → Stack Mapping

| JSON trigger | react-framer | GSAP (any) | html-css |
|-------------|--------------|------------|----------|
| `load` | `animate` prop | `gsap.from()` in `useEffect`/`onMounted` | CSS `animation` |
| `scroll-enter` | `whileInView` | `ScrollTrigger` | `animation-timeline` or `@starting-style` |
| `hover` | `whileHover` | CSS `:hover` or `mouseenter` listener | CSS `:hover` transition |
| `click` | `onClick` + state | `click` listener | CSS `:active` or checkbox hack |

---

## Spring Physics Cross-Stack Conversion

### Framer Motion (native spring)

```tsx
transition={{
  type: "spring",
  stiffness: 400,    // from spring.stiffness
  damping: 30,       // from spring.damping
  mass: 1,           // from spring.mass
  velocity: 0,       // from spring.velocity
}}
```

### GSAP (approximate via elastic ease)

GSAP doesn't have native spring physics. Approximate with:

```js
// For bouncy springs (low damping):
ease: "elastic.out(1, 0.3)"
// Parameters: elastic.out(amplitude, period)
// amplitude ≈ 1 / (damping / stiffness)
// period ≈ 2π / sqrt(stiffness / mass) (normalized)

// For snappy springs (high damping):
ease: "power4.out"
// High damping + high stiffness ≈ aggressive ease-out
```

**Conversion heuristic:**
| Spring feel | damping/stiffness ratio | GSAP approximation |
|------------|------------------------|---------------------|
| Very bouncy | < 0.1 | `elastic.out(1.2, 0.2)` |
| Bouncy | 0.1 - 0.3 | `elastic.out(1, 0.3)` |
| Snappy | 0.3 - 0.7 | `back.out(1.7)` |
| Critically damped | > 0.7 | `power3.out` or `power4.out` |

### CSS (no native spring)

Use cubic-bezier approximations:
```css
/* Snappy spring feel */
transition-timing-function: cubic-bezier(0.34, 1.56, 0.64, 1);

/* Bouncy spring feel */
transition-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55);
```

---

## Scroll System Implementation

### GSAP ScrollTrigger

```js
gsap.to(target, {
  // animation properties
  scrollTrigger: {
    trigger: target,
    start: startTrigger,   // from scrub_animations[].start_trigger
    end: endTrigger,       // from scrub_animations[].end_trigger
    scrub: scrubValue,     // from scrub_animations[].scrub
    pin: pinValue,         // from scrub_animations[].pin
  },
});
```

### Framer Motion (scroll-triggered)

```tsx
<motion.div
  initial={{ opacity: 0, y: 60 }}
  whileInView={{ opacity: 1, y: 0 }}
  viewport={{ once: true, margin: "-100px" }}
  transition={{ duration: 0.8 }}
/>
```

For scrub-like behavior, use `useScroll` + `useTransform`:

```tsx
import { useScroll, useTransform, motion } from "framer-motion";

const { scrollYProgress } = useScroll({ target: ref });
const y = useTransform(scrollYProgress, [0, 1], [0, -200]);

<motion.div style={{ y }} />
```

### Smooth Scroll (Lenis)

```js
import Lenis from "lenis";

const lenis = new Lenis({
  duration: 1.2,                    // from smooth_scroll.duration_ms / 1000
  easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)),  // from smooth_scroll.ease
  smoothWheel: true,
});

function raf(time) {
  lenis.raf(time);
  requestAnimationFrame(raf);
}
requestAnimationFrame(raf);
```

---

## Reduced Motion Handling

All generated code MUST include `prefers-reduced-motion` support.

### Strategy: `fade-only`

Replace all transform animations with simple opacity fade:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }

  /* Allow opacity-only transitions */
  .animated-element {
    transition: opacity 300ms ease !important;
  }
}
```

### Strategy: `instant`

Skip to final state immediately:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation: none !important;
    transition: none !important;
  }
}
```

### Strategy: `none`

No reduced motion handling (not recommended for production).

### JS detection (for GSAP / Framer Motion)

```js
const prefersReducedMotion = window.matchMedia("(prefers-reduced-motion: reduce)").matches;

if (prefersReducedMotion) {
  // Skip animations or reduce to opacity-only
  gsap.globalTimeline.timeScale(100); // instant completion
}
```

---

## Output Checklist

Before delivering generated code, verify:

- [ ] Every animation block has a `// motion-dna: {id}` comment
- [ ] All dependencies are listed with install commands
- [ ] `prefers-reduced-motion` handling is implemented
- [ ] Choreography sequences are wired in correct order
- [ ] Spring physics use stack-native implementation
- [ ] Code is self-contained and copy-paste ready
- [ ] Stagger `from` direction is correctly mapped
- [ ] Scroll triggers use correct start/end positions
