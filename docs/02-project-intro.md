# Motion DNA — 项目介绍

## 是什么

**motion-dna** 是一个 Claude Code Skill，用于对任意网页或前端界面的动效进行逆向工程，输出结构化的 Motion DNA JSON 规范，并基于该规范生成可运行的动效代码。

## 工作流程

```
参考来源                    Motion DNA JSON              代码输出
─────────────────          ─────────────────────         ──────────────────
URL / 源码 / 截图   ──►    结构化动效参数规范    ──►    任意目标技术栈代码
```

三个阶段：
1. **Analyze** — 从参考来源提取动效参数，输出 Motion DNA JSON
2. **Generate** — 将 Motion DNA JSON 转化为目标栈的可运行代码
3. **Full** — 同时提取设计 + 动效（与 design-dna 联动）

## 三种工作模式

| 模式 | 指令 | 用途 |
|------|------|------|
| `analyze motion` | `analyze motion [URL/代码/截图]` | 只提取动效 |
| `analyze design+motion` | `analyze design+motion [URL]` | 提取完整设计 + 动效 |
| `generate` | `generate [stack]` | 从 JSON 生成代码 |

## 支持的输入来源

| 来源 | 精度 | 说明 |
|------|------|------|
| 前端源码 (CSS/JS/TS) | **最高** | 直接解析，参数级精度 |
| 线上 URL | 高 | 抓取 DOM + computed styles + JS 库 |
| 截图 / 视频录屏 | 中-低 | 视觉估算，标注 `[estimated]` |

**最佳实践：URL + DevTools 导出的源码片段组合使用，精度最优。**

## 支持的输出栈

| Stack ID | 技术组合 |
|----------|---------|
| `react-framer` | React + Framer Motion |
| `react-gsap` | React + GSAP + ScrollTrigger |
| `html-gsap` | Vanilla HTML/CSS + GSAP |
| `html-css` | Vanilla HTML/CSS（无 JS 库）|
| `vue-gsap` | Vue 3 + GSAP |

## 核心捕捉维度

全部精确到参数级别：

- **Easing / Timing** — cubic-bezier 值、duration、delay
- **Transform 序列** — translateX/Y、scale、rotate、opacity、clip-path、blur
- **Spring 物理参数** — stiffness、damping、mass、velocity
- **Stagger / Choreography** — 子元素编排延迟、序列时序、并发/顺序逻辑
- **Scroll-driven 触发** — ScrollTrigger、IntersectionObserver、scroll-timeline、parallax
- **Visual Effects** — Shader、WebGL、particles、cursor effects、SVG animation、text effects

## 与 design-dna 的关系

| | design-dna | motion-dna |
|---|---|---|
| 定位 | 设计风格提取器 | 动效逆向工程引擎 |
| Motion 字段 | 附属，低精度 | 主体，参数级精度 |
| JSON 兼容性 | — | ✅ 完全兼容，可合并 |
| 安装方式 | `npx skills add zanwei/design-dna` | `npx skills add [repo]/motion-dna` |

联动使用时，输出结构为：
```json
{
  "design_system": { ... },
  "design_style": { ... },
  "visual_effects": { ... },
  "motion_dna": { ... }
}
```

## 核心设计决策

- **Spec 纯净** — JSON 只存参数，不混入代码，保证跨栈可移植性
- **单一 JSON** — 所有动效维度（animations、scroll、choreography、visual effects）统一在 `motion_dna` 一个块里
- **Confidence 标注** — 每条动效标注 High/Medium/Low，估算值标 `[estimated]`
- **Reduced Motion** — 所有生成代码内置 `@media (prefers-reduced-motion)` 支持
