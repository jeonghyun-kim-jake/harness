# Codex Harness Guide

This guide explains how to build a Harness target for Codex. Codex uses repository instructions from `AGENTS.md`, so Harness writes a concise root pointer plus detailed role and skill playbooks under `docs/harness/`.

## What Gets Generated

```text
AGENTS.md
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

2. Run Harness from the project root and ask for the Codex target.

```bash
claude "build a Codex harness for this project"
```

For a domain-specific harness, be explicit:

```bash
claude "build a Codex harness for repository maintenance and release readiness"
```

3. Verify the generated files.

```bash
ls -la AGENTS.md
ls -la docs/harness/
```

4. Open the project in Codex.

Codex should read `AGENTS.md` as the repository instruction entry point. The file should link to the generated orchestrator and role playbooks.

5. Run a real task in Codex.

Use a prompt that matches the harness domain:

```text
Use the repository maintenance harness to inspect the current branch and produce a release-readiness report.
```

### Option B: Build Directly In Codex Without Claude Code

If Claude Code is not available, use Codex itself to create the same repository instruction layout. This path does not install or execute the Harness plugin; it uses the Harness contract as a scaffold.

1. Create the target directories.

```bash
mkdir -p docs/harness/{domain}/agents
mkdir -p docs/harness/{domain}/skills
```

Replace `{domain}` with a short slug such as `repo-maintenance` or `release-readiness`.

2. Ask Codex to generate the harness files.

```text
Create a Codex harness for {domain}.

Use this file layout:
- AGENTS.md
- docs/harness/{domain}/orchestrator.md
- docs/harness/{domain}/agents/{agent-name}.md
- docs/harness/{domain}/skills/{skill-name}.md

Design 3-5 specialist roles, one orchestrator phase flow, and file-based handoffs through _workspace/{phase}_{agent}_{artifact}.md.
Keep AGENTS.md concise: include the harness trigger, the orchestrator path, role/playbook paths, validation expectations, and change history.
Put detailed behavior in docs/harness, not in AGENTS.md.
```

3. Ask Codex to verify its own instructions.

```text
Before doing any task, summarize the harness trigger, orchestrator path, and role files you see from AGENTS.md.
```

4. Run the verification checklist below.

## AGENTS.md Contract

`AGENTS.md` should stay short and stable. It should include:

- Harness name and domain
- Trigger condition
- Pointer to `docs/harness/{domain}/orchestrator.md`
- Pointer to generated role and skill playbooks
- `_workspace/` handoff rules
- Change history

Put detailed role behavior in `docs/harness/{domain}/agents/` and task procedures in `docs/harness/{domain}/skills/`.

## Verification Checklist

- [ ] `AGENTS.md` exists at the repository root
- [ ] `AGENTS.md` points to `docs/harness/{domain}/orchestrator.md`
- [ ] Each role has an agent file under `docs/harness/{domain}/agents/`
- [ ] Each reusable workflow has a skill playbook under `docs/harness/{domain}/skills/`
- [ ] Codex can summarize the harness trigger and orchestrator path before starting a matching task

## Known Limits

Codex repository instructions are not native parallel agents. Harness preserves team behavior through phased execution, role-specific playbooks, and file-based handoffs rather than `TeamCreate` or `SendMessage`.
