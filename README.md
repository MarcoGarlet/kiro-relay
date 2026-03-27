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
# Option 1: global install (recommended)
git clone https://github.com/YOUR_USER/kiro-relay.git ~/.kiro-relay
ln -s ~/.kiro-relay/relay ~/.local/bin/relay

# Option 2: clone inside project (add to .gitignore)
git clone https://github.com/YOUR_USER/kiro-relay.git .kiro/relay-tool
echo '.kiro/relay-tool/' >> .gitignore
```

No dependencies. Just bash.

## Usage

Run from anywhere inside a Kiro project (a directory tree containing `.kiro/`).

```
relay <command> [spec-name]
```

### Commands

| Command | What it does |
|---|---|
| `relay init` | Create a new spec from scratch — requirements, design, tasks via prompts |
| `relay status` | Show tasks progress, active sessions, spec files |
| `relay takeover` | Start a human session — picks a reason, snapshots tasks |
| `relay update` | Log changes: complete tasks, add tasks, update design/requirements, log code changes |
| `relay handback` | End the session, generate `HANDBACK.md` for the agent |
| `relay log` | Show relay history for a spec |

### Create a spec from scratch

When you want to define a feature (or bugfix) before the agent touches anything:

```bash
relay init user-authentication
# → pick feature or bugfix
# → fill in user story, acceptance criteria
# → describe the design approach and components
# → list implementation tasks
```

This creates a full `.kiro/specs/user-authentication/` folder with `requirements.md`, `design.md`, and `tasks.md` — all in Kiro's format. The agent can pick it up immediately with `#spec:user-authentication`.

### Take over from the agent

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

### How the agent picks up your work

Whether you created a spec from scratch with `relay init` or took over mid-flight with `relay takeover`, the agent needs to be told to look at it. Here's what to do in each scenario.

**After `relay init` (human-authored spec, no prior agent work):**

Open a spec session in Kiro and reference your spec:

```
#spec:user-authentication implement task 1
```

Or be more explicit if the agent hasn't seen the spec before:

```
I created a new spec for user-authentication. Read the requirements,
design, and tasks in #spec:user-authentication and start with task 1.
Do not modify the spec files — they are human-authored.
```

**After `relay handback` (resuming from a human session):**

```
I made some manual changes to #spec:user-authentication while you were away.
Read HANDBACK.md in the spec folder first — it describes what I changed.
Then continue with the next incomplete task.
```

Or more concisely:

```
#spec:user-authentication — read HANDBACK.md and resume from where I left off.
```

**After a token exhaustion (quick resume):**

If you only completed a task or two without changing the design:

```
#spec:user-authentication I completed tasks 3.1 and 3.2 manually.
Continue from task 4.1.
```

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

Copy `steering/relay.md` from this repo into your project's `.kiro/steering/` folder. It tells the agent two things:

1. **Handback protocol** — check for `HANDBACK.md` before resuming, don't redo completed tasks
2. **Human-authored specs** — when spec files contain the `authored-by: human` marker, don't rewrite them

```bash
cp ~/.kiro-relay/steering/relay.md your-project/.kiro/steering/relay.md
```

Specs created with `relay init` automatically include the marker comment. The agent will see it and treat the spec as authoritative — no unsolicited rewrites, no added sections, no notation changes.

## License

MIT
