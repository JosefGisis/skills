---
name: bug-session
description: >-
  Full bug session: establishes review context, runs parallel Bugbot-class
  reviewers (the user's choice, or three Bugbot reviewers when they have no
  preference), collects findings, then hands the list to bug-triage to finish.
  Use when explicitly asked for a bug session,
  multi-agent bug review, or parallel Bugbot-style discovery before triage and
  fix. Discovery only — does not triage or fix itself. Requires explicit
  invocation.
disable-model-invocation: true
---

# Bug Session

Explicit entry point for **discovery**. Surfaces issues via parallel Bugbot-class reviewers, then hands the finding list to `bug-triage`, which runs to completion.

This skill does **not** rank, propose, or implement fixes — `bug-triage` does that. Multi-agent review is one way to build a finding list; other lists go straight to `bug-triage`.

Disclose briefly once that bug-session is in use. After handoff, the same agent continues as orchestrator by following `bug-triage`.

## Workflow

Run these steps in order. Do not stop to ask which surfaced bugs to fix — `bug-triage` decides that.

### 1. Establish review context

Determine what will be reviewed: PR, branch, or diff scope. Default to branch changes. Resolve repo path and any PR URL/number. If context is ambiguous, ask once.

### 2. Choose reviewers

If the user already named which reviewers to use or how many, use that.

If they did not, ask once whether they have a preference, and state the suggestion: **3 identical Bugbot reviewers** on the same context. If they have no preference, run that suggestion. A missing preference is not a reason to stop the session.

Use the `review-bugbot` skill's launch shape for Bugbot subagents. Launch the chosen reviewers in parallel on the **same** context.

Do not add reviewer types the user did not ask for. The suggestion stays Bugbot.

### 3. Collect findings

Gather each reviewer’s report into one raw finding set (preserve which reviewer reported what). Do not fix anything here.

### 4. Hand off to bug-triage

Invoke the `bug-triage` skill with the collected finding list (and PR/context refs). Triage owns: dedupe/rank/confidence, decisions, proposal agents (using `bug-fix` under triage’s constraints), worktree implements, and PR resolve/reply.

## Composability

- **Session** = discovery → `bug-triage`
- **Triage alone** = already have a list → `bug-triage` only
- **One known bug** = `bug-fix` directly
