---
name: review
description: >-
  Reviews code against plan/feature intent and the implementation skill via two
  parallel sub-agents — reporting problems only, without making changes unless
  asked. Use when the user asks to review a change, PR, branch, diff, or
  implementation; to check whether the code matches the plan or feature; or to
  verify conformance to implementation standards. Distinct from planning
  (produces a plan) and implementation (writes code) — this skill audits
  finished or in-progress work.
---

# Review

Two-axis review of code against the planning and implementation skills. **Both axes always run as parallel sub-agents** so they do not pollute each other's context; this parent agent only prepares inputs and aggregates findings.

- **Plan / feature fidelity** — does the code do what the plan or feature asks for?
- **Implementation conformance** — does the code follow the implementation skill?

Do not invent a separate standards encyclopedia; those skills are the bar. Do not perform either axis yourself in the parent turn — delegate.

## Default: report only

**Only list problems.** Do not edit files, apply fixes, or rewrite the code unless the user explicitly asks you to make the changes. Suggestions may name what is wrong and where; they must not become unsolicited patches.

If there are no problems on an axis, say so in one short line for that axis — do not pad with praise or a tour of what went well.

## Conflict priority

1. **Report-only** (unless the user asked for fixes)
2. **Plan / feature fidelity** over stylistic nits when both appear
3. Within implementation findings, use the implementation skill's own conflict priority

## Process

### 1. Establish what to review

Identify the change set from what the user named: PR, branch, commit range, uncommitted diff, paths, or "what we just built." If the scope is unclear, ask once. Prefer a concrete diff (`git diff`, PR files, or the conversation's written code) over re-reading the whole repo.

Capture once, before spawning agents:

- How to obtain the diff (exact command, PR ref, or file paths)
- Commit list if applicable (`git log …`)
- Workspace / repo path

Confirm the scope resolves and the diff is non-empty. Fail here — not inside the sub-agents — if the ref is bad or there is nothing to review.

### 2. Establish the intent source (Plan / feature axis)

Resolve what "correct" means, in this order:

1. An explicit plan from this conversation or a linked plan doc
2. A feature/spec/issue the user points at
3. A clear feature description in the user request
4. If nothing exists, ask once. If they confirm there is no plan/spec, skip the Plan / feature sub-agent and note that — still run the Implementation sub-agent

Load the intent text (or a precise path) so it can be pasted into the Plan / feature sub-agent prompt. Infer only what the plan or feature would *seem to suggest* — do not invent requirements the source does not support.

### 3. Load skill paths for the agents

Resolve absolute paths (or install locations) to:

- The `implementation` skill (`SKILL.md`) — required for the Implementation agent
- Optionally the `planning` skill if it clarifies how plans should be read — not a second checklist for the Plan agent

The Implementation agent must receive the **full implementation skill body** (or a reliable path plus instruction to read it). Sub-agents do not inherit this chat's context.

### 4. Spawn both axes in parallel

Launch **two** sub-agents in the **same turn** (parallel Task calls). Use `generalPurpose` unless a narrower type clearly fits. Each agent is report-only: **no file edits**.

**Plan / feature sub-agent** — include:

- Repo path, diff command / paths, and commit list
- Full intent source (plan, spec, or feature description), quoted or attached by path
- Brief: Report **problems only**: (a) requirements the intent asked for that are missing, partial, or wrong; (b) material behaviour in the diff that the intent did not ask for (scope creep). Quote or paraphrase the intent line for each finding. Do not invent requirements. Do not comment on code style or implementation-skill rules. Under ~400 words. If nothing is wrong: `No problems found.`

**Implementation sub-agent** — include:

- Repo path, diff command / paths, and commit list
- The full `implementation` skill text (or path + "read this first and treat it as the checklist")
- Brief: Report **problems only**: concrete violations of the implementation skill. Cite the rule by name/number. Apply language-scoped rules only where they apply. Respect architectural conformity to the surrounding codebase. Skip findings tooling already enforces unless noted. Do not judge plan/feature completeness. Under ~400 words. If nothing is wrong: `No problems found.`

If the intent source is missing, spawn **only** the Implementation sub-agent and state that Plan / feature was skipped.

Do not resume one axis into the other. Do not let either agent apply fixes.

### 5. Aggregate

Present the two reports under these headings, verbatim or lightly cleaned — **do not merge or rerank** findings across axes:

```markdown
## Plan / feature
- [problem] …
(or: No problems found. / Skipped — no plan or feature source.)

## Implementation
- [problem] …
(or: No problems found.)
```

End with one line: counts per axis (e.g. `Plan/feature: 2 · Implementation: 1`). A change can pass one axis and fail the other — that separation is intentional.

### 6. Fixes only on request

If the user then asks to fix the issues (or a subset), implement those fixes in the parent (or a single fix agent) using the implementation skill. Do not expand into unrelated cleanup.
