---
name: bug-triage
description: >-
  Triages a finding list from any source end to end: dedupes and ranks it,
  decides which findings are real and how to fix them using bug-fix judgment,
  implements those fixes in isolated worktrees, and resolves linked PR comments
  when relevant. Asks the user only when the intended product behavior is
  ambiguous. Use when explicitly asked to triage bugs, process PR review
  comments, or handle a prefabricated error list. Does not discover issues — use
  bug-session for multi-agent review discovery. Requires explicit invocation.
disable-model-invocation: true
---

# Bug Triage

Explicit entry point. Takes a **finding list from any source** and runs it to completion: rank, decide, implement, and reply where relevant.

This skill does **not** surface new issues. For parallel Bugbot-class discovery, invoke `bug-session` (which hands its list to this skill).

Disclose briefly once that bug-triage is in use. The agent running this skill is the **orchestrator**. It makes every triage decision itself. Spawned agents never approve their own work, decide what to skip, or resolve comments.

## Workflow

Run these steps in order. Do not stop for approval between them.

### 1. Obtain the finding list

Accept any of:

- Output from `bug-session`
- A linked PR’s review / issue comments (fetch and normalize)
- A pasted list, CI log excerpt, or other prefabricated findings

If the source is unclear, ask once. Normalize into a common shape per finding: id, summary, location (file/line if known), source ref (e.g. comment URL), raw text.

### 2. Dedupe, rank, and score confidence

- Merge duplicates that describe the same underlying issue.
- **Confidence**: when multiple independent sources (e.g. multiple session reviewers) report the same issue, mark higher confidence (agreement count). When there is only one source (one PR comment, one pasted item), label confidence **`single-source`** — never invent consensus.
- Order findings by confidence, then severity and location. Use this order for the rest of the run.

### 3. Propose

For each finding, spawn a subagent. Tell it to follow `bug-fix` for judgment only: no edits, no commits. Give it the finding payload (and linked comment text if any). It returns:

```markdown
## Verdict
fix

## Rationale
Property `foo` was removed to silence the type error, but callers still send it for discounts.

## Proposal
In `src/order.ts`, map `foo` to `discountCode` before the API call. Do not drop the field.
```

Use `not a concern` and omit `## Proposal` when the issue is absent or immaterial. The proposal must be concrete enough to apply without rediscovering the bug. Trade-offs and “this needs a refactor” come back in the result — the agent does not start that work.

If a proposal depends on how the product is meant to behave and the code and report do not settle it, the agent returns that as an open question. It does not ask the user itself.

### 4. Decide

The orchestrator reviews every proposal and decides:

- **Implement** a `fix` proposal that is concrete, in scope, and not refactor-sized.
- **Skip** `not a concern`, weak proposals, and refactor-sized work. Record each skip and the reason.
- **Ask the user** only for an open product-behavior question (for example: "is this meant to work this way or another way?"). Ask those questions once, together, and continue with the rest of the list while waiting.

Do not ask which findings to pursue, whether to continue, or whether to approve a proposal.

### 5. Implement

For each finding marked for implementation:

1. Create a **separate git worktree**. If worktrees are unavailable, **stop and say so** — do not edit the main working tree instead.
2. Spawn a subagent in that worktree. Tell it to follow `bug-fix` and `implementation`, apply **only** the decided proposal, commit in that worktree, and not resolve or reply on PR comments. If the proposal cannot be applied as written, it stops and reports that — it does not invent a different fix.

Run implement agents in parallel when safe. After they finish, **integrate** (merge or cherry-pick) onto the session/target branch. On integrate conflict: **stop and ask** — do not force-resolve.

### 6. PR resolve / reply when relevant

When a finding was tied to a PR review comment (or similar) **and** the implement step succeeded on a change that will attach to that PR, the **orchestrator** resolves and replies on that comment.

Do **not** reply for: “not a concern,” skipped findings, or failed implements.

### 7. Report

Summarize what was fixed, what was skipped and why, what is pending a product question, and any integrate or worktree blockers.

## Composability

- **Solo one-off fix** → invoke `bug-fix` directly (not this skill).
- **Discovery then triage** → `bug-session` → this skill.
- **Already have a list** → this skill only.
