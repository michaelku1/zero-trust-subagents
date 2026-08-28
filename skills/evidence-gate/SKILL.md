---
name: evidence-gate
description: Use when dispatching parallel subagents for coding, refactoring, documentation, or audit work, or when about to act on a subagent's report — deleting code, merging changes, or editing docs based on claims you did not check yourself.
license: MIT
---

# Evidence Gate

## Overview

Agent contexts are trust boundaries. Everything that crosses one — your hints going down, worker claims coming up — is generated text: plausible, confident, sometimes wrong.

**Rule: generated text does not cross a trust boundary without independent verification.**

## The three rules

**1. DOWN — hints are marked UNVERIFIED.**
Never present your beliefs about the codebase as facts in a worker prompt. You cannot tell which of your beliefs are stale; the code is the only source of truth.

```text
Context (UNVERIFIED — verify against the code before using; if the code
disagrees, trust the code and report the contradiction): <your hints>
```

Never write "Known facts:", "For accuracy:", or instruct workers to "reflect" your beliefs in their output.

**2. UP — claims carry evidence status.**
Every worker prompt must mandate this report format:

```text
Do NOT report anything as fact you did not check in this session.
Tag every claim: [verified: <command/file you checked>] or [assumed].
Report anything that contradicts the provided context as a finding —
do not silently apply it or silently ignore it.
```

**3. ACT — one mechanical check before acting.**
Before deleting, merging, or editing anything based on a worker claim, run one mechanical check yourself against ground truth: repo-wide `grep -rn <symbol>`, run the import, run the tests, `ls` the path. A worker structurally cannot verify repo-wide claims — its "no callers found" covers only its slice and misses dynamic/string-based references. Claims tagged `[assumed]` are never actionable.

When one claim proves wrong, sweep all other worker outputs for the same error class.

## Rationalization table

| Excuse | Reality |
|---|---|
| "These are facts I know about this codebase" | You can't distinguish current from stale knowledge. Unlabeled beliefs get written into docs as fabricated facts (observed: "mark it deprecated" for a function with no deprecation marker in code). |
| "The worker already verified it" | Slice-local verification. "No imports found" misses string dispatch, reflection, config references. |
| "Deadline — just apply it" | The check is one grep: seconds vs shipping a breakage. |
| "All tests pass" | Whose tests, run where? Run them yourself and check they cover the changed path. |

## Red flags — STOP

- "Known facts:" (or any unmarked belief) in a dispatch prompt
- Acting on a claim tagged `[assumed]` — or not tagged at all
- Deleting or renaming a symbol without one repo-wide grep
- A worker report with no evidence tags
