# Herdr Team Template

A setup prompt that turns an AI coding agent into a team of agents running inside the **Herdr TUI** (`herdr`), a terminal workspace manager for AI coding agents.

> **Herdr TUI only.** This template drives Herdr through the `herdr` CLI in a terminal: tabs, panes, and `herdr agent` commands. It does not support the Herdr app.

The whole template is one file: [`herdr-team-setup.md`](herdr-team-setup.md). Give it to an agent CLI in your project root. The agent interviews you in a few short steps, then writes the team's instructions, roster, and workflow into your project. After that, a `lead` agent brings up the rest of the team in Herdr tabs and hands them work.

## Quickstart

1. **Install the Herdr skill** (one time, global):
   ```bash
   npx skills add herdrdev/herdr --skill herdr -g
   ```
2. **Copy the setup file into your project root:**
   ```bash
   cd /path/to/your/project
   curl -fsSLO https://raw.githubusercontent.com/aurumio/herdrtemplate/main/herdr-team-setup.md
   ```
   (Or copy `herdr-team-setup.md` from a clone of this repo.)
3. **Run the setup.** Start an agent CLI (`claude`, `codex`, `opencode`, or `pi`) in the project root and say:
   > Follow herdr-team-setup.md.

   Answer the four questions: project type, roles, agent CLI, files.
4. **Start the team.** Open the Herdr TUI, open a new tab for `lead`, and start it in that tab's pane:
   ```bash
   herdr agent start lead --kind <kind> --pane <pane_id>
   ```
   Then tell it:
   > Follow the Onboard section of .herdrteam/roles/lead/AGENTS.md.
5. **Give `lead` your first request.** Say *"team, leave"* when you're done.

The sections below explain each step in more detail.

## What you get

- **A `lead` agent you talk to.** It plans your request, writes handoffs, delegates them to the other roles, checks the results, and reports back. Only `lead` delegates and runs `herdr` commands.
- **Specialist roles in Herdr tabs.** Roles are organized into groups. Each group gets its own tab, with one pane per role and at most 3 roles per group. `lead` stays alone in its own tab.
- **Handoff files** in `.herdrteam/handoffs/` for every piece of delegated work, with a status lifecycle: `open` → `in-progress` ⇄ `question` → `done` | `blocked`.
- **A `work/` folder** with one subfolder per task, holding every role's output.
- **A shared `memory/` folder** with the team's knowledge sorted by category: project, decisions, conventions, domain, references, lessons, plus type-specific folders.
- **Questions instead of guesses.** A role that is uncertain asks its caller. `lead` answers only from settled facts and passes everything else up to you.
- **Temporary workers.** When a task fits no role, `lead` starts a short-lived `tmp-<role>` agent for it and removes it when the task is done.

## Requirements

- The Herdr TUI installed (`herdr --version`). The Herdr app is not supported.
- One or more agent CLIs: **Claude Code** (`claude`), **Codex** (`codex`), **OpenCode** (`opencode`), or **Pi** (`pi`)
- The Herdr skill, installed globally:
  ```bash
  npx skills add herdrdev/herdr --skill herdr -g
  ```
- *(Recommended)* Herdr's integration for each CLI the team uses, so `herdr agent wait` can see whether an agent is idle, working, or blocked:
  ```bash
  herdr integration install <kind>
  ```

The setup agent checks for all of these and offers to install the skill and integrations for you.

## Setup

In your project root, start an agent CLI and ask it to follow `herdr-team-setup.md`. It walks you through four steps, one at a time:

1. **Project type.** Pick *Creative Project* or *Development Project*, and describe the project in a sentence or two.
2. **Roles.** Approve the proposed team, or add, remove, rename, or regroup roles. Role names must match `[a-z][a-z0-9_-]{0,31}`.
3. **Agent CLI.** Pick one CLI for the whole team, or a different one for specific roles (for example, "codex for everyone, claude for reviewer"). The agent then checks Herdr, the Herdr skill, and the integrations.
4. **Files.** The agent creates `.herdrteam/`, `work/`, and `memory/`, then checks that nothing is missing.

If `.herdrteam/` already exists, the agent asks whether to merge or overwrite. Existing `work/` and `memory/` folders are always merged, never overwritten.

## Suggested roles

**Creative Project**

| Group (tab) | Roles |
|---|---|
| `lead` | `lead`: creative director and orchestrator |
| `research` | `researcher`: references, audience, market research, fact-checks · `strategist` *(optional)*: positioning, channels, campaigns |
| `create` | `writer`: copy, scripts, articles · `designer`: visual direction, layouts, asset specs, image and video prompts |
| `review` | `editor`: clarity, tone, brand voice, final quality pass |

**Development Project**

| Group (tab) | Roles |
|---|---|
| `lead` | `lead`: tech lead and orchestrator |
| `design` | `architect`: system design, interfaces, data models |
| `build` | `developer`: features and fixes · `devops` *(optional)*: CI/CD, environments, deployment · `docs` *(optional)*: READMEs, API docs, changelogs |
| `quality` | `reviewer`: diff reviews for correctness, security, maintainability · `tester`: tests, bug reproduction, acceptance checks |

The setup agent tailors these to your project description.

## Generated structure

```
<project root>/
├── work/                    # all task results: one subfolder per task
│   └── README.md
├── memory/                  # shared team knowledge, sorted into categories
│   ├── README.md            # index + memory rules
│   ├── project/  decisions/  conventions/  domain/  references/  lessons/
│   └── <type-specific>/     # Development: architecture/, environments/ · Creative: brand/, audience/
└── .herdrteam/
    ├── AGENTS.md            # team-wide rules + project type & description
    ├── TEAMS.md             # roster: roles, groups, CLIs, responsibilities
    ├── WORKFLOW.md          # how lead delegates and coordinates through Herdr
    ├── handoffs/
    │   └── README.md        # handoff convention + template
    └── roles/
        └── <role>/
            ├── AGENTS.md    # the role's own instructions
            ├── CLAUDE.md    # only for Claude Code roles: contains @AGENTS.md
            ├── logs/        # one log per task
            ├── context/     # role-private working state
            └── skills/      # role-specific playbooks
```

## Running the team

**Start.** Open the Herdr TUI in a terminal, open a new tab just for `lead`, and start the lead agent in that tab's pane:

```bash
herdr agent start lead --kind <kind> --pane <pane_id>
```

(Or run the lead CLI directly in a Herdr pane.) Then tell it:

> Follow the Onboard section of .herdrteam/roles/lead/AGENTS.md.

`lead` opens every group tab, starts and onboards each role, reports who is ready, and waits for your first request.

**Work.** Ask `lead` for what you need. For each request it:

1. Plans the work and creates a task folder, `work/<task-id>/`
2. Writes a handoff per role in `.herdrteam/handoffs/`
3. Prompts the role's agent through Herdr and waits
4. Verifies the result against the acceptance criteria
5. Reports back to you with links into `work/<task-id>/`, and files decisions in `memory/decisions/`

A typical creative flow: `lead` → researcher → writer + designer (in parallel) → editor → `lead`.
A typical development flow: `lead` → architect → developer → tester + reviewer (in parallel) → `lead`.

**Stop.** Say *"team, leave"* to `lead`. Each role saves its work, writes its log, and updates `memory/`. Then `lead` quits each CLI, closes its pane and group tab, writes a final summary, and closes its own pane last.

## Changing the team

Ask `lead`. It can add a role (after your approval), remove one (its folder is archived to `.herdrteam/roles/_archived/`, not deleted), or start a temporary worker for a one-off task. If the same kind of temporary worker is needed twice, `lead` suggests making it a permanent role.
