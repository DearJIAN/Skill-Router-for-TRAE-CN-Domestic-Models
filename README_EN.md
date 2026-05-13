# Skill Router for TRAE CN Domestic Models

[中文版](./README.md)

> Chinese name: Skill Routing for TRAE CN Domestic Models  
> A staged, auditable, multi-skill routing, collaboration, and governance framework for TRAE CN / domestic models.

This project provides a `skill-router` skill to help AI Coders in TRAE CN call available skills more reliably.

Its goal is not to let the model merely say it used a skill, but to make it re-route across task stages, invoke the right skills at the right time, sync the native Todo List, and leave behind an auditable stage ledger.

V3.4 builds on V3.3 by adding **Skill Invocation Proof**: every skill in the final ledger must be classified as Confirmed, Inferred, Planned only, or Unavailable / failed, so planned skills cannot be falsely reported as actual calls.

## Table of Contents

- [Suggested Companion Prompt (General Use)](#suggested-companion-prompt-general-use)
- [Repository Structure](#repository-structure)
- [Example Showcase](#example-showcase)
- [Why this skill-router exists](#why-this-skill-router-exists)
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

```text
Prioritize skill-router V3.4 for staged dynamic routing.

If TRAE native Todo List is available, sync router gates and actual work items into it.

Do not call skill-router only once at the beginning. Before entering each key stage, run a new router pass and select only the skills genuinely needed for the current stage.

The final answer must include a Stage Marker Ledger, and every skill must be labeled with an invocation evidence level: Confirmed / Inferred / Planned only / Unavailable failed.

Do not list any skill as actually used unless there is `toolName: Skill` evidence or clear skill loading output.

If a skill is claimed as used but has no invocation evidence, mark:
ROUTER_VIOLATION: SKILL_CLAIM_WITHOUT_INVOCATION
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
│   ├── SKILL-v3.3.md
│   └── SKILL-v3.4.md
└── 样例展示/
    ├── 调用过程展示1.png
    ├── 调用过程展示2.png
    ├── 输出总结展示1.png
    └── 输出总结展示2.png
```

### `skill-router/`

This is the **actual skill folder to use**.

For real use, place the current recommended V3.4 content into:

```text
skill-router/SKILL.md
```

### `所有版本/`

This folder keeps historical versions so people can trace how the design evolved. It is recommended to keep `SKILL-v3.4.md` here as well.

### `样例展示/`

This folder stores actual run screenshots, Todo List screenshots, Stage Marker Ledger screenshots, browser QA screenshots, and similar examples.

The README references the image files that already exist in this repository.

---

## Example Showcase

The screenshots below are examples already included in the current repository.

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

In V3.4, the final ledger should additionally show Confirmed / Inferred / Planned only / Unavailable failed to distinguish real calls, behavior-inferred guidance, and planned-only skills.

---

## Why this skill-router exists

This project comes from real frustrations I encountered while using domestic models inside TRAE CN.

The issue is not that TRAE CN domestic models cannot use skills at all. The problem is that they often show several obvious failure patterns:

1. **They do not proactively call skills**  
   Even when a task clearly fits certain skills, the model often does not call them on its own. The user usually has to explicitly say “please prioritize available skills” or “remember to call skill-router” before it is likely to trigger.

2. **They call the router only once at the beginning**  
   Even when reminded, the model often calls the router only once at task start. It may list “skills that might be used later,” but once the task moves into implementation, debugging, polish, QA, or documentation, it stops re-routing.

3. **They get lazy as task scale grows**  
   As tasks become larger, touch more files, or involve more stages, the model tends to minimize skill calls. It may call only one or two skills and then pretend multi-skill collaboration happened.

4. **They present planned usage as actual usage**  
   Future-stage skills are often written into the plan and then later reported as if they were actually invoked.

5. **They do not switch skills by stage**  
   Real development tasks usually include requirement clarification, planning, implementation, debugging, review, visual polish, QA, documentation, and handoff. Different stages need different skills, but weaker agents often try to reuse the same first-pass skill set for the whole task.

6. **Skill invocation is hard to audit**  
   The model may claim that certain skills were used, while the TRAE UI log does not show corresponding `toolName: Skill` calls. V3.4 therefore adds invocation evidence levels: Confirmed, Inferred, Planned only, and Unavailable / failed.

So this `skill-router` is not merely a “skill picker.” It is designed to make the model build a candidate pool, re-route at key stages, sync TRAE native Todo List when possible, and mark each skill’s invocation evidence level in the final Stage Marker Ledger.

In one sentence:

> This project exists to address TRAE CN domestic models being reluctant to call skills, calling them only once, calling too few, faking usage, failing to re-route by stage, and making skill invocation hard to audit.

---

## Current Recommended Version

Recommended version:

```text
SKILL-v3.4.md
```

For actual use, the effective file is:

```text
skill-router/SKILL.md
```

In other words, the current recommended version is V3.4. For real use, put the V3.4 content into `skill-router/SKILL.md`.

---

## Core Capabilities

This router mainly provides the following capabilities:

- **Available-skill routing**  
  Select appropriate skills for the current task from the real set of available skills.

- **Prevention of invented skills**  
  Prevent the model from inventing nonexistent skill names or claiming it used a skill that does not exist.

- **Multi-skill expert-team collaboration**  
  Instead of defaulting to the fewest possible skills, it organizes collaboration across planning, implementation, debugging, polish, QA, documentation, and other roles.

- **Stage-by-stage re-routing**  
  Do not allow a single router call only at task start. The model should re-route before key stages such as implementation, debugging, review, QA, and documentation.

- **Stage validation markers**  
  Each stage has independent markers such as `V3.4-1`, `V3.4-3`, `V3.4-4`, `V3.4-5`, and `V3.4-6`.

- **Todo-Bound Gates**  
  Router gates should not exist only in chat. They should be bound to the task plan / Todo List.

- **TRAE native Todo List sync**  
  If the current environment supports TRAE native task planning / Todo List, router gates and actual work items must be synced there first.

- **Resume-Safe continuation**  
  When re-routing mid-task, the model should not restart from scratch. It should recognize completed work and re-route only for the current unfinished work cluster.

- **Required UI Review / Polish gate**  
  For user-visible outputs such as webpages, dashboards, PDF covers, cards, and mobile screens, the default expectation is a visual review and small-scope polish pass after implementation and before QA.

- **QA evidence constraints**  
  Static code review must not pretend to be browser QA. If `gstack` and Chrome DevTools MCP are used, the final ledger should explicitly write:  
  `gstack (browser testing intent) + Chrome DevTools MCP`

- **Skill invocation evidence audit (new in V3.4)**  
  Every skill must be labeled with an evidence level:
  - Confirmed: TRAE UI explicitly shows `toolName: Skill`, or there is clear skill loading output
  - Inferred: the UI does not show a skill call, but the behavior clearly follows that skill’s rules
  - Planned only: the skill was only planned or listed as a candidate and must not be counted as actually used
  - Unavailable / failed: the skill was intended but unavailable or failed

- **Prevention of fake Actual skills used (new in V3.4)**  
  Planned only skills must not be listed as Actual skills used. If a skill is claimed as used but has no invocation evidence, mark:  
  `ROUTER_VIOLATION: SKILL_CLAIM_WITHOUT_INVOCATION`

- **Special workflow for creating / improving / validating skills**  
  When the user wants to create a new skill, the model should first clarify requirements, search for existing skills, decide whether to install or create, then use `skill-creator` or a standard `SKILL.md` flow, and include invocation validation and testing.

- **Final Stage Marker Ledger**  
  The final output should include each stage’s marker, actual skills, evidence level, skip reasons, and validation method so the user can audit the work.

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

V1 already included a special workflow for creating a new skill: clarify the requirement, search for an existing option with `find-skills`, and only create a new skill manually with `skill-creator` or a standard `SKILL.md` flow if nothing suitable exists.

Limitations:

- relatively conservative logic
- default tendency to choose only 1-3 skills
- easy to route only once at task start

---

### V2: Multi-Skill Expert Collaboration

V2 moved from “find the minimum number of skills needed” to “assemble a multi-skill expert team.”

Main improvements:

- introduced multi-skill collaboration
- expanded the candidate pool
- added stage markers
- required a final Stage Marker Ledger
- emphasized selecting only the skills truly needed for the current stage
- encouraged medium-to-high assertiveness in staged collaboration

Limitation:

- weaker models could still list future checkpoints without actually executing them

---

### V3: Todo-Bound Hard-Gated Router

V3 began binding router gates to the task plan.

Main improvements:

- added `ROUTER_STAGE_GATE: OPEN / CLOSED`
- required the relevant gate to open before each key stage
- required router checkpoints to be written into the task plan / todo list
- introduced a dynamic checkpoint generation algorithm
- disallowed using a fixed todo template for every task
- disallowed treating “future checkpoints” as proof of execution

Limitation:

- during mid-task rechecks, some models restarted from Router Pass 0 and rolled progress back

---

### V3.1: Resume-Safe Continuation Patch

V3.1 solved the problem of restarting from scratch during mid-task re-routing.

Main improvements:

- introduced resume-safe behavior
- required mid-task rechecks to continue from current progress
- locked completed todo items by default
- prevented sudden regeneration of a 0/6 task list when 4/6 was already complete
- added violation markers for rollback and static QA overclaiming

Key mode:

```text
ROUTER_RECHECK_MODE: RESUME_FROM_CURRENT_PROGRESS
```

Limitation:

- UI tasks could still jump directly from implementation to QA without an independent visual review stage

---

### V3.2: Required UI Review / Polish Gate

V3.2 added a polishing stage for user-facing artifacts.

If a task involves webpages, landing pages, dashboards, forms, cards, mobile pages, PDF covers, PDF report layouts, typography, spacing, hierarchy, color, or responsiveness, it should normally enter:

```text
ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH
```

The default chain for UI tasks becomes:

```text
TASK_MAP -> DESIGN/PLANNING -> IMPLEMENTATION -> REVIEW/POLISH -> QA/VERIFICATION
```

Limitation:

- the design stage could still fail to invoke an actual design skill
- QA attribution could still be unclear

---

### V3.3: Native Todo Sync + Minimum Design Skill Requirement + QA Attribution Consistency

V3.3 preserved and strengthened the workflow for creating / modifying / installing / validating skills, and treated this class of work as high-skill-density work.

Main improvements:

1. **Native Todo Sync Gate**: If TRAE supports a native Todo List / task plan, router gates and work items must be synced into it.
2. **Design Stage Skill Minimum**: For visual work such as UI, frontend, PDF layout, dashboards, landing pages, and report covers, the design stage should usually use at least one design-oriented skill.
3. **QA Skill Attribution Consistency**: If the QA stage selects `gstack` and uses Chrome DevTools MCP underneath, the final ledger must write `gstack（browser testing intent）+ Chrome DevTools MCP`.

V3.3 solved two practical problems:

- the model wrote beautiful gates in chat, but the official TRAE task list was never synced
- the model actually used `gstack` to trigger browser QA, but the final summary only wrote Chrome DevTools MCP

---

### V3.4 — Skill Invocation Proof

V3.4 fixes a key issue exposed by V3.3:

A model may already be doing staged routing and native Todo List sync, but still report “planned skills” as “actually invoked skills” in the final ledger.

For example:

- `diagnose` was only written into the plan, but never invoked;
- `full-output-enforcement` was only a candidate, but never invoked;
- `gstack` was claimed for QA, but no browser / MCP / `toolName: Skill` evidence existed.

V3.4 adds **Skill Invocation Proof**, requiring every skill to be labeled with an invocation evidence level:

```text
Confirmed: TRAE UI explicitly shows toolName: Skill, or there is clear skill loading output
Inferred: the UI does not show a skill call, but the behavior clearly follows that skill’s rules
Planned only: the skill was only planned or listed as a candidate and must not be counted as actually used
Unavailable / failed: the skill was intended but unavailable or failed
```

Core V3.4 rule:

```text
Do not list any skill as actually used unless there is toolName: Skill evidence or clear skill loading output.

If a skill is claimed as used but has no invocation evidence, mark:
ROUTER_VIOLATION: SKILL_CLAIM_WITHOUT_INVOCATION
```

In one sentence:

> V3.3 answers “did staged collaboration happen?” V3.4 answers “which skills were truly invoked?”

---

## Future Improvement Direction

### Future Direction: QA Evidence Precision Rule

V3.4 can already distinguish skill invocation evidence levels, but QA evidence can still become more precise.

For example, during responsive checks, if the model only checks:

```js
overflowX
```

that is not rigorous enough.

A better horizontal overflow detection method is:

```js
document.documentElement.scrollWidth <= document.documentElement.clientWidth
```

A future rule can require the model to use actual layout width checks when claiming “no horizontal scroll / no horizontal overflow,” instead of only checking CSS overflow values.

---

## Installation

### 1. Current latest file location

The effective skill file should be located at:

```text
skill-router/SKILL.md
```

The historical backup of `V3.4` is recommended at:

```text
所有版本/SKILL-v3.4.md
```

### 2. Confirm the skill name

The frontmatter at the top of `SKILL.md` should look roughly like:

```yaml
---
name: skill-router
description: "中文/English Skill Router V3.4 ..."
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
Please use skill-router in V3.4 Skill Invocation Proof mode.
If the current environment supports TRAE native task planning / Todo List, sync router gates and actual work items into the native plan list.
Do not call skill-router only once at the beginning.
The final output must include a Stage Marker Ledger and distinguish Confirmed / Inferred / Planned only / Unavailable failed.
```

---

## Recommended Test Prompt

### Test V3.4 Native Todo List + Multi-Stage Collaboration + Invocation Evidence Audit

```text
Please create a completely independent small demo and do not modify my existing project code.

Task: build a small LOVE-themed showcase page.

Requirements:
1. Create a new standalone directory: skill-router-test-love-page/
2. Use HTML, CSS, and JavaScript. Do not introduce a complex framework.
3. The page should include: a gentle hero section, 3 LOVE-themed cards, a "moments of love" timeline, and a button that randomly displays a short love message.
4. Visual style: gentle, clean, atmospheric, not cheesy, not too flashy.
5. The page must be responsive on mobile and desktop.
6. Do not write placeholders or pseudocode. Files must be directly runnable.
7. After finishing, open the page and check button interaction, responsive layout, and console errors.

Prioritize skill-router V3.4 for staged dynamic routing, and in the final Stage Marker Ledger distinguish Confirmed / Inferred / Planned only / Unavailable failed.
```

Expected behavior:

- the TRAE native Todo List gets updated
- design, implementation, Review / Polish, and QA appear when relevant
- the final output includes a Stage Marker Ledger
- every skill is labeled as Confirmed / Inferred / Planned only / Unavailable failed
- Planned only skills are not reported as Actual skills used

---

## How to Judge Whether It Is Qualified

A qualified run should usually show:

```text
ROUTER_CHECKPOINT_0_TASK_MAP
ROUTER_CHECKPOINT_2_PLANNING_OR_DESIGN
ROUTER_CHECKPOINT_3_IMPLEMENTATION
ROUTER_CHECKPOINT_4_REVIEW_OR_POLISH
ROUTER_CHECKPOINT_5_QA_OR_VERIFICATION
```

And markers like:

```text
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-1
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-3
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-4
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-5
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-6
```

The final output should also include:

```text
Stage Marker Ledger
```

and list:

```text
Confirmed skill calls
Behavior-inferred skill guidance
Planned only
Unavailable / failed
Violation
```

---

## Common Signs of Non-Compliance

### Only calling it once at the beginning

```text
Router Pass: ROUTER_CHECKPOINT_0_TASK_MAP
Later router checkpoints:
- implementation
- QA
```

Then the model completes the whole task directly. This is not compliant.

### Skipping polish for UI tasks

```text
Implementation complete, proceeding directly to QA
```

If the task is UI, a webpage, a PDF cover page, cards, or a dashboard, this is usually not compliant.

### Writing plans only in chat, without syncing native Todo

If TRAE supports a native Todo List but the model only writes `ROUTER_GATE ... WORK ...` in chat and does not trigger a native Todo List update, that does not count as fully compliant.

### Pretend QA

```text
Static code review passed, so browser QA passed
```

Static review cannot pretend to be browser QA.

### Unclear skill attribution

If `gstack` was selected and Chrome DevTools MCP was actually used for browser testing, the final output should write:

```text
gstack（browser testing intent）+ Chrome DevTools MCP
```

instead of only `Chrome DevTools MCP`.

### Claiming skill usage without invocation evidence

Non-compliant:

```text
Actual skills used: diagnose, full-output-enforcement, gstack
```

when the execution log does not contain corresponding `toolName: Skill` or clear skill loading output.

In V3.4, this should become:

```text
Confirmed: skill-router, impeccable
Planned only: diagnose, full-output-enforcement
Unavailable / failed: gstack, if browser QA was attempted but failed
Violation: ROUTER_VIOLATION: SKILL_CLAIM_WITHOUT_INVOCATION
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

Replace the author's example skills with your own skills. Do not keep any skill that does not exist in your environment.

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

For each skill, describe at least: the skill name, suitable scenarios, unsuitable scenarios, which stage it belongs to, and how it differs from other skills.

Example:

| Engineering scenario | Skill name | Skill package | Usage rule |
|---|---|---|---|
| Frontend / page implementation | `my-frontend-builder` | `my-skills` | Used to build complete pages, components, and dashboards. |
| UI / polish review | `my-ui-reviewer` | `my-skills` | Used after implementation to check spacing, hierarchy, color, and responsiveness. |
| Browser QA | `my-browser-qa` | `my-skills` | Used to open pages, take screenshots, click, and verify interactions. |

### Step 4: Replace stage preference rules

Search in `SKILL.md` for content like:

```text
Preferred skills:
- design-taste-frontend
- ui-ux-pro-max
- impeccable
- high-end-visual-design
```

Replace them with your own design skills. Do the same for implementation, debugging, QA, documentation, and skill creation rules.

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
Never list Planned only skills as Actual skills used.
```

These are the foundation that makes the router auditable.

### Step 7: Customize verification markers

The default V3.4 markers are:

```text
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-1
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-3
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-4
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-5
SKILL_ROUTER_APPLIED: ROUTER_20260512_CN_GSTACK——V3.4-6
```

You can change them to your own project token, but the replacement must be consistent across marker tables, trace rules, Stage Marker Ledger examples, and marker references in violation rules.

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
ROUTER_VIOLATION: SKILL_CLAIM_WITHOUT_INVOCATION
```

`ROUTER_VIOLATION: SKILL_CLAIM_WITHOUT_INVOCATION` is used when the model claims a skill was used but there is no `toolName: Skill`, clear skill loading output, or other auditable invocation evidence.

If these markers appear, the run is not fully compliant.

---

## Design Principles

### 1. Staged, not one-shot

Do not let the model choose all skills at the beginning and then never route again.

### 2. Use only current-stage skills in the current stage

Skills from later stages may be listed as candidates, but cannot be claimed as already used.

### 3. Multi-skill collaboration, not random skill dumping

The goal is planning skill + implementation skill + polish skill + QA skill, not piling up skill names.

### 4. Evidence first

Compared with “I used it,” what matters more is markers, Todo List, file changes, browser screenshots, test results, Stage Marker Ledger, and invocation evidence levels.

### 5. Native Todo List matters

Many models follow the official task plan more closely than the chat body.

### 6. V3.4 is about auditable failure

V3.4 cannot guarantee that a weak model will always call every required skill, but it makes fake calls, missing calls, and planned-only claims much easier to expose.

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

It targets practical lazy-model behaviors: calling the router only once, choosing too few skills, pretending later-stage skills were already used, not syncing the native Todo List, jumping straight to QA after UI work, using static review to pretend browser QA happened, restarting planning from scratch, overclaiming in the final summary, and reporting Planned only skills as Actual skills used.

If you have run into similar problems, you can directly use V3.4 or adapt it into your own version based on your available skills.

---

## Maintenance Notes

It is recommended to keep historical version files:

```text
所有版本/SKILL-v1.md
所有版本/SKILL-v2.md
所有版本/SKILL-v3.md
所有版本/SKILL-v3.1.md
所有版本/SKILL-v3.2.md
所有版本/SKILL-v3.3.md
所有版本/SKILL-v3.4.md
```

That way, people can clearly see how this router evolved step by step.

Possible future directions:

- more precise QA evidence, such as `scrollWidth <= clientWidth`
- automatic export of skill invocation evidence logs
- automatically generate a dedicated routing table based on the user's available skills
- a separate `skill-router-auditor` that checks the ledger after the task is complete
