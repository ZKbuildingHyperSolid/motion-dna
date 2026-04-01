# motion-dna

**从任意参考来源逆向工程、复刻 UI 动效 — 自动适配你的设计系统。**

一个 Agent Skill，从任意网站提取动效参数，将颜色和阴影适配到你的主题，然后将动效代码直接写入你的项目。

> [design-dna](https://github.com/zanwei/design-dna) 的动效伙伴。design-dna 捕捉视觉风格；motion-dna 捕捉事物如何运动。两者输出兼容的 JSON，可合并使用。

## 安装

```bash
npx skills add ZKbuildingHyperSolid/motion-dna
```

兼容 40+ 种 Agent，包括 Claude Code、Cursor、GitHub Copilot。

## 环境配置

首次使用时，motion-dna 会自动检测你的环境并显示配置清单。零配置也能用，但安装以下工具可以大幅提升精度：

| 工具 | 对 motion-dna 的作用 | 配置方式 |
|------|---------------------|---------|
| **Playwright + Chromium**（推荐） | 用真实浏览器加载页面，提取精确的 CSS 值、@keyframes 和动效库配置 | `npx playwright install chromium` |
| **Computer Use**（推荐） | 让 Claude 真正看到动效播放，验证参数准确性 | `/mcp` → 启用 `computer-use` → 授权 macOS 权限 |
| **Claude in Chrome**（可选） | 直接操控浏览器 | 安装 Chrome 扩展 → `claude --chrome` |

不做任何配置时，motion-dna 仍可通过 WebFetch + 截图工作 — 精度较低但可用。

## 工作原理

### Capture + Apply（默认模式）

主要工作流 — 指向你喜欢的动效，motion-dna 将它应用到你的项目：

```
1. 你: "把这个 hover 效果复制到我的 card 组件" + 截图
   ↓
2. motion-dna: 用 Playwright 加载页面，提取精确参数
   ↓
3. motion-dna: 将颜色/阴影适配到你的 design tokens
   ↓
4. motion-dna: 将动效代码直接写入你的组件文件
```

**怎么说：**
- "把这个动效复制到我的项目" + 截图/URL
- "让我的 hero 区域动起来像这个网站"
- "我想要这个 hover 效果用在我的卡片上"
- "这个动效是怎么实现的？" + URL

**你提供什么 → 你得到什么：**

| 你的输入 | 结果 |
|---------|------|
| 截图 + "我要这个淡入效果" | 识别动效，询问应用到哪个组件 |
| URL + "复制 hero 动效" | 加载页面，提取 hero 动效，写入你的文件 |
| 截图 + URL + 描述 | 最高精度 — 视觉参考 + 源码提取 |

### Motion Audit（动效审计）

全页动效提取，用于文档化或竞品分析：

```
你: "审计 linear.app 的所有动效"
   ↓
motion-dna: 分类汇总找到的 23 条动效
   ↓
你: "把 hero-title-enter 应用到我的项目"
   ↓
motion-dna: 写入你的组件
```

**怎么说：**
- "审计这个页面的所有动效"
- "分析这个网站的动效系统"
- "分析设计+动效"（与 design-dna 联动）

## 核心特性

### 智能提取

使用 Playwright 在真实浏览器中加载页面 — 处理 SPA、客户端渲染和动态加载的动效。Playwright 不可用时降级为 WebFetch。

所有参数精确到参数级别：
- **Easing / Timing** — cubic-bezier 值、duration、delay
- **Transform 序列** — translateX/Y、scale、rotate、opacity、clip-path、blur
- **Spring 物理参数** — stiffness、damping、mass、velocity
- **Stagger / 编排** — 子元素编排延迟、序列时序
- **Scroll 驱动** — ScrollTrigger、IntersectionObserver、parallax
- **视觉效果** — Shader、WebGL、粒子、光标效果、文字效果

### 设计感知适配

将动效应用到你的项目时，motion-dna 区分**运动参数**（直接复制）和**设计参数**（适配你的主题）：

| 直接复制 | 适配你的主题 |
|---------|-------------|
| Duration、easing、delay | 颜色、背景色 |
| Transform（translate、scale、rotate） | Box-shadow 色值 |
| Spring 物理参数 | 渐变色 |
| Stagger 节奏 | Border 色值 |
| Opacity 变化量 | Text-shadow 色值 |

深色主题的 hover 效果应用到你的浅色项目时，会使用你的 `--shadow-color` 和 `--color-surface` token，而不是硬编码的深色值。

### 直接写入代码

motion-dna 将动效代码直接写入你的项目文件 — 无需 copy-paste。它会：
- 从 `package.json` 自动检测你的技术栈
- 遵循你现有的代码风格
- 导入所需库
- 添加 `prefers-reduced-motion` 无障碍支持
- 包含 `// motion-dna: {id}` 可追溯注释

### 目标技术栈

| Stack ID | 技术组合 |
|----------|---------|
| `react-framer` | React + Framer Motion |
| `react-gsap` | React + GSAP + ScrollTrigger |
| `html-gsap` | 原生 HTML/CSS + GSAP |
| `html-css` | 原生 HTML/CSS（无 JS 库）|
| `vue-gsap` | Vue 3 + GSAP |

技术栈从你的项目自动检测。如果没有动效库，motion-dna 会推荐一个并在安装前征求你的同意。

## 与 design-dna 的关系

| | design-dna | motion-dna |
|---|---|---|
| 聚焦 | 视觉设计 Token | 动效参数 |
| 动效支持 | 极少 | 完整，参数级 |
| JSON 兼容 | — | ✅ 可合并 |

联动使用时（`analyze design+motion`）：

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
- [Schema 参考](references/schema.md) — Motion DNA JSON 完整字段说明
- [代码生成指南](references/generation-guide.md) — 各栈代码生成规范

## 许可证

[MIT](LICENSE)
