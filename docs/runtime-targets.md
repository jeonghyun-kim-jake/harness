# Runtime Targets

Harness now separates the team architecture model from the runtime-specific files that activate it. The same domain design can be emitted for Claude Code, Cursor, Codex, or multiple targets at once.

There are two ways to build a target:

- **Generated path:** run the Harness meta-skill from Claude Code and ask it to emit `claude`, `cursor`, or `codex` files.
- **Manual bootstrap path:** if Claude Code is unavailable, use `CURSOR.md` or `CODEX.md` as a scaffold prompt inside Cursor or Codex and create the same files directly.

## Target Matrix

| Target | Entry Point | Agent/Role Files | Skill/Playbook Files | Orchestration |
|--------|-------------|------------------|----------------------|---------------|
| `claude` | `CLAUDE.md` | `.claude/agents/{name}.md` | `.claude/skills/{skill}/SKILL.md` | Claude Code Agent Teams: `TeamCreate`, `SendMessage`, `TaskCreate`, `Agent` |
| `cursor` | `.cursor/rules/harness-{domain}.mdc` | `.cursor/rules/harness-{agent}.mdc` | `docs/harness/{domain}/skills/{skill}.md` | Cursor project rules plus file-based handoffs |
| `codex` | `AGENTS.md` | `docs/harness/{domain}/agents/{name}.md` | `docs/harness/{domain}/skills/{skill}.md` | Codex instructions plus file-based handoffs |

## Shared Design Contract

Every target starts from the same common model:

- Domain goal and scope
- 3-7 role definitions with responsibilities and boundaries
- Skill playbooks that describe how each role works
- One orchestrator flow with phases, inputs, outputs, handoffs, retries, and validation
- `_workspace/` artifact naming: `{phase}_{agent}_{artifact}.md`

The target adapter only changes where the instructions are written and how the runtime is expected to execute them.

## Claude Code

Claude remains the richest runtime because it can use native Agent Teams.

Generated files:

```text
.claude/
  agents/{agent-name}.md
  skills/{skill-name}/SKILL.md
CLAUDE.md
```

Rules:

- Prefer Agent Teams for two or more collaborating agents.
- Use `TeamCreate`, `TaskCreate`, and `SendMessage` for team orchestration.
- Use `Agent` with `run_in_background` only when direct result collection is enough.
- Keep `CLAUDE.md` as a pointer only: trigger, goal, and change history.

## Cursor

Cursor support uses project rules as the activation layer. The team does not run as native parallel agents; instead, the generated rules tell Cursor when to follow the harness and where to read the role and skill playbooks.

Build guide: [`../CURSOR.md`](../CURSOR.md)

Generated files:

```text
.cursor/
  rules/
    harness-{domain}.mdc
    harness-{agent}.mdc
docs/
  harness/{domain}/
    orchestrator.md
    agents/{agent-name}.md
    skills/{skill-name}.md
```

Rules:

- Use `.mdc` project rules with frontmatter.
- Put the broad trigger in `harness-{domain}.mdc`.
- Put role-specific guardrails in optional `harness-{agent}.mdc` files.
- Route all cross-role handoffs through `_workspace/` artifacts.
- If Claude Code is unavailable, follow the direct Cursor bootstrap prompt in `CURSOR.md`.

## Codex

Codex support uses repository instructions as the activation layer. `AGENTS.md` gives the root-level harness pointer and links to the detailed role and skill playbooks.

Build guide: [`../CODEX.md`](../CODEX.md)

Generated files:

```text
AGENTS.md
docs/
  harness/{domain}/
    orchestrator.md
    agents/{agent-name}.md
    skills/{skill-name}.md
```

Rules:

- Keep `AGENTS.md` concise and stable.
- Link to `docs/harness/{domain}/orchestrator.md` for detailed phase flow.
- Use role files for persona, responsibility boundaries, inputs, outputs, and validation.
- Use `_workspace/` artifacts for handoffs between phases and simulated roles.
- If Claude Code is unavailable, follow the direct Codex bootstrap prompt in `CODEX.md`.

## Multi-Target Generation

When the user requests more than one target, do not redesign the harness separately for each runtime. Build the common model once, then emit each target's files from that model.

Required final report:

- Selected target or targets
- Generated files grouped by target
- Known runtime limitations
- Verification performed
- Follow-up commands or prompts to try

## References

- Cursor project rules: https://docs.cursor.com/context/rules
- Codex AGENTS.md guide: https://developers.openai.com/codex/guides/agents-md/
