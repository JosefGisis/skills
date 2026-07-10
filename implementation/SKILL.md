---
name: implementation
description: Use this skill whenever writing, modifying, reviewing, or refactoring code — from high-level architectural choices down to line-level style. Trigger any time code is being produced or changed, whether implementing a planned feature, fixing a bug, extending existing functionality, writing a utility, or making a small targeted change. This skill corrects specific recurring bad habits — verbosity, premature abstraction, unsafe types, dependency tangles, and more — and its rules take precedence over default coding instincts whenever they conflict. This is not a general design-pattern encyclopedia; it is a targeted set of corrections based on observed mistakes.
---

# Implementation

This skill governs code as it's actually written — from architecture down to style. It does not dictate which design pattern or architectural approach to use in every situation; that's context-dependent. What it does is correct a specific set of recurring habits that consistently produce worse code than necessary. When these rules conflict with default instincts, these rules win.

## 1. Minimalism

Write only the code the problem requires. No extra abstractions, no defensive scaffolding that isn't earning its keep, no verbose patterns included out of habit. A human developer feels the friction of every line they type and instinctively writes the leanest correct solution. Match that instinct. The test is simple: could this solve the problem with less code without sacrificing clarity? If yes, it has too much.

## 2. Architectural conformity

Scale architecture to the feature — heavy structure for a simple task is waste. When working in an existing codebase, match its conventions and patterns, even imperfect ones. Implementing a feature does not implicitly authorize restructuring the surrounding architecture. That is a separate, explicit request. Work within the existing constraints. If the codebase has a pattern for routing, data access, or state management, use that pattern — do not introduce a different one because it's theoretically cleaner.

## 3. DRY over colocation

Do not repeat code. This is near-absolute. Colocating related code for readability matters too, but when DRY and colocation conflict, DRY wins. Code stays local only when it is genuinely specific to one feature and used nowhere else. The moment something is reused, it is by definition a separate concern — extract it to a shared utility, module, or library, even if that means it no longer sits beside its call site.

## 4. Reuse before writing

Before writing any helper, utility, or shared function, check whether equivalent functionality already exists — in an external library the project uses, or elsewhere in the codebase. If it exists and importing it makes architectural sense, use it. If it exists but importing from that location would be architecturally wrong (different feature boundary, wrong layer), extract a local copy. If consolidating the duplication is easy, do it. If not, extract locally and leave a brief note about the existing duplicate — cleaning up code outside the current scope is not this task's job.

## 5. Directional dependencies

Never create circular dependencies. Keep the dependency graph flowing in one direction: leaves and utilities are imported by services, services by handlers, handlers by routes — never the reverse. Indirect circularity (A → B → C → A) is a structural smell even when it doesn't cause a runtime failure. Avoid it.

Sibling modules at the same directory level may import from each other when the relationship is conceptually hierarchical and one-directional — an orchestration service importing from a utility service that happens to sit at the same folder depth. A flat directory structure is fine as long as dependency *direction* stays clear and consistent.

## 6. No premature deprecation or wrapping

Do not add `@deprecated` markers or backwards-compatibility wrappers unless explicitly asked. This is especially wrong mid-task: if the direction of a feature shifts during implementation, do not wrap or deprecate the earlier version. The feature is being actively built. Write it for its current shape and move on.

## 7. Extraction by judgment, not reflex

Do not extract a small helper function used exactly once. Inline it. The theoretical benefit of extraction — reuse, isolation — does not apply to something called once, and a tiny extracted function just adds a place the reader has to jump to for no payoff. However, if a single-use block is large or genuinely complex, extract it — a long inline blob hurts readability more than a well-named function.

When extracting, name functions for what they actually do. If two functions end up with nearly identical names differentiated by a subtle suffix to avoid a collision, stop. Either the abstraction is wrong, one should be inlined, or the names need to describe genuinely different behaviors — not just avoid a namespace clash.

## 8. Template Method for sequential processes

When a process has clear ordered steps, prefer a top-level orchestrator that calls each step in sequence over a single monolithic function or a cluster of vaguely-related helpers. Making the steps explicit — `validateInput`, `transformData`, `persistResult` — makes the flow readable at a glance, debuggable when a step fails, and extensible when a new step is needed. Not every function benefits from this, but it is underused where it would help.

## 9. Inheritance for shared structure

When multiple variants share the same skeleton of behavior, use a base or abstract class for the shared logic and extend it with subclasses that override only what differs. This eliminates duplicated logic and makes the variant relationships explicit in the code structure itself. Apply judgment — this is not always the right tool, but it is currently underreached for in situations where it would clarify things.

## 10. Deliberate error handling

Let errors bubble up to a centralized handler by default. Do not scatter try-catch blocks throughout the codebase catching errors ad hoc. When throwing, make error messages descriptive without leaking sensitive internals. For specific needs — retry logic, idempotency checks, or classifying errors to decide what's safe to surface via an API — create dedicated error classes extending the base Error type. Do not rely on parsing generic error message strings to infer meaning.

## 11. No `any`

`any` is a last resort, not a convenience. There are very few legitimate cases. Use the narrowest, most specific type the situation allows. If typing something correctly feels difficult, that difficulty is surfacing a design question worth answering — not a reason to escape-hatch with `any`.

## 12. Minimal, purposeful comments

Comment only what the code cannot communicate on its own: unconventional choices, non-obvious reasoning, genuinely complex logic. Do not comment self-evident code — it adds noise. Keep comments short. A comment that needs to be long is usually a sign the code itself should be restructured, not that more prose is needed to explain it.

## 13. Tests only where testing exists

Write or suggest tests only if the codebase already has established testing practices. Do not introduce a testing framework or convention as a side effect of a feature request — that is a separate initiative. When tests are written, keep them minimal: a small number of well-chosen cases that maximize coverage, not an exhaustive suite covering every permutation. Unreadable tests are unmaintainable tests.

## 14. Named constants, not magic values

Do not inline string literals or numeric constants directly in logic. Give them a name — at the top of the file for local use, or in a shared constants module if they appear in multiple places. The name should convey what the value represents, not just what it is.

## 15. Null means empty; undefined means absent

These are semantically different. `undefined` means a value was never set or is intentionally optional — its absence carries meaning. `null` means a value was deliberately set to nothing — its presence as null carries meaning. Preserve this distinction explicitly, especially in database writes and updates. Do not substitute one for the other, and do not let empty strings silently stand in for either.

## 16. Communicate before guessing

When something unexpected happens mid-implementation — an error that doesn't make sense, a dependency that behaves differently than documented, a design question not settled during planning — do not push forward on a guess. Stop and ask.

When something breaks, do not silently chain commands or attempt fix after fix without checking in. Stop. Explain what the problem appears to be, what you think is causing it, and what you intend to try. Then proceed. The user seeing "here's what went wrong and here's my plan" before you act is far more valuable than "here's a wall of changes I already made that may or may not be right."

## 17. Task specificity

Don't fix unrelated bugs, dead code, or style issues you notice along the way — mention them and ask first, even if the fix looks trivial.
