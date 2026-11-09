---
title: "Fix the harness, not the prompt: where each rule should live so your agent follows it"
date: 2027-01-05 00:00:00
lang: en
ref: harness-not-prompt
tags: [harness, hooks, claude-md, workflow]
read_min: 5
excerpt_text: "I had a rule in CLAUDE.md: don't hand-edit vendor/. The agent followed it almost every time. On the rare occasions it didn't, the patch passed the tests and vanished on the next go mod vendor. Rewording the rule did nothing. Moving it to a different part of the harness fixed it."
---

My team's Go services vendor their internal libraries. When a change touches `auditlib`, the right move is to edit the `auditlib` repo, point the `go.mod` in `orders` at that local copy with a `replace`, and regenerate with `go mod vendor`. I had a line in CLAUDE.md saying exactly that: don't hand-edit `vendor/`.

The agent followed it almost every time. On the few occasions it didn't, it patched `vendor/example.com/auditlib/...` directly, the tests passed and the change looked fine. Then someone ran `go mod vendor` again and the patch silently disappeared. The bug came back and nobody could tell why, because the code that fixed it no longer existed anywhere.

My first reaction was the usual one. I reworded the rule, put it in capitals and added an "IMPORTANT". Nothing changed. That's when I started to suspect the place I had put the rule.

## What the harness is

The model does the reasoning. The harness is everything around it: the loop that makes it call tools, the tools themselves, the instructions it loads, the permissions, the hooks, and what the agent sees when something fails. Part of it ships with Claude Code. The rest you build yourself, and that part shapes how the agent behaves in your codebase more than anything else.

So when the agent gets something wrong, I now ask which part of the harness let it through before I think about how to ask more nicely.

## The ladder

Each place you can put a rule gives a different guarantee at a different cost. I order them from soft to hard:

1. **CLAUDE.md.** Always in context and cheap to write. It's also one instruction among many, and the model weighs it against everything else.
2. **Skill.** A procedure that loads when the task matches its description. Good for the how-to, with steps and examples. It's still text the model can ignore, and with a weak description [it doesn't even load](/claude-code-skill-priority/).
3. **Feedback loop.** A test, the linter or the build fails and the agent sees the error. The signal is deterministic, though you still rely on the agent reacting well to it.
4. **Hook or permissions.** The action is blocked before it runs. The agent can't get around it however hard it tries.

The rule I follow is simple. If a rule gets broken more than once, it moves up a rung.

The `vendor/` rule climbed almost all of them. It started in CLAUDE.md. When I saw it slip, I moved it into a skill with the full procedure (sibling repo, `replace`, `go mod vendor`), and it slipped less often. Less often still isn't never, though, and a mistake that erases itself on the next `go mod vendor` is one of the most expensive kinds to chase. So it went up to a hook.

## The hook

It's a `PreToolUse` hook on file writes. If the path lands inside a `vendor/` directory with a `go.mod` next to it, the write is blocked:

```bash
#!/usr/bin/env bash
set -euo pipefail

file=$(jq -r '.tool_input.file_path // empty')
[[ "$file" == */vendor/* ]] || exit 0

root="${file%%/vendor/*}"
[[ -f "$root/go.mod" ]] || exit 0

cat >&2 <<EOF
Blocked: $file is inside a Go vendor/ directory. Manual edits there are lost on the next 'go mod vendor'.
Do this instead:
  1. Edit the source in the sibling repo (usually ../<lib>, next to $root).
  2. Point go.mod at it with: replace <module> => ../<lib>
  3. Run 'go mod vendor' in $root.
See the go-internal-deps skill for the full procedure.
EOF
exit 2
```

And in `~/.claude/settings.json`:

```json
"hooks": {
  "PreToolUse": [
    {
      "matcher": "Edit|Write|MultiEdit",
      "hooks": [{ "type": "command", "command": "bash ~/.claude/hooks/block-vendor-edits.sh" }]
    }
  ]
}
```

The `go.mod` check is there so a PHP project's `vendor/` isn't caught by mistake. And since I only watch Edit and Write, `go mod vendor` keeps working, which is exactly what I want the agent to run. If you've never set up a hook, I covered the basics with a notification hook in [this post](/claude-code-hooks-notification/).

## Three things I learned building it

### The error message is a prompt too

With `exit 2`, whatever the hook writes to stderr reaches the model as the tool's error. If all it says is "denied", the agent starts trying alternatives blindly, and one of them may be worse than what you blocked. That's why mine spells out what to do instead, step by step. I tried it in a real session: the Write into `vendor/` failed and the full message came through, three steps included. The agent doesn't have to guess the right path.

### Give it a command that defines "done"

Rung 3 only works if the agent knows what to run. My CLAUDE.md has a few lines asking it to always run the tests (`make test-unit`, `make test-integration`) and `make lint`. Without that, the agent decides on its own when it's finished, and it tends to decide early. With it, a lint failure usually goes back into the loop and gets fixed before I ever see it. It's the same idea as in [the post about the bug that nearly reached prod](/the-day-ai-nearly-shipped-a-bug-to-prod/): some check has to come from outside the model itself.

### Take out of CLAUDE.md what's already enforced

When a rule moves up a rung, I remove it from the rung below. The `vendor/` ban isn't in CLAUDE.md anymore. It lives in the skill as a procedure and in the hook as a block. Every line in CLAUDE.md competes for the model's attention with the others, and a rule a hook already enforces only takes weight away from the ones that still depend on the model reading them.

## Impact

- **A rare but expensive mistake can no longer happen the usual way.** I used to find out when the bug came back. Now the agent gets the block in the same turn, along with the right steps.
- **CLAUDE.md got shorter.** What's left are real guidelines, with no rules some other piece already enforces.
- **The hook is new.** I don't have "after" numbers worth sharing yet, and I'd rather not make them up.

## Decisions

- **User-level hook, not project-level.** I work with worktrees and sometimes open Claude inside a single repo. A workspace hook wouldn't cover those sessions.
- **Check for `go.mod` next to `vendor/`.** Blocking any path with `vendor/` in it was simpler, but it broke other languages.
- **Don't parse Bash commands.** Catching `sed -i` or `cp` into `vendor/` without blocking `go mod vendor` is fragile, and I wanted a hook small enough to read at a glance.
- **Keep the skill.** The hook says what not to do and the skill explains how to do it right. I need both.

## Limitations

- **Bash can get around it.** A `sed -i` into `vendor/` goes straight through. It's a known gap.
- **Every hook is code to maintain**, and it adds some latency to every write. Not every rule deserves one. Only the ones that break and cost you.
- **A block with no explanation teaches very little.** With a poor message, the agent would know what not to do but not what to do instead.
