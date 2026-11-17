---
name: bug-triage
description: >-
  Triages a finding list from any source, ranks by confidence, runs agents
  under bug-fix guidance to propose fixes, then after orchestrator approval
  implements them in worktrees and resolves linked PR comments when relevant. Use when explicitly
  asked to triage bugs, process PR review comments, or handle a prefabricated
  error list. Does not discover issues — use bug-session for multi-agent review
  discovery. Requires explicit invocation.
disable-model-invocation: true
---

# Bug Triage

Explicit entry point. Takes a **finding list from any source**, ranks it, asks agents that follow `bug-fix` to propose then implement, and handles PR resolve/reply when relevant. The constraints for each spawn (no edits, or apply this proposal in a worktree) come from this skill, not from `bug-fix`.

This skill does **not** surface new issues. For parallel Bugbot-class discovery, invoke `bug-session` (which hands its list to this skill).

Disclose briefly once that bug-triage is in use. The agent running this skill is the **orchestrator**: only it approves proposals, integrates worktrees, and replies on PRs. Spawned agents never approve their own work or resolve comments.

## Workflow

Follow these steps in order. Stop and wait where noted.

### 1. Obtain the finding list

Accept any of:

- Output from `bug-session`
- A linked PR’s review / issue comments (fetch and normalize)
- A pasted list, CI log excerpt, or other prefabricated findings

If the source is unclear, ask once. Normalize into a common shape per finding: id, summary, location (file/line if known), source ref (e.g. comment URL), raw text.

### 2. Dedupe, rank, and score confidence

- Merge duplicates that describe the same underlying issue.
- **Confidence**: when multiple independent sources (e.g. multiple session reviewers) report the same issue, mark higher confidence (agreement count). When there is only one source (one PR comment, one pasted item), label confidence **`single-source`** — never invent consensus.
- Rank for presentation: higher confidence and clearer severity/location first.

Present the ranked list to the user. **Wait for selection** (which items to pursue). Do not spawn fix agents until they choose.

### 3. Propose

For each selected finding, spawn a subagent. Tell it to follow `bug-fix` for judgment only: no edits, no commits. Give it the finding payload (and linked comment text if any). It returns:

```markdown
## Verdict
fix

## Rationale
Property `foo` was removed to silence the type error, but callers still send it for discounts.

## Proposal
In `src/order.ts`, map `foo` to `discountCode` before the API call. Do not drop the field.
```

Use `not a concern` and omit `## Proposal` when the issue is absent or immaterial. The proposal must be concrete enough to apply without rediscovering the bug. Trade-offs and “this needs a refactor” come back in the result — the agent does not start that work.

### 4. Orchestrator reviews proposals

Present proposals to the user. As orchestrator, select which fixes to implement only after that review. Reject or skip “not a concern,” weak proposals, and anything that needs a large refactor until the user decides.

Individual agents must not approve or start implementing on their own.

### 5. Implement

For each approved proposal:

1. Create a **separate git worktree**. If worktrees are unavailable, **stop and say so** — do not edit the main working tree instead.
2. Spawn a subagent in that worktree. Tell it to follow `bug-fix` and `implementation`, apply **only** the approved proposal, commit in that worktree, and not resolve or reply on PR comments. If the proposal cannot be applied as written, it stops and reports that — it does not invent a different fix.

Run implement agents in parallel when safe. After they finish, **integrate** (merge or cherry-pick) onto the session/target branch. On integrate conflict: **stop and ask** — do not force-resolve.

### 6. PR resolve / reply when relevant

When a finding was tied to a PR review comment (or similar) **and** the implement step succeeded on a change that will attach to that PR, the **orchestrator** resolves and replies on that comment.

Do **not** reply for: “not a concern,” rejected proposals, or failed implements.

## Composability

- **Solo one-off fix** → invoke `bug-fix` directly (not this skill).
- **Discovery then triage** → `bug-session` → this skill.
- **Already have a list** → this skill only.
