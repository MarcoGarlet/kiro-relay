# Human Relay Protocol

## Handback

When resuming work on a spec, always check for a `HANDBACK.md` file
in the spec folder. If present:

1. Read it completely before doing anything else
2. Do NOT redo any task marked as completed
3. Respect all "Human Amendment" sections in design.md and requirements.md
4. Delete HANDBACK.md after you have consumed it
5. Continue with the next incomplete task

If you see a `.kiro/relay/` directory with archived session files,
you can read them for historical context on past human interventions,
but focus on the current HANDBACK.md for your immediate next steps.

## Human-authored specs

Spec files may contain this HTML comment:

```
<!-- authored-by: human via kiro-relay — do not rewrite, refine only when explicitly asked -->
```

When you find this marker:

- Use the spec as-is. Do not rewrite, restructure, or "improve" it.
- Do not convert requirements to a different notation unless asked.
- Do not add sections, acceptance criteria, or tasks on your own.
- If something is unclear, ask the human before changing the spec.
- You may fix typos or formatting only if they break parsing.
