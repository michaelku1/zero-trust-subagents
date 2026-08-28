---
name: zero-trust-subagents
description: Use when dispatching parallel subagents for coding, refactoring, documentation, or audit work, or when about to act on a subagent's report — deleting code, merging changes, or editing docs based on claims you did not check yourself.
license: MIT
---

# Zero-Trust Subagents

Subagents run in separate contexts. Everything they send you is generated text. It sounds right. It can be wrong.

**Never let generated text cross between contexts without a check.**

## Rule 1 — Mark hints (you → worker)

Your beliefs about the codebase may be stale. Don't state them as facts. Pass them like this:

```text
Context (UNVERIFIED — verify against the code before using; if the code
disagrees, trust the code and report the contradiction): <your hints>
```

Never write "Known facts:".

## Rule 2 — Tag claims (worker → you)

Put this in every worker prompt:

```text
Do NOT report anything as fact you did not check in this session.
Tag every claim: [verified: <command/file you checked>] or [assumed].
Report anything that contradicts the provided context as a finding —
do not silently apply it or silently ignore it.
```

## Rule 3 — Check before acting (report → action)

Before you delete, merge, or edit based on a worker claim, run one check yourself:

| Worker says | You run |
|-------------|---------|
| "X is unused" | `grep -rn "X" .` |
| "imports fine" | run the import |
| "tests pass" | run the tests |
| "path is wrong" | `ls` the path |

A worker only sees its slice. It cannot verify a repo-wide claim. Never act on `[assumed]`. One claim wrong? Check the other workers' claims for the same mistake.

## Excuses

| Excuse | Answer |
|--------|--------|
| "I know this codebase" | Some of that knowledge is stale. Mark it UNVERIFIED. |
| "The worker already verified it" | Only its slice. You check the whole repo. |
| "No time" | The check is one grep. |

## Red flags — STOP

- "Known facts:" in a dispatch prompt
- Acting on an `[assumed]` or untagged claim
- Deleting or renaming a symbol without one repo-wide grep
