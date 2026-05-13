---
name: skill-router
description: "中文/English Skill Router V2 for Trae CN. Use when user asks to prioritize available skills, choose suitable skills, create/find/install/verify skills, or use gstack QA/browser testing/freeze/guard/review/ship abilities. Routes only current available top-level skills, prevents fake skill calls, and orchestrates multi-skill expert collaboration across task phases."
---

# Skill Router V2 / Multi-Skill Expert Orchestrator

## Purpose / 用途

This skill is a **routing and orchestration skill** for Trae CN AI Coder.

它不是用来直接完成业务任务的，而是用来决定：

1. 当前任务应该优先使用哪些 available skills。
2. 当前任务属于哪个阶段，以及该阶段应该调用哪些 skills。
3. 哪些 skills 只能作为候选，哪些 skills 应该立即参与协作。
4. 哪些 skill 不能使用，因为它们不在当前 available skills list 里。
5. 当任务推进到新阶段时，何时必须重新调用本 router。
6. 当用户想创建新 skill 时，是否应先搜索现成 skill。
7. 当需要 gstack 子能力时，如何通过顶层 `gstack` 触发。
8. 如何在任务结束时记录 skill 使用计划、使用总结和验证方式。

V2 的核心变化：

- 从“找最少够用 skills”改为“组建多 skill 专家团”。
- 从“任务开始只路由一次”改为“每个关键阶段重新路由一次”。
- 从“默认 1–5 个”改为“中高激进度、多阶段、非冗余协作”。
- 每次 router 调用只选择**当前阶段**真正该用的 skills，避免一开始把所有阶段混在一起。
- 每个阶段输出不同的 `SKILL_ROUTER_APPLIED` 后缀 marker，方便用户审计是否真的重新路由。

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
- 让多个 skill 协同
- 多调用几个 skill
- 组一个 skill 专家团
- 每个阶段重新选择 skill
- 重新路由 skill
- router recheck
- skill recheck
- multi-skill
- skill council
- expert council
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

## Core Rules / 核心规则

Before selecting or claiming any skill usage:

1. MUST inspect the current available skills list.
2. MUST only select skills that appear in the current available skills list.
3. MUST NOT invent skill names.
4. MUST NOT claim a skill was used unless it appears in available skills and was actually selected or invoked.
5. If a desired skill is not available, say so clearly and choose the closest available alternative.
6. If the user is Chinese, explain the routing plan and final summary in Chinese.
7. MUST prefer **multi-skill collaboration** over minimal skill selection for non-trivial tasks.
8. MUST avoid lazy routing. Do not stop at 1–3 skills for a non-trivial task unless there is a clear reason.
9. MUST use skills for different roles, not duplicates of the same role.
10. MUST re-run this router at stage boundaries instead of relying only on the first routing decision.
11. MUST NOT carry over old first-pass decisions blindly. Every phase needs a fresh stage-local router pass.

## Stage Verification Markers / 分阶段生效验证标记

V2 uses **stage-specific verification markers**.

Do NOT use one generic marker to prove the whole task.
Each marker proves that the router was actually invoked or rechecked for one specific stage.

Base proof token:

```text
ROUTER_20260512_CN_GSTACK——V2
```

When this skill is used at a stage, include the exact stage marker for that stage.

| Router checkpoint | Stage marker | What it proves |
|---|---|---|
| `ROUTER_CHECKPOINT_0_TASK_MAP` | `SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-1` | Initial task mapping, complexity classification, candidate pool, and first-stage plan were produced. |
| `ROUTER_CHECKPOINT_1_DISCOVERY` | `SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-2` | Discovery / requirement-stage routing was actually run. |
| `ROUTER_CHECKPOINT_2_PLANNING_OR_DESIGN` | `SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-3` | Planning, architecture, or design-stage routing was actually run. |
| `ROUTER_CHECKPOINT_3_IMPLEMENTATION` | `SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-4` | Implementation-stage routing was actually run. |
| `ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH` | `SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-5` | Review, critique, polish, or completeness-stage routing was actually run. |
| `ROUTER_CHECKPOINT_5_QA_OR_VERIFICATION` | `SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-6` | QA, testing, browser verification, or regression-stage routing was actually run. |
| `ROUTER_CHECKPOINT_6_DOCUMENTATION_OR_HANDOFF` | `SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-7` | Documentation, handoff, issue, PRD, or final context-preservation routing was actually run. |

### Marker rules

1. MUST output a stage marker immediately after each visible router pass or router recheck.
2. MUST NOT output a stage marker for a stage that did not happen.
3. MUST NOT claim a stage router pass happened unless that stage marker appears.
4. MUST NOT use the unsuffixed base token as proof that all stages ran.
5. If a stage is skipped, do not output that stage marker. Instead write:

```text
Stage skipped: <checkpoint-name> — <reason>
```

6. If the platform cannot visibly output intermediate markers during tool work, the final response MUST still include a **Stage Marker Ledger** listing exactly which markers were produced, which stages were skipped, and which skills were used at each stage.
7. For non-trivial tasks, the final response MUST include every stage marker that actually ran, not just the last marker.

### Trace file rule

When the user explicitly asks to verify whether `skill-router` is active, also create or append this project-root file if file writing is available:

```text
.skill_router_trace.log
```

Append one block per stage router invocation:

```text
=== SKILL ROUTER TRACE ===
Skill: skill-router
Version: V2
Checkpoint: <checkpoint-name>
StageMarker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-<n>
Task: <one-line summary>
StageSkills: <comma-separated skills actually used at this stage>
Skipped: <yes/no and reason if yes>
=== END TRACE ===
```

Do not claim the trace file was written unless it was actually created or updated.

## V2 Collaboration Philosophy / V2 协作哲学

The router should not ask: “What is the minimum number of skills that can solve this?”

The router should ask: “Which group of available skills can collaborate across stages to produce a better result?”

中文原则：

```text
不要默认找最少够用的 skill。
要默认组建足够多、分工清晰、互不重复的 skill 专家团。
```

Use more skills when they provide different perspectives:

- requirement clarification
- planning
- architecture
- design taste
- implementation
- debugging
- testing
- browser QA
- visual polish
- documentation
- handoff
- skill creation / skill verification

Do NOT use more skills when they only duplicate the same job.

## Collaboration Intensity / 协作强度

Classify task complexity before selecting skills.

### Trivial task / 琐碎任务

Examples: answer a simple question, rename one variable, explain one line of code.

- Candidate pool: 1–3 skills
- Actually selected: 0–2 skills
- Router recheck: optional
- It is acceptable to say no skill is needed.

### Simple but real task / 简单但真实任务

Examples: small code change, small prompt, small UI tweak, small bug fix.

- Candidate pool: 3–6 skills
- Actually selected across the full task: 2–4 skills
- Per router invocation: select only current-stage skills
- Router recheck: at least before final QA if the task includes code or UI

### Medium task / 中等任务

Examples: implement a feature, refactor a module, create or improve a skill, redesign a page, fix a non-obvious bug.

- Candidate pool: 6–10 skills
- Actually selected across the full task: 4–7 skills
- Per router invocation: select only current-stage skills
- Router recheck: mandatory before implementation and before QA/finalization

### Complex task / 复杂任务

Examples: multi-file feature, architecture change, full UI/UX redesign, production QA, browser testing, new skill system, release preparation.

- Candidate pool: 10–16 skills
- Actually selected across the full task: 7–12 skills
- Per router invocation: select only current-stage skills
- Router recheck: mandatory at every major stage boundary

### High-skill-density tasks / 高密度 skill 任务

These task types should usually use a larger collaboration set:

- UI / frontend / landing page / dashboard / app page
- redesign / visual polish / image-to-code
- debugging hard bugs
- architecture or refactoring
- browser QA / production verification
- creating, improving, testing, or installing skills
- prompt system design
- release / ship checks

For these, do not stop at fewer than 5 total skills unless many expected skills are unavailable.

## Expert Council Roles / 专家团角色

For every non-trivial task, try to fill multiple expert roles. One skill may fill more than one role only if necessary, but prefer distinct skills when available.

| Role / 角色 | Purpose / 用途 | Example skills |
|---|---|---|
| Lead / 主导 | Define goal, scope, constraints | `zoom-out`, `to-prd` |
| Requirement Challenger / 需求拷问 | Challenge assumptions, clarify ambiguity | `grill-me`, `grill-with-docs` |
| Architecture / Design Planner | Plan structure, boundaries, visual system | `improve-codebase-architecture`, `design-md`, `design-taste-frontend`, `brandkit` |
| Specialist / 专业执行 | Implement the actual task | `frontend-design`, `image-to-code`, `prototype`, `diagnose`, `skill-creator` |
| Taste / Polish Reviewer | Improve visual quality, hierarchy, UX | `impeccable`, `ui-ux-pro-max`, `gpt-taste`, `minimalist-ui`, `high-end-visual-design` |
| Completeness Enforcer | Prevent placeholders, TODOs, half-output | `full-output-enforcement` |
| Test / Verification | Tests, browser QA, regression checks | `tdd`, `gstack`, `diagnose` |
| Documentation / Handoff | Summarize context, decisions, next steps | `handoff`, `to-issues`, `to-prd`, `grill-with-docs` |
| Skill Ecosystem | Search, create, improve, verify skills | `find-skills`, `skill-creator`, `prompt-lookup`, `setup-matt-pocock-skills` |

## Stage-Local Router Invocation Rule / 分阶段重新调用规则

This is a critical V2 rule.

Many agents only call a routing skill once at the beginning. This is not enough.

For non-trivial tasks, the agent MUST use this router repeatedly at stage boundaries.

### Required behavior

1. At the beginning, run **Router Pass 0: Task Map**.
2. Router Pass 0 should identify the task type, complexity, candidate skill pool, expected stages, and first-stage skills.
3. Router Pass 0 MUST also write explicit future router checkpoints.
4. Before each major phase, run a new router pass for that phase.
5. Each router pass MUST select only skills for the current phase.
6. Do not select all future-phase skills as “used” during the first pass.
7. Do not claim future-phase skills were used until that phase actually happens.

### Mandatory router checkpoints

For medium and complex tasks, include these checkpoints when relevant:

```text
ROUTER_CHECKPOINT_0_TASK_MAP
ROUTER_CHECKPOINT_1_DISCOVERY
ROUTER_CHECKPOINT_2_PLANNING_OR_DESIGN
ROUTER_CHECKPOINT_3_IMPLEMENTATION
ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH
ROUTER_CHECKPOINT_5_QA_OR_VERIFICATION
ROUTER_CHECKPOINT_6_DOCUMENTATION_OR_HANDOFF
```

If a task does not have a stage, skip that checkpoint and say why.

### Stage-local selection format

At each checkpoint, use this format:

```text
Router Pass: <checkpoint-name>
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-<n>
当前阶段：<stage>
本阶段目标：<goal>
本阶段候选 skills：
- <skill>: <why candidate>
本阶段实际调用 / 使用 skills：
- <skill>: <role in this phase>
暂不调用但后续可能使用：
- <skill>: <future phase>
下一次 router checkpoint：<checkpoint-name or none>
```

### If the platform cannot literally call this skill again

If the skill system only supports one explicit skill call, the agent must still perform a **router recheck** by re-reading the routing rules and current available skills before entering a new stage.

Use both the recheck marker and the corresponding stage verification marker in the response or internal task log:

```text
ROUTER_RECHECK_APPLIED: <checkpoint-name>
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-<n>
```

This does not replace real skill invocation when real invocation is possible. It is only a fallback for limited agents.
The suffixed stage marker is still required so the user can audit which stages were actually re-routed.

## Candidate Pool Rule / 候选池规则

Do not jump directly to final skill selection.

For non-trivial tasks, first build a candidate pool:

```text
Skill 候选池：
- 强相关：<skills directly useful now>
- 中相关：<skills useful in later stages or secondary roles>
- 备用：<skills possibly useful if task changes>
```

Then select the current-stage expert council from that pool.

This prevents the model from lazily selecting only 1–2 obvious skills.

## Minimum Coverage Rule / 最低覆盖规则

For non-trivial tasks, cover at least 3 of these categories across the full task unless unavailable:

1. Discovery / Planning
2. Design / Architecture
3. Implementation / Specialist Work
4. Review / Polish
5. QA / Verification
6. Documentation / Handoff

For complex tasks, cover at least 4 categories.

If fewer categories are covered, explicitly explain why.

## Minimum Skill Count Rule / 最低 skill 数规则

These are not absolute quotas, but the agent must treat them as strong defaults.

- Simple real task: do not select fewer than 2 skills if skills were requested.
- Medium task: do not select fewer than 4 skills across the full task.
- Complex task: do not select fewer than 7 skills across the full task.
- UI/frontend/design task: do not select fewer than 5 skills across the full task.
- Skill creation/improvement task: do not select fewer than 4 skills across the full task.

If the agent selects fewer than the default minimum, it MUST include:

```text
为什么没有使用更多 skills：
- <reason>
```

Weak reasons are not allowed. “The task seems simple” is not enough for medium or complex tasks.

## Anti-Redundancy Rule / 反冗余规则

More skills are good only when they create meaningful collaboration.

Do not use five visual style skills when one style direction is enough.
Do not use three planning skills when requirements are already clear.
Do not use both `prototype` and full implementation skills unless the task benefits from prototyping first.

Preferred pattern:

```text
1 planning skill + 1 specialist skill + 1 quality skill + 1 verification skill + optional handoff/documentation skill
```

For UI-heavy tasks:

```text
1 planning/design-system skill + 1 visual taste skill + 1 implementation skill + 1 polish skill + 1 QA/browser skill + 1 completeness skill
```

For skill-writing tasks:

```text
1 requirement skill + 1 existing-skill search skill + 1 prompt/skill design skill + 1 verification/testing skill + optional handoff skill
```

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

## Main Workflow V2: When User Asks to Prioritize Skills

When the user says to prioritize available skills, follow this workflow.

### Router Pass 0: Inspect Available Skills and Build Task Map

Before starting the task:

1. Inspect the current available skills list.
2. Compare the current task against the routing table.
3. Classify task type and complexity.
4. Build a broad candidate pool.
5. Select only the first-stage skills.
6. Write future router checkpoints.

Do not rely only on memory.
Do not assume a skill exists.
Do not claim later-stage skills are already used.

Chinese output pattern:

```text
Router Pass: ROUTER_CHECKPOINT_0_TASK_MAP
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-1
任务类型：<type>
复杂度：<trivial/simple/medium/complex>
Skill 候选池：
- 强相关：...
- 中相关：...
- 备用：...
本阶段实际调用 / 使用 skills：
- <skill>: <role>
后续 router checkpoints：
- <checkpoint>: <when>
```

### Stage 1: Discovery / Requirement Routing

Use this stage when the task needs goal mapping, ambiguity reduction, scope control, or user-intent extraction.

Prefer multiple complementary skills when relevant:

- If the task needs high-level goal mapping, use `zoom-out`.
- If the requirement is unclear, use `grill-me`.
- If the requirement should become documentation, use `grill-with-docs`.
- If the user asks for PRD, use `to-prd`.
- If the user asks to split work into issues, use `to-issues`.
- If creating/improving a skill, consider `skill-creator`, `find-skills`, `prompt-lookup`, and `grill-me`.

For medium/complex tasks, do not use only one discovery skill if multiple discovery perspectives are useful.

Before leaving this stage, run or schedule:

```text
Next router checkpoint: ROUTER_CHECKPOINT_2_PLANNING_OR_DESIGN
```

### Stage 2: Planning / Architecture / Design Routing

Use this stage before implementation when structure, design, UX, architecture, or project boundaries matter.

Rules:

- If the task involves architecture or refactoring, use `improve-codebase-architecture`.
- If the task is UI-heavy or needs visual direction, consider `brandkit`, `design-md`, `design-taste-frontend`, `high-end-visual-design`, `imagegen-frontend-web`, `minimalist-ui`, or `stitch-design-taste`.
- If the task needs professional UI/UX judgment, consider `ui-ux-pro-max`.
- If the task is based on an image, screenshot, or design reference, prepare to use `image-to-code` in implementation.
- If the task is only a prototype, use `prototype`.
- If setting up a new environment with standard tools, use `setup-matt-pocock-skills`.

For UI/frontend tasks, usually choose at least 2 design/planning perspectives before implementation, such as:

```text
- `design-taste-frontend` for engineering-aware UI direction
- `impeccable` or `ui-ux-pro-max` for quality standards
- `design-md` if a persistent design system file is needed
```

Before implementation, MUST run:

```text
ROUTER_RECHECK_APPLIED: ROUTER_CHECKPOINT_3_IMPLEMENTATION
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-4
```

or literally re-invoke this skill if the platform supports it.

### Stage 3: Implementation Routing

During implementation, select current-stage specialist skills.

Rules:

- For frontend UI building, use `frontend-design`.
- For image/screenshot/reference implementation, use `image-to-code`.
- For UI polishing during implementation, use `impeccable`.
- For strict complete output, use `full-output-enforcement`.
- For TDD, use `tdd`.
- For debugging, use `diagnose`.
- For specialized visual styles, use `industrial-brutalist-ui`, `minimalist-ui`, or `stitch-design-taste` when relevant.
- For icons, use `better-icons`.
- For prompt creation or prompt optimization, use `prompt-lookup`.
- For skill creation or modification, use `skill-creator`.

Implementation-stage expert council should usually include:

```text
- 1 specialist implementation skill
- 1 completeness / anti-half-output skill
- 1 quality or taste skill if UI/user-facing output is involved
- 1 test/debug skill if code behavior matters
```

After the first implementation pass, MUST run:

```text
ROUTER_RECHECK_APPLIED: ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-5
```

or literally re-invoke this skill.

### Stage 4: Review / Polish Routing

Before finalizing substantial work, route to review and polish skills.

Rules:

- For UI visual review, use `impeccable`, `ui-ux-pro-max`, or `gpt-taste` as appropriate.
- For architecture review, use `improve-codebase-architecture`.
- For code review / engineering QA, use `gstack` with review intent when appropriate.
- For dangerous operations, use `gstack` with careful/freeze/guard intent if appropriate.
- For incomplete output risk, use `full-output-enforcement`.

Review should not repeat implementation. It should inspect, critique, and improve.

Before verification, MUST run:

```text
ROUTER_RECHECK_APPLIED: ROUTER_CHECKPOINT_5_QA_OR_VERIFICATION
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-6
```

or literally re-invoke this skill.

### Stage 5: QA / Verification Routing

Before finishing:

- For unit or logic verification, use `tdd` if relevant.
- For debugging verification, use `diagnose`.
- For browser/page testing, use `gstack`.
- For UI visual browser review, use `gstack` browser QA and/or `impeccable`.
- For “only report, do not modify” QA, use `gstack` and explicitly request QA-only behavior.
- For release checks, use `gstack` with ship/release intent.

QA-stage skill choice should focus on evidence:

```text
- tests run
- browser checked
- screenshots inspected
- git diff reviewed
- files changed verified
- known limitations stated
```

After QA, if the task needs summary/docs/context preservation, run:

```text
ROUTER_RECHECK_APPLIED: ROUTER_CHECKPOINT_6_DOCUMENTATION_OR_HANDOFF
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-7
```

or literally re-invoke this skill.

### Stage 6: Documentation / Handoff Routing

At the end:

- If the result should become a PRD, use `to-prd`.
- If the result should become issues, use `to-issues`.
- If the context should be handed to another agent, use `handoff`.
- If the task created or changed a skill, use `skill-creator` to review the skill output and verification behavior.
- If the user asks to document project context or decisions, use `grill-with-docs`.

When updating documentation, consider whether README, CHANGELOG, DESIGN.md, CONTEXT.md, ADR, or issue descriptions should be updated.

Do not update documentation unless it is useful or requested.

## Mid-task Re-evaluation Triggers / 中途重新路由触发器

Immediately re-run this router when any of these happen:

- The task changes type.
- A UI task becomes a bug hunt.
- A bug hunt becomes an architecture issue.
- A feature becomes visual polish.
- The user asks for higher quality.
- The model starts producing placeholders, TODOs, or incomplete output.
- A browser or runtime test is needed.
- A test fails.
- A screenshot or design reference appears.
- A new file type appears.
- A skill is unavailable.
- The task becomes about creating, modifying, installing, or verifying skills.

When re-evaluating, do not repeat the entire initial plan. Only select skills for the current phase.

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

## Special V2 Workflow: When User Wants to Create or Improve a Skill

When the user wants to create, modify, harden, or verify a skill, treat it as a high-skill-density task.

Default expert council across the full task should consider:

- `grill-me` — clarify target scenario, trigger rules, forbidden behavior, verification.
- `find-skills` — search whether an existing skill already solves the need.
- `skill-creator` — create or improve `SKILL.md`.
- `prompt-lookup` — find reusable prompt patterns or strengthen instruction wording.
- `full-output-enforcement` — prevent incomplete skill files, missing sections, or vague TODOs.
- `handoff` — summarize usage and next testing steps if the work will continue.

Do not necessarily call all of them in the first pass. Use stage-local router passes:

```text
ROUTER_CHECKPOINT_1_DISCOVERY: clarify skill behavior
ROUTER_CHECKPOINT_2_PLANNING_OR_DESIGN: search/adapt existing skills and design structure
ROUTER_CHECKPOINT_3_IMPLEMENTATION: write or modify SKILL.md
ROUTER_CHECKPOINT_5_QA_OR_VERIFICATION: canary test, negative test, verification marker
ROUTER_CHECKPOINT_6_DOCUMENTATION_OR_HANDOFF: summarize installation/testing steps
```

## Output Format When Routing Skills / 路由输出格式

When skills are requested or useful, start with a router pass block.

### Initial pass format

```text
Router Pass: ROUTER_CHECKPOINT_0_TASK_MAP
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-1
任务类型：<type>
复杂度：<trivial/simple/medium/complex>
协作强度：<low/medium/medium-high/high>

Skill 候选池：
- 强相关：
  - <skill>: <why>
- 中相关：
  - <skill>: <why>
- 备用：
  - <skill>: <why>

本阶段实际调用 / 使用 skills：
- <skill>: <role in this phase>
- <skill>: <role in this phase>

后续 router checkpoints：
- <checkpoint>: <when to re-run router>
```

### Later phase format

```text
Router Pass: <checkpoint-name>
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-<n>
当前阶段：<stage>
本阶段目标：<goal>

本阶段候选 skills：
- <skill>: <why candidate>

本阶段实际调用 / 使用 skills：
- <skill>: <role in this phase>

暂不调用但后续可能使用：
- <skill>: <future phase>

下一次 router checkpoint：<checkpoint-name or none>
```

### Final summary format

At the end, include:

```text
Skill 使用总结：

阶段调用台账 / Stage Marker Ledger:
- ROUTER_CHECKPOINT_0_TASK_MAP
  - Marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-1
  - 已使用 / 尝试使用 skills: <skills actually used in this stage>
  - 阶段目标: <goal>
  - 验证方式: <evidence or notes>
- ROUTER_CHECKPOINT_1_DISCOVERY
  - Marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-2 OR Stage skipped: <reason>
  - 已使用 / 尝试使用 skills: <skills actually used in this stage>
  - 阶段目标: <goal>
  - 验证方式: <evidence or notes>
- ROUTER_CHECKPOINT_2_PLANNING_OR_DESIGN
  - Marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-3 OR Stage skipped: <reason>
  - 已使用 / 尝试使用 skills: <skills actually used in this stage>
  - 阶段目标: <goal>
  - 验证方式: <evidence or notes>
- ROUTER_CHECKPOINT_3_IMPLEMENTATION
  - Marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-4 OR Stage skipped: <reason>
  - 已使用 / 尝试使用 skills: <skills actually used in this stage>
  - 阶段目标: <goal>
  - 验证方式: <evidence or notes>
- ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH
  - Marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-5 OR Stage skipped: <reason>
  - 已使用 / 尝试使用 skills: <skills actually used in this stage>
  - 阶段目标: <goal>
  - 验证方式: <evidence or notes>
- ROUTER_CHECKPOINT_5_QA_OR_VERIFICATION
  - Marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-6 OR Stage skipped: <reason>
  - 已使用 / 尝试使用 skills: <skills actually used in this stage>
  - 阶段目标: <goal>
  - 验证方式: <evidence or notes>
- ROUTER_CHECKPOINT_6_DOCUMENTATION_OR_HANDOFF
  - Marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-7 OR Stage skipped: <reason>
  - 已使用 / 尝试使用 skills: <skills actually used in this stage>
  - 阶段目标: <goal>
  - 验证方式: <evidence or notes>

未使用但考虑过：
- <skill-name> — <why not>

可追加协作 skill：
- <skill-name> — <when useful next>

总体验证方式：<tests, browser check, git diff, files changed, etc.>
```

If no skill is appropriate, say:

```text
当前任务没有明显需要调用的 skill。我将直接完成任务。
为什么没有使用更多 skills：
- <reason>
Stage skipped: ROUTER_CHECKPOINT_0_TASK_MAP — no skill routing was useful for this trivial task
```

## Anti-Hallucination Rules / 防幻觉规则

1. Never invent a skill.
2. Never claim to use a skill that is not in the available skills list.
3. Never claim `qa`, `review`, `careful`, `freeze`, or `ship` are top-level skills unless they appear in available skills.
4. For GStack internal abilities, call `gstack` and describe the intended internal ability.
5. If a skill is unavailable, say it is unavailable and choose another available skill.
6. If a skill was considered but not used, explain why.
7. Do not call skills only to inflate the count.
8. Do not under-call skills because of laziness.
9. At the end of substantial tasks, summarize which skills helped and how.
10. If fewer skills than the minimum default were used, explain why in a dedicated section.
11. Do not claim future-stage skills were used during the initial routing pass.
12. Do not skip router rechecks for medium/complex tasks unless the task ended before that stage.
13. Do not let the old “prefer 1–5 skills” behavior override V2. V2 prefers broad, role-based collaboration.

## Default Behavior / 默认行为

If the user simply asks:

```text
请优先调用你可用的 skill 去帮助你。
```

Then do this:

1. Read available skills list.
2. Run `ROUTER_CHECKPOINT_0_TASK_MAP`.
3. Classify task type and complexity.
4. Build a candidate pool, usually broader than the final first-stage selection.
5. Select current-stage skills only.
6. Write the next router checkpoint.
7. Execute the current stage.
8. Re-run router at each stage boundary.
9. Use verification or QA skills before finalizing when relevant.
10. Update docs or create handoff only when useful.
11. Summarize skill usage at the end.
12. Include the stage markers for every router checkpoint that actually ran. At minimum, if routing was used at the beginning, include:

```text
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-1
```

Do not include markers for stages that did not happen.

## Compact Mode / 紧凑模式

If the user asks for a short answer or token-saving mode, still follow V2 routing internally, but compress the visible routing output.

Minimum visible compact output:

```text
Skill 专家团：<skill1>, <skill2>, <skill3>, ...
当前阶段：<stage>
下一次 router checkpoint：<checkpoint>
```

Do not use compact mode to avoid the required router rechecks.

## V2 Self-Check / 最终自检

Before finalizing any non-trivial task, verify:

- [ ] I inspected the current available skills list.
- [ ] I did not invent skill names.
- [ ] I built a candidate pool before selecting current-stage skills.
- [ ] I selected skills by role, not by random count.
- [ ] I used multiple skills for non-trivial tasks unless there was a real reason not to.
- [ ] I re-ran or scheduled router checkpoints at phase boundaries.
- [ ] I did not claim future-stage skills were already used.
- [ ] I used `gstack` only as a top-level skill and described internal intent separately.
- [ ] I included the correct suffixed V2 stage markers for every router checkpoint that actually ran.
- [ ] I did not include stage markers for skipped stages.
- [ ] I listed the skills used at each stage in the final Stage Marker Ledger.
