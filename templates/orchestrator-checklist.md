# Orchestrator checklist — the ACT line

Before acting on any worker claim (delete, merge, edit, update docs):

1. **Treat every finding as a hypothesis** — including ones tagged `[verified]`. Worker verification is slice-local; only you can check repo-wide.
2. **Run one mechanical check against ground truth:**

   | Claim type | Check |
   |---|---|
   | "X is unused / dead code" | `grep -rn "X" .` (whole repo — catches string dispatch, reflection, configs) |
   | "module imports fine" | `python -c "import module"` (or equivalent) |
   | "all tests pass" | run the suite yourself; confirm it covers the changed path |
   | "file/path is wrong in docs" | `ls` the path; grep for what the live code actually loads |

3. **`[assumed]` claims are never actionable.** Send them back or verify them yourself.
4. **One wrong claim implies siblings.** When a claim proves wrong, sweep every other worker's output for the same error class before closing.
5. **Close with a mechanical audit** of any canonical docs you touched: path-existence check, residual-reference grep.
