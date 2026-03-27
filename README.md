# kiro-relay

Human ↔ agent handoff for [Kiro](https://kiro.dev) spec-driven development.

## The problem

When a human takes over from the agent — because tokens ran out, because you need a hotfix, or because your hands are itching — the Kiro spec files (`requirements.md`, `design.md`, `tasks.md`) go stale. The agent comes back and has to *guess* what changed. It re-does work, overwrites fixes, or loses track of where it was.

## The solution

`kiro-relay` is a zero-dependency bash script that gives you a structured way to:

1. **Take over** — formally signal that a human is in control
2. **Log changes** — update spec files through guided prompts, so your edits are consistent with Kiro's format
3. **Hand back** — generate a `HANDBACK.md` briefing that the agent can consume immediately

The agent finds `HANDBACK.md` inside the spec folder and knows exactly what happened, what changed, and where to resume.

## Install

```bash
# Clone
git clone https://github.com/MarcoGarlet/kiro-relay.git

# Add to PATH (pick one)
ln -s "$(pwd)/kiro-relay/relay" ~/.local/bin/relay
# or
export PATH="$PATH:$(pwd)/kiro-relay"
```

No dependencies. Just bash.

## Usage

Run from anywhere inside a Kiro project (a directory tree containing `.kiro/specs/`).

```
relay <command> [spec-name]
```

### Commands

| Command | What it does |
|---|---|
| `relay status` | Show tasks progress, active sessions, spec files |
| `relay takeover` | Start a human session — picks a reason, snapshots tasks |
| `relay update` | Log changes: complete tasks, add tasks, update design/requirements, log code changes |
| `relay handback` | End the session, generate `HANDBACK.md` for the agent |
| `relay log` | Show relay history for a spec |

### Typical workflow

```bash
# Agent ran out of tokens, you jump in
relay takeover user-authentication

# You fix a bug in AuthService.cs
relay update user-authentication
# → pick "Complete a task" or "Log a code change"
# → answer the prompts

# You also change the design approach
relay update user-authentication
# → pick "Update design"
# → describe what changed

# Done — hand back to the agent
relay handback user-authentication
# → HANDBACK.md is generated in .kiro/specs/user-authentication/
```

When the agent resumes, reference the spec with `#spec:user-authentication` in Kiro. The `HANDBACK.md` file is right there for it to read.

## What gets created

```
.kiro/
├── relay/                              # relay working directory
│   ├── my-feature.session.md           # active session (while human is in control)
│   └── my-feature.20250327-1430.session.md  # archived sessions
└── specs/
    └── my-feature/
        ├── requirements.md             # may get "Human Amendment" sections
        ├── design.md                   # may get "Human Amendment" sections
        ├── tasks.md                    # tasks get marked [x] by relay
        └── HANDBACK.md                 # generated on handback (agent reads this)
```

## Design decisions

- **Bash, zero deps** — works anywhere Kiro runs, no node/python needed
- **Interactive prompts, no editor** — quick and focused, you stay in the terminal
- **Markdown output** — human-readable, agent-readable, git-diffable
- **Append-only amendments** — relay never rewrites your spec files, it only appends clearly marked sections and toggles task checkboxes
- **Session archiving** — full history of every human intervention, useful for post-mortems

## Steering tip

Add this to `.kiro/steering/relay.md` so the agent knows about the handback protocol:

```markdown
# Human Relay Protocol

When resuming work on a spec, always check for a `HANDBACK.md` file
in the spec folder. If present:

1. Read it completely before doing anything else
2. Do NOT redo any task marked as completed
3. Respect all "Human Amendment" sections in design.md and requirements.md
4. Delete HANDBACK.md after you have consumed it
5. Continue with the next incomplete task
```

## License

MIT
