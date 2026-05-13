# Skill Router for TRAE CN Domestic Models

[中文版](./README.md)

> Chinese name: Skill Routing for TRAE CN Domestic Models
> A staged, auditable, multi-skill routing, collaboration, and governance framework for TRAE CN / domestic models.

This project provides a `skill-router` skill to help AI Coders in TRAE CN call available skills more reliably.

Its goal is not to let the model merely say it used a skill, but to make it truly re-route across task stages, invoke the right skills at the right time, sync the native Todo List, and leave behind an auditable stage ledger.

## Table of Contents

- [Suggested Companion Prompt (General Use)](#suggested-companion-prompt-general-use)
- [Repository Structure](#repository-structure)
- [Example Showcase](#example-showcase)
- [Why is this skill-router needed?](#why-is-this-skill-router-needed)
- [Current Recommended Version](#current-recommended-version)
- [Core Capabilities](#core-capabilities)
- [Version Evolution](#version-evolution)
- [Future Improvement Direction](#future-improvement-direction)
- [Installation](#installation)
- [How to Trigger It](#how-to-trigger-it)
- [Recommended Test Prompt](#recommended-test-prompt)
- [How to Judge Whether It Is Qualified](#how-to-judge-whether-it-is-qualified)
- [Common Signs of Non-Compliance](#common-signs-of-non-compliance)
- [How to Replace It with Your Own Skills](#how-to-replace-it-with-your-own-skills)
- [Common Violation Markers](#common-violation-markers)
- [Design Principles](#design-principles)
- [Open Source License](#open-source-license)
- [Acknowledgements](#acknowledgements)
- [Maintenance Notes](#maintenance-notes)

---

## Suggested Companion Prompt (General Use)

```
If the current environment supports TRAE's native task plan / Todo List, sync your router gates and actual work items into the native Todo List. Do not only write the plan in the chat message.

Dynamically choose skills based on the real task type and the current task stage. Before entering any new key stage, re-run skill-router. Each router pass should select only the skills that are genuinely needed for the current stage. Do not treat future-stage skills as already used.

The task plan must be generated dynamically for the current task. Do not reuse a fixed template. Different tasks may require different stages, such as requirement clarification, design/planning, architecture analysis, implementation, debugging, testing, review, visual polish, browser verification, documentation updates, and handoff summaries. Only add a dedicated visual review / polish stage when the task actually involves user-facing UI, webpages, PDF visuals, dashboards, cards, mobile screens, or visual design. Only add browser QA when the task actually requires real page behavior or browser-based verification.

Do not call skill-router only once at the beginning. Do not only list future router checkpoints in the chat message. Each key work item should have a corresponding router gate before it. If the native Todo List is available, sync router gates there first.
```

---

## Repository Structure

The recommended repository structure is:

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
│   └── SKILL-v3.3.md
└── 样例展示/
    ├── 调用过程展示1.png
    ├── 调用过程展示2.png
    ├── 输出总结展示1.png
    └── 输出总结展示2.png
```

### `skill-router/`

This is the **actual skill folder to use**.

What is inside now is already the current recommended latest version:

```text
skill-router/SKILL.md
```

In other words, the file currently placed in this folder is already the latest `V3.3`, and it is ready to use directly.

### `所有版本/`

This folder keeps historical versions so people can trace how the design evolved over time.

### `样例展示/`

This folder stores actual run screenshots, Todo List screenshots, Stage Marker Ledger screenshots, browser QA screenshots, and similar examples.

The README now references the image files that already exist in this repository.

---

## Example Showcase

The screenshots below are the examples already included in the current repository.

### Invocation Process Demo 1

![Invocation Process Demo 1](./样例展示/调用过程展示1.png)

This example mainly shows how the router performs staged routing and invocation during task execution.

---

### Invocation Process Demo 2

![Invocation Process Demo 2](./样例展示/调用过程展示2.png)

This example further shows how router passes, gates, and work items connect as the task continues.

---

### Final Output Summary Demo 1

![Final Output Summary Demo 1](./样例展示/输出总结展示1.png)

This example mainly shows how final output can present stage results, skill usage, and summary information.

---

### Final Output Summary Demo 2

![Final Output Summary Demo 2](./样例展示/输出总结展示2.png)

This example provides another real output style for the final summary section.

---

## Why is this skill-router needed?

This project comes from real frustrations I encountered while using domestic models inside TRAE CN.

The issue is not that TRAE CN domestic models cannot use skills. The issue is that they often show several very obvious failure patterns:

1. **They do not proactively call skills**  
   Even when the current task is clearly a good fit for certain skills, the model often does not use them on its own.  
   The user usually has to explicitly say things like "please prioritize available skills" or "remember to call skill-router" before it is likely to trigger.

2. **They call the router only once at the beginning**  
   Even when the user clearly reminds the model to use skills, it often calls the router only once at task start.  
   It may list a batch of "skills that might be used later," but once the task moves into implementation, debugging, polish, QA, documentation, and other real stages, it stops re-routing.

3. **They get lazy when the task scale grows**  
   As tasks become more complex, involve more steps, or touch more files, the model tends to trigger fewer skills.  
   It compresses skill usage as much as possible, sometimes calling only one or two skills and then pretending multi-skill collaboration already happened.

4. **They present "planned usage" as "actual usage"**  
   The model often writes future-stage skills into the plan at the beginning, then later talks about them in the final summary as if they were actually used.  
   This makes it hard for users to tell which skills were truly invoked and which ones were only written into the plan.

5. **They do not switch skills by stage**  
   A real development task usually has multiple stages: requirement clarification, planning, implementation, debugging, review, visual polish, QA, documentation, and handoff.  
   Different stages need different skills, but the model often tries to use the same set of skills selected at the beginning all the way through.

So the goal of this `skill-router` is not simply to "help the model choose skills." Its goal is to make the model:

- build a candidate skill pool at task start;
- dynamically choose only the skills that are truly needed for the current stage;
- call the router again before entering each new key stage;
- sync router gates and actual work items into the TRAE native Todo List;
- avoid counting future-stage skills as already used;
- output a final Stage Marker Ledger so users can audit which skills were actually used at each stage.

In one sentence:

> This project exists to solve the TRAE CN domestic model problem of being reluctant to call skills, calling them only once, calling too few of them, faking skill usage, and failing to re-route by stage.

---

## Current Recommended Version

Recommended version:

```text
SKILL-v3.3.md
```

For actual use, the effective file is:

```text
skill-router/SKILL.md
```

That means the `skill-router/` folder currently already contains the latest V3.3 version.

---

## Core Capabilities

This router mainly provides the following capabilities:

- **Available-skill routing**  
  It selects appropriate skills for the current task from the real set of available skills.

- **Prevention of invented skills**  
  It does not allow the model to invent nonexistent skill names, and it does not allow the model to claim it used a skill that does not actually exist.

- **Multi-skill expert-team collaboration**  
  It no longer defaults to using the fewest possible skills. Instead, it organizes staged collaboration across planning, implementation, debugging, polish, QA, documentation, and other roles based on task complexity.

- **Stage-by-stage re-routing**  
  It does not allow a single router call only at task start.  
  Before key stages such as implementation, debugging, review, QA, and documentation, the model is expected to call skill-router again.

- **Stage validation markers**  
  Each stage has independent markers such as `V3.3-1`, `V3.3-3`, `V3.3-4`, `V3.3-5`, and `V3.3-6`, making it easier to verify whether re-routing actually happened.

- **Todo-Bound Gates**  
  Router gates should not exist only in the chat message. They should be bound to the task plan / Todo List.

- **TRAE native Todo List sync**  
  If the current environment supports TRAE native task planning / Todo List, router gates and actual work items must be synced there first.

- **Resume-Safe continuation**  
  When re-routing mid-task, the model should not restart the old plan from scratch.  
  It should recognize what has already been completed and re-route only for the currently unfinished work cluster.

- **Forced UI Review / Polish gate**  
  For user-visible outputs such as webpages, dashboards, PDF front pages, cards, and mobile screens, the default expectation is a visual review and small-scope polish pass after implementation and before QA.

- **QA evidence constraints**  
  Static code inspection must not be used as a substitute for browser QA.  
  If `gstack` and Chrome DevTools MCP are used, the final ledger should explicitly record it as:  
  `gstack (browser testing intent) + Chrome DevTools MCP`

- **Special workflow for creating / improving / validating skills**  
  When the user wants to create a new skill, the model should not start writing immediately. It should first clarify requirements, search for an existing skill, decide whether to install or create, then use `skill-creator` or a standard `SKILL.md` flow, and include invocation validation and testing.

- **Final Stage Marker Ledger**  
  The final output should include each stage's marker, actually used skills, evidence, skip reasons, and validation method so the user can audit the work.

---

## Version Evolution

### V1: Basic Skill Router

V1 was the initial version. Its main capabilities were:

- selecting available skills according to the task
- preventing invented skill names
- providing a stage-organized routing table
- treating `gstack` as a top-level skill to trigger its internal QA / browser testing / review / guard / ship abilities
- adding a basic verification marker
- supporting special flows for creating, finding, installing, and validating skills

V1 already included a special workflow for creating a new skill: first clarify the requirement, then use `find-skills` to search for an existing option, and only if nothing suitable exists, create one manually with `skill-creator` or a standard `SKILL.md` flow, while also requiring invocation validation and testing steps.

Limitations:

- logic was relatively conservative
- default tendency was to choose only 1-3 skills
- it could easily route only once at task start

---

### V2: Multi-Skill Expert Collaboration

The core change in V2 was moving from:

```text
find the minimum number of skills needed
```

to:

```text
assemble a multi-skill expert team
```

Main improvements:

- introduced multi-skill collaboration
- expanded the candidate pool
- added stage markers
- required the final output to include a Stage Marker Ledger
- emphasized selecting only the truly needed skills for the current stage
- encouraged medium-to-high assertiveness in staged collaboration

Example markers:

```text
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-1
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-3
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-4
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V2-6
```

Limitations:

- weaker models could still list later checkpoints without actually executing them

---

### V3: Todo-Bound Hard-Gated Router

V3 began binding router gates to the task plan.

Main improvements:

- added `ROUTER_STAGE_GATE: OPEN`
- added `ROUTER_STAGE_GATE: CLOSED`
- required the relevant gate to be opened before each key stage
- required router checkpoints to be written into the task plan / todo list
- introduced a dynamic checkpoint generation algorithm
- disallowed using one fixed todo template for every task
- disallowed treating "future checkpoints" as proof of execution

Core execution loop:

```text
1. ROUTE_CURRENT_STAGE
2. OPEN_STAGE_GATE
3. DO_ONLY_THIS_STAGE_WORK
4. CLOSE_STAGE_GATE_WITH_EVIDENCE
5. RECHECK_NEXT_STAGE
6. Repeat until finalization
```

Limitations:

- during mid-task rechecks, some models restarted planning from Router Pass 0, causing progress rollback

---

### V3.1: Resume-Safe Continuation Patch

V3.1 solved the problem of restarting from the beginning during mid-task re-routing.

Main improvements:

- introduced resume-safe behavior
- required mid-task rechecks to continue from current progress rather than from scratch
- locked completed todo items by default
- prevented sudden regeneration of a 0/6 task list when 4/6 was already complete
- added violation markers for rollback and for static QA pretending to be full QA

Key mode:

```text
ROUTER_RECHECK_MODE: RESUME_FROM_CURRENT_PROGRESS
```

Typical violation:

```text
ROUTER_VIOLATION: MIDTASK_ROUTER_RESET_TO_TASK_MAP
```

Limitations:

- UI tasks could still jump directly from implementation to QA, without an independent visual review stage

---

### V3.2: UI Review / Polish Mandatory Gate

V3.2 added a polishing stage for user-facing artifacts.

As long as the task involves:

- websites
- landing pages
- dashboards
- forms
- card UI
- mobile pages
- PDF cover pages
- PDF report layouts
- report covers
- visual redesigns
- typography, spacing, hierarchy, color, or responsiveness

it should, by default, enter:

```text
ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH
```

The default chain for UI tasks becomes:

```text
TASK_MAP -> DESIGN/PLANNING -> IMPLEMENTATION -> REVIEW/POLISH -> QA/VERIFICATION
```

Why add this?

Because QA can only prove:

```text
whether it runs, whether it can be clicked, and whether there are obvious errors
```

But Review / Polish is responsible for:

```text
whether it looks good, whether the hierarchy is clear, whether the spacing feels right, and whether it looks like a finished product
```

Limitations:

- the design stage could still fail to invoke an actual design skill
- QA attribution could still be unclear, for example when the actual execution was `gstack + Chrome DevTools MCP` but only Chrome DevTools MCP was mentioned in the final summary

---

### V3.3: Native Todo Sync + Minimum Design Skill Requirement + QA Attribution Consistency

V3.3 is the currently recommended version.

V3.3 continues to preserve and strengthen the workflow for creating / modifying / installing / validating skills, and treats this class of work as a high-density skill task that often needs staged collaboration across multiple skills such as `find-skills`, `skill-creator`, `prompt-lookup`, `full-output-enforcement`, and `handoff`.

Main improvements:

1. **Native Todo Sync Gate**

   If the TRAE environment supports a native Todo List / task plan, router gates and work items must be synced into it.

   Writing gates only in chat does not count as fully todo-bound.
2. **Design Stage Skill Minimum**

   For visual tasks such as UI, frontend, PDF layout, dashboards, landing pages, and report covers, the design stage should by default use at least one design-oriented skill.

   Recommended design skills:

   ```text
   design-taste-frontend
   ui-ux-pro-max
   impeccable
   high-end-visual-design
   minimalist-ui (only when the style matches)
   ```
3. **QA Skill Attribution Consistency**

   If the QA stage selects `gstack` and uses Chrome DevTools MCP underneath, the final ledger must write:

   ```text
   gstack（browser testing intent）+ Chrome DevTools MCP
   ```

V3.3 solved two practical problems:

- the model wrote pretty gates in chat, but the official TRAE task list was never synced
- the model actually used `gstack` to trigger browser QA, but the final summary only wrote Chrome DevTools MCP, making auditing unclear

---

## Future Improvement Direction

### V3.3.1: QA Evidence Precision Rule

V3.3 is already usable, but QA evidence can still become more precise.

For example, during responsive checks, if the model only checks:

```js
overflowX
```

that is not rigorous enough.

A better horizontal overflow detection method is:

```js
document.documentElement.scrollWidth <= document.documentElement.clientWidth
```

Suggested future rule:

```text
When the model claims "no horizontal scroll / no horizontal overflow" on mobile, it must use scrollWidth and clientWidth to judge actual layout width, rather than only checking CSS overflow values.
```

This is not a blocker for V3.3, but it would make QA evidence stronger.

---

## Installation

### 1. Current latest file location

The latest version already placed in this repository is:

```text
skill-router/SKILL.md
```

The historical backup of `V3.3` is located at:

```text
所有版本/SKILL-v3.3.md
```

### 2. Confirm the skill name

The frontmatter at the top of `SKILL.md` should look roughly like:

```yaml
---
name: skill-router
description: "中文/English Skill Router V3.3 ..."
---
```

### 3. Confirm it is available in TRAE

Ask the model to list currently available skills:

```text
Please list your current available skills. Only list real skill names that actually exist, and do not guess.
```

If you can see:

```text
skill-router
```

then installation is basically successful.

---

## How to Trigger It

Common trigger prompts:

```text
Please prioritize using the skill / skills that are currently available to you.
```

or:

```text
Remember to call skill-router.
```

A stronger trigger prompt:

```text
Please use skill-router in V3.3 Native Todo Sync mode.
If the current environment supports TRAE native task planning / Todo List, sync the router gates and actual work items into the native plan list.
Do not call skill-router only once at the beginning.
```

---

## Recommended Test Prompt

### Test V3.3 Native Todo List + Multi-Stage Collaboration

```text
Please create a completely independent small demo and do not modify my existing project code.

Task: build a single-page web demo with the theme "Cyber Weather Console".

Requirements:
1. Create a new standalone directory.
2. Use HTML, CSS, and JavaScript.
3. Include a hero section, a current weather status card, a 4-hour forecast, an air quality / wind speed / humidity / UV data panel, and a city-switch button.
4. Visual style: dark high-tech dashboard, blue-purple neon, premium but restrained.
5. The page must be responsive and work well on desktop, tablet, and mobile.
6. After finishing, open the page and verify button interaction, visible page changes, responsive layout, and console errors.

Please prioritize using your currently available skill-router and suitable available skills to help complete the task.

If the current environment supports a TRAE native task plan / Todo List, sync router gates and actual work items into the native plan list instead of writing the plan only in chat.

Please use skills according to the real task stages: first plan the visual direction and structure, then implement, then do one round of visual review and small polishing, and finally perform browser verification.
```

Expected behavior:

- the TRAE native Todo List gets updated
- the design stage actually invokes a design skill
- the implementation stage invokes frontend implementation skills
- Review / Polish happens after implementation
- the QA stage uses `gstack（browser testing intent）+ Chrome DevTools MCP`
- the final output includes a Stage Marker Ledger

---

## How to Judge Whether It Is Qualified

A qualified run should show:

```text
ROUTER_CHECKPOINT_0_TASK_MAP
ROUTER_CHECKPOINT_2_PLANNING_OR_DESIGN
ROUTER_CHECKPOINT_3_IMPLEMENTATION
ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH
ROUTER_CHECKPOINT_5_QA_OR_VERIFICATION
```

And it should show markers like:

```text
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.3-1
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.3-3
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.3-4
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.3-5
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.3-6
```

And in the end it should also include:

```text
Stage Marker Ledger
```

with the actual skills used at each stage.

---

## Common Signs of Non-Compliance

### Only calling it once at the beginning

```text
Router Pass: ROUTER_CHECKPOINT_0_TASK_MAP
Later router checkpoints:
- implementation
- QA
```

Then the model just completes the whole task directly.

This is not compliant.

### Skipping polish for UI tasks

```text
Implementation complete, proceeding directly to QA
```

If the task is UI, a webpage, a PDF cover page, cards, or a dashboard, this is usually not compliant.

### Writing plans only in chat, without syncing native Todo

If TRAE supports a native Todo List, but the model only writes in chat:

```text
ROUTER_GATE ...
WORK ...
```

and does not trigger a native Todo List update, that does not count as fully compliant.

### Pretend QA

```text
Static code review passed, so browser QA passed
```

This is not compliant.

Static review cannot pretend to be browser QA.

### Unclear skill attribution

If `gstack` was selected and Chrome DevTools MCP was actually used for browser testing, the final output should write:

```text
gstack（browser testing intent）+ Chrome DevTools MCP
```

instead of only:

```text
Chrome DevTools MCP
```

---

## How to Replace It with Your Own Skills

The routing table and skill names in this README are examples from the author's own environment.

If you want to use it in your own environment, you must replace them with the skills you actually have.

### Step 1: List your available skills

Ask the model to run:

```text
Please list your current available skills. Only list real skill names that actually exist, and do not guess.
```

Copy down the real skill names.

### Step 2: Replace the Available Skills Routing Table

Open:

```text
skill-router/SKILL.md
```

Find:

```text
Available Skills Routing Table
```

Replace the author's example skills with your own skills.

Do not keep any skill that does not exist in your environment.

Wrong example:

```text
frontend-design
```

If your environment does not have that skill, it must not remain there.

Correct example:

```text
my-frontend-builder
```

provided that your environment really has that skill.

### Step 3: Categorize your skills by stage

It is recommended to categorize them by these stages:

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

For each skill, you should describe at least:

- the skill name
- suitable scenarios
- unsuitable scenarios
- which stage it belongs to
- how it differs from other skills

Example:

| Engineering scenario           | Skill name              | Skill package | Usage rule                                                                        |
| ------------------------------ | ----------------------- | ------------- | --------------------------------------------------------------------------------- |
| Frontend / page implementation | `my-frontend-builder` | `my-skills` | Used to build complete pages, components, and dashboards.                         |
| UI / polish review             | `my-ui-reviewer`      | `my-skills` | Used after implementation to check spacing, hierarchy, color, and responsiveness. |
| Browser QA                     | `my-browser-qa`       | `my-skills` | Used to open pages, take screenshots, click, and verify interactions.             |

### Step 4: Replace stage preference rules

Search in `SKILL.md` for content like:

```text
Preferred skills:
- design-taste-frontend
- ui-ux-pro-max
- impeccable
- high-end-visual-design
```

Replace them with your own design skills:

```text
Preferred skills:
- my-design-reviewer
- my-ux-critic
- my-visual-polisher
```

Likewise, also replace:

- implementation skills
- debugging skills
- QA skills
- documentation / handoff skills
- skill creation skills

### Step 5: If you do not have gstack

This router treats `gstack` as the top-level browser QA skill by default.

If your environment does not have `gstack`, replace all `gstack`-related rules with your own browser QA skill.

For example:

```text
Use `my-browser-test-skill` for browser QA.
When it uses Playwright or Chrome DevTools MCP underneath, attribute QA as:
my-browser-test-skill（browser testing intent）+ <actual execution mechanism>
```

If you do not have any browser QA skill, make the router explicitly say:

```text
No browser QA skill is currently available. Only static checks can be performed, and QA status is PARTIAL.
```

Do not let the model describe static checks as full QA.

### Step 6: Keep the anti-hallucination rules

Do not remove these rules:

```text
Never invent a skill.
Never claim a skill was used unless it exists and was actually selected or invoked.
Never claim future-stage skills were already used.
Never claim browser QA passed unless browser QA actually ran.
```

These are the foundation that makes the router auditable.

### Step 7: Customize verification markers

The default V3.3 markers are:

```text
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.3-1
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.3-3
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.3-4
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.3-5
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.3-6
```

You can change them to your own project token, for example:

```text
SKILL_ROUTER_APPLIED: ROUTER_20260512_MY_SKILL_SET——V3.3-4
```

But the replacement must be consistent across the whole file, including:

- marker tables
- trace rules
- Stage Marker Ledger examples
- marker references inside violation rules

---

## Common Violation Markers

This router includes built-in violation markers to expose lazy or fabricated skill usage behavior.

Common ones include:

```text
ROUTER_VIOLATION: ONLY_INITIAL_PASS_RAN
ROUTER_VIOLATION: STAGE_WORK_WITHOUT_STAGE_ROUTER_PASS
ROUTER_VIOLATION: MIDTASK_ROUTER_RESET_TO_TASK_MAP
ROUTER_VIOLATION: FIXED_TODO_PATTERN_REUSED
ROUTER_VIOLATION: REVIEW_POLISH_REQUIRED_BUT_SKIPPED
ROUTER_VIOLATION: QA_BEFORE_UI_POLISH_GATE
ROUTER_VIOLATION: NATIVE_TODO_SYNC_SKIPPED
ROUTER_VIOLATION: STATIC_QA_OVERCLAIM
```

If these markers appear, the run is not fully compliant.

---

## Design Principles

### 1. Staged, not one-shot

Do not let the model choose all skills at the beginning and then never route again.

### 2. Use only current-stage skills in the current stage

Skills from later stages may be listed as candidates, but cannot be claimed as already used.

### 3. Multi-skill collaboration, not random skill dumping

The goal is:

```text
planning skill + implementation skill + polish skill + QA skill
```

not:

```text
just pile up a bunch of skill names
```

### 4. Evidence first

Compared with "I used it," what matters more is:

- whether there is a marker
- whether there is a Todo List
- whether there are file changes
- whether there are browser screenshots
- whether there are test results
- whether there is a Stage Marker Ledger

### 5. Native Todo List matters

Many models follow the official task plan more closely than the chat body.

That is why V3.3 requires router gates to be written into the TRAE native Todo List whenever possible.

---

## Open Source License

Recommended:

```text
MIT License
```

You can also choose Apache-2.0, GPL, CC-BY, or another license if you prefer.

---

## Acknowledgements

This router was developed through repeated real-world testing and correction while using TRAE CN domestic models.

It is not targeting theoretical issues, but very practical lazy-model behaviors:

- calling the router only once at the beginning
- choosing too few skills
- pretending later-stage skills were already used
- not syncing the native Todo List
- jumping straight to QA after finishing UI
- using static review to pretend browser QA happened
- restarting planning from scratch in the middle
- overclaiming in the final summary

If you have run into similar problems, you can directly use V3.3, or adapt it into your own version based on your available skills.

---

## Maintenance Notes

It is recommended to continue keeping historical version files:

```text
所有版本/SKILL-v1.md
所有版本/SKILL-v2.md
所有版本/SKILL-v3.md
所有版本/SKILL-v3.1.md
所有版本/SKILL-v3.2.md
所有版本/SKILL-v3.3.md
```

That way, people can clearly see how this router evolved step by step.

Possible future directions:

- V3.3.1: more precise QA evidence, such as `scrollWidth <= clientWidth`
- V3.4: stronger trace-file auditing
- V4: automatically generate a dedicated routing table based on the user's available skills
