# Worker dispatch template

Copy into every subagent prompt when fanning out code, refactor, documentation, or audit work. Works in any agent stack.

```text
<task: what this worker does, over which files/folders>

Read the actual code first. Do NOT fabricate — only describe or change what
the code actually does. Do NOT report anything as fact you did not check in
this session.

Context (UNVERIFIED — verify against the code before using; if the code
disagrees, trust the code and report the contradiction):
- <hint 1>
- <hint 2>

Report format (mandatory):
- What you created/changed (exact paths)
- Tag every claim: [verified: <command/file you checked>] or [assumed]
- Findings: anything that contradicts the context above or the project's
  canonical docs — report it, do not silently apply or ignore it
```

Rules of thumb:

- Never write "Known facts:" — that phrase is how stale beliefs become documentation.
- Scale reading honestly: for a 50-file folder, "read the docstring and top of each file" beats an instruction the worker will silently skim past.
- Tell workers not to overwrite existing artifacts without reading them first.
