# skills

A small collection of [Claude Code Agent Skills](https://docs.claude.com/en/docs/claude-code/skills) that shape how an AI coding agent plans and implements work.

- **[deliberate](deliberate/SKILL.md)** — governs open-ended back-and-forth where the direction is still undecided: resist anchoring to the user's first framing, ask the question that would actually change the answer, name gaps in understanding, make scale concerns concrete, and converge on an answer instead of hedging indefinitely.
- **[planning](implementation/../planning/SKILL.md)** — governs the planning conversation before any code is written: clarify ambiguity up front, layer the plan by abstraction instead of front-loading detail, split oversized asks into phases, and consolidate the plan as understanding evolves.
- **[implementation](implementation/SKILL.md)** — governs code as it's written, from architecture to line-level style: minimalism, DRY over colocation, reuse before writing, directional dependencies, deliberate error handling, and other corrections for recurring bad habits.

Each skill is a single `SKILL.md` file with YAML frontmatter (`name`, `description`) describing when the skill should trigger, followed by the rules themselves.

The same content is also split into [Cursor Project Rules](.cursor/rules/) (`.mdc` files), grouped thematically so Cursor's agent can pull in just the relevant cluster:

- **[deliberate.mdc](.cursor/rules/deliberate.mdc)** — mirrors the deliberate skill directly.
- **[planning.mdc](.cursor/rules/planning.mdc)** — mirrors the planning skill directly (its 5 rules are already tightly related).
- **[implementation-structure.mdc](.cursor/rules/implementation-structure.mdc)** — architecture and structure: minimalism, conformity, DRY, reuse, dependency direction, deprecation, extraction, template method, inheritance.
- **[implementation-code-quality.mdc](.cursor/rules/implementation-code-quality.mdc)** — line-level quality: error handling, no `any`, comments, magic values, null vs. undefined.
- **[implementation-process.mdc](.cursor/rules/implementation-process.mdc)** — process discipline: tests, communicating before guessing, task specificity.

## Adding these skills to your local agent context

### Claude Code

Claude Code discovers skills from two locations:

- **Personal (all projects):** `~/.claude/skills/`
- **Project-specific (this repo only):** `<project>/.claude/skills/`

To install both skills globally, copy each skill folder into your personal skills directory:

```bash
mkdir -p ~/.claude/skills
cp -r deliberate ~/.claude/skills/deliberate
cp -r planning ~/.claude/skills/planning
cp -r implementation ~/.claude/skills/implementation
```

Or, to scope them to a single project instead, copy them into that project's `.claude/skills/` directory:

```bash
mkdir -p /path/to/your/project/.claude/skills
cp -r deliberate /path/to/your/project/.claude/skills/deliberate
cp -r planning /path/to/your/project/.claude/skills/planning
cp -r implementation /path/to/your/project/.claude/skills/implementation
```

Restart Claude Code (or start a new session) and the skills will be available. Claude invokes them automatically based on each skill's `description` trigger conditions — you can also invoke one explicitly with `/deliberate`, `/planning`, or `/implementation`.

### Cursor

Cursor discovers Project Rules from `.cursor/rules/` in the repository root. Each `.mdc` file here uses `description`-based activation ("Agent Requested") — Cursor's agent decides whether to pull a rule into context based on its `description` frontmatter, the same trigger conditions used by the Claude Code skills above.

Copy the whole directory into your project root:

```bash
mkdir -p /path/to/your/project/.cursor/rules
cp .cursor/rules/*.mdc /path/to/your/project/.cursor/rules/
```

Or copy only the clusters you want (e.g. just `planning.mdc`, or only the implementation files). Restart Cursor (or start a new chat) and the rules will be available; you can also check **Cursor Settings → Rules** to confirm they're indexed, or reference one explicitly with `@ruleName`.

If you'd rather have a rule always applied regardless of context (no agent judgment call), open the file in Cursor's rule editor and switch its type to **Always**, or set `alwaysApply: true` in the frontmatter directly.

### Other agents

Any agent harness that supports Markdown-based skill/instruction files can use these directly — point the harness at the `SKILL.md` files (or paste their contents into your system prompt / project instructions file) so the rules are loaded before planning or coding begins.
