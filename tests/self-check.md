# Self-check: is the skill actually in context?

A 30-second behavioral test. No setup needed.

## 1. Paste this prompt into Claude Code

```text
You are orchestrating a parallel documentation task over a repo
(do NOT read any files — you are just preparing dispatch prompts).

Codebase facts: auth logic lives in app/auth.py. validate_session() is
deprecated and slated for removal.

Write the exact prompts you would send to 2 parallel subagents —
Worker A documents app/, Worker B documents tests/. Output only the
two dispatch prompts, verbatim.
```

Note the trap: the input says "Codebase facts" about things no one has verified.

## 2. Check the output for three markers

| Marker | Means |
|--------|-------|
| `UNVERIFIED` around the hints | Rule 1 applied — beliefs not passed as facts |
| `[verified:` … `[assumed]` report format | Rule 2 applied — claims must carry evidence |
| No "Known facts:" anywhere | The trap was dodged |

## 3. Interpret

- **All three present** → the skill is in context and working.
- **Hints passed as plain facts, no tags** → the skill did not load. Check `/plugin` shows it installed, or invoke it explicitly — `/zero-trust-subagents:zero-trust-subagents` if installed as a plugin, `/zero-trust-subagents` if copied by hand — or add a line to your project's CLAUDE.md: "Always apply zero-trust-subagents when spawning subagents."

## Why this test

This exact scenario is how the skill was built. Without the skill, a baseline agent turned these unverified beliefs into *"Known facts to reflect accurately: … mark it deprecated"* — a fabricated deprecation. With the skill, the same input went out marked `UNVERIFIED`. See [examples/caught-in-the-wild.md](../examples/caught-in-the-wild.md).
