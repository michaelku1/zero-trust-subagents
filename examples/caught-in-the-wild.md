# Caught in the wild

Two real catches. One from testing this skill. One from the production session the pattern came from.

## Catch 1: "Known facts" that weren't facts

From this skill's own baseline tests. We tested agents *before* writing the skill, to see what they do naturally.

**Setup.** An orchestrator agent got some beliefs, stated as plain facts: *"validate_session() is deprecated and slated for removal."* The actual code has no deprecation marker anywhere. The agent's job: write dispatch prompts for two documentation workers.

**Without the skill.** The orchestrator passed its beliefs down as truth:

> "Known facts to reflect accurately: … validate_session() is deprecated and slated for removal — **mark it deprecated** and warn against new usage"

The workers would have written a deprecation into the docs. One that exists nowhere in the code. Nobody would have caught it — the reports required no evidence tags.

**With the skill.** Same input. The beliefs went down as `Context (UNVERIFIED — verify against the code…)`. The report format required `[verified]` / `[assumed]` tags. The fabrication never happened.

**The surprise.** In a separate test, rule 3 held even *without* the skill. An agent was told: "worker already verified it, deadline passed, just delete the function." It still ran a repo-wide grep. It found the string reference (`"app.auth:validate_session"`) that import analysis misses. It refused the delete.

Lesson: agents are careful about *acting*. They are careless about *dispatching*. Unlabeled beliefs flow down as facts far more easily than bad deletions get applied. That's why rule 1 exists.

## Catch 2: real evidence, wrong conclusion

From a production session: 5 parallel subagents writing ~39 READMEs across a mid-size internal Python codebase.

One worker reported, confidently:

> "The canonical project doc points at the wrong file. The live config actually lives in the cache directory. Confirmed by a path reference in an experiment config."

Sounds solid. It cites evidence. The orchestrator still didn't apply it — rule 3.

**The check.** Both files existed: a ~550-entry copy at the root, a ~150-entry stale copy in the cache dir. One repo-wide grep, plus reading the live loader, showed the truth: the code loads exactly the file the canonical doc named.

The worker's evidence was real — a stale fixture really did reference the cache copy. Its conclusion was wrong.

**The follow-up.** One wrong claim implies siblings (rule: sweep the error class). The orchestrator checked all 39 READMEs for the same mistake. Found one more. Fixed it.

Bonus: the same verification pass found genuinely dead code — imports that failed when actually executed. The checking machinery finds real bugs too, not just fabrications.
