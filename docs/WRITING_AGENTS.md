# Agents

Every `.md` file in `agents/` ships as a subagent when the Montage plugin is installed. Drop a file in, open a PR, done — nothing else registers it.

That is literal: a `README.md` in `agents/` would be loaded as an agent and fail frontmatter validation. That is why this guide lives in `docs/` and not next to the agents.

Claude Code exposes them as `montage:<name>` in the `@`-mention list. Codex loads the same files from the same directory.

## Writing one

Start from [`agents/clip-scout.md`](../agents/clip-scout.md) — it is a real agent, not a placeholder.

```markdown
---
name: transcript-auditor
description: Use when a Montage transcript needs checking against the audio — finds dropped speakers, garbled spans, and wrong language detection. Reports; does not edit.
model: sonnet
---

You are the Transcript Auditor. ...
```

### Frontmatter

`name` and `description` are the two that matter. The rest are optional.

| Field             | Notes |
|-------------------|-------|
| `name`            | kebab-case. Must match the filename to stay findable. |
| `description`     | **This is the routing logic.** Write when to invoke it and when not to, not what it is. Weak descriptions are the #1 reason an agent never fires. |
| `model`           | `haiku`, `sonnet`, `opus`, or `fable`. Omit to inherit the session model. Pick `haiku` only for mechanical work. |
| `effort`          | `low` / `medium` / `high` / `xhigh` / `max`. Omit to inherit. |
| `maxTurns`        | Hard cap on the agent's tool loop. Worth setting for anything that polls. |
| `tools`           | Allowlist. Omit for all tools. |
| `disallowedTools` | Denylist, e.g. `Write, Edit` for a read-only reviewer. |
| `skills`          | Skills this agent may invoke. |
| `memory`          | Persistent memory slug. |
| `background`      | `true` to run detached by default. |
| `isolation`       | Only valid value is `worktree` — its own git worktree. |

`hooks`, `mcpServers`, and `permissionMode` are **rejected** in plugin-shipped agents. That is a Claude Code security rule, not a repo convention: an installed plugin cannot silently grant itself hooks or servers. Declare MCP servers in the plugin's `.mcp.json` instead.

### Body

The body is the system prompt. What has actually worked for the Montage agents:

- **Name the MCP tools by name**, in the order they should be called. `get_transcript` before picking boundaries, not "gather context."
- **State the stop condition.** An agent that proposes should be told never to create; an agent that polls should be told when to give up.
- **Ban the failure mode you expect.** "Never invent a timestamp" is worth more than three paragraphs of encouragement.
- **Say what to do when the input is bad** — empty transcript, unfinished ingestion, ambiguous project name.

Anything the agent needs to look up rather than remember goes in a `references/` file next to it, referenced from the body.

## Agent or skill?

| Use a **skill** | Use an **agent** |
|-----------------|------------------|
| A workflow the main session should run itself | Work that should happen in a separate context and report back |
| The user's next turn depends on each step | A long read-heavy pass whose intermediate output nobody needs |
| Needs to ask the user a question mid-flow | Needs to be spawnable in parallel with siblings |

Ingestion is a skill: it polls and narrates, and the user reacts to each step. Clip scouting is an agent: it reads a lot, returns a short list, and three of them can run at once.

## Before you open the PR

```bash
claude plugin validate .claude-plugin/plugin.json      # manifest + component load check
claude plugin validate .claude-plugin/marketplace.json # `validate .` resolves to this one
```

Then install the local marketplace and confirm the agent actually shows up:

```bash
claude plugin marketplace add .
claude plugin install montage@montage
```

`@montage:` in a session should list your agent. If it does not appear, the frontmatter failed to parse.
