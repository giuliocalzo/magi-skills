---
name: magi-pr-review
description: Review and fix pull requests using an Evangelion MAGI-style workflow with three independent model agents: ChatGPT, Opus, and Nemotron. Use when the user asks for a MAGI PR review, multi-model PR review, consensus-based code review, or review-and-fix workflow for a pull request.
---

# MAGI PR Review

Use this skill to review a pull request with three independent MAGI agents, synthesize their findings, implement fixes, and run a critical consensus loop before completion.

## MAGI Roles

- **MELCHIOR / ChatGPT**: product behavior, user impact, integration risks, tests.
- **BALTHASAR / Opus**: correctness, architecture, maintainability, hidden edge cases.
- **CASPER / Nemotron or Gemini**: performance, infrastructure, concurrency, deployment and GPU/NVIDIA-domain risks when relevant. Use Nemotron when available; otherwise use Gemini.

Use real distinct models when the current environment exposes them. Do not silently substitute unavailable models except for the explicit CASPER fallback from Nemotron to Gemini. If another requested MAGI model is unavailable, tell the user which role cannot use the requested model and ask whether to continue with the closest available model or run that role without an explicit model override.

## Model Selection

When launching subagents and explicit model selection is supported:

- ChatGPT: use the available GPT model requested by the user, otherwise the latest available GPT model.
- Opus: use the available Claude Opus model requested by the user, otherwise the latest available Opus model.
- Nemotron: use a Nemotron-backed model if it is available in the current tool/model list; otherwise use the latest available Gemini model.

If the environment lists only a fixed set of supported model slugs, use only those exact slugs. Never invent a slug.

## Workflow

1. **Gather PR context**
   - Identify the PR, base branch, current branch, local changes, CI status, review comments, and the full diff.
   - Create or reuse a dedicated `git worktree` for the PR review and implementation. Run MAGI planning, review, fixes, and validation from that worktree unless the user explicitly asks to work in the current checkout.
   - Before creating the worktree, inspect `git status` in the current checkout and avoid moving, deleting, or reverting unrelated local changes.
   - If using GitHub, use `gh` for PR data and run `gh` commands outside the sandbox. If using GitLab, use `glab` for merge request data and run `glab` commands outside the sandbox. For other systems, only check out the PR/branch in the worktree and review the local diff unless the user explicitly provides another supported tool.
   - Do not implement until the planning and review phases have both completed.

2. **Independent plan phase**
   - Launch the three MAGI agents in parallel.
   - Give each agent the same PR context and ask for an implementation-neutral plan.
   - Instruct agents not to read each other's output.
   - Each plan must include: suspected risk areas, files to inspect, test strategy, and what evidence would change the agent's mind.

3. **Independent review phase**
   - Launch the three MAGI agents in parallel again, or resume the same agents only if their contexts remain isolated.
   - Each agent independently reviews the PR and produces findings.
   - Findings must be concrete, cite files/symbols, include severity, explain user impact, and state whether a fix is required before merge.

4. **MAGI synthesis**
   - Give all three plan reports and all three review reports to all MAGI agents.
   - Ask each agent to critique the other reports, identify duplicates, challenge weak claims, and vote on each finding.
   - Create one final summary from the combined reports.
   - The final summary must group items as:
     - **Consensus must-fix**: at least two MAGI agents agree it must be fixed.
     - **Disputed must-fix**: at least one MAGI agent says must-fix and another disagrees.
     - **Should-fix**: valuable but not merge-blocking.
     - **No-action**: rejected or duplicate findings with rationale.

5. **Sort unresolved items**
   - If there is no consensus for any summary item, sort each unresolved item by severity, confidence, blast radius, and reversibility.
   - Present the sorted list and ask the user before implementing disputed or ambiguous fixes.
   - Continue without asking only for clearly safe, consensus-backed fixes.

6. **Implementation**
   - Assign one MAGI role as the implementer. Prefer BALTHASAR / Opus for correctness-heavy fixes, MELCHIOR / ChatGPT for product/test-heavy fixes, and CASPER / Nemotron-or-Gemini for performance/infrastructure-heavy fixes.
   - Implement only items from **Consensus must-fix** unless the user approved disputed items.
   - Keep edits narrowly scoped. Do not refactor unrelated code.
   - Preserve unrelated local changes and never revert user work.

7. **Critical agreement loop**
   - All three MAGI agents review the implementation critically, including the implementing agent's self-review.
   - They must verify that each accepted summary item was fixed, no new regression was introduced, and tests cover the risk.
   - Proceed to validation only when at least two of the three MAGI agents approve the implementation.
   - If fewer than two MAGI agents approve, synthesize the objections, fix them, and repeat the critical review loop.
   - If quorum cannot be reached after a focused retry, sort the remaining disagreements by severity, confidence, blast radius, and reversibility, then ask the user for a decision.

8. **Validation and final response**
   - Run the most relevant tests, linters, or CI checks available locally.
   - Final response must include: what was fixed, what validation ran, remaining risks, and any unresolved MAGI disagreements.

## Report Template

Use this compact report format for each MAGI agent:

```markdown
## MAGI Report: [Role / Model]

### Plan
- Risk areas:
- Files to inspect:
- Test strategy:
- Evidence that would change this plan:

### Findings
- [Severity] [Must-fix/Should-fix/No-action]: [title]
  - Evidence:
  - Impact:
  - Suggested fix:
  - Confidence:
```

Use this final synthesis format:

```markdown
## MAGI Final Summary

### Consensus Must-Fix
- [severity] [title] - agreed by: [roles]

### Disputed Must-Fix
- [severity] [title] - positions: [role -> view]

### Should-Fix
- [title] - rationale:

### No-Action
- [title] - rejection rationale:

### Implementation Plan
- Implementer:
- Fix order:
- Validation:
```
