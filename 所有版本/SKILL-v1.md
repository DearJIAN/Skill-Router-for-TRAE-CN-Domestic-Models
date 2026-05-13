---
name: skill-router
description: "\"中文/English Skill Router for Trae CN. Use when user says: 优先调用可用skill/skills, 使用可用技能, 帮我选择skill, 根据available skills选择, 创建skill/新建skill/做一个skill, 找现成skill, 安装skill, 验证skill是否生效, 调用gstack的QA/browser testing/freeze/guard/review/ship能力. Routes tasks to current available skills only, prevents fake skill calls, and treats gstack as the top-level entry for its internal abilities.\""
---

# Skill Router

## Purpose / 用途

This skill is a routing and orchestration skill for Trae CN AI Coder.

它不是用来直接完成业务任务的，而是用来决定：

1. 当前任务应该优先使用哪些 available skills。
2. 这些 skills 应该在任务的哪个阶段使用。
3. 哪些 skill 不能使用，因为它们不在当前 available skills list 里。
4. 当用户想创建新 skill 时，是否应先搜索现成 skill。
5. 当需要 gstack 子能力时，如何通过顶层 `gstack` 触发。
6. 如何在任务结束时记录 skill 使用计划、使用总结和验证方式。

## Mandatory Trigger / 必须触发场景

When the user says any of the following Chinese or English phrases, this skill MUST be considered relevant:

- 请优先调用你可用的 skill 去帮助你
- 请优先调用你可用的 skills 去帮助你
- 优先使用可用 skill
- 优先使用可用 skills
- 优先使用可用技能
- 用你当前可用的 skills 辅助完成
- 用你当前可用的 skill 辅助完成
- 用你当前可用的技能辅助完成
- 结合 available skills
- 根据 available skills 选择
- 帮我判断该用哪些 skills
- 帮我选择合适的 skill
- 帮我选择合适的 skills
- 我想创建一个 skill
- 我想新建一个 skill
- 我想做一个 skill
- 帮我创建一个 skill
- 帮我写一个 SKILL.md
- 有没有现成的 skill
- 先找一下有没有现成 skill
- 找一个 skill
- 安装一个 skill
- 验证 skill 是否生效
- 测试 skill 是否被调用
- use available skills
- prioritize available skills
- choose suitable skills
- create a skill
- find an existing skill
- install a skill
- verify skill usage

## Core Rule / 核心规则

Before selecting or claiming any skill usage:

1. Inspect the current available skills list.
2. Only select skills that appear in the current available skills list.
3. Do not invent skill names.
4. Do not claim a skill was used unless it appears in available skills and was actually selected or invoked.
5. If a desired skill is not available, say so clearly and choose the closest available alternative.
6. If the user is Chinese, explain the routing plan and final summary in Chinese.

## Verification Marker / 生效验证标记

When this skill is used, include this exact line in the final response:

```text
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK
```

When the user explicitly asks to verify whether `skill-router` is active, also create or append this project-root file if file writing is available:

```text
.skill_router_trace.log
```

Append this block:

```text
=== SKILL ROUTER TRACE ===
Skill: skill-router
Proof: ROUTER_20260512_CN_GSTACK
Task: <one-line summary>
=== END TRACE ===
```

Do not claim this skill was used unless the final response marker is included.  
Do not claim the trace file was written unless it was actually created or updated.

## Available Skills Routing Table / 当前顶层 Skill 路由表

Use this table as the routing guide.  
这张表按**用途阶段**组织，只包含 Trae CN 当前 available skills list 中可直接调用的顶层 skill。

### 1. 启动与规划阶段 (Discovery & Planning)
| 工程用途场景 | Skill 名称 | 所属大 skill / skill 包 | 用途判断（功能描述） |
|---|---|---|---|
| 规划视角 / 抽象提级 | `zoom-out` | `mattpocock/skills` | 用于在实现细节之前拉高抽象层，梳理目标、约束与系统边界，减少局部最优导致的返工。 |
| 开始之前 / 需求拷问 | `grill-me` | `mattpocock/skills` | 用于开工前深入访谈用户、追问需求、挑战假设，避免需求没想清就写代码。 |
| 开始之前 / 文档化拷问 | `grill-with-docs` | `mattpocock/skills` | 用于边追问边沉淀文档，适合维护 `CONTEXT.md`、ADR、决策记录。 |
| 需求文档 / PRD | `to-prd` | `mattpocock/skills` | 用于把当前对话或需求整理成 PRD，并进一步提交为 issue。 |
| 需求拆解 / Issue | `to-issues` | `mattpocock/skills` | 用于把计划、PRD、spec 拆成可并行执行的 GitHub issues。 |
| 架构 / 重构规划 | `improve-codebase-architecture` | `mattpocock/skills` | 用于分析代码库架构、接口边界、模块关系，寻找重构和架构改进机会。 |
| 前端 / 设计系统规划 | `design-md` | `Google Stitch Skills / DESIGN.md` | 用于分析设计项目、生成或维护 `DESIGN.md`，作为项目设计系统和视觉规范源文件。 |

### 2. 设计与审美增强 (Design & Visuals)
| 工程用途场景 | Skill 名称 | 所属大 skill / skill 包 | 用途判断（功能描述） |
|---|---|---|---|
| 品牌 / 视觉系统 | `brandkit` | `Taste Skill` | 用于生成品牌视觉系统，包括 Logo 方向、配色、字体、品牌应用风格。 |
| 前端 / 设计工程规范 | `design-taste-frontend` | `Taste Skill` | 用于以工程约束驱动 UI/UX 设计落地，强调组件架构、性能约束、可维护样式与非模板化视觉输出。 |
| 前端 / 高端视觉参考图 | `high-end-visual-design` | `Taste Skill` | 用于生成高级、克制、精致、留白充足的高端网站视觉方向或参考图。 |
| Web / 参考图生成 | `imagegen-frontend-web` | `Taste Skill` | 用于生成网站、landing page、Web 页面参考图；偏视觉方向，不一定直接写代码。 |
| 移动端 / 参考图生成 | `imagegen-frontend-mobile` | `Taste Skill` | 用于生成移动端 App 页面、移动端流程参考图；偏设计参考，不一定直接写代码。 |
| 前端 / 极简 UI | `minimalist-ui` | `Taste Skill` | 用于 Notion、Linear、Vercel 风格的极简产品界面。 |
| 前端 / 粗野主义 UI | `industrial-brutalist-ui` | `Taste Skill` | 用于工业风、粗野主义、机械感、强对比、实验性布局 UI；普通商业项目慎用。 |
| 前端 / Stitch 设计口味 | `stitch-design-taste` | `Taste Skill` | 用于配合 Google Stitch 或 `DESIGN.md` 做统一设计口味输出。 |
| UI/UX 专家库 | `ui-ux-pro-max` | `UI UX Pro Max` | 用于专业 UI/UX 判断、产品体验评审、设计细节优化；适合补充设计审查。 |

### 3. 开发实现阶段 (Implementation)
| 工程用途场景 | Skill 名称 | 所属大 skill / skill 包 | 用途判断（功能描述） |
|---|---|---|---|
| 前端 / 高品质界面实现 | `frontend-design` | `anthropics/skills` | 用于创建高品质前端界面、页面、组件、dashboard、landing page。 |
| 前端 / 图转代码 | `image-to-code` | `Taste Skill` | 用于根据参考图、截图、视觉稿实现前端页面；适合“照着图写页面”。 |
| 代码输出 / 防半成品 | `full-output-enforcement` | `Taste Skill` | 用于强制 AI 输出完整代码，避免占位符、漏文件、TODO、半成品实现。 |
| 前端 / GPT 审美增强 | `gpt-taste` | `Taste Skill` | 用于加强 GPT/Codex 类模型的 UI/UX、布局变化、动效、anti-slop 输出。 |
| 前端 / UI UX 抛光 | `impeccable` | `Impeccable` | 用于设计、改进、重构前端界面，重点是审美、信息架构、视觉层级、间距、动效、响应式。 |
| 图标 / SVG / 前端资源 | `better-icons` | `Better Icons` | 用于搜索图标、获取 SVG、统一图标风格；适合替换粗糙图标、补图标库。 |
| 前端 / 原型验证 | `prototype` | `mattpocock/skills` | 用于快速构建一次性原型，验证 UI、状态流、交互和业务逻辑方向。 |
| 前端 / 旧项目改版 | `redesign-existing-projects` | `Taste Skill` | 用于改造已有项目 UI，先审计现状，再升级布局、间距、层级和样式。 |
| 测试 / TDD | `tdd` | `mattpocock/skills` | 用于测试驱动开发，强制红-绿-重构；适合写功能或修 bug 前先写失败测试。 |
| 修改 BUG / Debug | `diagnose` | `mattpocock/skills` | 用于硬 Bug、性能回归、复杂异常；按复现、最小化、假设、插桩、修复、回归测试流程处理。 |

### 4. 辅助与效能 (Utility & Meta)
| 工程用途场景 | Skill 名称 | 所属大 skill / skill 包 | 用途判断（功能描述） |
|---|---|---|---|
| 浏览器检测 / QA / GStack 总入口 | `gstack` | `garrytan/gstack` | Trae CN 识别到的 GStack 顶层 skill；用于无头浏览器 QA、真实页面检查、截图、点击、以及 GStack 内部工作流。 |
| Prompt / 提示词检索 | `prompt-lookup` | `Prompt Lookup` | 用于搜索、整理、复用提示词或提示词模板；适合让 AI 找合适 prompt、优化 prompt。 |
| 写 Skill / 管理 Skill | `skill-creator` | `anthropics/skills` | 用于创建、改进、评估自定义 skill；适合写 `SKILL.md`、设计调用验证、测试 prompt、benchmark/eval。 |
| 找 Skill / 安装 Skill | `find-skills` | `vercel-labs/skills` | 用于搜索、发现、筛选、安装 open agent skills 生态里的 skill。 |
| Skill 生态 / 快速初始化 | `setup-matt-pocock-skills` | `mattpocock/skills` | 用于初始化或批量配置 Matt Pocock 技能集合，适合在新环境快速补齐常用 skills 能力。 |
| 沟通风格 / 低废话 | `caveman` | `mattpocock/skills` | 用于极简沟通、减少废话和 token 消耗；适合让 AI 只输出关键结论。 |
| 交接 / 上下文压缩 | `handoff` | `mattpocock/skills` | 用于把当前对话压缩成 handoff 文档，让另一个 agent、新会话或未来的自己接手。 |

## Critical GStack Rule / GStack 子能力规则

Trae CN currently exposes `gstack` as a top-level skill.

当前 Trae CN 的 available skills list 里只有顶层 `gstack`，没有把它内部的子能力单独暴露成顶层 skill。

Therefore:

1. Do NOT directly claim you are calling `qa`, `qa-only`, `review`, `investigate`, `careful`, `freeze`, `guard`, `ship`, `browse`, or `browser testing` as top-level skills unless they appear in the current available skills list.
2. If the task needs those abilities, call the top-level skill: `gstack`.
3. In the task plan, clearly describe the intended gstack internal ability.

Chinese examples:

- 需要浏览器检测 / 页面 QA：调用 `gstack`，并说明“使用 gstack 的 browser testing / QA 能力”。
- 需要只检查不改代码：调用 `gstack`，并说明“使用 gstack 的 QA-only 行为，只输出报告，不修改代码”。
- 需要安全防误操作：调用 `gstack`，并说明“使用 gstack 的 freeze / guard / careful 意图限制修改范围”。
- 需要代码审查：调用 `gstack`，并说明“使用 gstack 的 review 意图进行工程审查”。
- 需要发布上线：调用 `gstack`，并说明“使用 gstack 的 ship / release 意图做发布检查”。

Preferred Chinese phrasing:

```text
我将调用顶层 skill：gstack，并在任务中使用它的 <QA/browser testing/freeze/guard/review/ship> 内部能力。
```

## Chrome DevTools MCP Preparation Rule

When using `gstack` browser testing, always prepare a clean runtime environment before calling Chrome DevTools MCP.

Before browser testing, do the cleanup first:

- Stop old frontend/backend dev servers started by previous AI tasks.
- Stop old Chrome DevTools MCP browser processes.
- Clean the old MCP browser profile cache:

```text
%USERPROFILE%\.cache\chrome-devtools-mcp\chrome-profile
```

- Restart the required frontend/backend service for the current task.
- Then run `gstack` browser testing with Chrome DevTools MCP.

If Chrome DevTools MCP returns:

```
The browser is already running for ... chrome-profile. Use --isolated to run multiple browser instances.
```

Do not fall back to static code review.

First clean old frontend/backend processes, old MCP browser processes, and the MCP browser profile cache again.

Then retry Chrome DevTools MCP.

Only fall back to static code review if browser testing still fails after cleanup and retry.

##  Main Workflow: When User Asks to Prioritize Skills

When the user says to prioritize available skills, follow this workflow.

### Stage 1: Inspect Available Skills

Before starting the task, check the current available skills list.

Then compare the current task against the routing table.

Do not rely only on memory.

Do not assume a skill exists.

### Stage 2: Select Skills

Choose suitable skills for each phase of the task.

Use this pattern in Chinese when the user uses Chinese:

```text
Skill 使用计划：
- 开始前：<skill-name> — <why>
- 规划阶段：<skill-name> — <why>
- 实现阶段：<skill-name> — <why>
- Review / QA：<skill-name> — <why>
- 验证阶段：<skill-name> — <why>
- 文档 / 交接：<skill-name> — <why>
```

Only include skills that are relevant.

Do not call every skill.

Prefer 1–5 of the most relevant skills by default. Complex tasks may use more than 5 skills, but you must explain the role of each skill and avoid calling skills just for the sake of calling them.

### Stage 3: Start-of-task Skill Choices

Use these rules before implementation:

- If the task needs high-level goal mapping, use `zoom-out`.
- If the requirement is unclear, use `grill-me`.
- If the requirement should become documentation, use `grill-with-docs`.
- If the user asks for PRD, use `to-prd`.
- If the user asks to split work into issues, use `to-issues`.
- If the task involves architecture or refactoring, use `improve-codebase-architecture`.
- If the task is UI-heavy or needs visual direction, consider `brandkit`, `design-md`, `design-taste-frontend`, `high-end-visual-design`, `imagegen-frontend-web`, `minimalist-ui`, or `stitch-design-taste`.
- If the task is based on an image, screenshot, or design reference, use `image-to-code`.
- If the task is only a prototype, use `prototype`.
- If setting up a new environment with standard tools, use `setup-matt-pocock-skills`.

### Stage 4: Implementation Skill Choices

During implementation:

- For frontend UI building, use `frontend-design`.
- For UI polishing, use `impeccable`.
- For strict complete output, use `full-output-enforcement`.
- For TDD, use `tdd`.
- For debugging, use `diagnose`.
- For specialized visual styles, use `industrial-brutalist-ui` or `stitch-design-taste`.
- For icons, use `better-icons`.
- For prompt creation or prompt optimization, use `prompt-lookup`.

### Stage 5: Mid-task Re-evaluation

If the task changes, stop and re-evaluate skills.

Examples:

- If a UI task becomes a bug hunt, switch or add `diagnose`.
- If a frontend task becomes visual polish, add `impeccable`.
- If a feature is ready and needs test discipline, add `tdd`.
- If browser verification is needed, use `gstack`.
- If the model starts producing incomplete code, use `full-output-enforcement`.

### Stage 6: Verification and QA

Before finishing:

- For unit or logic verification, use `tdd` if relevant.
- For debugging verification, use `diagnose`.
- For browser/page testing, use `gstack`.
- For UI visual review, use `impeccable` or `gstack` browser QA.
- For “only report, do not modify” QA, use `gstack` and explicitly request QA-only behavior.
- For dangerous operations, use `gstack` with careful/freeze/guard intent if appropriate.

### Stage 7: Documentation and Handoff

At the end:

- If the result should become a PRD, use `to-prd`.
- If the result should become issues, use `to-issues`.
- If the context should be handed to another agent, use `handoff`.
- If the task created or changed a skill, use `skill-creator`.
- If the user asks to document project context or decisions, use `grill-with-docs`.

When updating documentation, consider whether README, CHANGELOG, DESIGN.md, CONTEXT.md, ADR, or issue descriptions should be updated.

Do not update documentation unless it is useful or requested.

## Special Workflow: When User Wants to Create a New Skill

When the user says they want to create a skill, do NOT immediately create one.

Follow this order:

### Step 1: Clarify the desired skill

Extract:

- Skill name
- Target scenario
- Trigger conditions
- What the skill should do
- What the skill must not do
- Whether it should be Global or Project
- Whether it needs scripts, templates, examples, or reference files
- How to verify that it was actually called

If the description is incomplete, ask concise clarifying questions.

### Step 2: Search existing skills first

Use `find-skills` first if available.

Goal:

- Search whether an existing skill already matches the user's description.
- Prefer installing or adapting an existing skill if it is clearly suitable.
- Do not reinvent a skill if a trusted existing one already solves the problem.

If `find-skills` is unavailable, say so and continue with the best available alternative.

### Step 3: Decide install vs create

If a good existing skill is found:

- Summarize the candidate skill.
- Explain why it matches.
- Provide install command or import instructions.
- Ask whether to install/adapt it only if action requires user approval.

If no good existing skill is found:

- Use `skill-creator` if available.
- If `skill-creator` is unavailable but `write-a-skill` is available, use `write-a-skill`.
- If neither is available, create a standard `SKILL.md` manually.

### Step 4: Create the skill

When creating a skill, include:

- YAML frontmatter with `name` and `description`
- Clear trigger conditions
- Mandatory behavior
- Forbidden behavior
- Verification marker
- Example prompts
- Expected output format
- Safety constraints
- Anti-hallucination rule

### Step 5: Add call verification

Every newly created skill should include a lightweight verification mechanism.

Example:

```text
When this skill is used, include:
APPLIED_SKILL: <skill-name>
```

For stronger verification, create or append:

```text
.skill_trace.log
```

With:

```text
Skill: <skill-name>
Rule: <rule-id>
Proof: <random-token>
Task: <summary>
```

Do not claim the skill was used unless the verification behavior actually happened.

### Step 6: Test the skill

After creating a skill:

1. Ask Trae CN to list current available skills.
2. Confirm the new skill appears.
3. Run a canary test.
4. Run a real task test.
5. Disable or remove the skill and run a negative test if needed.

## Output Format When Routing Skills

When skills are requested or useful, start with:

```text
Skill 使用计划：
- <skill-name>: <why>
- <skill-name>: <why>
```

Then continue the task.

At the end, include:

```text
Skill 使用总结：
- 已使用 / 尝试使用：<skill-name>
- 用途：<what it helped with>
- 未使用但考虑过：<skill-name> — <why not>
- 验证方式：<tests, browser check, git diff, files changed, etc.>
```

If no skill is appropriate, say:

```text
当前任务没有明显需要调用的 skill。我将直接完成任务。
```

## Anti-Hallucination Rules

1. Never invent a skill.
2. Never claim to use a skill that is not in the available skills list.
3. Never claim `qa`, `review`, `careful`, `freeze`, or `ship` are top-level skills unless they appear in available skills.
4. For GStack internal abilities, call `gstack` and describe the intended internal ability.
5. If a skill is unavailable, say it is unavailable and choose another available skill.
6. If a skill was considered but not used, explain why.
7. Do not call skills just to call skills. Use them only when they help the task.
8. At the end of substantial tasks, summarize which skills helped and how.

## Default Behavior

If the user simply asks:

```text
请优先调用你可用的 skill 去帮助你。
```

Then do this:

1. Read available skills list.
2. Select relevant skills from the routing table.
3. Explain the skill plan briefly in Chinese.
4. Execute the task.
5. Re-evaluate skills during the task if the task changes.
6. Use verification or QA skills before finalizing when relevant.
7. Update docs or create handoff only when useful.
8. Summarize skill usage at the end.
9. Include the verification marker:
   `SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK`