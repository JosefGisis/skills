# skills

A collection of Agent Skills and Cursor Rules that shape how an AI coding agent plans, deliberates, writes, reviews, and fixes code.

## Skills

Each skill is a folder containing a `SKILL.md` with YAML frontmatter (`name`, `description`) describing its purpose and trigger conditions:

- **[deliberate](deliberate/SKILL.md)** — governs open-ended back-and-forth where the direction is still undecided: resist anchoring, ask load-bearing questions, name gaps, make scale concerns concrete, and converge deliberately.
- **[planning](planning/SKILL.md)** — governs the planning conversation before code is written: clarify ambiguity up front, layer plans by abstraction, split oversized asks, and consolidate as understanding evolves.
- **[implementation](implementation/SKILL.md)** — orchestrates writing code: plans against clean-code and language standards, implements under process rules (task specificity, communicate before guessing, tests only where established), and triggers the review skill.
- **[clean-code](clean-code/SKILL.md)** — language-agnostic code quality principles: minimalism, intentional structure, architectural conformity, DRY over colocation, directional dependencies, and deliberate error handling.
- **[javascript](javascript/SKILL.md)** — JavaScript and TypeScript typing rules: no `any` and semantic distinction between `null` and `undefined`.
- **[review](review/SKILL.md)** — two-axis report-only review via parallel sub-agents: plan/feature fidelity and coding standards conformance (`clean-code` + language skills).
- **[bug-fix](bug-fix/SKILL.md)** — rules for diagnosing and resolving a single targeted bug without band-aids, collateral damage, or unsolicited refactors.
- **[bug-triage](bug-triage/SKILL.md)** — triages finding lists from any source, dedupes, ranks by confidence, runs proposal agents, then implements approved fixes in isolated worktrees and resolves linked PR comments. Explicit invoke only.
- **[bug-session](bug-session/SKILL.md)** — runs parallel Bugbot-class reviewers over a PR or diff, gathers findings, and hands the list to `bug-triage`. Explicit invoke only.
- **[prompt-auditor](prompt-auditor/SKILL.md)** — audits a draft prompt, closes material gaps with quick questions, and returns a sectioned improved prompt runnable with a single yes. Explicit invoke only (`/prompt-auditor`).

## Cursor Project Rules (`.cursor/rules/`)

Always-on project rules configured with `alwaysApply: true`:

- **[disclose-rule-usage.mdc](.cursor/rules/disclose-rule-usage.mdc)** — requires disclosing rule usage when discretionary rules apply.
- **[clean-code.mdc](.cursor/rules/clean-code.mdc)** — always-applied clean code standards.
- **[javascript.mdc](.cursor/rules/javascript.mdc)** — always-applied JS/TS typing standards.
- **[planning.mdc](.cursor/rules/planning.mdc)** — always-applied implementation planning discipline.
- **[deliberate.mdc](.cursor/rules/deliberate.mdc)** — always-applied decision-making and pushback discipline.
- **[bug-fix.mdc](.cursor/rules/bug-fix.mdc)** — always-applied single bug fix judgment rules.

## Adding these skills to your local agent context

Run commands from this repository's root.

### Claude Code

Claude Code discovers skills from:

- **Personal (all projects):** `~/.claude/skills/`
- **Project-specific (this repo only):** `<project>/.claude/skills/`

To install or update skills globally:

```bash
mkdir -p ~/.claude/skills
for s in deliberate planning implementation clean-code javascript review bug-fix bug-triage bug-session prompt-auditor; do
  cp -r "$s" ~/.claude/skills/
done
```

Or for a specific project:

```bash
mkdir -p /path/to/your/project/.claude/skills
for s in deliberate planning implementation clean-code javascript review bug-fix bug-triage bug-session prompt-auditor; do
  cp -r "$s" /path/to/your/project/.claude/skills/
done
```

### Cursor

Copy all rules into your project's `.cursor/rules/`:

```bash
mkdir -p /path/to/your/project/.cursor/rules
cp .cursor/rules/*.mdc /path/to/your/project/.cursor/rules/
```

Restart Cursor or start a new chat session to load the updated rules and skills.
