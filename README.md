# Wisecom Coding Skill

A language-agnostic coding skill for AI agents: correct, secure, maintainable software with explicit contracts, clear ownership, and verified behavior.

## Install

Run in the repository where you want to use the skill. Requires Node.js/npm and Git.

```sh
npx skills add wisecom-oy/wisecom-code-skill --skill wisecom-coding
```

The [skills CLI](https://www.npmjs.com/package/skills) prompts for your coding agent. To target an agent without prompts:

```sh
npx skills add wisecom-oy/wisecom-code-skill --skill wisecom-coding --agent codex --yes
```

Use `--agent claude-code` for Claude Code, or `--global` to install for your user rather than one repository.

## What it does

Guides the agent through understanding the task, designing the smallest correct change, protecting behavior, implementing, refactoring, verifying, and checking production failure modes.

- Clear names, functions, control flow, modules, APIs, comments, and source layout.
- Contracts, invariants, state ownership, safe types, validation, security, and error handling.
- Transactions, concurrency, idempotency, retries, timeouts, and distributed effects.
- Testing, dependencies, performance, observability, architecture, scaling, review, and Git hygiene.
- Evidence-based completion: report checks actually run; never invent APIs or claim unobserved success.

Ask your agent: **“Use wisecom-coding to implement this change”** or **“Review this change using wisecom-coding.”** Activation depends on your agent; installing the skill does not enforce rules through hooks or CI.

## Contents

[SKILL.md](skills/wisecom-coding/SKILL.md) is the short entry point and topic index. Its `references/` directory contains the introduction and all 31 chapters of the Wisecom Coding Playbook. Detailed chapters load as needed.

This is an instruction-only [Agent Skill](https://agentskills.io/specification), not an npm package or application. No runtime dependencies, custom installer, or agent-specific configuration.

## License

[MIT](LICENSE).
