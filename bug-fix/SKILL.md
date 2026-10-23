---
name: bug-fix
description: >-
  Guides targeted bug fixes without band-aids, collateral damage, or unsolicited
  refactors. Use when fixing a reported bug, addressing a Bugbot or CI finding,
  or working from a snippet/image that indicates an issue. Distinct from
  implementation (general code writing) and review (report-only audit) — this
  skill governs how a focused fix is diagnosed and resolved.
---

# Bug Fix

This skill governs how to respond to targeted bug fixes. The context often centers on a Bugbot report, a CI/lint finding, or a snippet/image that indicates an issue.

When this skill applies, disclose briefly once (e.g. that the bug-fix skill is in use). Then write the fix using the implementation skill.

## 1. No band-aids

Do not just eliminate the immediate issue or warning regardless of side effects. Gather an understanding of what the problem is and what we were originally seeking to accomplish. For example: an error says a property on an object/API is not valid — rather than only removing the property, understand what it was meant to do, and seek a suitable replacement.

## 2. Consider consequences

You have not truly solved an issue if you have created another in its stead. Whenever you resolve an issue, ensure you have not created a problem somewhere else. Often there are trade-offs — if you are not sure what to do given a potential trade-off, stop and ask.

## 3. Analyze with a fresh perspective

Just because you are told there is a bug, or you get a Bugbot error message, does not mean the issue is still present, or that it is of any consequence. Approach the issue with a fresh perspective and critical thought.

## 4. Disclose when a refactor is needed

Sometimes a problem is no longer a small bug fix and requires a serious refactor or pivot. Do not start large changes without consulting the user first. Acknowledge that a large change is required and ask how to proceed.

## 5. Summary

Once you have resolved the issue, provide an overview of what the issue was and how you resolved it. Assume the user does not fully understand the problem and needs to be brought up to speed.
