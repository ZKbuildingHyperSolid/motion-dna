# Motion DNA — 执行交接文档

> 本文档用于向 Claude Opus 或 Claude Code 交接项目，包含所有已完成的决策、设计结果和下一步执行指令。

---

## 项目状态

**当前阶段：** 全部文件已完成，进入测试和发布阶段。

**repo 结构：**
```
motion-dna/
├── SKILL.md                        ✅ 已完成
├── README.md                       ✅ 已完成
├── README.zh-CN.md                 ✅ 已完成
├── LICENSE                         ✅ 已完成（MIT）
├── references/
│   ├── schema.md                   ✅ 已完成
│   └── generation-guide.md         ✅ 已完成
└── docs/
    ├── 01-brief.md                 ✅ 已完成
    ├── 02-project-intro.md         ✅ 已完成
    ├── 03-goals.md                 ✅ 已完成
    └── 04-handoff.md               ✅ 本文件
```

---

## 所有已确认的设计决策

### 产品定位
- Claude Code Skill，开源，MIT 协议
- 动效逆向垂直工具，不做通用设计提取
- 与 design-dna（zanwei/design-dna）JSON 兼容，作为 `motion_dna` 扩展字段

### 工作模式（三种，用户通过指令选择）
- `analyze motion` → 只提取动效，输出 motion_dna JSON
- `analyze design+motion` → 调用 design-dna + motion-dna，合并输出
- `generate` → 已有 JSON，生成目标栈代码

### 输入源（三种，自动路由）
- URL → 抓取页面，扫描库、CSS、JS
- 源码（CSS/JS/TS）→ 直接解析，最高精度
- 截图/视频 → 视觉估算，标注 `[estimated]`

### 输出设计
- **Spec 与代码分离**：JSON 是纯净参数，代码在 Generate 阶段按需生成
- **全收一个 JSON**：不拆模块，一个 `motion_dna` 块包含所有维度
- **置信度透明**：每条动效有 High/Medium/Low，估算值内联标注

### 目标代码栈（5个）
- `react-framer` — React + Framer Motion
- `react-gsap` — React + GSAP + ScrollTrigger
- `html-gsap` — 原生 HTML/CSS + GSAP
- `html-css` — 原生 HTML/CSS 无库
- `vue-gsap` — Vue 3 + GSAP

---

## 关键参考资料

- design-dna repo：https://github.com/zanwei/design-dna
- Agent Skills 规范：https://agentskills.io
- skills CLI：https://github.com/vercel-labs/skills
