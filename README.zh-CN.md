# motion-dna

**从任意参考来源逆向工程、结构化、复刻 UI 动效。**

一个 Agent Skill，从 URL、源码或截图中提取动效参数，输出结构化的 Motion DNA JSON 规范，再基于规范生成任意目标技术栈的可运行动效代码。

> [design-dna](https://github.com/zanwei/design-dna) 的动效伙伴。design-dna 捕捉视觉风格；motion-dna 捕捉事物如何运动。两者输出兼容的 JSON，可合并为单一可移植规范。

## 安装

```bash
npx skills add <your-username>/motion-dna
```

兼容 40+ 种 Agent，包括 Claude Code、Cursor、GitHub Copilot。

## 工作原理

```
参考来源                    Motion DNA JSON              代码输出
───────────────            ─────────────────            ───────────
URL / 源码 / 截图    →     结构化动效参数规范     →     任意技术栈代码
```

### 三种模式

| 模式 | 使用指令 | 功能 |
|------|----------|------|
| **分析动效** | "analyze motion [URL]" | 仅提取 Motion DNA JSON |
| **分析设计+动效** | "analyze design+motion [URL]" | design-dna + motion-dna 合并输出 |
| **生成代码** | "generate react-framer" | 将现有 JSON 转为可运行代码 |

### 输入来源

| 来源 | 精度 | 说明 |
|------|------|------|
| 前端源码 (CSS/JS/TS) | **最高** | 直接解析，参数级精度 |
| 线上 URL | 高 | 抓取 DOM + computed styles + JS 库 |
| 截图 / 视频录屏 | 中-低 | 视觉估算，标注 `[estimated]` |

**最佳实践：** URL + DevTools 导出的源码片段组合使用，精度最优。

### 目标技术栈

| Stack ID | 技术组合 |
|----------|---------|
| `react-framer` | React + Framer Motion |
| `react-gsap` | React + GSAP + ScrollTrigger |
| `html-gsap` | 原生 HTML/CSS + GSAP |
| `html-css` | 原生 HTML/CSS（无 JS 库）|
| `vue-gsap` | Vue 3 + GSAP |

## 捕捉维度

每条动效精确到参数级别：

- **Easing / Timing** — cubic-bezier 值、duration、delay
- **Transform 序列** — translateX/Y、scale、rotate、opacity、clip-path、blur
- **Spring 物理参数** — stiffness、damping、mass、velocity
- **Stagger / 编排** — 子元素编排延迟、序列时序、并发/顺序逻辑
- **Scroll 驱动** — ScrollTrigger、IntersectionObserver、scroll-timeline、parallax
- **视觉效果** — Shader、WebGL、粒子、光标效果、SVG 动画、文字效果

## 示例输出

```json
{
  "motion_dna": {
    "meta": {
      "source": "https://example.com",
      "captured_at": "2026-04-01T12:00:00Z",
      "overall_confidence": "high",
      "libraries_detected": ["gsap", "ScrollTrigger", "Lenis"],
      "motion_personality": "干脆利落，带有滚动驱动的深度感"
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
        "label": "首屏标题加载淡入上移",
        "trigger": "load",
        "target": ".hero h1",
        "duration_ms": 800,
        "delay_ms": 200,
        "easing": "cubic-bezier(0.16, 1, 0.3, 1)",
        "properties": {
          "opacity": ["0", "1"],
          "translateY": ["40px", "0px"]
        },
        "confidence": "high"
      }
    ]
  }
}
```

## 设计原则

- **Spec 纯净** — JSON 只存参数，不混入代码，保证跨栈可移植性
- **置信度透明** — 每个值标注 High/Medium/Low；估算值标记 `[estimated]`
- **不伪造数据** — 未知值为 `null`，绝不猜测
- **无障碍内置** — 所有生成代码包含 `@media (prefers-reduced-motion)` 支持
- **可追溯** — 生成代码包含 `// motion-dna: {id}` 注释，链接回规范

## 与 design-dna 的关系

| | design-dna | motion-dna |
|---|---|---|
| 聚焦 | 视觉设计 Token | 动效参数 |
| 动效支持 | 极少，低精度 | 完整，参数级 |
| JSON 兼容 | — | ✅ 可合并 |

联动使用时：

```json
{
  "design_system": { "..." },
  "design_style": { "..." },
  "visual_effects": { "..." },
  "motion_dna": { "..." }
}
```

## 文档

- [项目背景](docs/01-brief.md) — 背景与定位
- [项目介绍](docs/02-project-intro.md) — 完整功能概览
- [项目目标](docs/03-goals.md) — 技术与产品目标
- [Schema 参考](references/schema.md) — 完整字段说明
- [代码生成指南](references/generation-guide.md) — 各栈代码生成规范

## 许可证

[MIT](LICENSE)
