---
name: magi-build-feature
description: Implement a new feature using an Evangelion MAGI-style workflow with three independent model agents: ChatGPT, Opus, and Nemotron. Use when the user asks for a MAGI feature build, multi-model feature implementation, consensus-based design-and-build workflow, or wants a new feature implemented with cross-model review.
---

# MAGI Feature Build

Use this skill to design and implement a new feature with three independent MAGI agents: produce competing designs, synthesize them into one plan, implement it, and run a critical consensus loop before completion.

## MAGI Roles

- **MELCHIOR / ChatGPT**: product behavior, user experience, requirements coverage, edge cases, tests.
- **BALTHASAR / Opus**: correctness, architecture, maintainability, API and data-model design.
- **CASPER / Nemotron or Gemini**: performance, infrastructure, concurrency, deployment and GPU/NVIDIA-domain risks when relevant. Use Nemotron when available; otherwise use Gemini.

Use real distinct models when the current environment exposes them. Do not silently substitute unavailable models except for the explicit CASPER fallback from Nemotron to Gemini. If another requested MAGI model is unavailable, tell the user which role cannot use the requested model and ask whether to continue with the closest available model or run that role without an explicit model override.

## Model Selection

When launching subagents and explicit model selection is supported:

- ChatGPT: use the available GPT model requested by the user, otherwise the latest available GPT model.
- Opus: use the available Claude Opus model requested by the user, otherwise the latest available Opus model.
- Nemotron: use a Nemotron-backed model if it is available in the current tool/model list; otherwise use the latest available Gemini model.

If the environment lists only a fixed set of supported model slugs, use only those exact slugs. Never invent a slug.

## Output Artifacts

Every MAGI report is saved as a markdown file under `docs/magi/<magi-name>/`, where `<magi-name>` is the lowercase role name: `melchior`, `balthasar`, or `casper`. The synthesized output is saved at the `docs/magi/` root.

```
docs/magi/
├── melchior/
│   ├── design.md
│   └── review.md
├── balthasar/
│   ├── design.md
│   └── review.md
├── casper/
│   ├── design.md
│   └── review.md
└── final-design.md
```

Each agent writes only to its own `docs/magi/<magi-name>/` directory. Use these files as the source of truth handed between phases.

## Workflow

1. **Gather feature context**
   - Clarify the feature's goal, scope, users, acceptance criteria, and any constraints (performance, deadlines, compatibility).
   - Identify the relevant codebase areas, existing patterns, frameworks, build/test commands, and integration points.
   - Create or reuse a dedicated `git worktree` or feature branch for the work. Run MAGI design, implementation, and validation from there unless the user explicitly asks to work in the current checkout.
   - Before creating the worktree, inspect `git status` in the current checkout and avoid moving, deleting, or reverting unrelated local changes.
   - Do not implement until the design and review phases have both completed.

2. **Independent design phase**
   - Launch the three MAGI agents in parallel.
   - Give each agent the same feature context and ask for an independent implementation design.
   - Instruct agents not to read each other's output.
   - Each design must include: proposed approach, files/modules to create or change, data model and interfaces, test strategy, risks and trade-offs, and what evidence would change the agent's mind.
   - Each agent saves its design to `docs/magi/<magi-name>/design.md`.

3. **Independent design review phase**
   - Launch the three MAGI agents in parallel again, or resume the same agents only if their contexts remain isolated.
   - Each agent independently reviews the feature context and produces concerns about correctness, scope, UX, performance, and maintainability before any code is written.
   - Concerns must be concrete, cite files/symbols/patterns, include severity, explain user impact, and state whether they must be resolved before implementation.
   - Each agent saves its review to `docs/magi/<magi-name>/review.md`.

4. **MAGI synthesis**
   - Read the per-agent reports from `docs/magi/<magi-name>/design.md` and `docs/magi/<magi-name>/review.md`.
   - Give all three designs and all three review reports to all MAGI agents.
   - Ask each agent to critique the other reports, identify duplicates, challenge weak claims, and vote on each design decision.
   - Create one final design from the combined reports.
   - The final design must group decisions as:
     - **Consensus**: at least two MAGI agents agree on the approach.
     - **Disputed**: at least one MAGI agent disagrees with another on a decision.
     - **Should-do**: valuable but not required for the first implementation.
     - **No-action**: rejected or duplicate ideas with rationale.
   - Save the synthesized result to `docs/magi/final-design.md`.

5. **Sort unresolved decisions**
   - If there is no consensus for any design decision, sort each unresolved item by severity, confidence, blast radius, and reversibility.
   - Present the sorted list and ask the user before implementing disputed or ambiguous decisions.
   - Continue without asking only for clearly safe, consensus-backed decisions.

6. **Implementation**
   - Assign one MAGI role as the implementer. Prefer BALTHASAR / Opus for correctness-heavy features, MELCHIOR / ChatGPT for product/UX/test-heavy features, and CASPER / Nemotron-or-Gemini for performance/infrastructure-heavy features.
   - Implement only the **Consensus** design unless the user approved disputed items.
   - Build incrementally and keep edits scoped to the feature. Do not refactor unrelated code.
   - Add tests that cover the acceptance criteria and the risks raised during review.
   - Preserve unrelated local changes and never revert user work.

7. **Critical agreement loop**
   - All three MAGI agents review the implementation critically, including the implementing agent's self-review.
   - They must verify that the acceptance criteria are met, each accepted design decision was implemented, no regression was introduced, and tests cover the risks.
   - Proceed to validation only when at least two of the three MAGI agents approve the implementation.
   - If fewer than two MAGI agents approve, synthesize the objections, fix them, and repeat the critical review loop.
   - If quorum cannot be reached after a focused retry, sort the remaining disagreements by severity, confidence, blast radius, and reversibility, then ask the user for a decision.

8. **Validation and final response**
   - Run the most relevant tests, linters, type checks, or builds available locally.
   - Final response must include: what was built, how it maps to the acceptance criteria, what validation ran, remaining risks, and any unresolved MAGI disagreements.

## Report Template

Use this compact report format for each MAGI agent:

```markdown
## MAGI Report: [Role / Model]

### Design
- Approach:
- Files/modules to create or change:
- Data model and interfaces:
- Test strategy:
- Risks and trade-offs:
- Evidence that would change this design:

### Review Concerns
- [Severity] [Blocker/Should-fix/No-action]: [title]
  - Evidence:
  - Impact:
  - Suggested resolution:
  - Confidence:
```

Use this final synthesis format:

```markdown
## MAGI Final Design

### Consensus
- [decision] - agreed by: [roles]

### Disputed
- [decision] - positions: [role -> view]

### Should-Do
- [item] - rationale:

### No-Action
- [item] - rejection rationale:

### Implementation Plan
- Implementer:
- Build order:
- Acceptance criteria:
- Validation:
```
