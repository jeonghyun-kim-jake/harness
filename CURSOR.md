# Cursor Harness Guide

This guide explains how to build a Harness target for Cursor. Cursor does not run Claude Code Agent Teams directly; Harness translates the same team architecture into Cursor project rules and supporting playbooks.

## What Gets Generated

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

## Build Steps

### Option A: Generate From Claude Code

1. Install Harness in Claude Code.

```bash
claude plugin marketplace add revfactory/harness
claude plugin install harness@harness
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

2. Run Harness from the project root and ask for the Cursor target.

```bash
claude "build a Cursor harness for this project"
```

For a domain-specific harness, be explicit:

```bash
claude "build a Cursor harness for frontend refactoring in this repository"
```

3. Verify the generated files.

```bash
ls -la .cursor/rules/
ls -la docs/harness/
```

4. Open the project in Cursor.

Cursor will load project rules from `.cursor/rules`. The broad `harness-{domain}.mdc` rule should point Cursor to the orchestrator and role playbooks.

5. Run a real task in Cursor.

Use a prompt that matches the harness domain:

```text
Use the frontend refactoring harness to review the dashboard components and produce a phased refactor plan.
```

### Option B: Build Directly In Cursor Without Claude Code

If Claude Code is not available, use Cursor itself to create the same file layout. This path does not install or execute the Harness plugin; it uses the Harness contract as a scaffold.

1. Create the target directories.

```bash
mkdir -p .cursor/rules
mkdir -p docs/harness/{domain}/agents
mkdir -p docs/harness/{domain}/skills
```

Replace `{domain}` with a short slug such as `frontend-refactor` or `release-readiness`.

2. Ask Cursor to generate the harness files.

```text
Create a Cursor harness for {domain}.

Use this file layout:
- .cursor/rules/harness-{domain}.mdc
- docs/harness/{domain}/orchestrator.md
- docs/harness/{domain}/agents/{agent-name}.md
- docs/harness/{domain}/skills/{skill-name}.md

Design 3-5 specialist roles, one orchestrator phase flow, and file-based handoffs through _workspace/{phase}_{agent}_{artifact}.md.
Use a producer-reviewer or fan-out/fan-in pattern if the domain needs multiple reviewers.
Keep the .mdc rule concise and point to the docs/harness files for details.
```

3. Review the generated `.mdc` rule.

The rule should include frontmatter like:

```markdown
---
description: Use this harness for {domain} work.
globs:
  - "**/*"
alwaysApply: false
---
```

4. Run the verification checklist below.

## Rule Authoring Contract

`harness-{domain}.mdc` should contain:

- Frontmatter with `description`, `globs`, and `alwaysApply`
- A short trigger rule
- A pointer to `docs/harness/{domain}/orchestrator.md`
- `_workspace/` handoff rules
- Change history

Role-specific `.mdc` files should stay focused and small. Put large instructions in `docs/harness/{domain}/agents/` or `docs/harness/{domain}/skills/` and reference those files from the rule.

## Verification Checklist

- [ ] `.cursor/rules/harness-{domain}.mdc` exists
- [ ] The rule has valid MDC frontmatter
- [ ] `docs/harness/{domain}/orchestrator.md` exists
- [ ] Each role has an agent file under `docs/harness/{domain}/agents/`
- [ ] Each reusable workflow has a skill playbook under `docs/harness/{domain}/skills/`
- [ ] Cursor can answer which harness rule it loaded for a matching task

## Known Limits

Cursor rules provide persistent instructions, not native parallel agents. Harness preserves team behavior through phases, role switching, and file-based handoffs rather than `TeamCreate` or `SendMessage`.
