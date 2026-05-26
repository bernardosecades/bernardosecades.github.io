---
title: "How a per-team plugin marketplace stopped Claude skill duplication across teams"
date: 2026-05-25 00:00:00
lang: en
ref: claude-code-team-marketplace
tags: [claude-code, plugins, marketplace, workflow]
read_min: 5
excerpt_text: "Three teams, three incompatible ways of sharing Claude skills, zero reuse. A per-team plugin marketplace fixed it — here's the layout, the template, and what it actually changed."
---

Three engineering teams were sharing Claude skills in three incompatible ways and nobody was reusing anyone else's work. Same skills written twice, the good ones invisible to the people who needed them, and an `~/.claude/` folder per developer drifting from everyone else's.

A per-team plugin marketplace, organised the way teams already work, fixed it. This post documents the layout, the template you can fork today, and what concretely changed once it was in place.

## Three incompatible solutions, no marketplace

Before any of this was centralised, I watched three patterns coexist on the same engineering org:

- **A repo for skills only.** One team created a Git repo with nothing but a `skills/` folder and a README telling people to symlink it under `~/.claude/`. It worked for that team — and nobody else ever installed it.
- **Skills inside the service repo.** Another team committed their skills into the same repo as the service they owned. The skill only loaded when someone happened to be working *inside* that repo. Useful, but locked to one cwd.
- **Personal copies in `~/.claude/`.** A third pair of engineers kept theirs personal and shared by pasting `SKILL.md` files into Slack threads. Two weeks later three slightly different versions of the same skill were in flight.

Three solutions, none of them composable. The skills were good — the distribution was a mess. The marketplace idea showed up exactly here: a single place the org installs once, with everything organised so it doesn't collapse into the junk drawer it replaces.

## The marketplace is just one repo

A Claude Code plugin marketplace is a Git repo with a `.claude-plugin/marketplace.json` at the root. That file lists every plugin in the repo, each plugin is a folder, and folders contain skills / agents / commands / hooks. Once someone publishes the repo, anyone in the org runs:

```bash
claude plugin marketplace add yourorg/ai-marketplace
claude plugin install <plugin-name>@<marketplace-name>
```

That's the whole interface. The interesting decision is what goes inside the repo.

## The trap: one big plugin

The first instinct is to make one plugin called `engineering` and pile everything in. Six months in:

- Two teams have written subtly different `code-review` skills.
- A skill the backend team needs every day is mixed with twenty Android-only commands.
- Every PR touches the same folder, every PR has merge conflicts.
- Nobody wants to delete anyone else's thing, so nothing ever gets deleted.

The plugin grows into a junk drawer that everyone is afraid to clean.

## The layout that works: one folder per team

Give each team its own plugin folder. The team owns it the way they own a microservice — naming, contents, lifecycle.

```text
ai-marketplace/
├── .claude-plugin/
│   └── marketplace.json
└── plugins/
    ├── orders/         ← orders team owns this folder
    │   ├── .claude-plugin/plugin.json
    │   ├── skills/
    │   ├── agents/
    │   ├── commands/
    │   └── hooks/
    ├── billing/        ← billing team owns this folder
    │   └── …
    ├── notifier/       ← notifier team owns this folder
    │   └── …
    └── common/         ← cross-team plugin (more on this below)
        └── …
```

Three things fall out of this layout almost for free:

- **No merge friction.** Two teams editing two folders never conflict.
- **CODEOWNERS just works.** `plugins/orders/* @org/orders-team` and reviews route themselves.
- **Install is à la carte.** A frontend engineer can install `orders@ai-marketplace` and skip the rest. They don't have to opt into another team's hooks.

The plugin name in `plugin.json` should match the folder (`name: "orders"`). The folder becomes the unit of ownership, the install handle, and the discovery surface — three things lined up on one boundary.

## Trying another team's plugin is the point

The reason a team marketplace beats a personal pile of scripts isn't speed. It's that **you can see what other teams figured out** without asking them.

I install `notifier@ai-marketplace` for a week. They have a [subagent](/claude-code-subagents/) that scans Slack-style templates for variables that never got resolved. I don't have notifications in my service, but the *technique* — a read-only subagent that walks a folder and reports missing references — is exactly what I need for my SQL fixtures.

I steal the pattern, write it for my own context, ship it in `plugins/orders/`. Two months later somebody on `billing` does the same thing for their config files. Now three teams have a variant of the same idea.

That's the signal to promote.

## The `common` plugin (call it whatever you want)

When the same shape of skill, agent, or command shows up in three team plugins, it stops being a team thing. Pull it out into a cross-team plugin and let everyone install it:

```text
plugins/common/
├── skills/
│   ├── secrets-no-commit/      ← security rule everyone wants
│   ├── pr-checklist/           ← review hygiene everyone wants
│   └── unresolved-references/  ← the technique that emerged in three places
└── hooks/
    └── hooks.json              ← block commits to main, warn on .env, etc.
```

Naming is a religious war. `common`, `shared`, `org`, `misc`, `platform` — I've seen all of them. Pick one and write a README sentence explaining what belongs in it. The rule I use:

> A plugin goes in `common` when at least two teams use it **and** removing it would feel wrong to all of them.

Security rules and review checklists almost always end up here. They were never going to belong to one team.

## Promotion is the only governance you need

Most marketplace governance docs read like a procurement policy. You don't need any of that. The flow is:

1. **Experiment** inside your team's plugin. Nobody else has to care.
2. **Use other teams' plugins** for a week. Notice what you reach for.
3. **Promote** to `common` only when the same idea has surfaced in two or three teams independently.

No central committee, no naming bikeshed up front, no "request to add a plugin" form. Teams are the unit, `common` is the after-the-fact synthesis. The marketplace ends up looking like the org chart, which is exactly what you want — because that's how reviews, ownership, and on-call already work.

## Updates ride the same channel

The marketplace doesn't just centralise *where* plugins live — it centralises *how they change*. Same channel for install and for every update after it.

Pre-marketplace, an update to a skill meant: edit your local copy, paste the new `SKILL.md` into a Slack thread, hope the other teams pull it down. Two weeks later three drifted versions were in flight. The good fix didn't propagate; the broken one didn't either.

With one repo and team-owned folders, every plugin declares a `version` in its `plugin.json`. Bumping `plugins/orders/.claude-plugin/plugin.json` from `0.4.0` to `0.5.0` and pushing to main is the only step. Every installer of `orders@ai-marketplace` picks up the change the next time they refresh the marketplace.

- **One update, every team.** A fix to `common/skills/secrets-no-commit/SKILL.md` lands in every developer's environment on their next marketplace refresh. No more "did you pull the latest version?" Slack thread.
- **Rollback is a pinned version, not a frantic cleanup.** A team ships a broken hook in `common/hooks.json`? Pin to the previous version of the plugin until they fix it. Pre-marketplace, rollback was "everyone manually delete the bad file from `~/.claude/`".
- **New hires start at parity.** Onboarding becomes one `claude plugin marketplace add yourorg/ai-marketplace`, plus the team plugins they want. Five seconds in, the new engineer is at the same baseline as someone who's been here two years — no `~/.claude/` zip handoff, no Slack archaeology.
- **The changelog is git history.** Want to know what changed in `plugins/billing/` last month? `git log` on that folder. Pre-marketplace, the "history of org Claude standards" lived in personal repos, Slack threads, and people's memories — partial, distributed, lossy.

The folder-per-team layout was about *who owns what*. The version + update channel is about *how change moves*. Owning your folder is wasted if updates still have to be pasted into Slack.

## A template you can fork today

If you want to try the layout without designing it from scratch, I open-sourced a minimal template at [github.com/bernardosecades/ai-marketplace](https://github.com/bernardosecades/ai-marketplace). It ships with:

- `.claude-plugin/marketplace.json` already wired.
- A complete `plugins/example/` showing every primitive — skill, agent, command, hook.
- An empty `plugins/my-team/` skeleton ready to copy as `plugins/<your-team>/`.

Fork it, rename the team folders, push it to your org, run `claude plugin marketplace add yourorg/ai-marketplace`. The folder-per-team layout above is what the template already ships with — this post is the *why*, the repo is the *how*.

## Impact

Two to three months in, what changed in concrete terms:

- **One install instead of three.** Before: each team distributed differently and nobody installed anyone else's work. After: every engineer runs `claude plugin marketplace add` once and picks plugins à la carte.
- **Updates propagate instead of drift.** A fix to `common/` lands in every developer's next marketplace refresh — two to three months in, zero version-drift incidents across the four team plugins.
- **Reuse where there was zero.** The pattern from one team (unresolved-template-variables) became the seed for two others, and ultimately a `common/` plugin every team now uses.
- **Cross-team conventions, finally enforceable.** Secret-leak prevention and PR review hygiene moved from "we should all do this" to a single `common/` hooks bundle every developer pulls down.
- **Bottom-up beats committee.** Zero meetings to decide what is "official". Patterns prove themselves in two or three team plugins first, then get promoted by code review.

## Technical decisions

- **Per-team folder, not one big plugin.** The big-plugin variant concentrates conflicts on a shared folder and nobody dares delete anyone else's content. One folder per team gives CODEOWNERS something to point at and à la carte installs.
- **`common` lives inside the same marketplace, not a separate one.** A second marketplace turns promotion into a publish-then-install dance. Same repo means promotion is a folder move plus a PR.
- **Promotion is bottom-up, not top-down.** A central committee picking "standards" produces a junk drawer of approved-but-unused plugins. Waiting for the same shape to surface in two or three team plugins keeps `common` short and earned.

## Real limitations

- **Plugins still sit at the bottom of the [priority order](/claude-code-skill-priority/).** A personal skill in `~/.claude/skills/` silently wins over a marketplace plugin with the same name. Pick distinctive plugin names — `orders-review`, not `review`.
- **No per-plugin permission scoping.** Once you install a plugin you trust every hook in it. `common/hooks.json` has org-wide blast radius and deserves the same review rigor as infrastructure code.
- **Folder ownership is not runtime isolation.** CODEOWNERS routes code review, but nothing prevents a member of one team from installing and running another team's hooks. The boundary is social and review-time, not enforced by the tool.
- I would not run third-party hooks from outside the org without a manual audit. The plugin format makes drive-by trust very easy.

## Why this matters

A plugin you wrote alone is a personal trick. A plugin your team owns is a convention. A plugin in `common` is a standard.

The folder layout decides which of the three a piece of Claude knowledge gets to become. One folder per team is the cheapest structure that lets all three exist at the same time — and lets a good idea travel from "personal trick" to "standard" without anyone organising a meeting.
