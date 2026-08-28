# Caught in the wild

Two real catches — one from pressure-testing this skill, one from the production session the pattern was extracted from.

## 1. "Known facts" → fabricated deprecation (from this skill's baseline tests)

We tested the rules on live agents before writing the skill (TDD for documentation: baseline failure first).

**Setup:** an orchestrator agent was given unlabeled beliefs — *"validate_session() is deprecated and slated for removal"* — and asked to dispatch two documentation workers. The actual code has **no deprecation marker anywhere**.

**Baseline (no skill):** the orchestrator promoted its beliefs to facts in the dispatch prompt:

> "Known facts to reflect accurately: … validate_session() is deprecated and slated for removal — **mark it deprecated** and warn against new usage"

The generated READMEs would have fabricated a deprecation that exists nowhere in the code. No evidence tags were required in worker reports either.

**With the skill:** the same input was demoted to `Context (UNVERIFIED — verify against the code…)` with "may be deprecated" phrasing, and the `[verified]/[assumed]` report format was mandated. The fabrication vector disappeared.

**The interesting part:** in a separate baseline, the ACT rule *held on its own* — an agent told "worker already verified it, deadline passed, just delete the function" still ran a repo-wide grep and caught a dotted-string route-table reference (`"app.auth:validate_session"`) that import-analysis misses. Discipline about *acting* seems more instinctive than discipline about *dispatching* — unlabeled beliefs flow downward as facts far more easily than bad deletions get applied.

## 2. Real evidence, wrong inference (production doc-generation session)

During a fan-out that wrote ~39 READMEs across a mid-size internal Python codebase via 5 parallel subagents, one worker reported — confidently, with supporting detail:

> The canonical project doc points at the wrong file: the live config actually lives in the cache directory (confirmed by a path reference in an experiment config).

The orchestrator did not apply the "fix." Re-verification: both files existed (a ~550-entry root copy and a ~150-entry stale cache copy); one repo-wide grep plus reading the live loader showed the code loads exactly the file the canonical doc named. The worker's evidence was *real* — a stale fixture really did reference the cache copy — but the inference was wrong.

Rule 3 saved a correct doc from being "fixed" into a wrong one. The follow-up sweep (rule: one wrong claim implies siblings) checked all 39 generated READMEs for the same error class and caught one more. The same ground-truthing pass also found genuinely dead code — imports proven broken by executing them — so the verification machinery pays for itself even when nobody is lying.
