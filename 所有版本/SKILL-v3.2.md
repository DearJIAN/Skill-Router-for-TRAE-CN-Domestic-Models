---
name: skill-router
description: "中文/English Skill Router V3.2 UI-Polish-Required Resume-Safe Todo-Bound Hard-Gated for Trae CN. Use when user asks to prioritize available skills, choose suitable skills, create/find/install/verify skills, or use gstack QA/browser testing/freeze/guard/review/ship abilities. Routes only current available top-level skills, prevents fake skill calls, orchestrates multi-skill expert collaboration across task phases, and enforces visible stage-by-stage router checkpoints with hard gates, and binds router checkpoints to the task plan / todo list dynamically."
---

# Skill Router V3.2 UI-Polish-Required Resume-Safe Todo-Bound Hard-Gated / Multi-Skill Expert Orchestrator

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

V3.2 的核心变化：

- 从“找最少够用 skills”改为“组建多 skill 专家团”。
- 从“任务开始只路由一次”改为“每个关键阶段重新路由一次”。
- 从“默认 1–5 个”改为“中高激进度、多阶段、非冗余协作”。
- 每次 router 调用只选择**当前阶段**真正该用的 skills，避免一开始把所有阶段混在一起。
- 每个阶段输出不同的 `SKILL_ROUTER_APPLIED` 后缀 marker，方便用户审计是否真的重新路由。
- **新增 Todo-Bound Gates**：必须把 router checkpoint 写进任务规划 / todo list 作为独立阻塞项，而不是只在文字里说“后续会调用”。
- **新增 Dynamic Plan Rule**：todo gate 必须根据当前任务动态生成，禁止照抄固定模板或把任何任务都当成同一种任务。
- **新增 UI Review/Polish Required Gate**：任何创建或明显修改用户可见界面、PDF视觉、报告封面、卡片、dashboard、landing page、移动端页面的任务，默认必须在实现后、QA前插入 Review/Polish gate，不能用“实现阶段同步完成”或“QA 已覆盖”跳过。

## V3.2 Hard-Gate Patch / V3.2 硬闸门补丁

This patch exists because weak agents often do this bad pattern:

```text
Router Pass: ROUTER_CHECKPOINT_0_TASK_MAP
Stage marker: ...V3.2-1
后续 router checkpoints:
- ROUTER_CHECKPOINT_2_PLANNING_OR_DESIGN
- ROUTER_CHECKPOINT_3_IMPLEMENTATION
- ROUTER_CHECKPOINT_5_QA_OR_VERIFICATION
```

Then they continue the whole task without actually running those later checkpoints.

That is NOT compliant.

中文硬规则：

```text
列出“后续 router checkpoints”不等于执行了这些 checkpoints。
只有在进入某阶段之前，重新输出该阶段 Router Pass + 对应 V3.2-n marker，才算该阶段真的重新路由。
```

### Hard-gated execution loop / 硬闸门执行循环

For every non-trivial task, especially medium/complex tasks, the agent MUST follow this loop:

```text
1. ROUTE_CURRENT_STAGE
2. OPEN_STAGE_GATE
3. DO_ONLY_THIS_STAGE_WORK
4. CLOSE_STAGE_GATE_WITH_EVIDENCE
5. RECHECK_NEXT_STAGE
6. Repeat until finalization
```

Do NOT run this weaker loop:

```text
1. Route once at the beginning
2. Mention future checkpoints
3. Do all work
4. Summarize as if all checkpoints happened
```

### Stage gate states / 阶段闸门状态

Before doing any meaningful work for a stage, output a gate block:

```text
ROUTER_STAGE_GATE: OPEN
Router Pass: <checkpoint-name>
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-<n>
当前阶段：<stage>
本阶段允许使用 skills：<only current-stage skills>
本阶段禁止提前声称使用的后续 skills：<future-stage skills>
```

After finishing that stage, output or record a close block:

```text
ROUTER_STAGE_GATE: CLOSED
Completed checkpoint: <checkpoint-name>
本阶段实际使用 skills：<skills actually used>
阶段证据：<files changed / tests run / review notes / browser QA / docs updated>
Next required router checkpoint: <checkpoint-name or none>
```

If the agent starts implementation, QA, review, browser testing, documentation, or final summary while the corresponding gate was never opened, it MUST treat that as a router violation.

### Mandatory pre-action rule / 动作前强制规则

Before the first meaningful action in each stage, the agent MUST run the stage router pass.

Meaningful actions include:

- editing files
- writing implementation code
- redesigning UI
- debugging a runtime issue
- running tests
- launching browser QA
- reviewing code or visuals
- updating docs
- writing the final answer for a substantial task

Therefore:

- Before code edits, MUST run `ROUTER_CHECKPOINT_3_IMPLEMENTATION` and output `...V3.2-4`.
- Before polish/review, MUST run `ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH` and output `...V3.2-5`.
- Before tests/browser QA/final verification, MUST run `ROUTER_CHECKPOINT_5_QA_OR_VERIFICATION` and output `...V3.2-6`.
- Before docs/handoff/final context summary, MUST run `ROUTER_CHECKPOINT_6_DOCUMENTATION_OR_HANDOFF` and output `...V3.2-7` when relevant.

### Future checkpoint is not proof / 未来 checkpoint 不是证据

The following does NOT count as a router pass:

```text
后续 router checkpoints:
- ROUTER_CHECKPOINT_3_IMPLEMENTATION: 开始编码实现前
```

It is only a reminder.

The checkpoint counts only when the agent later outputs the current-stage block, for example:

```text
ROUTER_STAGE_GATE: OPEN
Router Pass: ROUTER_CHECKPOINT_3_IMPLEMENTATION
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-4
当前阶段：实现阶段
本阶段实际调用 / 使用 skills：
- frontend-design: 前端实现
- full-output-enforcement: 防止半成品
- diagnose: 下载链路 bug 定位
```

### Router violation rule / 路由违规规则

For a medium or complex task:

- If only `...V3.2-1` appears, the agent MUST NOT claim multi-stage routing happened.
- If implementation happened but `...V3.2-4` never appeared before implementation, mark it as a violation.
- If QA/testing/browser checks happened but `...V3.2-6` never appeared before QA, mark it as a violation.
- If final summary claims stages that have no markers, mark them as skipped or violation, not as completed.

Use this exact violation marker when applicable:

```text
ROUTER_VIOLATION: ONLY_INITIAL_PASS_RAN
```

or:

```text
ROUTER_VIOLATION: STAGE_WORK_WITHOUT_STAGE_ROUTER_PASS
```

### Minimum stage-pass count / 最低阶段调用次数

For complex tasks, the agent should normally produce at least these visible router passes unless the task ends early:

```text
V3.2-1: task map
V3.2-3: planning/design
V3.2-4: implementation
V3.2-5: review/polish
V3.2-6: QA/verification
```

If the complex task includes docs, handoff, or skill work, also use:

```text
V3.2-7: documentation/handoff
```

If the agent produces fewer than 4 stage markers for a complex task, it MUST include:

```text
为什么复杂任务没有产生至少 4 个阶段 marker：
- <real reason>
```

Weak reasons such as “I already planned it at the beginning” are invalid.


## V3.2 UI Review / Polish Required Gate Patch / V3.2 用户界面抛光强制闸门补丁

This patch exists because V3.1 can already call multiple stage routers, but weak agents may still jump from implementation directly to QA for UI tasks.

V3.2 fixes that by making `ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH` a **default required gate** for user-visible artifacts.

中文目标：

```text
只要任务创建或明显修改了用户能看到的东西，就不能“写完代码直接 QA”。
必须先做一轮 Review / Polish，再进入 QA。
```

### 1. UI artifact trigger / 用户可见产物触发条件

A task is considered UI / visual / user-facing if it creates or significantly changes any of these:

- webpage, landing page, dashboard, form, card UI, mobile screen, modal, component, widget
- PDF cover, PDF report layout, document template, report card, generated visual output
- screenshot-matched implementation, design reference implementation, style overhaul
- typography, spacing, visual hierarchy, color, motion, responsive layout
- anything the user describes with words like: 好看, 精致, 现代, 高级, 视觉, 审美, 排版, 字体, 卡片, 页面, 首页, 封面, 抛光, UI, UX, design, visual, polish, responsive

If any trigger matches, the router MUST treat `ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH` as required unless the task is explicitly logic-only.

### 2. Required gate chain for UI work / UI 工作强制闸门链

For UI / visual / user-facing work, the default gate chain is:

```text
TASK_MAP -> DESIGN/PLANNING -> IMPLEMENTATION -> REVIEW/POLISH -> QA/VERIFICATION
```

Therefore, after closing implementation for a UI task, the next required checkpoint MUST normally be:

```text
Next required router checkpoint: ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH
```

It MUST NOT jump directly from implementation to QA unless the task is purely logic-only or the user explicitly requests no visual review.

Bad:

```text
Completed checkpoint: ROUTER_CHECKPOINT_3_IMPLEMENTATION
Next required router checkpoint: ROUTER_CHECKPOINT_5_QA_OR_VERIFICATION
```

for a UI/page/report-cover/card/dashboard task.

Good:

```text
Completed checkpoint: ROUTER_CHECKPOINT_3_IMPLEMENTATION
Next required router checkpoint: ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH
```

### 3. QA cannot substitute polish / QA 不能代替抛光

Browser QA, unit tests, static code review, screenshots, or DOM snapshots do NOT replace Review / Polish.

Review / Polish answers:

```text
Is the artifact visually good, readable, balanced, aligned, polished, coherent, and suitable for the user goal?
```

QA answers:

```text
Does it run, render, interact, respond, and avoid obvious failures?
```

These are different gates.

A UI task that goes directly from implementation to QA MUST mark:

```text
ROUTER_VIOLATION: QA_BEFORE_UI_POLISH_GATE
```

### 4. No silent merge rule / 禁止“同步完成”跳过

For UI / visual / user-facing tasks, these skip reasons are invalid:

```text
在实现阶段同步完成
QA 阶段会一起检查
代码已经看起来不错
任务比较简单
已经使用了 frontend-design
```

If the task is UI-facing and the agent wants to skip Review / Polish, it MUST provide a concrete non-visual reason, such as:

```text
用户明确要求只修逻辑 bug，不允许改视觉。
本次修改完全发生在后端计算函数，没有用户可见产物。
```

Otherwise, skipping the gate is a violation:

```text
ROUTER_VIOLATION: REVIEW_POLISH_REQUIRED_BUT_SKIPPED
```

### 5. Review / Polish gate must inspect the built artifact / 抛光阶段必须检查已实现产物

`ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH` is not a second design-planning stage.

It happens **after implementation** and MUST inspect the actual artifact that was created or changed.

During this gate, the agent should use one or more current-stage skills such as:

- `impeccable`
- `ui-ux-pro-max`
- `design-taste-frontend`
- `gpt-taste`
- `minimalist-ui` / `high-end-visual-design` only when the style matches

The stage evidence should include at least one of:

- visual review notes against the built page/PDF/component
- screenshot inspection
- layout / spacing / hierarchy critique
- targeted CSS/layout edits made after critique
- explicit “no polish change needed” decision with concrete reasons

Weak evidence is invalid:

```text
Loaded design rules.
Looks fine.
QA passed.
```

### 6. Targeted polish only / 只做定向抛光，不重开任务

Review / Polish MUST NOT restart the task.

It may only:

- improve spacing, alignment, typography, hierarchy, contrast, rhythm, responsive behavior
- remove visual clutter
- make components more consistent
- fix awkward layout or readability issues
- make small targeted code changes based on review findings

It MUST NOT recreate the entire app/page unless the artifact is unusable.

If a major rewrite is required, the agent must open a new implementation gate after explaining why.

### 7. Todo-bound polish insertion / 任务列表必须插入抛光项

For UI-facing tasks, the todo list MUST contain a first-class blocking item for Review / Polish between implementation and QA.

Generic form:

```text
ROUTER_GATE ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH :: <task-specific visual review reason> :: after <implemented artifact> before <QA/checking>
WORK REVIEW/POLISH :: inspect and polish <specific artifact>
```

The wording MUST be task-specific. Do not copy this example literally.

If the todo list has implementation and QA items but no polish gate for a UI task, the plan is invalid and MUST be repaired before continuing:

```text
ROUTER_PLAN_REPAIR_REQUIRED: true
ROUTER_VIOLATION: UI_POLISH_GATE_MISSING_FROM_TODO
```

### 8. Minimum markers for UI tasks / UI 任务最低 marker

For a simple-but-real UI task, the normal minimum visible markers are:

```text
V3.2-1: task map
V3.2-3: design/planning
V3.2-4: implementation
V3.2-5: review/polish
V3.2-6: QA/verification
```

For a UI continuation / local visual tweak, the agent may skip V3.2-1 if it is already mid-task, but it should still normally produce:

```text
V3.2-4: implementation
V3.2-5: review/polish
V3.2-6: QA/verification
```

and use resume mode instead of starting over.

### 9. Final ledger requirement / 最终台账要求

For UI-facing tasks, the final Stage Marker Ledger MUST show the Review / Polish row separately from QA.

Bad ledger:

```text
Implementation: done
QA: gstack done
```

Good ledger:

```text
Implementation: files changed
Review / Polish: visual critique + targeted polish edits or no-change rationale
QA: browser/PDF/render/interaction verification
```

If the Review / Polish row is missing for a UI task, mark:

```text
ROUTER_VIOLATION: FINAL_LEDGER_MISSING_UI_POLISH_STAGE
```


## V3.2 Todo-Bound Dynamic Gate Patch / V3.2 任务列表绑定动态闸门补丁

This patch exists because weak agents often obey only the visible task plan / todo list. If router checkpoints are only written in prose, they are often ignored during execution.

这个补丁的目标是：把 skill-router 从“开头说一下”变成“任务规划系统里的阻塞闸门”。

### 1. Todo-bound rule / 路由必须绑定任务列表

For any non-trivial task, router checkpoints MUST be inserted into the agent's task plan / todo list as **first-class blocking tasks**.

The agent MUST NOT only write future checkpoints in prose.

A router checkpoint counts only when BOTH are true:

1. It appears as a visible task-plan / todo item before the related work item.
2. It later outputs the matching `ROUTER_STAGE_GATE: OPEN` block and the matching `SKILL_ROUTER_APPLIED` stage marker.

If either side is missing, the router pass is incomplete.

中文硬规则：

```text
如果任务列表里没有 ROUTER_CHECKPOINT todo，就不能声称该阶段会重新路由。
如果输出了 marker 但任务列表里没有对应 checkpoint，也只能算临时补救，不算规范执行。
如果任务列表里有 checkpoint 但没有输出 marker，也不算已经执行。
```

### 2. No fixed todo pattern / 禁止固定任务模板

Do NOT use a hardcoded todo list.

Do NOT assume every task has the same stages.

Do NOT copy any example todo list from this skill as the actual plan.

The todo-bound router plan MUST be dynamically synthesized from the current user's task.

每次生成任务列表前，先做动态判断：

```text
1. 当前任务到底包含哪些工作类型？
2. 哪些工作类型需要独立阶段？
3. 哪些阶段需要不同 skill 组合？
4. 哪些阶段可以跳过？为什么？
5. 哪些阶段虽然同属 implementation，但目标不同，应该拆成多个 gate？
```

A task plan is invalid if it looks generic, copied, or unrelated to the user's actual task.

Use this violation marker when applicable:

```text
ROUTER_VIOLATION: FIXED_TODO_PATTERN_REUSED
```

### 3. Dynamic checkpoint synthesis algorithm / 动态 checkpoint 生成算法

When building or updating the todo list, follow this algorithm:

```text
Step A — Extract task-specific work clusters
- Parse the user's request into concrete work clusters.
- A work cluster is a meaningful unit of work that may need a different skill mix.
- Examples of cluster types: bug diagnosis, UI redesign, API change, data migration, browser QA, documentation, skill creation, refactor planning, visual polish.

Step B — Classify each cluster by stage
Possible stages:
- discovery / requirement
- planning / architecture / design
- implementation / modification
- review / polish / completeness check
- QA / verification / regression
- documentation / handoff

Step C — Decide whether a router gate is needed
Insert a router gate before a cluster when:
- the next cluster enters a new stage;
- the skill mix should change;
- the risk level changes;
- the agent is about to edit files, run tests, perform browser QA, update docs, or finalize a substantial task;
- the task is medium/complex and the next cluster is meaningful work.

Step D — Insert the gate immediately before the work cluster
The checkpoint must be a separate todo item.
Do not merge it into the business work item.

Step E — Select only current-cluster skills
The router pass for this gate selects skills for the next cluster only.
It may list future candidate skills, but it MUST NOT claim they are already used.

Step F — Close the gate with evidence
After the cluster finishes, close the gate and record actual skills used and evidence.
```

### 4. Adaptive todo item format / 动态 todo 项格式

Use task-specific todo wording.

Preferred abstract format:

```text
[ ] ROUTER_GATE <checkpoint-name> :: <task-specific reason> :: before <specific work cluster>
[ ] WORK <stage-name> :: <specific user-task action>
```

This is a schema, not a fixed plan.

Good properties:

- The router gate names the exact upcoming work cluster.
- The work item uses words from the actual user task.
- Different domains get different gates when their skill needs differ.
- The list does not include irrelevant stages.

Bad properties:

- Every task always gets the same five todos.
- The plan says only “implement feature” without task-specific details.
- Router checkpoints are grouped at the top instead of placed before the related work.
- A single initial router checkpoint tries to cover the whole task.

### 5. Same-stage split rule / 同阶段多次拆分规则

The same stage checkpoint may appear multiple times if the skill mix changes.

For example, two implementation clusters may both use `ROUTER_CHECKPOINT_3_IMPLEMENTATION`, but one may need `diagnose + tdd`, while another may need `frontend-design + impeccable + full-output-enforcement`.

When this happens, keep the same official stage marker, but add a task-specific Gate ID:

```text
ROUTER_STAGE_GATE: OPEN
Router Pass: ROUTER_CHECKPOINT_3_IMPLEMENTATION
Gate ID: implementation/pdf-download-bug
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-4
当前阶段：实现 / 修复 PDF 下载链路
本阶段允许使用 skills：diagnose, tdd, full-output-enforcement
```

A later implementation gate could be:

```text
ROUTER_STAGE_GATE: OPEN
Router Pass: ROUTER_CHECKPOINT_3_IMPLEMENTATION
Gate ID: implementation/assessment-card-visual-upgrade
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-4
当前阶段：实现 / 在线测评卡片视觉升级
本阶段允许使用 skills：frontend-design, impeccable, design-taste-frontend, full-output-enforcement
```

The exact Gate ID must be derived from the current task, not from a fixed template.

### 6. Pre-work todo validation / 工作前任务列表校验

Before marking any non-router work item as `in_progress`, the agent MUST check:

```text
Does this work item have a directly related ROUTER_GATE todo before it?
Has that gate been completed with the correct stage marker?
Does the gate select skills for this work item rather than for the entire project?
```

If not, the agent MUST pause and rewrite the todo list before doing the work.

Use this violation marker if work has already started without the required todo-bound gate:

```text
ROUTER_VIOLATION: TODO_GATE_MISSING_BEFORE_STAGE
```

### 7. Todo update timing / 任务列表更新时间

The task plan / todo list must be updated at these moments:

1. Immediately after `ROUTER_CHECKPOINT_0_TASK_MAP`.
2. Before entering a new major work cluster.
3. When the user changes scope.
4. When debugging reveals a different root cause.
5. When implementation turns into design/review/QA/doc work.
6. Before final answer on a medium or complex task.

If the environment has a visible todo/update-plan tool, use it.
If the environment has no visible todo tool, print a compact `Router-bound task plan` block in the response.

### 8. Current-cluster-only skill selection / 只为当前工作簇选 skill

Each router gate may include:

- current-stage skills: allowed to use now;
- future candidate skills: maybe useful later;
- explicitly forbidden early claims: skills not yet used.

The agent MUST NOT count a future candidate as used.

At the end of each gate, the close block must distinguish:

```text
本阶段实际使用 skills：<actually used now>
候选但未使用 skills：<considered but not used now>
后续阶段可能使用 skills：<future only, not counted>
```

### 9. Stage Marker Ledger must bind todos to markers / 阶段台账必须绑定 todo 与 marker

At the end of a medium or complex task, output a ledger with one row per router-bound gate that actually existed in the todo list.

Required fields:

```text
阶段调用台账 / Stage Marker Ledger:
- Todo item: <exact router gate todo title>
  Checkpoint: <ROUTER_CHECKPOINT_x>
  Gate ID: <task-specific gate id>
  Marker: <SKILL_ROUTER_APPLIED ... V3.2-n OR skipped/violation>
  Current-stage skills actually used: <skills>
  Work cluster protected by this gate: <work item>
  Evidence: <files changed / tests run / browser QA / review notes / docs updated>
```

If a major work item has no matching ledger entry, mark it as violation or skipped with reason.

### 10. Plan repair rule / 任务计划修复规则

If the current todo list already exists and lacks router gates, do not continue blindly.

First repair it:

```text
ROUTER_PLAN_REPAIR_REQUIRED: true
原因：已有任务列表包含实现 / QA / 文档等工作项，但缺少对应 ROUTER_GATE todo。
修复动作：重新插入 task-specific router gates before the affected work items.
```

Then continue from the next unfinished work item.

### 11. Minimality without laziness / 不写死，也不偷懒

V3.2 is middle-high aggressive, not template-driven, and stricter for user-facing UI/visual artifacts.

Rules:

- Do not call every skill.
- Do not force every stage onto every task.
- Do not use a fixed todo plan.
- Do not stop at one initial router call for medium/complex tasks.
- Prefer multiple complementary skills when they bring different perspectives.
- For medium/complex tasks, prefer multiple router-bound gates distributed across actual work clusters.
- For trivial tasks, a single router pass may be enough, but explain why no additional gates are needed.


## V3.2 Resume-Safe Router Patch / V3.2 防回退续跑补丁

This patch exists because weak agents may correctly add todo-bound router gates, but then make a new mistake:

```text
1. The task is already 4/6 completed.
2. The agent enters QA or docs.
3. It calls skill-router again.
4. It restarts from ROUTER_CHECKPOINT_0_TASK_MAP and rebuilds the whole plan from zero.
```

That is NOT compliant.

中文硬规则：

```text
中途 router recheck 不是“从头重新规划”。
中途 router recheck 必须读取当前任务进度，只为下一个未完成工作簇重新路由。
已完成的 todo / work cluster 默认锁定，不得因为重新路由而回到未完成状态。
```

### 1. Resume anchor rule / 续跑锚点规则

Before every mid-task router pass, the agent MUST identify a **resume anchor**.

The resume anchor is the current progress state:

```text
ROUTER_RESUME_ANCHOR:
- Completed work clusters: <exact completed todos or observable completed work>
- Current active / next unfinished work cluster: <the next real work item>
- Existing router gates already completed: <markers already seen>
- Remaining work clusters only: <what still needs work>
```

If a task is already in progress, the agent MUST NOT run a fresh global `ROUTER_CHECKPOINT_0_TASK_MAP` unless one of these is true:

1. The user explicitly asks to restart planning from scratch.
2. The existing task plan is invalid or dangerously wrong.
3. The scope changed so much that the old plan no longer matches the task.

Even then, the agent MUST output:

```text
ROUTER_REPLAN_FROM_SCRATCH_ALLOWED: true
原因：<specific reason>
已完成工作保护：<which completed work must not be repeated>
```

### 2. No mid-task reset rule / 禁止中途重置规则

If any work item is already completed, a new router pass MUST operate in **resume mode**:

```text
ROUTER_RECHECK_MODE: RESUME_FROM_CURRENT_PROGRESS
```

It MUST NOT operate as:

```text
ROUTER_RECHECK_MODE: START_FROM_ZERO
```

unless `ROUTER_REPLAN_FROM_SCRATCH_ALLOWED: true` was explicitly justified.

Use this violation marker if the agent restarts from checkpoint 0 after meaningful progress already exists:

```text
ROUTER_VIOLATION: MIDTASK_ROUTER_RESET_TO_TASK_MAP
```

### 3. Completed work lock / 已完成工作锁定规则

Completed non-router work items are locked.

A completed work item MUST NOT be reopened, duplicated, or moved back to pending unless:

1. A test, browser QA, user feedback, or error proves it is not actually complete.
2. The user requests a change to that completed item.
3. The agent explicitly marks it as a regression fix.

If reopening is necessary, the agent MUST write:

```text
COMPLETED_WORK_REOPEN_REASON:
- Work item: <completed item being reopened>
- Reason: <test failure / user change / discovered regression>
- New router gate required before rework: <checkpoint>
```

Use this violation marker if completed work is silently reopened or the plan is restarted:

```text
ROUTER_VIOLATION: COMPLETED_WORK_REOPENED_WITHOUT_CAUSE
```

### 4. Current-stage recheck, not global replan / 当前阶段复检，不是全局重排

When entering a new stage after partial completion, the router pass must be stage-local.

Bad:

```text
Router Pass 0: 任务映射
重新生成完整 todo：
1. 修复 Bug
2. 重构模板
3. 升级 UI
4. QA
```

Good:

```text
ROUTER_RECHECK_MODE: RESUME_FROM_CURRENT_PROGRESS
ROUTER_RESUME_ANCHOR:
- Completed work clusters: PDF 下载修复；PDF 报告模板重构；在线测评卡片视觉升级
- Current active / next unfinished work cluster: 整体 QA
- Remaining work clusters only: gstack 浏览器 QA；README 更新

ROUTER_STAGE_GATE: OPEN
Router Pass: ROUTER_CHECKPOINT_5_QA_OR_VERIFICATION
Gate ID: qa/full-regression-after-pdf-and-card-changes
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-6
当前阶段允许使用 skills：gstack, diagnose, tdd
```

### 5. Todo update must be in-place / Todo 必须原地更新

When a visible todo list already exists, router recheck MUST update it in place.

Rules:

- Keep completed items completed.
- Insert missing router gate immediately before the next unfinished work item.
- Do not create a second independent todo list unless the old list is invalid.
- Do not move `ROUTER_CHECKPOINT_0_TASK_MAP` back to active after progress exists.
- Do not mark the overall task as restarted unless the user requested a restart.

If the agent must repair the todo list, it should say:

```text
ROUTER_PLAN_REPAIR_REQUIRED: true
ROUTER_RECHECK_MODE: RESUME_FROM_CURRENT_PROGRESS
修复范围：只修复剩余 todo 的 router gate，不重做已完成工作。
```

### 6. Router pass numbering discipline / Router Pass 编号纪律

After the initial task map has already completed, the agent MUST NOT label a later recheck as:

```text
Router Pass 0
```

unless it is a justified full restart.

Mid-task rechecks must use the real current checkpoint:

```text
Router Pass: ROUTER_CHECKPOINT_3_IMPLEMENTATION
Router Pass: ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH
Router Pass: ROUTER_CHECKPOINT_5_QA_OR_VERIFICATION
Router Pass: ROUTER_CHECKPOINT_6_DOCUMENTATION_OR_HANDOFF
```

For repeated same-stage gates, add a Gate ID instead of resetting to pass 0.

### 7. Review / polish cannot be silently merged / Review 抛光不可随口合并

For UI-heavy or design-heavy medium/complex tasks, `ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH` is normally required.

The agent MUST NOT skip review/polish with vague reasons such as:

```text
跳过（在实现阶段同步完成）
```

That reason is invalid for substantial UI work.

Valid options:

1. Run a real review/polish router gate and output `...V3.2-5`.
2. Mark review/polish as skipped only with a concrete reason, such as user time constraint, no UI/design changes, or unavailable required files.
3. If review/polish was genuinely performed inside implementation, still output a close block that lists the concrete review evidence.

Use this violation marker when a UI-heavy complex task skips review/polish by claiming it was “同步完成” without evidence:

```text
ROUTER_VIOLATION: REVIEW_POLISH_SKIPPED_BY_SYNC_CLAIM
```

### 8. QA evidence strength rule / QA 证据强度规则

Static code review is not equivalent to browser QA, runtime QA, or download verification.

If `gstack` or Chrome DevTools MCP is unavailable, the agent MUST NOT say:

```text
整体 QA 通过，所有改动已验证
```

unless it actually ran equivalent runtime checks.

Instead it must distinguish:

```text
QA 状态：PARTIAL
已完成：静态代码审查 / 文件差异检查 / 可运行的单元测试
未完成：浏览器交互验证 / 真实下载验证 / 视觉截图验证
阻塞原因：Chrome DevTools MCP 不可用 / gstack browser QA failed
下一步：需要恢复 MCP 后重新运行 gstack browser QA，或请用户手动验证指定流程
```

Use this violation marker if the agent claims full QA based only on static review for a UI/browser/download task:

```text
ROUTER_VIOLATION: STATIC_QA_OVERCLAIM
```

### 9. Actual skill invocation vs considered skill / 实际调用与候选区分

The final ledger must separate these three categories:

```text
实际调用 / 使用 skills：<the skills actually invoked or demonstrably followed>
候选但未实际调用 skills：<considered but not used>
应调用但失败 / 不可用 skills：<skill unavailable or tool failed, with reason>
```

If a skill is only named in a plan, it is not “used”.

If `gstack` was called but Chrome DevTools MCP failed, write:

```text
gstack: attempted / partially blocked
Browser QA: not completed unless screenshots/interactions succeeded
```

### 10. Final ledger must preserve progress chronology / 最终台账必须保留进度顺序

The final Stage Marker Ledger must reflect the real chronology.

Required additional fields:

```text
Stage Marker Ledger:
- Chronology index: <actual order>
- Resume mode: START_FROM_ZERO only for first pass; RESUME_FROM_CURRENT_PROGRESS for later passes
- Todo status before gate: <e.g. 3/6 completed, current item QA>
- Completed work preserved: <yes/no>
- Reopened completed work: <none or reason>
```

If the final ledger shows `ROUTER_CHECKPOINT_0_TASK_MAP` after 2+ completed work items without a justified restart, mark:

```text
ROUTER_VIOLATION: MIDTASK_ROUTER_RESET_TO_TASK_MAP
```

### 11. V3.2 recommended user-facing instruction / V3.2 推荐用户提示词

When the user wants strict multi-stage router behavior, the agent should understand this compact instruction:

```text
请使用 skill-router 的 V3.2 UI-Polish-Required Resume-Safe Todo-Bound Hard-Gated 模式。
每个 router gate 必须进入 todo list，且必须在对应业务工作前完成。
如果任务已经完成到中途，后续 router recheck 必须使用 RESUME_FROM_CURRENT_PROGRESS，只为下一个未完成工作簇路由，禁止重新从 Router Pass 0 开始规划。
已完成 todo 默认锁定，除非测试失败或用户改需求，否则不得回退。
UI/浏览器/下载类任务不能用静态审查冒充完整 QA；gstack 或 Chrome MCP 失败时只能标记 PARTIAL QA。
最终输出 Stage Marker Ledger，列出每个 gate 的 marker、实际 skills、todo 状态、证据、跳过原因和任何 violation。
```


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

V3.2 uses **stage-specific verification markers**.

Do NOT use one generic marker to prove the whole task.
Each marker proves that the router was actually invoked or rechecked for one specific stage.

Base proof token:

```text
ROUTER_20260512_CN_GSTACK——V3.2
```

When this skill is used at a stage, include the exact stage marker for that stage.

| Router checkpoint | Stage marker | What it proves |
|---|---|---|
| `ROUTER_CHECKPOINT_0_TASK_MAP` | `SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-1` | Initial task mapping, complexity classification, candidate pool, and first-stage plan were produced. |
| `ROUTER_CHECKPOINT_1_DISCOVERY` | `SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-2` | Discovery / requirement-stage routing was actually run. |
| `ROUTER_CHECKPOINT_2_PLANNING_OR_DESIGN` | `SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-3` | Planning, architecture, or design-stage routing was actually run. |
| `ROUTER_CHECKPOINT_3_IMPLEMENTATION` | `SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-4` | Implementation-stage routing was actually run. |
| `ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH` | `SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-5` | Review, critique, polish, or completeness-stage routing was actually run. |
| `ROUTER_CHECKPOINT_5_QA_OR_VERIFICATION` | `SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-6` | QA, testing, browser verification, or regression-stage routing was actually run. |
| `ROUTER_CHECKPOINT_6_DOCUMENTATION_OR_HANDOFF` | `SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-7` | Documentation, handoff, issue, PRD, or final context-preservation routing was actually run. |

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
8. A future checkpoint listed in Pass 0 is NOT proof that the stage router pass ran.
9. A final ledger alone is NOT proof if the corresponding stage marker never appeared before or during that stage's work.
10. For medium/complex tasks, if implementation, review, QA, or documentation happens without its corresponding stage marker, the agent MUST mark a router violation instead of pretending compliance.

### Trace file rule

When the user explicitly asks to verify whether `skill-router` is active, also create or append this project-root file if file writing is available:

```text
.skill_router_trace.log
```

Append one block per stage router invocation:

```text
=== SKILL ROUTER TRACE ===
Skill: skill-router
Version: V3
Checkpoint: <checkpoint-name>
StageMarker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-<n>
Task: <one-line summary>
StageSkills: <comma-separated skills actually used at this stage>
Skipped: <yes/no and reason if yes>
=== END TRACE ===
```

Do not claim the trace file was written unless it was actually created or updated.

## V3 Collaboration Philosophy / V3 协作哲学

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

This is a critical V3 rule.

Many agents only call a routing skill once at the beginning. This is not enough.

For non-trivial tasks, the agent MUST use this router repeatedly at stage boundaries.

### Required behavior

1. At the beginning, run **Router Pass 0: Task Map**.
2. Router Pass 0 should identify the task type, complexity, candidate skill pool, expected stages, and first-stage skills.
3. Router Pass 0 MUST also write explicit future router checkpoints.
4. Future checkpoints are only reminders. They are NOT counted as executed.
5. Before each major phase, run a new router pass for that phase and open the corresponding `ROUTER_STAGE_GATE`.
6. Each router pass MUST select only skills for the current phase.
7. Do not select all future-phase skills as “used” during the first pass.
8. Do not claim future-phase skills were used until that phase actually happens.
9. Do not begin implementation, review, QA, or documentation until the corresponding stage router pass has appeared.
10. After the stage work, close the stage gate with evidence.

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
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-<n>
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
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-<n>
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

## Main Workflow V3: When User Asks to Prioritize Skills

When the user says to prioritize available skills, follow this workflow.

### Router Pass 0: Inspect Available Skills and Build Task Map

Before starting the task:

1. Inspect the current available skills list.
2. Compare the current task against the routing table.
3. Classify task type and complexity.
4. Build a broad candidate pool.
5. Select only the first-stage skills.
6. Dynamically synthesize router-bound todo gates based on the actual task clusters.
7. Insert each router gate as a separate blocking todo before the related work item.
8. Write future router checkpoints only as reminders; they do not count until they become todo-bound gates and output markers.

Do not rely only on memory.
Do not assume a skill exists.
Do not claim later-stage skills are already used.

Chinese output pattern:

```text
Router Pass: ROUTER_CHECKPOINT_0_TASK_MAP
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-1
任务类型：<type>
复杂度：<trivial/simple/medium/complex>
Skill 候选池：
- 强相关：...
- 中相关：...
- 备用：...
本阶段实际调用 / 使用 skills：
- <skill>: <role>
Router-bound task plan / 路由绑定任务计划：
- ROUTER_GATE <checkpoint> :: <task-specific reason> :: before <specific work cluster>
- WORK <stage> :: <specific task action>
后续 router checkpoints：
- <checkpoint>: <when, reminder only, not proof>
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
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-4
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
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-5
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
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-6
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
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-7
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

## Special V3 Workflow: When User Wants to Create or Improve a Skill

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

## Bad Pattern and Good Pattern / 错误与正确示例

### Bad pattern: only initial routing

This is NOT compliant for a complex task:

```text
Router Pass: ROUTER_CHECKPOINT_0_TASK_MAP
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-1
任务类型：UI 体验升级 + PDF 报告模板重构 + 下载链路 Bug 修复
复杂度：complex

后续 router checkpoints：
- ROUTER_CHECKPOINT_2_PLANNING_OR_DESIGN
- ROUTER_CHECKPOINT_3_IMPLEMENTATION
- ROUTER_CHECKPOINT_5_QA_OR_VERIFICATION
```

Why this is bad:

- It only proves Pass 0 ran.
- It does not prove design-stage routing ran.
- It does not prove implementation-stage routing ran.
- It does not prove QA-stage routing ran.
- It listed future checkpoints but did not execute them.

If the agent then performs the whole task, final output must include:

```text
ROUTER_VIOLATION: ONLY_INITIAL_PASS_RAN
```

### Good pattern: stage-gated routing

```text
ROUTER_STAGE_GATE: OPEN
Router Pass: ROUTER_CHECKPOINT_0_TASK_MAP
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-1
当前阶段：任务映射
本阶段实际使用 skills：skill-router
ROUTER_STAGE_GATE: CLOSED
Next required router checkpoint: ROUTER_CHECKPOINT_2_PLANNING_OR_DESIGN
```

Before design work:

```text
ROUTER_STAGE_GATE: OPEN
Router Pass: ROUTER_CHECKPOINT_2_PLANNING_OR_DESIGN
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-3
当前阶段：设计 / 架构 / UI 方向
本阶段实际使用 skills：design-taste-frontend, ui-ux-pro-max, high-end-visual-design
ROUTER_STAGE_GATE: CLOSED
Next required router checkpoint: ROUTER_CHECKPOINT_3_IMPLEMENTATION
```

Before code work:

```text
ROUTER_STAGE_GATE: OPEN
Router Pass: ROUTER_CHECKPOINT_3_IMPLEMENTATION
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-4
当前阶段：实现
本阶段实际使用 skills：frontend-design, diagnose, full-output-enforcement
ROUTER_STAGE_GATE: CLOSED
Next required router checkpoint: ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH
```

Before QA:

```text
ROUTER_STAGE_GATE: OPEN
Router Pass: ROUTER_CHECKPOINT_5_QA_OR_VERIFICATION
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-6
当前阶段：QA / 验证
本阶段实际使用 skills：gstack, tdd, diagnose
ROUTER_STAGE_GATE: CLOSED
Next required router checkpoint: ROUTER_CHECKPOINT_6_DOCUMENTATION_OR_HANDOFF
```

## Stage Gate Output Format / 阶段闸门输出格式

Use this format at the beginning of every stage for non-trivial tasks:

```text
ROUTER_STAGE_GATE: OPEN
Router Pass: <checkpoint-name>
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-<n>
当前阶段：<stage>
本阶段目标：<goal>
本阶段候选 skills：
- <skill>: <why candidate>
本阶段实际调用 / 使用 skills：
- <skill>: <role in this phase>
本阶段禁止提前声称使用的后续 skills：
- <skill>: <future stage>
```

Use this format after the stage work:

```text
ROUTER_STAGE_GATE: CLOSED
Completed checkpoint: <checkpoint-name>
本阶段实际使用 skills：<skills actually used>
阶段证据：<evidence>
Next required router checkpoint: <checkpoint-name or none>
```

## Output Format When Routing Skills / 路由输出格式

When skills are requested or useful, start with a router pass block.

### Initial pass format

```text
Router Pass: ROUTER_CHECKPOINT_0_TASK_MAP
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-1
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
Stage marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-<n>
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
  - Marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-1
  - 已使用 / 尝试使用 skills: <skills actually used in this stage>
  - 阶段目标: <goal>
  - 验证方式: <evidence or notes>
- ROUTER_CHECKPOINT_1_DISCOVERY
  - Marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-2 OR Stage skipped: <reason>
  - 已使用 / 尝试使用 skills: <skills actually used in this stage>
  - 阶段目标: <goal>
  - 验证方式: <evidence or notes>
- ROUTER_CHECKPOINT_2_PLANNING_OR_DESIGN
  - Marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-3 OR Stage skipped: <reason>
  - 已使用 / 尝试使用 skills: <skills actually used in this stage>
  - 阶段目标: <goal>
  - 验证方式: <evidence or notes>
- ROUTER_CHECKPOINT_3_IMPLEMENTATION
  - Marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-4 OR Stage skipped: <reason>
  - 已使用 / 尝试使用 skills: <skills actually used in this stage>
  - 阶段目标: <goal>
  - 验证方式: <evidence or notes>
- ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH
  - Marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-5 OR Stage skipped: <reason>
  - 已使用 / 尝试使用 skills: <skills actually used in this stage>
  - 阶段目标: <goal>
  - 验证方式: <evidence or notes>
- ROUTER_CHECKPOINT_5_QA_OR_VERIFICATION
  - Marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-6 OR Stage skipped: <reason>
  - 已使用 / 尝试使用 skills: <skills actually used in this stage>
  - 阶段目标: <goal>
  - 验证方式: <evidence or notes>
- ROUTER_CHECKPOINT_6_DOCUMENTATION_OR_HANDOFF
  - Marker: SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-7 OR Stage skipped: <reason>
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
13. Do not let the old “prefer 1–5 skills” behavior override V3. V3 prefers broad, role-based collaboration.
14. Do not use a fixed todo list pattern. Router-bound todo gates must be dynamically derived from the user's actual task.
15. Do not treat prose-only future checkpoints as completed router passes. A completed pass needs both a todo gate and a stage marker.

## Default Behavior / 默认行为

If the user simply asks:

```text
请优先调用你可用的 skill 去帮助你。
```

Then do this:

1. Read available skills list.
2. Run `ROUTER_CHECKPOINT_0_TASK_MAP` and open `ROUTER_STAGE_GATE` for task mapping.
3. Classify task type and complexity.
4. Build a candidate pool, usually broader than the final first-stage selection.
5. Select current-stage skills only.
6. Close the task-map gate with evidence and write the next required router checkpoint.
7. Before doing the next stage's work, run that stage's router pass and open that stage gate.
8. Execute only that current stage.
9. Close that stage gate with evidence.
10. Repeat route/open/do/close at each stage boundary.
11. Use verification or QA skills before finalizing when relevant.
12. Update docs or create handoff only when useful.
13. Summarize skill usage at the end.
14. Include the stage markers for every router checkpoint that actually ran. At minimum, if routing was used at the beginning, include:

```text
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.2-1
```

Do not include markers for stages that did not happen.

## Compact Mode / 紧凑模式

If the user asks for a short answer or token-saving mode, still follow V3 routing internally, but compress the visible routing output.

Minimum visible compact output:

```text
Skill 专家团：<skill1>, <skill2>, <skill3>, ...
当前阶段：<stage>
下一次 router checkpoint：<checkpoint>
```

Do not use compact mode to avoid the required router rechecks.

## V3 Self-Check / 最终自检

Before finalizing any non-trivial task, verify:

- [ ] I inspected the current available skills list.
- [ ] I did not invent skill names.
- [ ] I built a candidate pool before selecting current-stage skills.
- [ ] I selected skills by role, not by random count.
- [ ] I used multiple skills for non-trivial tasks unless there was a real reason not to.
- [ ] I re-ran router checkpoints at phase boundaries; merely scheduling them was not counted as execution.
- [ ] I opened a `ROUTER_STAGE_GATE` before each implementation/review/QA/documentation phase.
- [ ] I closed each executed stage gate with evidence.
- [ ] I did not claim future-stage skills were already used.
- [ ] I used `gstack` only as a top-level skill and described internal intent separately.
- [ ] I included the correct suffixed V3 stage markers for every router checkpoint that actually ran.
- [ ] For non-trivial tasks, I inserted router checkpoints into the todo/task plan as separate blocking items.
- [ ] I did not reuse a fixed todo pattern; the gates were derived from the actual work clusters.
- [ ] Each major work item had a matching preceding router gate or a clear skipped/violation reason.
- [ ] I did not include stage markers for skipped stages.
- [ ] I listed the skills used at each stage in the final Stage Marker Ledger.
