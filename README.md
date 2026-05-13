# Skill Router for TRAE CN Domestic Models

[English Version](./README_EN.md)

> 中文名：对 TRAE CN 国内模型的技能路由  
> 一个面向 TRAE CN / 国内模型的多 Skill 分阶段路由、协同与审计方案。

这个项目提供一个 `skill-router` 技能，用来帮助 TRAE CN 里的 AI Coder 更稳定地调用可用 skills。

它的目标不是让模型“嘴上说用了 skill”，而是让模型在不同任务阶段重新路由、调用不同 skills、同步原生 Todo List，并在最后留下可审计的阶段台账。

V3.4 在 V3.3 的 Native Todo Sync、UI Polish、QA Attribution 基础上，进一步加入 **Skill Invocation Proof**：最终台账必须区分 Confirmed / Inferred / Planned only / Unavailable failed，避免模型把“计划使用的 skill”冒充成“实际调用的 skill”。

## 目录

- [建议搭配 Prompt（通用版）](#建议搭配-prompt通用版)
- [项目目录说明](#项目目录说明)
- [样例展示](#样例展示)
- [为什么需要这个 skill-router？](#为什么需要这个-skill-router)
- [当前推荐版本](#当前推荐版本)
- [核心能力](#核心能力)
- [版本演进](#版本演进)
- [后续优化方向](#后续优化方向)
- [如何安装](#如何安装)
- [如何触发](#如何触发)
- [推荐测试 Prompt](#推荐测试-prompt)
- [如何判断是否合格](#如何判断是否合格)
- [常见不合格表现](#常见不合格表现)
- [如何替换成你自己的 skills](#如何替换成你自己的-skills)
- [常见违规标记](#常见违规标记)
- [设计原则](#设计原则)
- [开源许可](#开源许可)
- [致谢](#致谢)
- [维护建议](#维护建议)

---

## 建议搭配 Prompt（通用版）

```text
请优先使用 skill-router V3.4 做分阶段动态路由。

如果支持 TRAE 原生 Todo List，必须把 router gate 和实际工作项同步进去。

不要只在开头调用一次 skill-router；每进入关键阶段前都要重新 router pass，并只选择当前阶段真正需要的 skills。

最终必须输出 Stage Marker Ledger，并按 Confirmed / Inferred / Planned only / Unavailable failed 区分每个 skill 的调用证据等级。

不要把没有 toolName: Skill 或明确 skill 加载输出的 skill 写成实际调用。

如果声称使用了某个 skill 但没有调用证据，必须标记：
ROUTER_VIOLATION: SKILL_CLAIM_WITHOUT_INVOCATION
```

---

## 项目目录说明

推荐仓库结构如下：

```text
Skill Router for TRAE CN Domestic Models/
├── README.md
├── README_EN.md
├── skill-router/
│   └── SKILL.md
├── 所有版本/
│   ├── SKILL-v1.md
│   ├── SKILL-v2.md
│   ├── SKILL-v3.md
│   ├── SKILL-v3.1.md
│   ├── SKILL-v3.2.md
│   ├── SKILL-v3.3.md
│   └── SKILL-v3.4.md
└── 样例展示/
    ├── 调用过程展示1.png
    ├── 调用过程展示2.png
    ├── 输出总结展示1.png
    └── 输出总结展示2.png
```

### `skill-router/`

这是**真正要使用的 skill 文件夹**。

正式使用时，把当前推荐版本 V3.4 的内容放入：

```text
skill-router/SKILL.md
```

### `所有版本/`

这里保存历史版本备份，方便回溯每一版的设计变化。建议将 `SKILL-v3.4.md` 也放入此目录。

### `样例展示/`

这里放实际运行截图、Todo List 截图、Stage Marker Ledger 截图、浏览器 QA 截图等。

README 已经引用仓库里现有的图片文件。

---

## 样例展示

下面展示的是当前仓库里已经放好的样例截图。

### 调用过程展示 1

![调用过程展示 1](./样例展示/调用过程展示1.png)

这个示例主要展示 router 在任务执行过程中进行阶段化路由与调用的样子。

---

### 调用过程展示 2

![调用过程展示 2](./样例展示/调用过程展示2.png)

这个示例补充展示了不同阶段继续推进时，router / gate / work item 的衔接方式。

---

### 输出总结展示 1

![输出总结展示 1](./样例展示/输出总结展示1.png)

这个示例主要展示最终输出中对阶段结果、技能调用和总结信息的呈现方式。

---

### 输出总结展示 2

![输出总结展示 2](./样例展示/输出总结展示2.png)

这个示例补充展示了最终总结区块的另一种实际输出效果。

V3.4 的最终台账应额外展示 Confirmed / Inferred / Planned only / Unavailable failed，用于区分真实调用、行为推断和仅计划使用的 skills。

---

## 为什么需要这个 skill-router？

这个项目来自我在 TRAE CN 国内模型里的真实使用困扰。

TRAE CN 里的国内模型并不是完全不会使用 skills，而是经常存在几个明显问题：

1. **不主动调用 skills**  
   很多任务明明适合调用 skills，模型却不会主动调用。用户必须在 prompt 里明确说“请优先调用可用 skills”“记得调用 skill-router”，它才更可能触发。

2. **只在开头调用一次**  
   即使用户明确提醒它使用 skills，模型也经常只在任务开始时调用一次 router。它会列出一堆“后续可能使用的 skills”，但真正进入实现、调试、抛光、QA、文档等阶段时，就不再重新调用。

3. **任务一复杂就偷懒**  
   当任务规模变大、文件变多、阶段变多时，模型会倾向于少触发 skills。它会尽量压缩 skill 调用次数，甚至只调用一两个 skill，然后假装已经完成多 skill 协作。

4. **把“计划使用”说成“已经使用”**  
   模型常常在开头把未来阶段的 skills 写进计划里，最后总结时却把这些 skills 当成已经实际使用过。

5. **不会按阶段换 skill**  
   一个真实开发任务通常包含多个阶段：需求澄清、规划、实现、调试、Review、视觉抛光、QA、文档、交接。不同阶段需要不同 skills，但模型经常只用一套开头选好的 skills 贯穿到底。

6. **调用真假难审计**  
   模型可能声称使用了某些 skills，但 TRAE UI 的工具日志里并没有对应的 `toolName: Skill` 调用记录。V3.4 因此新增了 skill 调用证据等级，要求区分 Confirmed、Inferred、Planned only、Unavailable / failed。

所以这个 `skill-router` 的目标不是简单地“帮模型选 skill”，而是让模型在任务开始时构建候选池，在关键阶段重新路由，尽量同步 TRAE 原生 Todo List，并在最终 Stage Marker Ledger 中标注每个 skill 的调用证据等级。

一句话：

> 这个项目是为了解决 TRAE CN 国内模型“懒得调用 skills、只调用一次、少调用、假调用、不会分阶段调用、调用真假难审计”的问题。

---

## 当前推荐版本

推荐使用：

```text
SKILL-v3.4.md
```

正式使用时，它对应的生效文件就是：

```text
skill-router/SKILL.md
```

也就是说，当前推荐版本是 V3.4。正式使用时，应将 V3.4 内容放入 `skill-router/SKILL.md`。

---

## 核心能力

这个 router 主要提供这些能力：

- **可用 skill 路由**  
  根据当前任务，从真实 available skills 里选择合适的 skills。

- **防止虚构 skill**  
  不允许模型编造不存在的 skill 名称，也不允许声称使用了不存在的 skill。

- **多 skill 专家团协作**  
  不再默认找最少 skill，而是根据任务复杂度组织规划、实现、调试、抛光、QA、文档等不同角色的 skill 协作。

- **分阶段重新路由**  
  不允许只在任务开头调用一次。在实现、调试、Review、QA、文档等关键阶段前，需要重新调用 skill-router。

- **阶段化验证 marker**  
  每个阶段都有独立 marker，例如 `V3.4-1`、`V3.4-3`、`V3.4-4`、`V3.4-5`、`V3.4-6`，方便判断它到底有没有重新路由。

- **Todo-Bound Gates**  
  router gate 不能只写在聊天正文里，而应绑定到任务计划 / Todo List。

- **TRAE 原生 Todo List 同步**  
  如果当前环境支持 TRAE 原生任务计划 / Todo List，必须优先把 router gate 和实际 work item 同步进去。

- **Resume-Safe 续跑**  
  中途重新路由时，不允许从头规划旧任务。应该识别已完成事项，只为当前未完成工作簇重新路由。

- **UI Review / Polish 强制闸门**  
  对网页、dashboard、PDF 首页、卡片、移动端页面等用户可见产物，默认要求在实现后、QA 前做一轮视觉审查与小范围抛光。

- **QA 证据约束**  
  不允许用静态代码审查冒充浏览器 QA。如果使用 `gstack` 和 Chrome DevTools MCP，应在最终台账里明确写成：  
  `gstack（browser testing intent）+ Chrome DevTools MCP`

- **Skill 调用证据等级审计（V3.4 新增）**  
  每个 skill 必须标注证据等级：
  - Confirmed：TRAE UI 明确出现 `toolName: Skill` 或有明确 skill 加载输出
  - Inferred：UI 未显示调用，但行为明显受该 skill 规则指导
  - Planned only：只是计划/候选，不能算实际使用
  - Unavailable / failed：想用但不可用或调用失败

- **防止伪造 Actual skills used（V3.4 新增）**  
  不允许把 Planned only 写成 Actual skills used。如果某个 skill 被声称使用但没有调用证据，必须标记：  
  `ROUTER_VIOLATION: SKILL_CLAIM_WITHOUT_INVOCATION`

- **创建 / 改进 / 验证 skill 的特殊流程**  
  当用户想创建新 skill 时，不会直接开写，而是先澄清需求、搜索现成 skill、判断安装还是创建，再用 `skill-creator` 或标准 `SKILL.md` 流程创建，并加入调用验证和测试流程。

- **Stage Marker Ledger 最终台账**  
  最后输出每个阶段的 marker、实际使用 skills、证据等级、跳过原因和验证方式，方便用户审计。

---

## 版本演进

### V1：基础 Skill Router

V1 是最初版本，主要能力是：

- 根据任务选择 available skills
- 防止虚构 skill 名称
- 提供按阶段组织的 skill 路由表
- 将 `gstack` 视为顶层 skill，用于触发其内部 QA / browser testing / review / guard / ship 等能力
- 加入基础验证标记
- 支持创建、查找、安装、验证 skill 的特殊流程

V1 已经包含“创建新 skill”的特殊流程：先澄清需求，再用 `find-skills` 搜索现成 skill，没有合适方案时再用 `skill-creator` 或标准 `SKILL.md` 手动创建，并要求加入调用验证和测试步骤。

局限：

- 逻辑偏保守
- 默认倾向只选 1–3 个 skill
- 容易只在任务开始时路由一次

---

### V2：多 Skill 专家团协作

V2 的核心变化是从“找最少够用的 skills”改成“组建多 skill 专家团”。

主要增强：

- 引入多 skill 协作理念
- 增加候选池
- 增加阶段化 marker
- 要求最终输出 Stage Marker Ledger
- 强调每个阶段只选择当前阶段真正需要的 skills
- 鼓励中高激进度的多阶段协作

局限：

- 弱模型仍然可能只列出“后续 checkpoint”，但不真正执行

---

### V3：Todo-Bound Hard-Gated Router

V3 开始把 router gate 和任务计划绑定。

主要增强：

- 加入 `ROUTER_STAGE_GATE: OPEN / CLOSED`
- 要求每个关键阶段开始前必须打开对应 gate
- 要求 router checkpoint 写入任务计划 / todo list
- 引入动态 checkpoint 生成算法
- 禁止使用固定 todo 模板套所有任务
- 禁止把“后续 checkpoint”当成已执行证据

局限：

- 中途 recheck 时，部分模型会重新从 Router Pass 0 开始规划，导致任务进度回退

---

### V3.1：Resume-Safe 续跑安全补丁

V3.1 解决的是“中途重新路由时从头开始”的问题。

主要增强：

- 引入 resume-safe 行为
- 中途 recheck 必须基于当前进度继续，而不是从零开始
- 已完成 todo 默认锁定
- 禁止在 4/6 已完成时突然重新生成 0/6 任务列表
- 对中途回退、静态 QA 冒充完整 QA 等行为加入违规标记

关键模式：

```text
ROUTER_RECHECK_MODE: RESUME_FROM_CURRENT_PROGRESS
```

局限：

- UI 类任务仍可能从实现阶段直接跳到 QA，缺少独立视觉审查

---

### V3.2：UI Review / Polish 强制闸门

V3.2 增加了用户可见产物的抛光阶段。

只要任务涉及网页、landing page、dashboard、表单、卡片 UI、移动端页面、PDF 首页、PDF 报告布局、报告封面、视觉重构、字体、间距、层级、颜色、响应式，就默认必须进入：

```text
ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH
```

UI 类任务默认链路变成：

```text
TASK_MAP -> DESIGN/PLANNING -> IMPLEMENTATION -> REVIEW/POLISH -> QA/VERIFICATION
```

局限：

- 设计阶段仍可能没有实际调用设计类 skill
- QA 阶段归因可能不清楚，例如实际是 `gstack + Chrome DevTools MCP`，但最终只写了 Chrome DevTools MCP

---

### V3.3：Native Todo Sync + 设计 Skill 最低要求 + QA 归因一致性

V3.3 继续保留并强化了创建 / 修改 / 安装 / 验证 skill 的流程，并将这类任务视为高密度 skill 任务，通常需要 `find-skills`、`skill-creator`、`prompt-lookup`、`full-output-enforcement`、`handoff` 等多个 skills 分阶段协作。

主要增强：

1. **Native Todo Sync Gate**：如果 TRAE 环境支持原生 Todo List / 任务计划，router gate 和 work item 必须同步进去。只在聊天正文里写 gate，不算完整 todo-bound。
2. **Design Stage Skill Minimum**：对 UI、前端、PDF layout、dashboard、landing page、report cover 等视觉任务，设计阶段默认至少使用一个设计类 skill。
3. **QA Skill Attribution Consistency**：如果 QA 阶段选择了 `gstack`，底层使用 Chrome DevTools MCP，那么最终台账必须写成 `gstack（browser testing intent）+ Chrome DevTools MCP`。

V3.3 解决了两个实际问题：

- 模型只在聊天里写漂亮的 gate，但 TRAE 官方计划列表没有同步
- 模型实际用了 `gstack` 触发浏览器 QA，但最终总结只写 Chrome DevTools MCP，导致审计不清楚

---

### V3.4：Skill Invocation Proof / 调用证据等级审计

V3.4 解决的是 V3.3 暴露出的一个关键问题：

模型可能已经分阶段路由，也同步了 Todo List，但仍然会在最终台账里把“计划使用的 skill”写成“实际调用的 skill”。

例如：

- `diagnose` 只是被写进计划，但没有真实调用；
- `full-output-enforcement` 只是作为候选出现，但没有真实调用；
- `gstack` 被声称用于 QA，但没有浏览器 / MCP / `toolName: Skill` 证据。

V3.4 新增 **Skill Invocation Proof**，要求每个 skill 都必须标注调用证据等级：

```text
Confirmed：TRAE UI 明确出现 toolName: Skill 或有明确 skill 加载输出
Inferred：UI 未显示调用，但行为明显受该 skill 规则指导
Planned only：只是计划/候选，不能算实际使用
Unavailable / failed：想用但不可用或调用失败
```

V3.4 的核心规则：

```text
不要把没有 toolName: Skill 或明确 skill 加载输出的 skill 写成实际调用。

如果声称使用了某个 skill 但没有调用证据，必须标记：
ROUTER_VIOLATION: SKILL_CLAIM_WITHOUT_INVOCATION
```

一句话：

> V3.3 解决“有没有分阶段协同”；V3.4 解决“到底哪些 skill 真调用了”。

---

## 后续优化方向

### 未来方向：QA Evidence Precision Rule

V3.4 已经能区分 skill 调用证据等级，但 QA 证据还可以更精确。

例如响应式检查里，如果模型只看：

```js
overflowX
```

还不够严谨。

更好的横向溢出检测方式是：

```js
document.documentElement.scrollWidth <= document.documentElement.clientWidth
```

未来可以要求模型在声称“移动端无横向滚动 / 无水平溢出”时，用真实布局宽度判断，而不是只看 CSS overflow 值。

---

## 如何安装

### 1. 当前最新版位置

正式使用文件应位于：

```text
skill-router/SKILL.md
```

历史版本 `V3.4` 建议备份于：

```text
所有版本/SKILL-v3.4.md
```

### 2. 确认 skill 名称

`SKILL.md` 顶部 frontmatter 应类似：

```yaml
---
name: skill-router
description: "中文/English Skill Router V3.4 ..."
---
```

### 3. 在 TRAE 中确认可用

让模型列出当前可用 skills：

```text
请列出你当前可用的 available skills，只列真实存在的 skill 名称，不要猜测。
```

如果能看到：

```text
skill-router
```

说明安装基本成功。

---

## 如何触发

常用触发语：

```text
请优先调用你当前可用的 skill / skills 来辅助完成。
```

或者：

```text
记着调用 skill-router。
```

更强一点的触发语：

```text
请使用 skill-router 的 V3.4 Skill Invocation Proof 模式。
如果当前环境支持 TRAE 原生任务计划 / Todo List，请把 router gate 和实际 work item 同步到原生计划列表里。
不要只在开头调用一次 skill-router。
最终输出 Stage Marker Ledger，并区分 Confirmed / Inferred / Planned only / Unavailable failed。
```

---

## 推荐测试 Prompt

### 测试 V3.4 原生 Todo List + 多阶段协作 + 调用证据审计

```text
请新建一个完全独立的小 demo，不要修改我现有项目代码。

任务：做一个 LOVE 主题的小型展示页。

要求：
1. 新建独立目录：skill-router-test-love-page/
2. 使用 HTML、CSS、JavaScript，不要引入复杂框架。
3. 页面需要包含：温柔 hero 区域、3 张 LOVE 主题卡片、一个“爱的瞬间”时间线、一个按钮，点击后随机显示一句简短的 love message。
4. 视觉风格：温柔、干净、有氛围感，不要土味，不要太花。
5. 页面要响应式，手机和桌面都能正常查看。
6. 不要写占位符，不要只写伪代码，文件要能直接打开运行。
7. 完成后请打开页面检查按钮交互、响应式布局和控制台错误。

请优先使用 skill-router V3.4 做分阶段动态路由，并在最终 Stage Marker Ledger 中区分 Confirmed / Inferred / Planned only / Unavailable failed。
```

预期表现：

- TRAE 原生 Todo List 被更新
- 设计、实现、Review / Polish、QA 阶段按需出现
- 最终有 Stage Marker Ledger
- 每个 skill 都有 Confirmed / Inferred / Planned only / Unavailable failed 证据等级
- Planned only 不会被写成 Actual skills used

---

## 如何判断是否合格

一个合格的运行通常应该看到：

```text
ROUTER_CHECKPOINT_0_TASK_MAP
ROUTER_CHECKPOINT_2_PLANNING_OR_DESIGN
ROUTER_CHECKPOINT_3_IMPLEMENTATION
ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH
ROUTER_CHECKPOINT_5_QA_OR_VERIFICATION
```

并且看到类似 marker：

```text
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-1
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-3
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-4
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-5
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-6
```

最终还应该有：

```text
Stage Marker Ledger
```

并列出：

```text
Confirmed skill calls
Behavior-inferred skill guidance
Planned only
Unavailable / failed
Violation
```

---

## 常见不合格表现

### 只开头调用一次

```text
Router Pass: ROUTER_CHECKPOINT_0_TASK_MAP
后续 router checkpoints:
- implementation
- QA
```

然后直接做完整个任务。这是不合格。

### UI 任务跳过抛光

```text
实现完成，直接进入 QA
```

如果任务是 UI、网页、PDF 首页、卡片、dashboard，这通常不合格。

### 只写聊天计划，不同步原生 Todo

如果当前 TRAE 支持原生 Todo List，但模型只在聊天正文写 `ROUTER_GATE ... WORK ...`，却没有触发原生 Todo List 更新，那不算完整合格。

### QA 冒充

```text
静态代码审查通过，所以浏览器 QA 通过
```

静态审查不能冒充浏览器 QA。

### skill 归因不清

如果选择了 `gstack`，并实际用 Chrome DevTools MCP 做浏览器测试，最终应该写：

```text
gstack（browser testing intent）+ Chrome DevTools MCP
```

而不是只写 `Chrome DevTools MCP`。

### 声称使用 skill，但没有调用证据

不合格：

```text
Actual skills used: diagnose, full-output-enforcement, gstack
```

但执行日志里没有对应的 `toolName: Skill` 或明确 skill 加载输出。

V3.4 中应改为：

```text
Confirmed: skill-router, impeccable
Planned only: diagnose, full-output-enforcement
Unavailable / failed: gstack, if browser QA was attempted but failed
Violation: ROUTER_VIOLATION: SKILL_CLAIM_WITHOUT_INVOCATION
```

---

## 如何替换成你自己的 skills

README 里的 routing table 和 skill 名称是作者自己环境里的示例。

如果你要在自己的环境使用，必须替换成你真实拥有的 skills。

### 第一步：列出你的 available skills

让模型执行：

```text
请列出你当前可用的 available skills，只列真实存在的 skill 名称，不要猜测。
```

把真实存在的 skill 名称复制下来。

### 第二步：替换 Available Skills Routing Table

打开：

```text
skill-router/SKILL.md
```

找到：

```text
Available Skills Routing Table
```

把里面的作者示例技能替换成你自己的技能。不要保留你环境里不存在的 skill。

### 第三步：按阶段给你的 skills 分类

建议按这些阶段分类：

```text
Discovery / Planning
Design / Visuals
Implementation
Debugging / Testing
Review / Polish
QA / Browser Verification
Documentation / Handoff
Skill Creation / Skill Management
```

每个 skill 至少写清楚：skill 名称、适用场景、不适用场景、属于哪个阶段、和其他 skill 的分工区别。

示例：

| 工程用途场景 | Skill 名称 | 所属 skill 包 | 用途判断 |
|---|---|---|---|
| 前端 / 页面实现 | `my-frontend-builder` | `my-skills` | 用于创建完整页面、组件、dashboard。 |
| UI / 抛光审查 | `my-ui-reviewer` | `my-skills` | 用于实现后检查间距、层级、配色和响应式。 |
| 浏览器 QA | `my-browser-qa` | `my-skills` | 用于打开页面、截图、点击、验证交互。 |

### 第四步：替换阶段偏好规则

在 `SKILL.md` 中搜索类似内容：

```text
Preferred skills:
- design-taste-frontend
- ui-ux-pro-max
- impeccable
- high-end-visual-design
```

替换成你自己的设计类 skills。实现、调试、QA、文档、skill creation 等规则也应同步替换。

### 第五步：如果你没有 gstack

这个 router 默认把 `gstack` 当成顶层浏览器 QA skill。

如果你的环境没有 `gstack`，需要把所有 `gstack` 相关规则替换成你自己的浏览器 QA skill。

例如：

```text
Use `my-browser-test-skill` for browser QA.
When it uses Playwright or Chrome DevTools MCP underneath, attribute QA as:
my-browser-test-skill（browser testing intent）+ <actual execution mechanism>
```

如果你没有任何浏览器 QA skill，就让 router 明确说明：

```text
当前没有可用 browser QA skill，只能进行静态检查，QA 状态为 PARTIAL。
```

不要让模型把静态检查说成完整 QA。

### 第六步：保留反幻觉规则

这些规则不要删：

```text
Never invent a skill.
Never claim a skill was used unless it exists and was actually selected or invoked.
Never claim future-stage skills were already used.
Never claim browser QA passed unless browser QA actually ran.
Never list Planned only skills as Actual skills used.
```

这是整个 router 能被审计的基础。

### 第七步：自定义验证标记

默认 V3.4 marker 是：

```text
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-1
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-3
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-4
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-5
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-6
```

你可以改成自己的项目 token，但必须全文件统一替换，包括 marker 表、trace 规则、Stage Marker Ledger 示例、违规判断中引用的 marker。

---

## 常见违规标记

本 router 内置了一些违规标记，用来暴露模型偷懒或伪造 skill 使用的行为。

常见包括：

```text
ROUTER_VIOLATION: ONLY_INITIAL_PASS_RAN
ROUTER_VIOLATION: STAGE_WORK_WITHOUT_STAGE_ROUTER_PASS
ROUTER_VIOLATION: MIDTASK_ROUTER_RESET_TO_TASK_MAP
ROUTER_VIOLATION: FIXED_TODO_PATTERN_REUSED
ROUTER_VIOLATION: REVIEW_POLISH_REQUIRED_BUT_SKIPPED
ROUTER_VIOLATION: QA_BEFORE_UI_POLISH_GATE
ROUTER_VIOLATION: NATIVE_TODO_SYNC_SKIPPED
ROUTER_VIOLATION: STATIC_QA_OVERCLAIM
ROUTER_VIOLATION: SKILL_CLAIM_WITHOUT_INVOCATION
```

`ROUTER_VIOLATION: SKILL_CLAIM_WITHOUT_INVOCATION` 用于：模型声称使用了某个 skill，但没有 `toolName: Skill`、明确 skill 加载输出或其他可审计调用证据。

如果出现这些标记，说明本轮执行不完全合规。

---

## 设计原则

### 1. 分阶段，而不是一次性

不要让模型在开头选完所有 skills，然后全程不再路由。

### 2. 当前阶段只用当前阶段的 skill

未来阶段的 skill 可以列为候选，但不能说已经使用。

### 3. 多 skill 协作，不是乱用 skill

目标是“规划 skill + 实现 skill + 抛光 skill + QA skill”，不是随便堆 skill 名字。

### 4. 证据优先

比起“我用了”，更重要的是：marker、Todo List、文件改动、浏览器截图、测试结果、Stage Marker Ledger、调用证据等级。

### 5. 原生 Todo List 很重要

很多模型更听官方任务计划，而不是聊天正文。

### 6. V3.4 的核心是可审计失败

V3.4 不保证弱模型每次都真的调用所有该用的 skills，但它能让假调用、少调用、只计划不调用更容易暴露。

---

## 开源许可

推荐使用：

```text
MIT License
```

你也可以根据需要选择 Apache-2.0、GPL、CC-BY 等其他许可证。

---

## 致谢

这个 router 是在 TRAE CN 国内模型实际使用中不断测试和修正出来的。

它针对的不是理论问题，而是非常实际的模型偷懒行为：只开头调用一次 router、skill 选得太少、假装后续 skill 已经使用、不同步原生 Todo List、UI 写完直接 QA、静态审查冒充浏览器 QA、中途重新从头规划、最终总结过度声明、Planned only 冒充 Actual skills used。

如果你也遇到类似问题，可以直接使用 V3.4，或者根据自己的 available skills 改造成属于你自己的版本。

---

## 维护建议

建议后续版本继续保留历史文件：

```text
所有版本/SKILL-v1.md
所有版本/SKILL-v2.md
所有版本/SKILL-v3.md
所有版本/SKILL-v3.1.md
所有版本/SKILL-v3.2.md
所有版本/SKILL-v3.3.md
所有版本/SKILL-v3.4.md
```

这样别人可以清楚看到这个 router 是如何一步步进化的。

未来可能方向：

- 更精确的 QA evidence，例如 `scrollWidth <= clientWidth`
- Skill 调用证据日志自动导出
- 自动根据用户 available skills 生成专属路由表
- 独立 `skill-router-auditor`，在任务结束后专门查账
