---
name: implementation
description: >-
  Corrects recurring implementation habits — verbosity, premature abstraction,
  unsafe types, dependency tangles, and scope creep. Use when writing, modifying,
  reviewing, or refactoring code (features, bugs, utilities, or small targeted
  changes). Not a design-pattern encyclopedia; a targeted correction set that
  takes precedence over default coding instincts when they conflict.
---

# Implementation

This skill governs code as it's actually written — from architecture down to style. It does not dictate which design pattern to use in every situation; that's context-dependent. It corrects recurring habits that consistently produce worse code than necessary. When these rules conflict with default instincts, these rules win.

## Conflict priority

When two rules in this skill pull in different directions, apply this order (higher wins):

1. **Task specificity** and **communicate before guessing** (process)
2. **Architectural conformity** (match the codebase you are in)
3. **Minimalism**
4. Everything else

Language-specific rules (§11, §15) apply only in languages where those constructs exist.

## 1. Minimalism

Write only the code the problem requires. No extra abstractions, no defensive scaffolding that isn't earning its keep, no verbose patterns included out of habit. A human developer feels the friction of every line they type and instinctively writes the leanest correct solution. Match that instinct. The test is simple: could this solve the problem with less code without sacrificing clarity? If yes, it has too much.

## 2. Architectural conformity

Scale architecture to the feature — heavy structure for a simple task is waste. When working in an existing codebase, match its conventions and patterns, even imperfect ones. Implementing a feature does not implicitly authorize restructuring the surrounding architecture. That is a separate, explicit request. Work within the existing constraints. If the codebase has a pattern for routing, data access, or state management, use that pattern — do not introduce a different one because it's theoretically cleaner.

## 3. DRY over colocation

Do not repeat code when the same logic is (or is about to be) used in more than one place. Colocating related code for readability matters too, but when DRY and colocation conflict, DRY wins. Code stays local only when it is genuinely specific to one feature and used nowhere else. Once something is shared, extract it to a shared utility, module, or library — even if that means it no longer sits beside its call site.

## 4. Reuse before writing

Before writing any helper, utility, or shared function, check whether equivalent functionality already exists — in an external library the project uses, or elsewhere in the codebase. If it exists and importing it makes architectural sense, use it. If it exists but importing from that location would be architecturally wrong (different feature boundary, wrong layer), prefer a thin local wrapper or a carefully scoped local copy over a cross-boundary import. If consolidating the duplication is easy and in-scope, do it. If not, keep the local version and leave a brief note about the existing duplicate — cleaning up code outside the current scope is not this task's job.

## 5. Directional dependencies

Never create circular dependencies. Keep the dependency graph flowing in one direction: leaves and utilities are imported by services, services by handlers, handlers by routes — never the reverse. Indirect circularity (A → B → C → A) is a structural smell even when it doesn't cause a runtime failure. Avoid it.

Sibling modules at the same directory level may import from each other when the relationship is conceptually hierarchical and one-directional — an orchestration service importing from a utility service that happens to sit at the same folder depth. A flat directory structure is fine as long as dependency *direction* stays clear and consistent.

## 6. No premature deprecation or wrapping

Do not add `@deprecated` markers or backwards-compatibility wrappers unless explicitly asked. This is especially wrong mid-task: if the direction of a feature shifts during implementation, do not wrap or deprecate the earlier version. The feature is being actively built. Write it for its current shape and move on.

## 7. Extraction by judgment, not reflex

Do not extract a small helper function used exactly once. Inline it. The theoretical benefit of extraction — reuse, isolation — does not apply to something called once, and a tiny extracted function just adds a place the reader has to jump to for no payoff. However, if a single-use block is large or genuinely complex, extract it — a long inline blob hurts readability more than a well-named function.

When extracting, name functions for what they actually do. If two functions end up with nearly identical names differentiated by a subtle suffix to avoid a collision, stop. Either the abstraction is wrong, one should be inlined, or the names need to describe genuinely different behaviors — not just avoid a namespace clash.

## 8. Explicit steps for sequential processes

When a process has clear ordered steps, prefer a top-level orchestrator that calls each step in sequence over a single monolithic function or a cluster of vaguely-related helpers — *if* that clarity is worth the extra surface area (see Minimalism). Making the steps explicit — `validateInput`, `transformData`, `persistResult` — makes the flow readable at a glance, debuggable when a step fails, and extensible when a new step is needed. Do not force this shape onto simple functions.

## 9. Shared structure only when variants share a skeleton

When multiple variants already share the same skeleton of behavior, a base/abstract type (or equivalent shared core) that overrides only what differs can eliminate duplication and make relationships explicit. Use this when the shared skeleton is real and stable — not as a default for every similar-looking pair of functions. Prefer simpler options (shared helpers, composition) when inheritance would be speculative.

## 10. Deliberate error handling

If the project has a centralized error-handling path, let errors bubble up to it by default. Do not scatter try-catch blocks catching errors ad hoc. When throwing, make error messages descriptive without leaking sensitive internals. For specific needs — retry logic, idempotency checks, or classifying errors to decide what's safe to surface via an API — create dedicated error types rather than parsing generic error message strings. If the codebase has no centralized handler, follow its existing error style; do not invent a new global strategy mid-task.

## 11. No `any` (TypeScript / gradual-typing languages)

In TypeScript and similar gradually typed languages, `any` is a last resort, not a convenience. There are very few legitimate cases. Use the narrowest, most specific type the situation allows. If typing something correctly feels difficult, that difficulty is surfacing a design question worth answering — not a reason to escape-hatch with `any`. Skip this rule in languages without an `any`-like escape hatch. If the surrounding codebase already relies on `any` heavily, match local convention for consistency unless the task is specifically to tighten types.

## 12. Minimal, purposeful comments

Comment only what the code cannot communicate on its own: unconventional choices, non-obvious reasoning, genuinely complex logic. Do not comment self-evident code — it adds noise. Keep comments short. A comment that needs to be long is usually a sign the code itself should be restructured, not that more prose is needed to explain it.

## 13. Tests only where testing exists

Write or suggest tests only if the codebase already has established testing practices. Do not introduce a testing framework or convention as a side effect of a feature request — that is a separate initiative. When tests are written, keep them minimal: a small number of well-chosen cases that maximize coverage, not an exhaustive suite covering every permutation. Unreadable tests are unmaintainable tests.

## 14. Named constants, not magic values

Do not inline unexplained string or numeric literals in logic when a name would clarify meaning. Give them a name — at the top of the file for local use, or in a shared constants module if they appear in multiple places. The name should convey what the value represents, not just what it is.

Exceptions: self-evident values (`0`, `1`, `-1`, empty string in a clear context), idiomatic literals (HTTP methods already readable at the call site), and test fixtures where the literal *is* the point. Prefer Minimalism over ritual naming.

## 15. Null means empty; undefined means absent (JavaScript / TypeScript)

In JS/TS, these are semantically different. `undefined` means a value was never set or is intentionally optional — its absence carries meaning. `null` means a value was deliberately set to nothing — its presence as null carries meaning. Preserve this distinction explicitly, especially in database writes and updates. Do not substitute one for the other, and do not let empty strings silently stand in for either. In other languages, follow that language's idiomatic empty/absent conventions instead.

## 16. Communicate before guessing

When something unexpected happens mid-implementation — an error that doesn't make sense, a dependency that behaves differently than documented, a design question not settled during planning — do not push forward on a guess. Stop and ask.

When something breaks, do not silently chain commands or attempt fix after fix without checking in. Stop. Explain what the problem appears to be, what you think is causing it, and what you intend to try. Then proceed. The user seeing "here's what went wrong and here's my plan" before you act is far more valuable than "here's a wall of changes I already made that may or may not be right."

## 17. Task specificity

Don't fix unrelated bugs, dead code, or style issues you notice along the way — mention them and ask first, even if the fix looks trivial.
