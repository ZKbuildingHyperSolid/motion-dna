# Motion DNA — Project Brief

## 背景与痛点

现有 AI 工具（包括 Claude Code）在处理 UI 动效复刻任务时，缺乏一套结构化的动效逆向工程流程。具体问题：

- 动效是动态的，无法像静态设计稿一样直接"截图分析"
- 现有工具（如 design-dna）把 motion 作为设计风格的附属字段，精度严重不足
- 没有标准化的动效参数规范（easing、spring、stagger、choreography），导致 AI 复现质量不稳定
- 开发者无法将"这个网站的动效感觉"转化为可复用、可版本控制的工程资产

## 项目起点

调研了现有开源项目 **design-dna**（zanwei/design-dna），该项目做的是：

> 从参考 UI（截图、图片、URL）提取视觉设计特征，输出结构化 Design DNA JSON，再由 AI 按 JSON 生成 UI。

**结论：design-dna 部分满足需求，但动效维度有明显短板：**

| 维度 | design-dna 现状 |
|------|----------------|
| 动效参数精度 | 仅 "if observable, note easing curves and duration feel"，无独立结构 |
| Choreography / Stagger | 归入 composite_notes，模糊处理 |
| Scroll 触发逻辑 | 无专项字段 |
| Spring 物理参数 | 不支持 |
| 动效复现代码生成 | 泛化处理，无精确参数到代码的映射 |

## 差异化定位

**motion-dna 是 design-dna 的动效垂直超集。**

- design-dna 做设计风格提取，motion 是附属物
- motion-dna 以动效为主体，做动效精准逆向工程
- 两者 JSON 结构兼容，可合并使用

## 核心理念

> 把主观的"让它动起来像那个网站"，转化为精确、可复现、可版本控制的 Motion Spec。

## 目标用户

开发者——需要复刻或参考特定网站动效风格的前端工程师、独立开发者、设计工程师。
