---
title: "MCP: when it's worth it, when the CLI wins (the GitHub case)"
date: 2026-06-23 00:00:00
lang: en
ref: mcp-when-to-use
tags: [claude-code, mcp, tooling, workflow]
read_min: 4
excerpt_text: "The GitHub MCP ships around forty tools you mostly won't touch. If you have a shell and `gh`, the CLI does the same job with one tool instead of forty. MCP still earns its place — just not here. Here's the decision rule."
---

You install the GitHub MCP because it feels like the *proper* way to give Claude access to GitHub — typed tools, an official server, the protocol everyone's talking about. A week later you're staring at it wondering whether it's actually pulling its weight, or whether `gh pr list` would have done the same thing without the ceremony.

Honest answer, for GitHub specifically: the CLI usually wins. But MCP genuinely earns its place in other cases. Here's the trade-off and the rule I landed on.

## What an MCP actually buys you

- **A typed tool interface, reused across clients.** The same server works in Claude Code, Cursor, the web app. Define it once, use it everywhere.
- **Structured output.** Tools return JSON-shaped data, not free text you have to parse and hope the format didn't change.
- **Auth handled server-side.** Tokens live in the server config, not in every command you run.
- **Per-tool permission gating.** Each tool is addressable as `mcp__<server>__<tool>`, so you can allow the read side and deny the write side — the [permission-tuning post](/claude-code-permission-tuning/) goes deep on this.
- **Reach into things that have no good CLI.** Notion, Jira, Sentry, an internal SaaS with a web API and nothing else. This is the real win.

## What it costs

- **Tool-definition surface.** A single server can ship dozens of tools. The GitHub MCP exposes **around forty** (`mcp__github__*`). That's selection complexity for the model and permission surface for you, whether or not you ever call them.
- **Context, on clients that load schemas up front.** Forty tool definitions aren't free. Modern Claude Code *defers* them (more on that below), but not every client does.
- **Another process to keep alive.** A server that can hang, crash, or drift out of date.
- **Blast radius.** Write tools are real: `merge_pull_request`, `delete_file`, `create_or_update_file`. The convenient ones are also the dangerous ones.
- **Latency and a layer that can break** between you and the thing you're actually trying to do.

## The GitHub case

The GitHub MCP exposes ~40 tools. Modern Claude Code **defers** their schemas — it loads them on demand via tool-search instead of dumping all forty into context at startup. So the classic "MCP bloats your context window" critique is real, but version-dependent. Either way, you rarely need forty GitHub tools in a session.

And if you have a shell with `gh` authenticated, one Bash tool covers nearly all of it — and the model already knows `gh` cold:

```bash
gh pr list
gh pr view 123
gh pr create --fill
gh issue list --label bug
gh api repos/:owner/:repo/commits
```

Side by side, the same operation:

```text
MCP:  mcp__github__list_pull_requests   ← 1 of ~40 tools, needs the server running
CLI:  gh pr list --json number,title,state  ← 1 Bash call, structured JSON, no server
```

Both give you structured data. The CLI does it with one tool the model already has, zero extra schemas, and nothing to keep running.

So for GitHub, reach for the MCP only when:

1. **You're on a shell-less client** — claude.ai on the web, a desktop integration — where `gh` isn't an option.
2. **You want per-tool permission gating** — allow `mcp__github__get_file_contents`, deny `mcp__github__merge_pull_request`. That's cleaner than a Bash string pattern, which isn't a shell AST and can't reliably tell a safe `git` invocation from a dangerous one.
3. **You want guaranteed structure** without remembering the right `--json` fields.

Outside those three, `gh` via Bash is the lighter tool.

## The decision rule

**Use an MCP when:**

- the capability has no mature CLI (Notion, Jira, Sentry, an internal API);
- you're on a client with no shell;
- you want typed output *and* per-tool permission control;
- you'll reuse the same server across multiple clients.

**Prefer the CLI / skip the MCP when:**

- a mature CLI already exists (`gh`, `aws`, `kubectl`, `psql`) **and** you have a shell;
- the server exposes a big surface you'll use 5% of;
- token budget matters and your client doesn't defer schemas.

## Impact

- Dropped the GitHub MCP on my shell-based setup. `gh` via Bash covers essentially all of it — fewer tools in play, fewer write tools to gate, one less server to babysit.
- Kept MCP for the SaaS in my stack that has no CLI at all. That's exactly where it buys something the shell can't.

## Decisions

- **Default to the CLI when a good one exists and I have a shell.** It's the smaller, more legible tool.
- **MCP for shell-less clients and for per-tool permission gating.** Those are the cases the CLI genuinely can't cover.
- **Gate write tools with deny rules regardless of which path I'm on** — see the [permission-tuning post](/claude-code-permission-tuning/).

## Limitations

- **Deferred tool loading is version- and client-dependent.** On clients that load every schema at startup, the token argument against big MCP servers is much stronger.
- **`gh` output is free text by default** — you reach for `--json`/`--jq` to get structure, whereas an MCP tool hands it to you typed.
- **Per-tool deny is cleaner with MCP than with Bash.** `deny mcp__github__merge_pull_request` is exact; a Bash pattern is a string match, not an understanding of the command.
- **The rule flips entirely for capabilities with no CLI.** There, "skip the MCP" isn't on the table — MCP *is* the access path, and the trade-off becomes which tools to enable, not whether to.
