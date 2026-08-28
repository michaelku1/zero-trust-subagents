# zero-trust-subagents

**Your subagents lie confidently. Verify before you act.**

## The problem

You split work across parallel subagents. Each one reports back: *"done, all tests pass."*

Reports are generated text. They sound right. Sometimes they are wrong.

Act on a wrong report, and you ship the breakage.

## The fix: 3 rules

| # | Rule | Direction | What you do |
|---|------|-----------|-------------|
| 1 | **Mark hints** | you → worker | Label every hint you pass down as `UNVERIFIED` |
| 2 | **Tag claims** | worker → you | Worker tags each claim `[verified: <command>]` or `[assumed]` |
| 3 | **Check before acting** | report → action | Run one check yourself: grep, run the import, run the tests |

That's the whole framework. One idea behind it: **subagent reports are untrusted input. Check them like you'd check user input.**

## Example

A subagent reports:

> "`validate_session()` is dead code. No callers found. Removed it."

True in its folder. But another folder calls it — through a string:

```python
ROUTES = {"GET /session/validate": "app.auth:validate_session"}
```

The worker can't see this. It only read its own slice. **No worker can verify a repo-wide claim.**

The orchestrator runs one check:

```bash
grep -rn validate_session .
```

Two seconds. Catch found. Merge saved.

## Install

As a Claude Code plugin:

```
/plugin marketplace add michaelku1/zero-trust-subagents
/plugin install zero-trust-subagents@zero-trust-subagents
```

Or copy the skill by hand:

```bash
cp -r skills/zero-trust-subagents ~/.claude/skills/   # global
cp -r skills/zero-trust-subagents .claude/skills/     # this project only
```

Not on Claude Code? Use the raw prompt text in [`templates/`](templates/). It works in any agent stack.

## How do I know it's working?

Skills load on demand — installing puts the skill's name and description in the agent's context; the rules load when the skill is invoked (by you, or by Claude when the task matches).

Three ways to check:

1. `/plugin` lists it as installed; typing `/zero-` autocompletes the skill. Note the plugin-install command is namespaced: `/zero-trust-subagents:zero-trust-subagents`. (The bare `/zero-trust-subagents` only exists if you copied the skill by hand.)
2. **Behavioral test** (30 seconds): run the canned prompt in [`tests/self-check.md`](tests/self-check.md). If the dispatch prompts come back with `UNVERIFIED` hints and the `[verified:]/[assumed]` format, the rules are in context.
3. **Hard guarantee:** add one line to your project's `CLAUDE.md` — *"Always apply zero-trust-subagents when spawning subagents."* This makes application deterministic instead of description-matched.

## Does it actually help?

We tested on live agents before writing the skill. Two findings:

1. **Rule 3 held on its own.** Under deadline pressure, the agent still grepped before deleting. Good instinct. The skill locks it in.
2. **Rule 1 is where discipline broke.** Given unlabeled beliefs, the orchestrator wrote them into worker prompts as *"Known facts"* — including a deprecation that exists nowhere in the code. The docs would have fabricated it. With the skill, the same beliefs went down marked `UNVERIFIED`. No fabrication.

Details: [`examples/caught-in-the-wild.md`](examples/caught-in-the-wild.md)

## What's in the box

| File | What it is |
|------|-----------|
| `skills/zero-trust-subagents/SKILL.md` | The Claude Code skill |
| `templates/worker-prompt.md` | Dispatch template for any agent stack |
| `templates/orchestrator-checklist.md` | The before-you-act checklist |
| `examples/caught-in-the-wild.md` | Two real catches |
| `tests/self-check.md` | 30-second test that the skill is loaded |

## License

MIT
