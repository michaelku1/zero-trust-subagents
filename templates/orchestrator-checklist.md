# Orchestrator checklist

Run this before acting on any worker claim. Acting = delete, merge, edit, or update docs.

## 1. Every finding is a hypothesis

Even `[verified]` ones. The worker verified its slice. Only you can check the whole repo.

## 2. Run one mechanical check

| Worker says | You run |
|-------------|---------|
| "X is unused / dead code" | `grep -rn "X" .` — whole repo. Catches string dispatch, reflection, configs. |
| "module imports fine" | Run the import: `python -c "import module"` |
| "all tests pass" | Run the suite yourself. Check it covers the change. |
| "docs point at the wrong file" | `ls` the path. Grep for what the live code actually loads. |

## 3. Never act on `[assumed]`

Send it back, or verify it yourself.

## 4. One wrong claim implies siblings

A claim proved wrong? Sweep every other worker's output for the same error class before closing.

## 5. Audit what you touched

Finished editing canonical docs? Do a final pass: check paths exist, grep for leftover references.
