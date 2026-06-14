---
name: magi-query
description: Answer a question or research query using an Evangelion MAGI-style workflow with three independent model agents: ChatGPT, Opus, and Nemotron. Use when the user asks to query the MAGI, wants a multi-model answer, a consensus-based research answer, or asks a question that should be answered by independent agents who then reconcile into a single authoritative answer.
---

# MAGI Query

Use this skill to answer a query with three independent MAGI agents: each agent researches and answers on its own and writes its own document, then all agents review each other's documents, discuss disagreements, and reconcile them into a single most-accurate answer that is returned to the user.

## MAGI Roles

- **MELCHIOR / ChatGPT**: breadth, practical framing, user intent, examples, and clarity.
- **BALTHASAR / Opus**: correctness, rigor, logical consistency, and hidden assumptions or edge cases.
- **CASPER / Nemotron or Gemini**: technical depth, performance, infrastructure, and GPU/NVIDIA-domain accuracy when relevant. Use Nemotron when available; otherwise use Gemini.

Use real distinct models when the current environment exposes them. Do not silently substitute unavailable models except for the explicit CASPER fallback from Nemotron to Gemini. If another requested MAGI model is unavailable, tell the user which role cannot use the requested model and ask whether to continue with the closest available model or run that role without an explicit model override.

## Model Selection

When launching subagents and explicit model selection is supported:

- ChatGPT: use the available GPT model requested by the user, otherwise the latest available GPT model.
- Opus: use the available Claude Opus model requested by the user, otherwise the latest available Opus model.
- Nemotron: use a Nemotron-backed model if it is available in the current tool/model list; otherwise use the latest available Gemini model.

If the environment lists only a fixed set of supported model slugs, use only those exact slugs. Never invent a slug.

## Output Artifacts

Every MAGI answer is saved as a markdown file under `docs/magi/<magi-name>/`, where `<magi-name>` is the lowercase role name: `melchior`, `balthasar`, or `casper`. The reconciled answer is saved at the `docs/magi/` root.

The document filename is a `<slug>.md`, where `<slug>` is a 2-4 word, lowercase, kebab-case summary of the original question (for example, the query "How should we tune GPU memory for inference?" becomes `gpu-memory-tuning.md`). Derive the slug once at the start and reuse the exact same slug for all three agent documents and the final document.

```
docs/magi/
├── melchior/
│   └── <slug>.md
├── balthasar/
│   └── <slug>.md
├── casper/
│   └── <slug>.md
└── <slug>.md
```

Each agent writes only to its own `docs/magi/<magi-name>/` directory. Use these files as the source of truth handed between phases.

## Workflow

1. **Gather query context**
   - Restate the query and clarify scope, audience, depth, and any constraints (format, length, domain, deadline).
   - Derive a `<slug>`: a 2-4 word, lowercase, kebab-case summary of the original question. Reuse this exact slug as the filename for every agent document and the final document.
   - Identify any relevant local context: files, codebase areas, prior documents, or data the answer should be grounded in.
   - Ask the user only if the query is ambiguous enough that the agents would otherwise diverge on intent rather than substance.

2. **Independent answer phase**
   - Launch the three MAGI agents in parallel.
   - Give each agent the same query and context, and ask each to produce its own complete answer.
   - Instruct agents not to read each other's output.
   - Each answer must include: the direct response, supporting reasoning or evidence, sources or assumptions, known limitations, and a confidence level.
   - Each agent saves its answer to `docs/magi/<magi-name>/<slug>.md`.

3. **Cross-review and discussion phase**
   - Once all three answers are complete, give all three documents to all MAGI agents.
   - Ask each agent to review the other two documents, identify agreements, contradictions, gaps, and factual errors, and challenge weak or unsupported claims.
   - Each agent updates its own `docs/magi/<magi-name>/<slug>.md` with corrections and a short critique of the other answers.

4. **MAGI synthesis**
   - Read the per-agent answers from `docs/magi/<magi-name>/<slug>.md`.
   - Reconcile the three answers into one authoritative document, preferring claims that are corroborated by multiple agents or backed by the strongest evidence.
   - Group the content as:
     - **Consensus**: at least two MAGI agents agree on the claim.
     - **Disputed**: at least one MAGI agent disagrees with another; record both positions.
     - **Single-source**: a claim only one agent raised that survived review.
     - **Discarded**: rejected or unsupported claims with rationale.
   - Save the reconciled result to `docs/magi/<slug>.md`.

5. **Resolve disagreements**
   - For each disputed item, prefer the position with stronger evidence and higher cross-agent confidence.
   - If a material disagreement cannot be resolved from evidence, keep both positions in the final answer and flag the open question for the user rather than guessing.

6. **Final response**
   - Respond to the original query with the contents of `docs/magi/<slug>.md`.
   - Lead with the direct, reconciled answer. Then include key supporting reasoning, any unresolved MAGI disagreements, and remaining limitations or open questions.

## Report Template

Use this compact format for each MAGI agent answer:

```markdown
## MAGI Answer: [Role / Model]

### Answer
- Direct response:

### Reasoning and Evidence
- Key points:
- Sources / assumptions:

### Limitations
- Known gaps or caveats:

### Confidence
- Level and why:

### Critique of Other MAGI (cross-review phase)
- Agreements:
- Disagreements / corrections:
```

Use this final synthesis format:

```markdown
## MAGI Final Answer

### Answer
- Reconciled response:

### Consensus
- [claim] - agreed by: [roles]

### Disputed
- [claim] - positions: [role -> view]

### Single-Source
- [claim] - raised by: [role], survived review because:

### Discarded
- [claim] - rejection rationale:

### Open Questions
- [unresolved item for the user]
```
