# evidence-gate

**Your subagents lie confidently. Verify before you act.**

You split a refactor across 5 subagents, one folder each. Each reports back: *"done, all tests pass."* Their reports are generated text — plausible, evidence-citing, sometimes wrong. Merge on a wrong one and you ship the breakage.

## Install

Claude Code plugin:

```
/plugin marketplace add michaelku1/evidence-gate
/plugin install evidence-gate@evidence-gate
```

Or copy the skill directly:

```bash
cp -r skills/evidence-gate .claude/skills/     # project-local
cp -r skills/evidence-gate ~/.claude/skills/   # global
```

Not using Claude Code? The same rules as raw prompt text: [`templates/`](templates/).

## The three rules

Agent contexts are trust boundaries. Generated text doesn't cross one without independent verification.

1. **DOWN** — every hint you pass a worker is marked `UNVERIFIED — check against the code`. Your beliefs about the codebase may be stale, and you can't tell which ones; don't let workers echo them back as fact.
2. **UP** — workers must tag each claim `[verified: <command they ran>]` or `[assumed]`.
3. **ACT** — before merging, deleting, or editing anything a report told you, run one mechanical check yourself: repo-wide grep, run the import, run the tests.

## Why: a worker can't verify a repo-wide claim

A subagent reports: *"`validate_session()` is dead code — no callers found — removed it."* True inside its assigned folder. But the function is called from a route table in **another worker's slice**, via a dotted string that import-analysis never sees:

```python
ROUTES = {"GET /session/validate": "app.auth:validate_session"}
```

Each worker sees only its own context, so *no worker can ever verify a repo-wide claim*. One `grep -rn validate_session .` by the orchestrator (2 seconds) catches it before the merge does.

## Pressure-tested, not vibes

We baseline-tested these rules on live agents before writing the skill ([details](examples/caught-in-the-wild.md)):

- The **ACT** rule held even without the skill in our runs — the agent grepped before deleting under deadline pressure. Good; the skill locks that in.
- The **DOWN** rule is where discipline evaporated: given unlabeled beliefs, the orchestrator wrote *"Known facts to reflect accurately: validate_session() is deprecated — mark it deprecated"* into worker prompts. The code has no deprecation marker — the docs would have fabricated one. With the skill, the same input was demoted to UNVERIFIED hints and the fabrication vector disappeared.

The asymmetry is the whole trick: generating work is expensive, checking a claim is cheap and mechanical. Subagent reports are untrusted input crossing a trust boundary — taint that only clears with a check.

## What's in the box

```
skills/evidence-gate/SKILL.md      # the Claude Code skill (the deliverable)
templates/worker-prompt.md         # fill-in dispatch template, any agent stack
templates/orchestrator-checklist.md# the ACT-line checks
examples/caught-in-the-wild.md     # real catches from testing this skill
```

## License

MIT
