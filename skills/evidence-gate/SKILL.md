---
name: evidence-gate
description: Use when dispatching parallel subagents for coding, refactoring, documentation, or audit work, or when about to act on a subagent's report — deleting code, merging changes, or editing docs based on claims you did not check yourself.
license: MIT
---

# Evidence Gate

## Overview

Subagents run in separate contexts. Everything they send you is generated text. It sounds right. It can be wrong.

**Rule: generated text never crosses between contexts without a check.**

## The three rules

### 1. Mark hints (you → worker)

Never state your beliefs about the codebase as facts in a worker prompt. Some of your beliefs are stale. You can't tell which. The code is the only source of truth.

Use this exact framing:

```text
Context (UNVERIFIED — verify against the code before using; if the code
disagrees, trust the code and report the contradiction): <your hints>
```

Never write "Known facts:". Never tell workers to "reflect" your beliefs in their output.

### 2. Tag claims (worker → you)

Put this in every worker prompt:

```text
Do NOT report anything as fact you did not check in this session.
Tag every claim: [verified: <command/file you checked>] or [assumed].
Report anything that contradicts the provided context as a finding —
do not silently apply it or silently ignore it.
```

### 3. Check before acting (report → action)

Before you delete, merge, or edit anything based on a worker claim: run one mechanical check yourself.

- "X is unused" → `grep -rn "X" .` (whole repo)
- "imports fine" → run the import
- "tests pass" → run the tests
- "path is wrong" → `ls` the path

Why: a worker only sees its own slice. "No callers found" misses string dispatch, reflection, and config references. **No worker can verify a repo-wide claim.**

`[assumed]` claims are never actionable.

One claim proved wrong? Sweep all other worker outputs for the same error class.

## Excuses and answers

| Excuse | Answer |
|--------|--------|
| "These are facts I know" | You can't tell current from stale. Mark them UNVERIFIED. Unlabeled beliefs become fabricated docs. |
| "The worker already verified it" | It verified its slice. Only you can check the whole repo. |
| "Deadline — just apply it" | The check is one grep. Seconds vs shipping a breakage. |
| "All tests pass" | Whose tests? Run them yourself. Check they cover the change. |

## Red flags — STOP

- "Known facts:" in a dispatch prompt
- Acting on an `[assumed]` claim — or an untagged one
- Deleting or renaming a symbol without one repo-wide grep
- A worker report with no evidence tags
