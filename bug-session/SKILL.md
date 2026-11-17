---
name: bug-session
description: >-
  Full bug session: establish review context, ask which Bugbot-class agents to
  run in parallel (cost-aware), collect findings, then hand the list to
  bug-triage. Use when explicitly asked for a bug session, multi-agent bug
  review, or parallel Bugbot-style discovery before triage and fix. Discovery
  only — does not triage or fix itself. Requires explicit invocation.
disable-model-invocation: true
---

# Bug Session

Explicit entry point for **discovery**. Surfaces issues via parallel Bugbot-class reviewers, then hands the finding list to `bug-triage`.

This skill does **not** rank, propose, or implement fixes — `bug-triage` does that. Multi-agent review is one way to build a finding list; other lists go straight to `bug-triage`.

Disclose briefly once that bug-session is in use. After handoff, the same agent continues as orchestrator by following `bug-triage`.

## Workflow

### 1. Establish review context

Determine what will be reviewed: PR, branch, or diff scope (e.g. branch changes vs uncommitted). Resolve repo path and any PR URL/number. If context is ambiguous, ask once.

### 2. Ask which agents to run

Ask the user:

- Which Bugbot-class reviewer(s) to use (default: Bugbot via the `review-bugbot` skill / `bugbot` subagent)
- How many to run **in parallel**

Note cost: more parallel reviewers raise confidence via agreement but multiply cost. **Default recommendation: 3 identical Bugbot reviewers** on the same context (enough for consensus, bounded cost).

Do not launch until they answer (or accept the default).

### 3. Launch parallel reviewers

Detectors: **Bugbot via the `review-bugbot` skill only**. Do not also fan out `review`, security-review, or CI investigators.

Launch the agreed number of reviewers in parallel on the **same** determined context. Prefer the `review-bugbot` skill’s launch shape when using Bugbot subagents.

### 4. Collect findings

Gather each reviewer’s report into one raw finding set (preserve which reviewer reported what). Do not fix anything here.

### 5. Hand off to bug-triage

Invoke the `bug-triage` skill with the collected finding list (and PR/context refs). Triage owns: dedupe/rank/confidence, selection, proposal agents (using `bug-fix` under triage’s constraints), orchestrator approval, worktree implements, and PR resolve/reply.

## Composability

- **Session** = discovery → `bug-triage`
- **Triage alone** = already have a list → `bug-triage` only
- **One known bug** = `bug-fix` directly
