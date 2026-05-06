---
name: clean-docs
description: >
  Reviews and cleans up agent-instruction Markdown files (AGENTS.md, CLAUDE.md, and similar)
  using empirically-derived best practices. Applies length limits, progressive disclosure,
  procedural workflows, decision tables, paired do/don't rules, and reference-file modularity.
---

# Clean Markdown Agent-Instruction Docs

You are reviewing one or more Markdown files that instruct AI coding agents (e.g. `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, module-level `README.md` consumed by agents). Your job is to **rewrite them in place** so they measurably improve agent performance, following the empirical findings below.

If the user did not specify target files, ask which file(s) to clean. If they pointed at a directory, find the agent-instruction files within it (`AGENTS.md`, `CLAUDE.md`, root `README.md` if it doubles as agent guidance, plus any module-level equivalents).

---

## Operating Principles (non-negotiable)

A high-quality agent-instruction file performs like upgrading the underlying model from Haiku to Opus. A bloated one performs **worse than no file at all**. Optimize ruthlessly.

1. **Length cap: 100–150 lines for the main file.** Performance gains reverse past 150 lines. If the file exceeds this, the default action is to cut, not to reorganize. Push detail into reference files the agent loads on demand.
2. **Progressive disclosure beats comprehensiveness.** The main file covers common workflows at a high level; deeper detail lives in linked reference files (≤10–15 references total).
3. **Discovery is unreliable outside AGENTS.md.** AGENTS.md is auto-loaded (100% discovery). Module README.md hits ~80% when the agent works in that directory. Nested READMEs hit ~40%. Orphan `_docs/` folders hit <10%. **Critical content must live in or be linked from the AGENTS-equivalent file**, not stranded in deep folders.
4. **Module-level files outperform repo-root cross-cutting files.** When the codebase is large, prefer per-module instruction files over one giant root file.
5. **The surrounding documentation environment matters.** If 500K+ tokens of unrelated specs surround the file, the focused file's signal is diluted. Note this risk; recommend pruning if observed.

---

## The Seven Patterns That Work

Apply these patterns aggressively. Each is paired with the metric it most improves so you can prioritize based on the user's stated goal (if any).

### 1. Procedural Workflows → improves *feature wiring* and *completeness*
Convert prose descriptions of multi-step tasks into **numbered step lists**. A six-step deployment workflow cut missing wiring files from 40% to 10%, raised correctness by 25%, and completeness by 20%.

- Each step is one imperative sentence.
- Branching cases (>3 branches) move to a reference file; the main file shows only the happy path with a pointer.

### 2. Decision Tables → improves *adherence to conventions*
Whenever the docs name two or more competing tools, libraries, or patterns, render the choice as a table. Decision tables raised best-practices scores by 25%.

| Use this | When |
|---|---|
| `React Query` | Server state, cache, network fetch |
| `Zustand` | Pure client state, no server sync |

Add a table whenever you see prose like "we usually use X but sometimes Y" — that is a decision table waiting to happen.

### 3. Production Code Examples → improves *code reuse*
Embed 3–10 line snippets **lifted from the actual codebase**, not invented. Improved code reuse by 20%. Cap at the most relevant 3–5 snippets — too many cause pattern-matching on the wrong elements. If the file shows fabricated or generic snippets, replace them with real ones (search the repo) or remove them.

### 4. Pair every "Don't" with a "Do" → improves *gotcha handling*
Warning-only docs underperform. 15+ sequential "don'ts" without "dos" caused 2× longer PRs and 20% less completeness.

- Bad: "Don't instantiate HTTP clients directly."
- Good: "Don't instantiate HTTP clients directly. Use the shared `apiClient` from `lib/http` with the retry middleware."

Scan for every "don't / never / avoid / do not" and ensure each has a same-paragraph alternative. If no alternative exists, either find one in the codebase or delete the rule.

### 5. Domain-Specific, Enforceable Rules → improves *correctness*
Rules like "Use `Decimal` instead of `float` for all financial calculations" beat generic guidance. **Hard cap: ~30 stacked rules across the file.** Beyond 30, effectiveness diminishes and the file starts to harm performance. If you find 30+, group them by subsystem and move all but the most cross-cutting rules into module-level reference files.

### 6. Concise Architecture Overview → prevents *over-exploration*
Extensive service-topology prose causes agents to read 12+ docs on simple tasks, loading 80K+ irrelevant tokens.

- Keep architecture sections to bullet points naming **boundaries and entry points**, not narratives.
- Describe **what** components are, not **why** they were built that way.
- If an architecture section runs >20 lines, cut it to ≤10 and move the rest to a reference file.

### 7. Modular References → enables *progressive disclosure*
The main file links to reference files. **10–15 references is the maximum.** Each reference is a focused doc (one workflow, one subsystem). Audit existing links: dead ones get removed; orphan docs that aren't linked from any AGENTS-equivalent file get either linked or deleted.

---

## Anti-Patterns to Strip Out

When you encounter these, remove or rewrite without asking:

- **Marketing prose**, project history, or contributor-acknowledgement sections — agents do not need them.
- **Human-skimming flourishes**: emoji-heavy headers, table-of-contents for short files, tagline boxes.
- **Hedged guidance** ("you may want to consider possibly using…") — rewrite as a directive or delete.
- **Stale patterns that block new architecture**: e.g., a file documenting REST+polling that would steer the agent away from a now-preferred WebSocket solution. Flag these and ask the user before rewriting, since the right fix may be a separate spec rather than an AGENTS.md edit.
- **Long "don't" lists** with no paired "do" (per pattern 4).
- **Architecture deep-dives** explaining historical rationale (per pattern 6).
- **Duplicated guidance** — if the same rule appears in three sections, keep one canonical version and delete the rest.
- **Sections that exist only because of a README convention** (Contributing, License, Badges) when the file is purely an agent-instruction file. Move them to a separate `README.md` if needed.

---

## Execution Workflow

Run these steps in order for each target file:

1. **Read the file fully and count lines.** Note the line count up front.
2. **Inventory sections.** List every `##`/`###` heading and its line range.
3. **Diagnose against the seven patterns.** For each pattern, mark Present / Missing / Violated. For each anti-pattern, mark instances with line numbers.
4. **Plan the rewrite.** Decide: which sections to delete, which to compress, which prose to convert into tables or numbered steps, what to extract into reference files. If extracting to reference files, name them and state where they will live.
5. **Verify code examples.** For every code snippet, grep the codebase to confirm it reflects real code. Replace fabricated snippets with real ones (cite the source path) or delete them.
6. **Rewrite the file in place** using `Edit` or `Write`. Target ≤150 lines for the main file. Preserve any project-specific facts that are still accurate.
7. **Create or update reference files** for extracted detail. Link them from the main file. Do not exceed 15 references.
8. **Re-count lines and re-check** the seven patterns. If still over 150 lines after compression, cut further — do not stop at 151.
9. **Report** to the user with: original line count → new line count, sections removed, sections added, patterns now present, reference files created, and any stale-pattern concerns deferred for their decision.

---

## Renaming and Migration

If the user asks to convert a `README.md` into an `AGENTS.md`:

- Aggressively trim human-skimming sections (badges, install-from-source narratives, contributor lists).
- Keep code examples, decision criteria, and procedural steps.
- The new `AGENTS.md` follows the 100–150 line cap from day one.
- If the original README is still useful for humans, leave a slimmed `README.md` in place alongside the new `AGENTS.md`.

---

## Things to Confirm With the User Before Acting

- Removing a section that contains project-specific facts you cannot verify from the codebase.
- Splitting a single file into multiple module-level files (changes the repo layout).
- Deleting `_docs/` folders or other low-discovery doc trees, even when content seems redundant.
- Rewriting guidance that documents a now-stale architectural pattern — the right fix may be a new spec, not an AGENTS.md edit.

For all other cleanups (length, tables, paired do/don'ts, removing hedges, compressing architecture prose, fixing fabricated snippets), proceed without asking.

---

## Output Format for the Final Report

```
clean-docs report
  file: <path>
  lines: <before> → <after>
  patterns added: <list>
  anti-patterns removed: <list>
  reference files created: <list, or "none">
  deferred decisions: <list, or "none">
```

Keep the report under 15 lines. The diff speaks for itself.