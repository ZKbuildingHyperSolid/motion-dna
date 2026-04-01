# Motion DNA — 项目介绍

## 是什么

**motion-dna** 是一个 Claude Code Skill，用于从任意网站逆向工程动效参数，适配到用户的设计系统，然后将动效代码直接写入用户的项目。

## 工作流程

```
参考来源                    识别 + 提取                    适配 + 写入
─────────────────          ─────────────────────         ──────────────────
截图 + 描述 + URL   ──►    锁定目标动效，提取参数   ──►    适配设计系统，写入用户文件
```

## 两种工作模式

| 模式 | 怎么说 | 用途 |
|------|--------|------|
| **Capture + Apply**（默认） | "把这个动效复制到我的组件" | 截图指向 → 提取单条动效 → 适配主题 → 写入代码 |
| **Motion Audit** | "审计这个页面的所有动效" | 全页动效提取，输出分类 Motion DNA JSON |

## 支持的输入来源

| 来源 | 精度 | 说明 |
|------|------|------|
| 截图 + 描述 + URL | **最高** | 视觉定位 + 浏览器提取 |
| URL + 描述 | 高 | Playwright 加载 SPA，描述锁定目标 |
| 截图 + 描述 | 中 | 视觉估算，标注 `[estimated]` |
| URL only | 中 | 提取全部，询问用户要哪个 |

## 提取引擎

- **Playwright（首选）** — 用真实 Chrome 浏览器加载页面，处理 SPA 和客户端渲染，提取 computed styles、@keyframes、动效库配置
- **WebFetch（降级）** — 抓取 SSR 内容，对 SPA 有限

## 设计感知适配

动效中附带的设计参数（颜色、阴影、渐变）会自动适配到用户的 design tokens，而不是生硬复制。

| 直接复制（运动参数） | 适配用户主题（设计参数） |
|-------------------|---------------------|
| duration、easing、delay | color、background |
| translate、scale、rotate | box-shadow 色值 |
| spring 物理参数 | gradient 色值 |
| stagger 节奏 | border、text-shadow 色值 |

## 支持的输出栈

| Stack ID | 技术组合 |
|----------|---------|
| `react-framer` | React + Framer Motion |
| `react-gsap` | React + GSAP + ScrollTrigger |
| `html-gsap` | Vanilla HTML/CSS + GSAP |
| `html-css` | Vanilla HTML/CSS（无 JS 库）|
| `vue-gsap` | Vue 3 + GSAP |

技术栈从 `package.json` 自动检测，无需手动指定。

## 与 design-dna 的关系

| | design-dna | motion-dna |
|---|---|---|
| 定位 | 设计风格提取器 | 动效逆向工程 + 应用引擎 |
| Motion 字段 | 附属，低精度 | 主体，参数级精度 |
| JSON 兼容性 | — | ✅ 完全兼容，可合并 |
| 安装方式 | `npx skills add zanwei/design-dna` | `npx skills add ZKbuildingHyperSolid/motion-dna` |

## 核心设计决策

- **截图驱动** — 用户通过截图 + 描述精确指向想要的动效，而不是全页倾倒
- **运动与设计分离** — 运动参数直接复制，设计参数适配用户主题
- **直接写入** — 代码直接写入用户的项目文件，不是 copy-paste 代码块
- **Confidence 标注** — 估算值标记 `[estimated]`，未知值为 `null`
- **Reduced Motion** — 所有生成代码内置 `@media (prefers-reduced-motion)` 支持
