---
title: "Claude Code: permission rules — wildcards, allow/deny, priority, and plugins"
date: 2026-06-02 00:00:00
lang: en
ref: claude-code-permission-tuning
tags: [claude-code, permissions, settings, plugins, workflow]
read_min: 6
excerpt_text: "Approving Bash(ls) forty times a day numbs you into mashing Enter. Allowing Bash(*) wakes you up one Tuesday to a wiped repo. The fix is wildcards at the right granularity — and the realisation that permissions cover more than Bash."
---

The [previous post on scope vs permissions](/claude-code-scope-vs-permissions/) split the world into "what Claude can see" and "what Claude can do". This one is about the second half — how to tune those rules so they protect you without burying you in dialogs.

Two failure modes. Both common, both painful, neither obvious to a beginner until they've been bitten by one.

**Too closed.** You approve `Bash(ls)` forty times a day. Every prompt halts on a permission dialog. After two days you're hitting Enter without reading what you're approving — and that's exactly how a real destructive command slips through, the day Claude builds something wrong.

**Too open.** You set `Bash(*)` to make the friction stop. Three weeks later, on a Tuesday at 18:42, Claude composes a command incorrectly during a refactor and runs `rm -rf` against a path you cared about. There was no second pair of eyes because you had explicitly removed the second pair of eyes.

The fix is not "more approvals" or "fewer approvals". It's **wildcards at the right granularity**, **deny rules that catch the obvious traps**, and the realisation that **permissions are not only about `Bash`**.

## Wildcards are the dial, not the switch

A permission rule has the shape `Tool(pattern)`. The pattern accepts `*`. The dial goes from "useless precision" to "irresponsible openness", and you pick where to sit.

```text
Bash(ls)              ← matches exactly the literal "ls"
                        useless: ls /tmp won't match
Bash(ls *)            ← ls with any args. Most of what you want.
Bash(git *)           ← any git subcommand. Wide but safe-ish.
Bash(git status:*)    ← git status with any args. Same intent,
                        canonical Claude Code shape with the colon.
Bash(*)               ← everything. Don't.
```

The trap people fall into: they start with `Bash(ls)` (too narrow, useless), give up and jump to `Bash(*)` (too wide, dangerous). The middle is what works.

A reasonable user-level baseline I run in `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(ls *)",
      "Bash(cat *)",
      "Bash(grep *)",
      "Bash(find *)",
      "Bash(cd *)",
      "Bash(git status)",
      "Bash(git diff)",
      "Bash(git log)",
      "Bash(npm test)",
      "Bash(npm run lint)"
    ]
  }
}
```

Read-only or idempotent commands go in. Anything that mutates state stays out and goes through the approval flow. Everything else lives at project level where the team can shape it together.

## /fewer-permission-prompts — bootstrap from your own history

Writing this from scratch is dull. Claude Code ships a skill that does it for you:

```text
/fewer-permission-prompts
```

It scans your session transcripts, finds the commands you approved repeatedly, and proposes an allowlist sorted by frequency. You accept or skip each one; the chosen rules are appended to `.claude/settings.json`.

You typically run it once after a couple of weeks of normal use, then every month or so to catch new patterns. It will not propose `Bash(*)` no matter how often you approved it — it favours specific patterns over the wide ones, by design.

It does have a blind spot: it suggests allow rules, never deny rules. Tightening the dangerous side is on you.

## Wildcards work both ways — deny is the other half

The same wildcard syntax applies to `deny`. And **`deny` always wins over `allow`**, so it is your hard guardrail for things that should never happen unattended:

```json
{
  "permissions": {
    "deny": [
      "Bash(sudo *)",
      "Bash(rm -rf /*)",
      "Bash(chown *)",
      "Bash(chmod *)",
      "Bash(systemctl *)",
      "Bash(curl * | sh)",
      "Bash(curl * | bash)",
      "Read(./**/.env*)",
      "Read(./**/secrets.*)",
      "Edit(./prod/**)",
      "Edit(./infra/**/*.tf)"
    ]
  }
}
```

What people forget when they write `allow`-only configs: there is no rule that says `Bash(git *)` excludes `Bash(rm -rf:*)`. If Claude builds an awkward command, the wide allow doesn't catch it. The deny does.

Symmetric mental model: `allow` removes the friction you trust yourself with; `deny` removes the autonomy you don't trust yourself with at 2 am.

## Priority — when rules collide

Permissions stack from four sources, in this order:

1. **Enterprise managed** — pushed centrally by your org. Highest priority.
2. **Personal** — `~/.claude/settings.json`.
3. **Project (shared)** — `<repo>/.claude/settings.json`, committed.
4. **Project (local)** — `<repo>/.claude/settings.local.json`, gitignored.

Two rules of resolution:

- A `deny` anywhere in the stack **always wins** over any `allow`.
- For two rules that match the same operation, the **more specific pattern wins**: `Bash(git push origin main)` beats `Bash(git push:*)` beats `Bash(git *)`.

You can use this. A team can ship a project-level `deny` for production paths that survives any personal `allow`. An individual can ship a personal-level `deny` that no project config can undo.

If a permission seems ignored, the cause is almost always one of these two rules — usually a wider `deny` somewhere in the stack you'd forgotten about.

## It is not only Bash

This is the part most people miss. Allow/deny rules apply to **every tool Claude can invoke**, not just `Bash`. The grammar is `Tool(pattern)`:

- `Read(./.env)` — block reading a specific file.
- `Read(./secrets/**)` — block reading an entire directory tree.
- `Edit(./prod/**)` — block edits to production paths.
- `Write(./vendor/**)` — block writes (e.g., to a Go `vendor/` directory).
- `WebFetch(domain:internal.company.tld)` — restrict outbound fetches.
- `mcp__github__get_file_contents` — allow a specific MCP tool.
- `mcp__github__merge_pull_request` — leave the dangerous MCP tool in the approval flow.

The MCP shape is `mcp__<server>__<tool>`, no parentheses. That's how you give Claude read access to GitHub through the MCP but keep the write side — issues, PRs, releases — gated.

A practical, mixed allowlist might look like:

```json
{
  "permissions": {
    "allow": [
      "Bash(git *)",
      "Bash(go test:*)",
      "Bash(ls *)",
      "Read(./**)",
      "mcp__github__get_file_contents",
      "mcp__github__search_repositories"
    ],
    "deny": [
      "Bash(rm -rf /*)",
      "Edit(./prod/**)",
      "Read(./**/.env*)",
      "mcp__github__merge_pull_request"
    ]
  }
}
```

Same dial, different tools. The mental model from `Bash` carries over to all of them.

## Plugins and their skills — the layer everyone ignores

Plugins ship as a bundle: skills, agents, slash commands, sometimes MCP servers. Installing one is "load all of the above". Most people install a plugin once and forget it has surface area.

There are two levers.

**Lever 1 — disable the whole plugin.** `settings.json` has an `enabledPlugins` map:

```json
{
  "enabledPlugins": {
    "slack@claude-plugins-official": false,
    "gopls-lsp@claude-plugins-official": true,
    "marketplace-internal@yourcompany-plugins": true
  }
}
```

Setting a plugin to `false` removes everything it contributes — skills, agents, the lot — without uninstalling it. Handy when you only want a plugin's skills available in some projects and not others (a personal `false`, then `true` in the project's `.claude/settings.local.json`).

**Lever 2 — keep the plugin, deny specific pieces.** This is the one people miss. The plugin loads, but you opt out of individual surface. Plugin-namespaced skills appear as `plugin:skill-name` in the harness; the same shape goes in a deny rule:

```json
{
  "permissions": {
    "deny": [
      "Skill(marketplace-internal:risky-deploy)",
      "Skill(marketplace-internal:db-restore)"
    ]
  }
}
```

Same idea for the agents and MCP tools a plugin brings along — they all sit behind the same `Tool(pattern)` syntax you already know.

Why this matters: when you install a marketplace plugin, you are accepting whatever skills the maintainer ships, *and* whatever skills they push in the next update. A surgical deny lets you keep the 90% of the plugin you want and stop the 10% you don't want to authorise on autopilot. That conversation never gets had if you only think about `Bash`.

A close cousin: if you collide on names between a personal skill and a plugin skill, [the priority order in this post](/claude-code-skill-priority/) decides which one runs. Personal beats project beats plugins — but enterprise beats all three.

## A worked example

I work across ~25 Go microservices. Mixed wildcards, an MCP, a plugin in the stack. The relevant slice of `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(git *)",
      "Bash(go *)",
      "Bash(ls *)",
      "Bash(cat *)",
      "Bash(grep *)",
      "Bash(find *)",
      "Read(./**)",
      "mcp__github__get_file_contents",
      "mcp__github__search_repositories"
    ],
    "deny": [
      "Bash(rm -rf /*)",
      "Bash(sudo *)",
      "Bash(curl * | sh)",
      "Read(./**/.env*)",
      "Edit(./infra/**/*.tf)",
      "Skill(marketplace-internal:db-restore)"
    ]
  },
  "enabledPlugins": {
    "slack@claude-plugins-official": false,
    "marketplace-internal@yourcompany-plugins": true
  }
}
```

Read: any file, any path. Write/edit: no Terraform under `./infra/`, ever. Bash: anything starting with `git`, `go`, `ls`, `cat`, `grep`, `find` runs without asking; anything starting with `sudo`, `rm -rf /`, or a curl-pipe-to-shell is hard-blocked. GitHub MCP: read side allowed, write side falls through to the approval flow. Two skills from the internal marketplace plugin are blacklisted but the rest of the plugin still works.

A week of work with this setup: about three or four prompts a day go to approval — the new patterns I haven't seen before — and none of them are `ls`.

## Impact

- Permission prompts went from ~40/day to ~3/day after the first wave of `/fewer-permission-prompts` + a hand-curated deny block. The 3/day is what I want: it forces me to read what I'm about to approve.
- Two near-incidents prevented in the first month after adding `Bash(rm -rf /*)` and `Edit(./prod/**)` to deny — Claude proposed both during refactors, the hard block stopped them.
- One plugin update changed a skill's behaviour silently; the per-skill deny I had on the risky one kept it inert until I had time to read the new version.

## Technical decisions

- **Wildcards over per-command rules.** Maintaining `Bash(git status)`, `Bash(git diff)`, `Bash(git log)`, `Bash(git checkout *)`, etc. is unsustainable. `Bash(git *)` plus a tight deny block on the dangerous half is what I converged on.
- **/fewer-permission-prompts to bootstrap, never as the source of truth.** It seeds the allow list. The deny list is hand-written: the skill never proposes those.
- **enabledPlugins for off-by-default, deny for surgical opt-out.** Disabling a whole plugin is the right call when you don't trust the publisher; denying specific skills/tools is the right call when you trust the plugin overall but not one or two pieces of it. Both levers exist for a reason — don't only use the first one.
- **Project-local for risky paths, user-level for personal habit.** The deny on `./prod/**` lives in the project's committed settings so it applies to every teammate. The allow on `Bash(go *)` lives in my personal config; not everyone wants it.

## Real limitations

- **Wildcards in `Bash()` are string patterns, not shell ASTs.** `Bash(git *)` does not understand that `git diff | xargs rm` is dangerous — the prefix matches. The deny block on `rm -rf` catches it; the allow alone would not.
- **`/fewer-permission-prompts` does not know what your plugins expose.** It only learns from `Bash` history. Skills, agents, and MCP tools you've approved show up as "approved" but the skill won't synthesise an allow for them. You add those by hand.
- **A new plugin update can ship new skills.** Your deny list only covers what existed when you wrote it. The first turn after a plugin update is the turn to actually read the changelog instead of mashing Enter.
- **No "preview the policy" command.** There's no `claude settings explain Bash(rm -rf X)` that tells you which layer would win. When something feels wrong, you grep the four `settings.json` files by hand.

The dial is a habit, not a one-time config. Treat the file like a `.bashrc`: edit it as you learn what your real day looks like, not what your first day told you it would look like.
