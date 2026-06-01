# Quickstart — 5 Minutes to Your First Harness

> **Time budget: 5 minutes (strict).** If you are not at Step 5 within 5 minutes, stop and file an issue — that is a bug in this document, not a bug in you.

<!-- TODO: Loom embed — 60s screen recording showing Steps 1→5 end-to-end. Replace this comment with the `<iframe>` once recorded. -->

**What you will have at the end:** a working harness for one target: Claude Code (`.claude/agents/` and `.claude/skills/`), Cursor (`.cursor/rules/` and `docs/harness/`), or Codex (`AGENTS.md` and `docs/harness/`).

**Prerequisites (check before starting):**
- Claude Code **v2.x or later** (`claude --version` should return `2.x` or higher)
- A shell that persists `export` across commands (bash, zsh, or fish)
- Network access to `github.com` and `api.anthropic.com`

For Cursor or Codex generation, the easiest path is to run this meta-skill from Claude Code, then commit the generated target files into the project where Cursor or Codex will use them. If Claude Code is unavailable, use the direct bootstrap instructions in [`../CURSOR.md`](../CURSOR.md) or [`../CODEX.md`](../CODEX.md).

---

## Step 1 — Add the marketplace (60 seconds)

```bash
claude plugin marketplace add revfactory/harness
```

**What this does:** Registers the `harness` marketplace so Claude Code can discover plugins published by `revfactory`.

**Expected output:** `Added marketplace: revfactory/harness`

---

## Step 2 — Install the plugin and enable the Experimental flag (40 seconds)

```bash
claude plugin install harness@harness
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

*(To persist the flag across shell sessions, append the `export` line to `~/.zshrc` or `~/.bashrc`.)*

**What this does:** Installs the `harness` plugin from the `harness` marketplace, then enables Agent Teams — the Claude Code API harness uses to orchestrate multi-agent workflows. See [`docs/experimental-dependency.md`](./experimental-dependency.md) for why the flag is required.

**Failure FAQ #1 — `AGENT_TEAMS not found` / teams don't instantiate**
**Cause:** Claude Code version is older than v2.x (Agent Teams was introduced in v2.0).
**Fix:** Run `claude --version`. If below 2.0, upgrade via `npm i -g @anthropic-ai/claude-code` (or your distribution's installer), then repeat Step 2.

---

## Step 3 — Generate a harness from one sentence (2 minutes)

This step assumes Claude Code is available. Without Claude Code, skip to [`../CURSOR.md`](../CURSOR.md) or [`../CODEX.md`](../CODEX.md) and use the manual bootstrap prompt.

```bash
claude "build a harness for a fintech risk-assessment team"
```

**What this does:** Invokes the `/harness:harness` meta-skill, which analyzes your domain sentence and scaffolds a team of specialized agents plus their runtime-specific files.

**Try these alternate prompts** — any of them work:
- `claude "하네스 구성해줘 — 핀테크 리스크 평가 팀"` (Korean also works)
- `claude "build a harness for an e-commerce fraud-detection workflow"`
- `claude "design an agent team for technical due diligence on open-source repos"`
- `claude "build a Cursor harness for a frontend refactoring workflow"`
- `claude "build a Codex harness for repository maintenance"`

**Expected output:** A streaming plan, then confirmation that 3–5 role files and their skills/playbooks were written for the selected target.

**Failure FAQ #2 — The Korean prompt returns nothing / the English one succeeds but Korean doesn't**
**Cause:** Locale or tokenizer misrouting; harness's orchestrator matches on Korean trigger words ("하네스 구성"), which are built into the skill definition.
**Fix:** If Korean fails, re-run with the English prompt above — the underlying skill is identical. If both fail, jump to Failure FAQ #3.

---

## Step 4 — Verify the generated files (30 seconds)

Claude target:

```bash
ls -la .claude/agents/
ls -la .claude/skills/
```

Cursor target:

```bash
ls -la .cursor/rules/
ls -la docs/harness/
```

Codex target:

```bash
ls -la AGENTS.md
ls -la docs/harness/
```

**What this does:** Confirms the meta-skill wrote files to the expected locations.

**Expected output:** 3–5 role files and matching skill/playbook files, with names reflecting your domain (e.g., `risk-analyst.md`, `compliance-reviewer.md`, `portfolio-monitor.md` for the fintech example).

**Failure FAQ #3 — "Nothing was generated" / directories are empty**
**Cause:** The plugin is not actually installed or is not active in the current project.
**Fix:** Run `claude plugin list`. If `harness@harness` is absent, repeat Step 2. If present but inactive, run `claude plugin enable harness@harness`, then repeat Step 3.

---

## Step 5 — Run a sample task against the new team (90 seconds)

Copy a realistic Jira-ticket-style prompt and hand it to your fresh team:

```bash
claude "Ticket FIN-427: A new corporate customer (mid-cap manufacturer, \$80M revenue, South Korea) has applied for a \$5M working-capital line. Produce a risk assessment covering (1) credit-history red flags, (2) sector concentration vs. our existing book, (3) regulatory exposure (KFTC, FSC). Output: a 1-page memo with a go/no-go recommendation."
```

**What this does:** Claude Code detects the new agents in `.claude/agents/`, routes the task through the team patterns harness generated (typically Producer-Reviewer or Expert-Pool for risk work), and returns a structured memo.

For Cursor or Codex, open the generated project in that tool and use the same realistic task. Cursor will pick up `.cursor/rules/*.mdc`; Codex will pick up `AGENTS.md`.

**Failure FAQ #4 — "The team doesn't execute / only one agent responds"**
**Cause:** `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` was set in the shell that ran Step 3 but not in the shell running Step 5 (happens when opening a new terminal).
**Fix:** Re-export in the current shell: `export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`, then re-run Step 5. To make permanent, add the line to your shell rc file.

**Failure FAQ #5 — "Too many API calls / cost anxiety"**
**Cause:** Multi-agent teams can fan out to 5+ parallel Claude calls per task. A single complex ticket can consume 50K–200K tokens.
**Fix:** Limit to a single task per run (don't chain `&&` multiple harness invocations), and use the `--max-turns` flag if your Claude Code version supports it. For production, gate harness invocations behind a cost-aware wrapper — see `docs/cost-controls.md` *(forthcoming)*.

---

## You're done

At this point you should have:

- [x] Target-specific harness entry files
- [x] Domain-specialized role files
- [x] Supporting skills or playbooks
- [x] One successful sample-task execution
- [x] For Claude target, a working `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` environment

**Next reads:**
- [`docs/experimental-dependency.md`](./experimental-dependency.md) — Why the flag, and what we'll do when it changes
- [`docs/runtime-targets.md`](./runtime-targets.md) — Target-specific file layout for Claude, Cursor, and Codex
- [`revfactory/harness-100`](https://github.com/revfactory/harness-100) — Catalog of 100+ pre-built domain harnesses, if you'd rather clone than generate
- [`revfactory/claude-code-harness`](https://github.com/revfactory/claude-code-harness) — The A/B test harness we used to measure +60% quality on 15 tasks

**If you hit something this guide didn't cover:** open an issue with the `quickstart-gap` label and include: (a) which step failed, (b) `claude --version`, (c) the exact error message. The SLA for quickstart-gap issues is **48 hours** to first response (see `CONTRIBUTING.md`).
