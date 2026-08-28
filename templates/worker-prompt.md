# Worker dispatch template

Copy this into every subagent prompt. Fill in the `<...>` parts. Works in any agent stack.

```text
<task: what this worker does, over which files/folders>

Read the actual code first. Do NOT fabricate — only describe or change
what the code actually does. Do NOT report anything as fact you did not
check in this session.

Context (UNVERIFIED — verify against the code before using; if the code
disagrees, trust the code and report the contradiction):
- <hint 1>
- <hint 2>

Report format (mandatory):
- What you created/changed (exact paths)
- Tag every claim: [verified: <command/file you checked>] or [assumed]
- Findings: anything that contradicts the context above — report it,
  do not silently apply it or ignore it
```

## Three rules of thumb

1. **Never write "Known facts:".** That phrase is how stale beliefs become documentation.
2. **Scale the reading honestly.** For a 50-file folder, say "read the docstring and top of each file." An impossible instruction gets silently skipped.
3. **Protect existing files.** Tell workers: don't overwrite an existing artifact without reading it first.
