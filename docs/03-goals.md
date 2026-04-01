# Motion DNA — 项目目标

## 核心目标

**打造一个完全可行、可落地的动效逆向工程 Claude Code Skill，供开发者开箱即用。**

## 具体目标拆解

### 1. 技术目标

- [ ] 能从任意 URL 提取完整动效参数，精度达到 cubic-bezier 级别
- [ ] 能解析主流动效库（GSAP、Framer Motion、Lottie、Three.js 等）的配置
- [ ] 能识别 Scroll-driven 动效的触发条件和 scrub 逻辑
- [ ] 能捕捉 Spring 物理参数（stiffness/damping/mass）
- [ ] 能理解 Choreography 编排顺序和 Stagger 节奏
- [ ] 能基于 Motion DNA JSON 生成至少 3 种主流栈的可运行代码

### 2. 产品目标

- [ ] 发布为开源 Claude Code Skill，可通过 `npx skills add` 一键安装
- [ ] 与 design-dna 的 JSON 结构完全兼容，支持联动使用
- [ ] 用户指令简单直接：3 个模式，记一次就会用
- [ ] 输出的 Motion DNA JSON 可版本控制、可跨项目复用

### 3. 质量目标

- [ ] 所有估算值必须标注 `[estimated]` 和置信度
- [ ] 生成代码必须包含 `// motion-dna: {id}` 注释，可追溯
- [ ] 所有生成代码内置 `prefers-reduced-motion` 无障碍支持
- [ ] 无法确定的参数标 `null`，不允许伪造数据

## 不做的事（范围边界）

- ❌ 不做纯设计（颜色/字体/间距）的提取和复刻 — 那是 design-dna 的职责
- ❌ 不做视频自动逐帧解析的工程化工具 — 视频输入作为低精度兜底选项
- ❌ 不绑定特定技术栈 — Motion DNA JSON 保持栈无关
- ❌ 不做 GUI 工具 — 纯 Claude Code Skill，命令行交互

## 交付物清单

| 文件 | 状态 | 说明 |
|------|------|------|
| `SKILL.md` | ✅ 完成 | 核心 Skill 执行文件 |
| `docs/01-brief.md` | ✅ 完成 | 项目背景与定位 |
| `docs/02-project-intro.md` | ✅ 完成 | 项目完整介绍 |
| `docs/03-goals.md` | ✅ 完成 | 项目目标 |
| `docs/04-handoff.md` | ✅ 完成 | 执行交接文档 |
| `README.md` | ✅ 完成 | 面向开发者的公开文档 |
| `README.zh-CN.md` | ✅ 完成 | 中文版 README |
| `references/schema.md` | ✅ 完成 | Motion DNA JSON Schema 详细说明 |
| `references/generation-guide.md` | ✅ 完成 | 代码生成规范文档 |
