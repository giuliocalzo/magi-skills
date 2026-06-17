---
name: magi-review-full
description: Perform an exhaustive multi-pass code review using an Evangelion MAGI-style workflow with three independent model agents: ChatGPT, Opus, and Nemotron. Each agent reviews the code independently across three iterations, swapping review lens each iteration, producing nine reviews total, then reconciles them into a single agreed final document. Use when the user asks for a MAGI full review, a deep or exhaustive multi-model code review, or a rotating multi-pass consensus review.
---

# MAGI Full Review

Use this skill to review code with three independent MAGI agents across three iterations. Each agent reviews independently and swaps its review lens on every iteration, so the code is examined from every perspective by every model. This produces nine independent reviews, which are then reconciled into a single final document that all three MAGI agents agree on.

## MAGI Roles

- **MELCHIOR / ChatGPT**: product behavior, user impact, integration risks, tests.
- **BALTHASAR / Opus**: correctness, architecture, maintainability, hidden edge cases.
- **CASPER / Nemotron or Gemini**: performance, infrastructure, concurrency, deployment and GPU/NVIDIA-domain risks when relevant. Use Nemotron when available; otherwise use Gemini.

The MAGI agents are the fixed model identities above. The **review lens** (product, correctness, performance) is what rotates between iterations, so every model reviews from every lens across the three passes.

Use real distinct models when the current environment exposes them. Do not silently substitute unavailable models except for the explicit CASPER fallback from Nemotron to Gemini. If another requested MAGI model is unavailable, tell the user which role cannot use the requested model and ask whether to continue with the closest available model or run that role without an explicit model override.

## Model Selection

When launching subagents and explicit model selection is supported:

- ChatGPT: use the available GPT model requested by the user, otherwise the latest available GPT model.
- Opus: use the available Claude Opus model requested by the user, otherwise the latest available Opus model.
- Nemotron: use a Nemotron-backed model if it is available in the current tool/model list; otherwise use the latest available Gemini model.

If the environment lists only a fixed set of supported model slugs, use only those exact slugs. Never invent a slug.

## Review Lenses and Rotation

There are three review lenses:

- **P — Product**: behavior, user impact, integration risks, tests.
- **C — Correctness**: correctness, architecture, maintainability, hidden edge cases.
- **I — Infrastructure**: performance, infrastructure, concurrency, deployment, domain risks.

The lens assigned to each MAGI rotates every iteration:

| Iteration | MELCHIOR | BALTHASAR | CASPER |
| --- | --- | --- | --- |
| 1 | P | C | I |
| 2 | C | I | P |
| 3 | I | P | C |

By the end, every MAGI agent has reviewed the code through all three lenses.

## Output Artifacts

Every review is saved as a markdown file under `docs/magi/<magi-name>/`, where `<magi-name>` is the lowercase MAGI name: `melchior`, `balthasar`, or `casper`. The final agreed review is saved at the `docs/magi/` root.

`<slug>` is a 2-4 word, lowercase, kebab-case summary of the review target (for example, reviewing the auth refactor becomes `auth-refactor`). Derive the slug once at the start and reuse the exact same slug for every document.

```
docs/magi/
├── melchior/
│   ├── review-1-<slug>.md
│   ├── review-2-<slug>.md
│   └── review-3-<slug>.md
├── balthasar/
│   ├── review-1-<slug>.md
│   ├── review-2-<slug>.md
│   └── review-3-<slug>.md
├── casper/
│   ├── review-1-<slug>.md
│   ├── review-2-<slug>.md
│   └── review-3-<slug>.md
└── <slug>.md
```

Each agent writes only to its own `docs/magi/<magi-name>/` directory. The nine `review-<n>-<slug>.md` files are the source of truth handed to the final synthesis.

## Workflow

1. **Gather review context**
   - Identify the code under review: scope, files, branch or diff, and any acceptance criteria or constraints.
   - Derive a `<slug>`: a 2-4 word, lowercase, kebab-case summary of the review target. Reuse this exact slug for every document.
   - Always create a dedicated `git worktree` with the related branch checked out before starting the review, unless the request explicitly says otherwise (for example, asks to work in the current checkout). Run all MAGI reviews from that worktree.
   - Before creating the worktree, inspect `git status` in the current checkout and avoid moving, deleting, or reverting unrelated local changes.

2. **Iteration 1 — independent review**
   - Launch the three MAGI agents in parallel with the lens assignment for iteration 1 (MELCHIOR=P, BALTHASAR=C, CASPER=I).
   - Once the three MAGI agents are created, each one must be made aware of whether it is running inside a dedicated worktree or the current checkout, including the worktree path and branch name when applicable.
   - Give each agent the same review context and its assigned lens, and instruct agents not to read each other's output.
   - Each agent independently reviews the code and saves its review to `docs/magi/<magi-name>/review-1-<slug>.md`.

3. **Iteration 2 — swap lenses and review again**
   - Rotate the lens assignment (MELCHIOR=C, BALTHASAR=I, CASPER=P).
   - Launch the three MAGI agents in parallel again, keeping their contexts isolated, and instruct them not to read each other's output.
   - Each agent reviews the code through its new lens and saves its review to `docs/magi/<magi-name>/review-2-<slug>.md`.

4. **Iteration 3 — swap lenses and review again**
   - Rotate the lens assignment once more (MELCHIOR=I, BALTHASAR=P, CASPER=C).
   - Launch the three MAGI agents in parallel again, keeping their contexts isolated, and instruct them not to read each other's output.
   - Each agent reviews the code through its third lens and saves its review to `docs/magi/<magi-name>/review-3-<slug>.md`.

5. **Final synthesis and agreement**
   - Once all nine reviews are complete, give all nine `review-<n>-<slug>.md` documents to all three MAGI agents.
   - Ask each agent to consolidate the findings, identify duplicates, challenge weak or unsupported claims, and vote on each finding.
   - Produce one final document and require all three MAGI agents to agree on its content before it is finalized.
   - If an agent disagrees, synthesize the objection, revise the document, and re-confirm. If unanimous agreement cannot be reached after a focused retry, record the remaining disagreements explicitly and ask the user for a decision.
   - Group findings as:
     - **Consensus must-fix**: agreed by all three MAGI agents.
     - **Majority must-fix**: agreed by two of three; record the dissent.
     - **Should-fix**: valuable but not blocking.
     - **No-action**: rejected or duplicate findings with rationale.
   - Save the agreed final document to `docs/magi/<slug>.md`.

6. **Final response**
   - Respond with the contents of `docs/magi/<slug>.md`.
   - State that all nine reviews completed, confirm whether unanimous agreement was reached, and list any unresolved MAGI disagreements.

## Report Template

Use this compact format for each per-iteration review:

```markdown
## MAGI Review: [MAGI Name / Model] — Iteration [n], Lens [P/C/I]

### Findings
- [Severity] [Must-fix/Should-fix/No-action]: [title]
  - Evidence:
  - Impact:
  - Suggested fix:
  - Confidence:

### Notes
- Areas not covered by this lens:
```

Use this final synthesis format:

```markdown
## MAGI Full Review — Final

### Agreement
- Unanimous: [yes/no]
- Agreed by: [roles]

### Consensus Must-Fix
- [severity] [title] - agreed by: all

### Majority Must-Fix
- [severity] [title] - agreed by: [roles], dissent: [role -> view]

### Should-Fix
- [title] - rationale:

### No-Action
- [title] - rejection rationale:

### Open Disagreements
- [item] - positions: [role -> view]
```
