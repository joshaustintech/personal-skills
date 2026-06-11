---
name: tdd-test-plan
description: Break a feature, plan, or spec into small, independently testable TDD tasks and assign a model (opus/sonnet/haiku) to each based on its difficulty. Output is compact XML to save tokens. Use when the user says "tdd plan", "break this into tasks", "test plan", "make a task list", "plan the implementation", or invokes /tdd-test-plan. Each task is sized to one red-green-refactor cycle.
---

# TDD Test Plan

Turn a plan or spec into an ordered list of **small, testable** tasks, each scoped to one test-first cycle, with a model assigned per task. Emit the list as compact XML.

## Steps

1. **Get the plan.** Use the user's spec/plan/feature description. If too vague to decompose (no acceptance criteria, unclear scope), ask 1-2 sharp questions, then proceed. Don't pad.

2. **Decompose into TDD tasks.** Each task must be:
   - **Small** — one red-green-refactor cycle, ideally <1 test file's worth of behavior.
   - **Testable** — has a concrete, observable pass condition (a test you can name *now*).
   - **Ordered** — list dependencies; put foundational tasks first.
   - **Independent where possible** — split anything that bundles two behaviors.
   If a "task" can't state its test, it's not a task yet — break it down further or mark it `<analysis>` (see below).

3. **Assign a model per task** using the rules below.

4. **Emit XML** in the schema below. XML only — no prose tasklist duplicate. Add a one-line summary after.

## Model assignment

Default bias: **cheap unless the task is genuinely hard.** Opus is reserved for rough tasks. Honor any explicit user override (e.g. "use opus for everything").

| Model | Use for |
|---|---|
| `haiku` | Mechanical, low-ambiguity: boilerplate, simple CRUD, trivial pure functions, wiring, renames, obvious tests. |
| `sonnet` | Standard implementation: most feature work, multi-step logic with a clear shape, normal refactors, typical test suites. Default when unsure between haiku/sonnet. |
| `opus` | Only when **particularly rough**: deep design/analysis, tricky algorithms, concurrency/race reasoning, security-sensitive logic, ambiguous specs needing judgment, cross-cutting architecture decisions. |

Rules:
- **Default to sonnet.** Drop to haiku only when the task is obviously mechanical. Raise to opus only when the task is genuinely hard — not "important", *hard*.
- **Analysis/design tasks → opus.** Anything labeled deep thought, tradeoff analysis, or architecture goes opus regardless of size.
- **User override wins.** If the user names a model or policy, follow it and skip these heuristics.
- State the *reason* for opus/haiku in the task's `model` choice via the `<why>` field — keeps assignments honest.

## XML schema

Compact tags, attributes over child elements where short. Emit exactly this shape:

```xml
<plan goal="short goal">
  <task id="1" model="haiku|sonnet|opus" deps="none">
    <do>what to implement, imperative, one line</do>
    <test>the failing test to write first — name the assertion</test>
    <done>observable pass condition</done>
    <why>only if model=opus or haiku: 1-line justification</why>
  </task>
  <task id="2" model="sonnet" deps="1">
    <do>...</do>
    <test>...</test>
    <done>...</done>
  </task>
  <analysis id="3" model="opus" deps="2">
    <do>design decision needing judgment, no test</do>
    <done>decision recorded / approach chosen</done>
    <why>tricky tradeoff X vs Y</why>
  </analysis>
</plan>
```

Rules for the XML:
- `<task>` = has a test (TDD). `<analysis>` = thinking step with no test; use sparingly.
- `deps` lists prerequisite ids, comma-separated, or `none`.
- Omit `<why>` when `model="sonnet"` (the default needs no defense).
- No closing prose inside tags, no markdown inside XML, keep each field one line.

## Rules

- **Test-first.** Every `<task>` names the test before the implementation. No test → not a task.
- **Small over clever.** Prefer more small tasks to fewer big ones. A task spanning 3 files is two tasks.
- **Cheap by default.** Most tasks are haiku/sonnet. A plan that is mostly opus is wrong unless the user said so.
- **Order honestly.** Foundations and shared types first; UI/integration last.
- **No fabrication.** If the spec lacks detail for a task's test, say so in `<test>` rather than inventing acceptance criteria.

## After output

One-line summary: task count, model split (e.g. `5 tasks: 2 haiku, 2 sonnet, 1 opus`), and the critical-path dependency chain.
