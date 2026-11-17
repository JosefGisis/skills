---
name: implementation
description: >-
  Orchestrates writing code: plan against clean-code and relevant language
  skills, write under process rules, then run the review skill. Use when
  implementing, modifying, or fixing code (features, bugs, utilities, or
  small targeted changes). Invoke this skill rather than writing from default
  instincts alone.
---

# Implementation

This skill orchestrates how code gets written. It does not hold coding-style rules — those live in `clean-code` and in language skills (e.g. `javascript`). When these process rules conflict, the priority below wins.

## Conflict priority

1. **Task specificity** and **communicate before guessing**
2. **Tests only where testing exists**

## Workflow

Follow these steps in order. Do not skip the review step.

1. **Load standards** — Read the `clean-code` skill. Also read any language skill that applies to the files you will touch (for example `javascript` for JS/TS). Those skills are the coding bar for this task.
2. **Plan** — Outline the change against those rules before editing. Keep the plan tight enough that the work stays inside the request.
3. **Write** — Implement under the process rules below. Apply clean-code and any loaded language skill while writing.
4. **Review** — Invoke the `review` skill on what you just wrote. Treat its report as the check that the work matches intent and standards.

## 1. Communicate before guessing

When something unexpected happens mid-implementation — an error that doesn't make sense, a dependency that behaves differently than documented, a design question not settled during planning — do not push forward on a guess. Stop and ask.

When something breaks, do not silently chain commands or attempt fix after fix without checking in. Stop. Explain what the problem appears to be, what you think is causing it, and what you intend to try. Then proceed. The user seeing "here's what went wrong and here's my plan" before you act is far more valuable than "here's a wall of changes I already made that may or may not be right."

## 2. Task specificity

Don't fix unrelated bugs, dead code, or style issues you notice along the way — mention them and ask first, even if the fix looks trivial.

## 3. Tests only where testing exists

Write or suggest tests only if the codebase already has established testing practices. Do not introduce a testing framework or convention as a side effect of a feature request — that is a separate initiative. When tests are written, keep them minimal: a small number of well-chosen cases that maximize coverage, not an exhaustive suite covering every permutation. Unreadable tests are unmaintainable tests.
