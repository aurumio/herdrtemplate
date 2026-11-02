# Herdr Team Setup

You are setting up a multi-agent team that runs inside **Herdr** (a terminal workspace manager for AI coding agents). You will interview the user in four short steps, then create a `.herdrteam/` folder in the current working directory that describes the team, its roles, and how the roles delegate work to each other through Herdr. Next to it, in the same directory, you also create `work/`, where every task's results are saved, and `memory/`, the team's shared knowledge sorted into categories.

Work through the steps in order. **Ask one step at a time and stop to wait for the user's answer before continuing.** Do not create any files until Step 4.

---

## Step 1 — Project type

Ask the user:

> What kind of project is this team for?
> 1. **Creative Project** — writing, content, design, marketing, storytelling, media
> 2. **Development Project** — software, apps, APIs, infrastructure, tooling
>
> Also, in one or two sentences: what is the project about?

Wait for the answer. Keep the project type and the description; both go into `.herdrteam/AGENTS.md`.

---

## Step 2 — Suggest roles and get approval

Based on the project type, propose a team. Always include `lead`, because it is the orchestrator the user talks to and the only role that delegates work. Tailor the rest to the project description: drop roles that clearly do not fit, and add one if the description calls for it.

**Creative Project — suggested roles**

| Role | Responsibility |
|---|---|
| `lead` | Creative director and orchestrator. Owns the brief, splits work into handoffs, delegates, reviews results, and reports to the user. |
| `researcher` | Gathers references, audience insights, competitor and market research, and fact-checks. |
| `writer` | Drafts copy, scripts, stories, articles, and long-form content. |
| `designer` | Visual direction, layouts, mood boards, asset specs, and image or video prompts. |
| `editor` | Edits for clarity, tone, consistency, and brand voice; gives the final quality pass. |
| `strategist` *(optional)* | Positioning, channels, campaign planning, and distribution. |

**Development Project — suggested roles**

| Role | Responsibility |
|---|---|
| `lead` | Tech lead and orchestrator. Breaks down requirements, writes handoffs, delegates, integrates results, and reports to the user. |
| `architect` | System design, technical decisions, interfaces, and data models. Writes design notes and does not ship features. |
| `developer` | Implements features and fixes according to handoffs and the architect's design. |
| `reviewer` | Reviews diffs for correctness, security, and maintainability, and reports actionable findings only. |
| `tester` | Writes and runs tests, reproduces bugs, and verifies acceptance criteria. |
| `devops` *(optional)* | CI/CD, environments, deployment, and observability. |
| `docs` *(optional)* | READMEs, API docs, changelogs, and user guides. |

**Groups (Herdr layout).** `lead` always works alone in its own tab, with a single pane. All other roles are organized into **groups**. Each group gets its own Herdr tab, labeled with the group name, and each role in a group gets its own pane inside that tab. Suggested groups:

| Project type | Group → roles |
|---|---|
| Creative | `research` → researcher, strategist · `create` → writer, designer · `review` → editor |
| Development | `design` → architect · `build` → developer, devops, docs · `quality` → reviewer, tester |

Keep groups to at most 3 roles each so the panes stay readable. Group names follow the same naming rule as roles.

Present the proposed team grouped by tab, with one line per role. For example:

```
tab lead     → lead
tab build    → developer | devops
tab quality  → reviewer  | tester
```

Then ask:

> Approve this team? You can **approve**, **remove**, **add** (name + one-line description), or **rename** roles, or **move** a role to another group.

Wait for approval. Apply any changes and show the final list again before you move on.

Role names become Herdr agent names, so each one must match `[a-z][a-z0-9_-]{0,31}`: lowercase, starting with a letter, and unique. Normalize names that don't fit (for example `QA Engineer` → `qa-engineer`) and tell the user.

---

## Step 3 — Choose the agent CLI

Ask:

> Which agent CLI should the team run?
> 1. **Claude Code** (`claude`)
> 2. **Codex** (`codex`)
> 3. **OpenCode** (`opencode`)
> 4. **Pi** (`pi`)
>
> One CLI for the whole team, or a different one for specific roles? (Example: "codex for everyone, claude for reviewer")

Wait for the answer. Record the Herdr `--kind` value for each role: `claude`, `codex`, `opencode`, or `pi`.

Then check Herdr itself:

1. **Herdr is installed.** Run `herdr --version`. If it fails, tell the user to install Herdr and stop.
2. **Herdr skill.** The agents need the Herdr skill to drive Herdr. Check whether it is installed globally:
   ```bash
   ls -d ~/.agents/skills/herdr ~/.claude/skills/herdr ~/.codex/skills/herdr ~/.config/opencode/skills/herdr ~/.pi/agent/skills/herdr 2>/dev/null
   ```
   If the folder is missing for any CLI the team uses, ask:
   > The Herdr skill isn't installed for `<kind>`. Install it globally? (`npx skills add herdrdev/herdr --skill herdr -g`)

   On a yes, run `npx skills add herdrdev/herdr --skill herdr -g`, pick the team's CLIs when it asks, and check again. Restart any agent CLI that was already running so it loads the skill. On a no, note it; `lead` falls back to `herdr --skill` (below).
3. **Herdr integrations.** Herdr has a built-in integration for each agent CLI that reports the agent's state (idle, working, blocked) to Herdr. `herdr agent wait` relies on that state, so without the integration, waits can hang or show `unknown`. For each distinct `--kind` in the team, ask:
   > Install Herdr's integration for `<kind>`? It changes that CLI's global configuration. (`herdr integration install <kind>`)

   Run `herdr integration install <kind>` only for the CLIs the user approves. Note any CLI without the integration; `lead` must then check that role with `herdr agent read` rather than trusting `agent wait`.

---

## Step 4 — Create `.herdrteam/`, `work/` and `memory/`

If `.herdrteam/` already exists, tell the user what is in it and ask whether to **merge** (only add missing files) or **overwrite**. If `work/` or `memory/` already exist in the project root, always merge: never overwrite or delete their contents, because they hold the team's results and knowledge. Never delete existing files without explicit permission.

Create this structure in the current working directory, the project root where `lead` starts. `work/` and `memory/` sit in the project root, next to `.herdrteam/`, not inside it:

```
<project root>/              # where lead starts; every agent's working directory
├── work/                    # all task results: one subfolder per task
│   └── README.md            # work folder convention + task README template
├── memory/                  # shared team knowledge, sorted into categories
│   ├── README.md            # index of every entry + memory rules
│   ├── project/             # goals, scope, requirements, constraints, stakeholders
│   ├── decisions/           # one file per decision, YYYYMMDD-<slug>.md
│   ├── conventions/         # standards and style: code style or brand voice, naming, formats
│   ├── domain/              # domain knowledge; glossary.md for terms
│   ├── references/          # sources, external docs, research findings, links
│   ├── lessons/             # gotchas, mistakes, what worked and what didn't
│   └── <type-specific>/     # Development: architecture/, environments/ · Creative: brand/, audience/
└── .herdrteam/
    ├── AGENTS.md            # team-wide instructions + project type & description
    ├── TEAMS.md             # roster of all roles, their CLI, and responsibilities
    ├── WORKFLOW.md          # how roles delegate and coordinate through Herdr
    ├── handoffs/            # task handoffs between roles
    │   └── README.md        # handoff file convention + template
    └── roles/
        └── <role>/          # one folder per approved role
            ├── AGENTS.md    # the role's own instructions
            ├── CLAUDE.md    # only if this role runs Claude Code: contains `@AGENTS.md`
            ├── logs/        # task logs written by this role
            │   └── .gitkeep
            ├── context/     # role-private working state only (e.g. lead's panes.md); shared knowledge goes in memory/
            │   └── .gitkeep
            └── skills/      # skills / playbooks specific to this role
                └── .gitkeep
```

Put a `.gitkeep` in every empty `memory/` category folder.


Fill each file using the templates below. Replace every `{{placeholder}}` with real content, and write actual role descriptions rather than generic filler.

**Paths.** Every agent runs with the project root as its working directory. Inside generated files, always write paths in full from the project root (`work/<task-id>/`, `memory/decisions/`, `.herdrteam/roles/<role>/logs/`), never relative to `.herdrteam/` or to a role folder (`roles/<role>/…` or `logs/`). The one exception is Markdown links in `TEAMS.md`, which resolve from the file's own folder. Each role's folder name must be exactly its normalized Herdr agent name.

**Verify before Step 5.** For every role in `TEAMS.md`, check that `.herdrteam/roles/<role>/AGENTS.md` exists, that `work/README.md` and `memory/README.md` exist in the project root, and that no `{{` placeholder is left in any generated file except the templates in the README files:
```bash
for r in <role> <role> ...; do test -f ".herdrteam/roles/$r/AGENTS.md" || echo "MISSING: $r"; done
test -f work/README.md && test -f memory/README.md || echo "MISSING: work/ or memory/ README"
ls -d ~/.agents/skills/herdr ~/.claude/skills/herdr ~/.codex/skills/herdr ~/.config/opencode/skills/herdr ~/.pi/agent/skills/herdr 2>/dev/null || echo "MISSING: herdr skill (npx skills add herdrdev/herdr --skill herdr -g)"
grep -rn '{{' .herdrteam memory --exclude README.md || true   # README templates keep their {{placeholders}}
```
Fix anything missing before you continue.

### `.herdrteam/AGENTS.md`

```markdown
# {{Project name}} — Herdr Team

**Project type:** {{Creative Project | Development Project}}

## About the project
{{User's project description, expanded into 1 short paragraph.}}

## What a {{Creative | Development}} project means for this team
{{3–5 bullets describing what "done" looks like for this project type, e.g. for Development: working code, tests passing, reviewed diffs; for Creative: on-brief, on-voice, edited deliverables.}}

## Rules for every role
- Read `.herdrteam/TEAMS.md` and `.herdrteam/WORKFLOW.md` before your first task.
- Your own instructions live in `.herdrteam/roles/<your-role>/AGENTS.md`.
- Work only on tasks assigned to you through a handoff in `.herdrteam/handoffs/`.
- Save everything a task produces in that task's folder, `work/<task-id>/<your-role>/` in the project root. The handoff's `workdir:` gives the path. Never save task output in your role folder or loose in the project root. In a Development project, source-code changes go where the code lives in the repo. Notes, designs, reports and test results go in the work folder, and the handoff's Result lists the code files you changed.
- Log every task in `.herdrteam/roles/<your-role>/logs/`.
- Anything the team needs to know later goes in the shared `memory/` in the project root, filed under the right category. Read `memory/README.md` first; it is the index. Follow its rules: one topic per file, update instead of duplicating, add a line to the index, and delete what is no longer true. Your role's `context/` folder holds only private working state.
- Stay inside your role. If work belongs to another role, say so in your handoff result. Only `lead` delegates.
- Only `lead` runs `herdr` commands. The *Session start*, *Start a role's agent*, *Onboard* and *Leave* procedures in `WORKFLOW.md` are `lead`'s to run. Other roles read them for context only and never prompt any agent, including themselves.
- **When you are uncertain, pass the question to your caller. Never guess.** Your caller is whoever gave you the task: the handoff's `from:` (usually `lead`). `lead`'s caller is the user. See *Questions go to the caller* in `WORKFLOW.md`.
- Never close Herdr panes, tabs, or workspaces outside this team. On Leave, `lead` quits every role's CLI and closes its pane and group tab, then closes its own pane last (see *Leave* in `WORKFLOW.md`).
```

### `.herdrteam/TEAMS.md`

```markdown
# Team Roster

| Role | Group (Herdr tab) | Herdr agent name | CLI (`--kind`) | Herdr integration | Responsibility | Instructions |
|---|---|---|---|---|---|---|
| lead | `lead` (own tab, single pane) | `lead` | `{{kind}}` | {{yes | no}} | Orchestrator | [AGENTS.md](roles/lead/AGENTS.md) |
| {{role}} | `{{group}}` | `{{role}}` | `{{kind}}` | {{yes | no}} | {{one line}} | [AGENTS.md](roles/{{role}}/AGENTS.md) |
...

## Layout
```
tab lead       → lead
tab {{group}}  → {{role}} | {{role}}
...
```
`lead`'s pane is never split. Each group is a separate tab, with one pane per role.

## Delegation map
- `lead` → {{list of roles lead delegates to}}
- {{role}} → reports back to `lead`
...

## Temporary workers
Workers `lead` creates for a task that no role above fits (see *Temporary workers* in `WORKFLOW.md`). They run in the `tmp` tab and are removed when their task is done.

| Worker | Task | CLI | Pane | Started |
|---|---|---|---|---|
<!-- - tmp-<role> | <task-id> | <kind> | <pane_id> | <ISO timestamp> -->

## Escalation
Anything ambiguous, blocked, or out of scope goes back to `lead`. `lead` escalates to the user.
```

### `.herdrteam/roles/<role>/AGENTS.md`

```markdown
# Role: {{role}}

**Herdr agent name:** `{{role}}`  ·  **Group / tab:** `{{group}}`  ·  **CLI:** `{{kind}}`  ·  **Reports to:** {{lead | user (for lead)}}

## Mission
{{2–3 sentences: why this role exists in this specific project.}}

## Responsibilities
- {{concrete responsibility}}
- ...

## Not your job
- {{things this role must hand back instead of doing}}

## Inputs
- Handoffs addressed to `{{role}}` in `.herdrteam/handoffs/`
- The task folder given in the handoff's `workdir:`, including other roles' output for the same task
- {{other inputs, e.g. design notes from architect, brief from lead}}

## Outputs / definition of done
- {{what a finished task from this role looks like}}
- Output saved in `<workdir>/{{role}}/`, the handoff's `workdir:` plus your role name
- Handoff file updated with `status: done` and a Result section that lists the files you produced
- Log entry written in `.herdrteam/roles/{{role}}/logs/`

## When uncertain — ask your caller
Your caller is whoever gave you the task: the handoff's `from:`. {{For lead: your caller is the user.}}
Ask instead of guessing when the requirements are ambiguous, there are several valid options with different trade-offs, the work needs a decision outside your role, an action is risky or irreversible, or something contradicts `memory/` or another handoff.
1. Stop at a safe point. Don't build on an assumption.
2. In the handoff, set `status: question` and add your question under **Questions**. Make it specific, list the options you see with your recommendation, and say what it blocks.
3. Reply `question: {{role}} → <caller>: <one-line question>`, then wait.
4. When the answer comes, record it under **Questions**, save it to `memory/` under the right category (decisions go in `memory/decisions/`) if it will matter again, set `status: in-progress`, and continue.

## Memory
- `memory/` in the project root is the team's shared knowledge, filed by category (see `memory/README.md`). At the start of each task, read the index and the entries that matter for the task. When you learn something future tasks will need, save it there, in the right category, and add it to the index.
- `.herdrteam/roles/{{role}}/context/` holds only this role's private working state. Anything another role might need goes in `memory/`.
- `.herdrteam/roles/{{role}}/skills/` holds reusable playbooks/checklists for this role. Use them; add new ones when a procedure repeats.

## Log format
Write one file per task: `.herdrteam/roles/{{role}}/logs/YYYY-MM-DD-<handoff-id>.md`, covering what you did, files touched, decisions, open issues, and time-ordered notes.

## Onboard (every time you join a session)
Onboarding only reads files. Don't run `herdr` commands, edit files, or start any work while onboarding. All paths are from the project root. If a file is missing, don't search for it or retry. Reply `not ready: {{role}} — missing <path>` and stop.
1. Read `.herdrteam/AGENTS.md`, `.herdrteam/TEAMS.md`, `.herdrteam/WORKFLOW.md`, and this file. The `herdr` procedures in `WORKFLOW.md` are `lead`'s. Don't run them yourself.
2. Read `memory/README.md` (the index), `memory/project/` and `memory/conventions/`, plus anything in `.herdrteam/roles/{{role}}/context/`. Check `.herdrteam/roles/{{role}}/skills/` for the playbooks you have.
3. Look in `.herdrteam/handoffs/` for files with `to: {{role}}` and status `open`, `in-progress` or `blocked`. Note them; don't resume them yet.
4. Reply `ready: {{role}}`, followed by one line per handoff you found (`in-progress` first). Then **end your turn** and wait. Don't start or resume any handoff until `lead` tells you to.

If you are asked to onboard again in a session where you already replied `ready`, don't redo these steps. Just send the `ready` reply again.

## Leave (when told to leave, or before the session ends)
1. Finish the current step, or stop at a safe point. Don't leave files half-edited.
2. For every handoff you own that isn't `done`, set `status: blocked` or leave it `in-progress`, and add a **Result** note saying where you stopped and what's next.
3. Write or close your log in `.herdrteam/roles/{{role}}/logs/`.
4. Save anything a future session will need (decisions, conventions, gotchas) to `memory/` under the right category, and update the index. Keep it short and current, and delete entries that are no longer true.
5. Reply `left: {{role}}` with a 1–3 line summary, then end your turn. `lead` then quits your CLI and closes your pane, so don't start anything new after `left`.
```

For the `lead` role, add a **Delegation** section to its AGENTS.md that points to `.herdrteam/WORKFLOW.md` and summarizes the loop: plan, write handoff, prompt agent, wait, verify, report. It also says that when no role fits a task, `lead` creates a temporary worker for it (*Temporary workers* in `WORKFLOW.md`) instead of doing the work itself or forcing it onto the wrong role.

Also extend `lead`'s **Onboard** section. Unlike the other roles, `lead` runs `herdr` commands while onboarding. After its own steps 1–3, `lead` runs *Session start — onboard the whole team* in `WORKFLOW.md` **once**. That means opening every group tab, starting and onboarding every other role, and reporting the team status. No task is needed. `lead` then waits for the user's first request. Make these rules explicit in `lead`'s Onboard:
- Before any `herdr` command, `lead` loads the global `herdr` skill. If the skill isn't available, `lead` reads `herdr --skill` output instead and tells the user to run `npx skills add herdrdev/herdr --skill herdr -g`.
- *Session start* refers back only to steps 1–3. Don't re-enter the Onboard section from there.
- `lead` never sends an onboarding prompt to itself (`herdr agent prompt lead …`).
- If `lead` is asked to onboard again in the same session, it doesn't restart. It checks `herdr agent list` and onboards only the roles that are missing.

Also, in `lead`'s **When uncertain** section, make clear that `lead`'s caller is the user. When `lead` receives a question from a role, it answers only if the answer is already settled: in the user's request, in `memory/` (especially `memory/decisions/`), or in an earlier decision. Otherwise it passes the question up to the user unchanged.

### `.herdrteam/roles/<role>/CLAUDE.md` (only for roles whose CLI is `claude`)

```markdown
@AGENTS.md
```

### `.herdrteam/handoffs/README.md`

````markdown
# Handoffs

Every unit of delegated work is one Markdown file in this folder.

**File name:** `YYYYMMDD-HHMM-<from>-to-<to>-<short-slug>.md`
(e.g. `20261102-1530-lead-to-developer-login-form.md`)

**Status lifecycle:** `open` → `in-progress` ⇄ `question` → `done` | `blocked`

- `question`: the assignee is uncertain and waiting for its caller (`from:`) to answer. See **Questions**.
- `blocked`: the assignee can't continue for a reason an answer won't fix, such as a missing dependency or work that belongs to another role.

## Template

```markdown
---
id: {{YYYYMMDD-HHMM-slug}}
from: lead
to: {{role}}
status: open
priority: normal   # low | normal | high
created: {{ISO timestamp}}
depends_on: []     # other handoff ids
task: {{task-id}}                       # the user request this belongs to
workdir: work/{{task-id}}/              # in the project root; save output in <workdir>/<role>/
---

## Task
{{What to do, in 1–3 sentences.}}

## Context
{{Links to files, prior handoffs, relevant entries in `memory/`.}}

## Acceptance criteria
- [ ] {{verifiable criterion}}

## Questions
<!-- The assignee asks here when uncertain; the caller answers below each one. -->
<!-- Q1 (role, timestamp): question · options · recommendation · what it blocks -->
<!-- A1 (caller, timestamp): answer -->

## Result
<!-- Filled in by the assignee when done or blocked -->
```
````

### `work/README.md` (in the project root)

````markdown
# Work

This folder is in the project root, where `lead` starts. Every task (one user request) gets one folder here, and all of its output goes there.

**Folder name:** `YYYYMMDD-HHMM-<short-slug>` (e.g. `20261102-1530-login-form`). `lead` creates it.

```
work/<task-id>/
├── README.md      # task brief, handoffs, status, deliverables (lead keeps it current)
├── <role>/        # one subfolder per role that works on the task
└── ...
```

- Roles write only into their own `<role>/` subfolder. They may read the other roles' subfolders for the same task.
- Don't delete or move task folders. They are the team's history.
- In Development projects, source code stays where it lives in the repo. The task folder holds notes, designs, reports and test results, and points to the code that changed.

## Task README template

```markdown
# {{task-id}} — {{title}}

**Status:** open | in-progress | done | blocked
**Requested:** {{ISO timestamp}}  ·  **Request:** {{the user's request, verbatim or close}}

## Handoffs
- [{{handoff-id}}](../../.herdrteam/handoffs/{{handoff-id}}.md) → {{role}} · {{status}}

## Deliverables
- {{path}} — {{what it is}}

## Summary
{{Filled in by lead when the task is done.}}
```
````

### `memory/README.md` (in the project root)

Fill in the category list for the actual project type, and add the type-specific folders.

````markdown
# Memory

The team's shared knowledge. Everything a role needs to know and can't read from the code or the task output lives here, sorted by category. Every role reads it and every role writes to it.

## Categories
| Folder | What goes there |
|---|---|
| `project/` | Goals, scope, requirements, constraints, stakeholders |
| `decisions/` | One file per decision: `YYYYMMDD-<slug>.md` with context, options, the decision, and who made it |
| `conventions/` | Standards and style: {{code style, naming, review rules | brand voice, tone, formats}} |
| `domain/` | Domain knowledge; `glossary.md` for terms |
| `references/` | Sources, external docs, research findings, links |
| `lessons/` | Gotchas, mistakes, what worked and what didn't |
| {{type-specific, e.g. `architecture/`, `environments/` or `brand/`, `audience/`}} | {{...}} |

## Rules
- **Read first.** Read this index at the start of every task, then open the entries that matter for the task.
- **One topic per file**, in the right category, with a short descriptive file name (`conventions/api-errors.md`).
- **Update, don't duplicate.** If an entry on the topic exists, edit it.
- **Say where it came from.** End each entry with `Source: <user | handoff-id | role>, <date>`.
- **Decisions** go in `decisions/` only once the user or `lead` has settled them. Open questions stay in the handoff.
- **Keep it true.** Delete or fix entries that turn out to be wrong, and update the index.
- **Keep the index current.** Every entry has one line below. Add yours in the same edit.
- No secrets or credentials.

## Index
<!-- - [title](category/file.md) — one-line hook -->
````

### `.herdrteam/WORKFLOW.md`

Write this file for the actual roster and CLIs, following this structure:

````markdown
# Workflow — Delegation with Herdr

All roles run as named Herdr agents in panes of the same Herdr workspace. The user talks to `lead`. `lead` delegates through handoff files and the `herdr` CLI. Other roles do the work and report back by updating the handoff.

## 0. Preconditions
- The lead agent runs inside a Herdr pane: `test "${HERDR_ENV:-}" = 1`. If that check fails, stop and tell the user.
- `lead` has loaded the `herdr` skill, installed globally with `npx skills add herdrdev/herdr --skill herdr -g`. If it isn't installed, read `herdr --skill` instead and tell the user.
- Treat the installed `herdr` binary as the authority on syntax. Use `herdr agent` and `herdr pane` (no subcommand) to see help.
- For roles whose CLI has no Herdr integration (see `TEAMS.md`), the agent state can be wrong. After each wait, confirm with `herdr agent read <role>` and the handoff file.
- Read IDs from the JSON that commands return. Never guess pane IDs.

## Session start — onboard the whole team (no task needed)
When `lead` onboards, it brings up the **entire team before any work is assigned**. Agents don't wait for a first task to be started. Only `lead` runs this section, once per session.

1. `lead` labels its own pane and tab `lead`. Always do this, even if they're already labelled, because the user or a restart may have renamed them:
   ```bash
   herdr pane rename "$HERDR_PANE_ID" lead
   herdr tab rename "$HERDR_TAB_ID" lead
   ```
   Record `lead`'s tab and pane in `.herdrteam/roles/lead/context/panes.md` as *created by user*. They are closed only as the very last step when the whole team leaves.
2. `lead` runs steps 1–3 of its own Onboard section (read files, `memory/`, `.herdrteam/roles/lead/context/`, handoffs). If `work/` or `memory/` is missing in the project root, create it from the setup structure first. Don't re-enter the Onboard section or this section from there.
3. For every group in `TEAMS.md`, one group at a time: create the group's tab, then start each of its roles (§3 *Start a role's agent*) and onboard it (see **Onboard** below). Skip `lead`, and skip roles already listed by `herdr agent list`. Don't start temporary workers at session start. If *Temporary workers* in `TEAMS.md` still lists any from an earlier session, report them and their unfinished handoffs to the user. Restart one only when its task continues.
4. Onboard roles in the same group in parallel: send each onboarding prompt without `--wait`, then run `herdr agent wait <role> --timeout 120000` **once** for each one. If a wait times out or the reply has no `ready: <role>`, read the pane once (`herdr agent read`), mark the role `not ready` with what you saw, and move on. Don't re-send the prompt or wait again during onboarding.
5. Don't create or send any handoff during team onboarding. Agents end up idle at `ready: <role>`.
6. Report the team status to the user as one line per role (role, group/tab, pane, `ready` or the problem), followed by any unfinished handoffs the roles reported. Then wait for the user's first request.

If a role fails to start or onboard, report that role, keep onboarding the others, and don't retry blindly.

## 1. Plan
`lead` breaks the user's request into tasks, one per role, and notes dependencies. If a task doesn't fit any role in `TEAMS.md`, `lead` assigns it to a temporary worker (see *Temporary workers*). It never does the work itself or gives it to a role it doesn't fit.

Then `lead` creates the task folder. One user request is one task and gets one folder:
```bash
mkdir -p work/<task-id>     # in the project root; task-id: YYYYMMDD-HHMM-<short-slug>
```
`lead` writes `work/<task-id>/README.md` from the template in `work/README.md`. Each role saves its output in `work/<task-id>/<role>/`. Before planning, `lead` checks `memory/` for earlier decisions and conventions that apply. A follow-up request on the same topic is a new task with its own folder, and its README links to the earlier one.

## 2. Write the handoff
`lead` creates `.herdrteam/handoffs/<id>.md` from the template with `status: open`, setting `task:` and `workdir:` to the task folder. It also adds the handoff to the task README.

## 3. Start a role's agent
Normally every role is already running from team onboarding at session start. Use these steps during team onboarding, or when a role has to be started or restarted later.

### Layout
- **`lead` has its own tab with a single pane, and both are always labelled `lead`.** Never split `lead`'s pane, and never start another role in `lead`'s tab. If `lead` isn't alone in its tab, tell the user.
- **Every other role runs in its group's tab**, one pane per role, as listed in `TEAMS.md`.
- All tabs are in `lead`'s workspace (`$HERDR_WORKSPACE_ID`), and every pane uses the project root as its working directory.
- Keep a layout registry in `.herdrteam/roles/lead/context/panes.md`. List each group with its tab ID, and each role with its pane ID. Note for each tab and pane whether `lead` created it.

### Steps
Check whether the agent is already running:

```bash
herdr agent list
```

If it is not running:

1. **Find or create the group's tab.** Check the registry first, then `herdr tab list --workspace "$HERDR_WORKSPACE_ID"`. If the tab doesn't exist, create it in the background:
   ```bash
   herdr tab create --workspace "$HERDR_WORKSPACE_ID" --label <group> --cwd "$PWD" --no-focus
   # → .result.tab.tab_id and .result.root_pane.pane_id
   ```
   The first role in a group uses the tab's `root_pane`.
2. **Add more panes for the other roles in the group.** Split the group's last pane, never `lead`'s. Use `right` for the 2nd role and `down` for the 3rd:
   ```bash
   herdr pane layout --pane <last_group_pane_id>
   herdr pane split <last_group_pane_id> --direction right --cwd "$PWD" --no-focus   # → .result.pane.pane_id
   ```
3. **Label the pane and start the agent:**
   ```bash
   herdr pane rename <pane_id> <role>
   herdr agent start <role> --kind <kind> --pane <pane_id>
   ```
4. Record the tab and pane IDs in `.herdrteam/roles/lead/context/panes.md`, then onboard the agent (see **Onboard** below).

Read every ID from the JSON a command returns. If a field path differs from the comments above, follow the actual output.

## 4. Delegate
```bash
herdr agent prompt <role> "New handoff: .herdrteam/handoffs/<id>.md. Set status to in-progress, do the task, save all output in <workdir>/<role>/, write your log, then set status to done (or blocked) and fill in Result." --wait --timeout 600000
```

You can delegate independent tasks to several roles in parallel. Send each prompt without `--wait`, then wait on each one:

```bash
herdr agent wait <role> --timeout 600000
```

## 5. Collect and verify
- Read the handoff file's `status` and `Result` section. Treat the file as the source of truth, not the terminal output.
- If you need more detail: `herdr agent read <role> --source recent-unwrapped --lines 120`
- Check the result against the acceptance criteria. If it falls short, write a follow-up handoff instead of silently editing the work yourself.

## 6. Handle states
| State | Meaning | Action |
|---|---|---|
| `idle` / `done` | Ready for input | Read the handoff and continue |
| `working` | Still busy | Keep waiting; don't re-send the prompt |
| `blocked` | Approval or question dialog | Run `herdr agent read <role>`, then **ask the user** before answering |
| handoff `status: question` | The role is uncertain | Follow *Questions go to the caller* below |
| `unknown` | Herdr can't classify the state | Inspect with `agent get` / `agent read` before acting |

A timeout does **not** prove the prompt was lost. Inspect the agent before you re-send anything.

## Questions go to the caller
Whenever a role is uncertain, it asks the role that called it instead of guessing. Questions travel up the chain of callers, and answers travel back down the same way:

```
role ──question──▶ lead ──(if lead can't answer from settled facts)──▶ user
role ◀──answer──── lead ◀──────────────────answer─────────────────────── user
```

1. The role writes the question in the handoff's **Questions** section, sets `status: question`, replies `question: <role> → lead: …`, and waits.
2. After every wait, `lead` checks the handoff's `status`. A role waiting on a question looks `idle` or `done` in Herdr, so the agent state alone doesn't show it.
3. `lead` answers only from settled facts: the user's request, `memory/` (especially `memory/decisions/`), or `TEAMS.md`. **If `lead` is uncertain too, it passes the question to the user unchanged**, together with the role's options and recommendation, and doesn't pick one itself.
4. `lead` writes the answer into the handoff, then sends it to the role:
   ```bash
   herdr agent prompt <role> "Answer to your question in .herdrteam/handoffs/<id>.md: <answer>. Continue." --wait --timeout 600000
   ```
5. If questions from several roles arrive together, `lead` collects them into one message to the user.
6. Herdr's `blocked` state (a CLI approval dialog) is different from this. It always goes to the user.

## 7. Report
`lead` updates the task README (status, the handoffs, the final deliverables and where they are), summarizes completed handoffs to the user with links into `work/<task-id>/`, and logs the cycle in `.herdrteam/roles/lead/logs/`. It records every decision the user made in `memory/decisions/`, and checks that the roles filed what they learned in `memory/` and updated the index.

## Onboard

### An agent joins the session
`lead` runs this every time **another** role's agent starts, including restarts. `lead` never runs it for itself, and other roles never run it.

```bash
herdr agent prompt <role> "You are <role>. Follow the Onboard section of .herdrteam/roles/<role>/AGENTS.md. Only read files; don't run herdr commands. Reply ready: <role>, then stop." --wait --timeout 120000
herdr agent read <role> --source recent-unwrapped --lines 40   # expect "ready: <role>" and its open handoffs
```

Before prompting, check that `.herdrteam/roles/<role>/AGENTS.md` exists (`test -f`). If it doesn't, don't start or prompt that role. Report it as `not ready: missing .herdrteam/roles/<role>/AGENTS.md`.

Send the onboarding prompt once. If the reply doesn't say `ready: <role>`, inspect the agent once, report it to the user, and don't delegate to it. Don't re-send the prompt in a loop.

### A new role joins the team
1. `lead` proposes the role name (`[a-z][a-z0-9_-]{0,31}`), responsibility, group (existing or new, at most 3 roles per group), and CLI to the user, then waits for approval.
2. Create `.herdrteam/roles/<role>/` with `AGENTS.md`, `logs/`, `context/` and `skills/` inside it. Add `CLAUDE.md` containing `@AGENTS.md` if the CLI is `claude`.
3. Add the role to `TEAMS.md`, both the roster and the delegation map, and update **Typical flow** below if it changes.
4. Start the agent and onboard it as described above.

### Temporary workers
When a task doesn't fit any role in `TEAMS.md`, `lead` creates a temporary worker with the right role, runs it, and removes it when the task is done. Examples: a one-off data migration in a Creative project, legal wording, a translation. No user approval is needed, but `lead` tells the user in one line: `tmp worker: tmp-<role> for <task-id>`. (A *permanent* role still needs approval; see *A new role joins the team*.)

1. **Name and define it.** Name it `tmp-<role>`, e.g. `tmp-translator`. It must match `[a-z][a-z0-9_-]{0,31}` and not clash with an existing agent. Write `.herdrteam/roles/tmp-<role>/AGENTS.md` from the role template, with a Mission, Responsibilities and "Not your job" that fit this one task. Add `temporary: true` and `task: <task-id>` under the title. Create `logs/` and `context/`, and add `CLAUDE.md` with `@AGENTS.md` if the CLI is `claude`.
2. **Pick the CLI.** Use `lead`'s own `--kind`, unless the user said otherwise for this kind of work.
3. **Start it in the `tmp` tab.** Temporary workers never share a group tab and never run in `lead`'s tab. Create a `tmp` tab if there isn't one (§3 *Start a role's agent*, with group `tmp`). It holds at most 3 workers; open `tmp-2` and so on for more. Record the tab and pane in `.herdrteam/roles/lead/context/panes.md` as created by `lead`, and add the worker to *Temporary workers* in `TEAMS.md`.
4. **Onboard it** as in *An agent joins the session* (send the prompt once, wait once).
5. **Delegate and verify** as for any role. It gets a normal handoff with `task:` and `workdir:`, saves output in `work/<task-id>/tmp-<role>/`, and asks `lead` when uncertain. It can file knowledge in `memory/` like any role.
6. **Remove it when its handoffs are done.** Run *An agent leaves the session* (Leave, quit the CLI, close the pane, and close the `tmp` tab if it is empty). Then move `.herdrteam/roles/tmp-<role>/` to `.herdrteam/roles/_archived/tmp-<role>-<task-id>/` and remove it from `TEAMS.md`. Its output stays in `work/`.

Keep at most 3 temporary workers running at once; queue the rest. If the same kind of temporary worker is needed a second time, `lead` suggests to the user that it become a permanent role.

## Leave

### An agent leaves the session
Use this at the end of the day, before a restart, or to free a pane:

```bash
herdr agent prompt <role> "Follow the Leave section of .herdrteam/roles/<role>/AGENTS.md." --wait --timeout 300000
herdr agent read <role> --source recent-unwrapped --lines 40   # expect "left: <role>"
```

After you confirm `left: <role>`, **quit the agent's CLI and close its pane**. Do this for every role pane, including panes the user opened.

1. Quit the CLI by sending its exit command as a prompt, without `--wait`:
   ```bash
   herdr agent prompt <role> "<exit command>"
   ```
   | `--kind` | Exit command |
   |---|---|
   | `claude` | `/exit` |
   | `codex` | `/quit` |
   | `opencode` | `/exit` |
   | `pi` | `/quit` |

   If you're not sure of a CLI's exit command, check its help. Don't guess repeatedly.
2. Confirm the CLI has exited: the role is gone from `herdr agent list`, or `herdr pane process-info <pane_id>` no longer shows the CLI. Check at most 3 times, a few seconds apart. If the CLI is still running, send `herdr pane send-keys <pane_id> C-c` twice and check once more.
3. Close the pane, even if the CLI didn't exit cleanly (closing ends the process). Then remove the pane from the registry.
   ```bash
   herdr pane close <pane_id>
   ```

When every role in a group has left, close the group's tab if it is still open, and remove it from the registry:

```bash
herdr tab close <tab_id>
```

If a role doesn't reply `left: <role>` within the timeout, inspect it once and ask the user before you quit it. Its handoffs or log may be half-written.

### The whole team leaves
1. `lead` sends Leave to every group, one tab at a time, temporary workers in the `tmp` tab included. It quits and closes each role's pane as above, then closes the group tabs. It archives each temporary worker as in *Temporary workers*, step 6.
2. `lead` runs its own Leave steps and writes the final team summary to its log in `.herdrteam/roles/lead/logs/`.
3. `lead` tells the user the team summary and says it is closing its own session.
4. Last, `lead` closes its own pane, which also ends its own CLI. This must be `lead`'s final command, because nothing after it runs:
   ```bash
   herdr pane close "$HERDR_PANE_ID"
   ```

### A role leaves the team
1. `lead` confirms with the user, then makes the agent leave the session as above.
2. Reassign the role's unfinished handoffs: set `to:` to the new owner and add a note.
3. Move `.herdrteam/roles/<role>/` to `.herdrteam/roles/_archived/<role>/`. Don't delete it, so logs and context are kept.
4. Remove the role from `TEAMS.md` and the delegation map, and update **Typical flow** below.

## Role-to-role rules
- When uncertain, ask the caller and don't guess. Only the user answers questions `lead` can't settle.
- Only `lead` delegates. Other roles never prompt each other directly. If a role needs another role's help, it sets `status: blocked` and explains in Result, and `lead` re-routes the work, to a temporary worker if no role fits.
- Use `--no-focus` for every background tab and pane, so the user's focus stays on `lead`. Close only this team's panes and tabs, and only during Leave. Never close workspaces.
- Respect the layout: `lead` stays alone in its tab, and each role stays in its group's tab.
- Agent names = role names (see TEAMS.md).

## Typical flow for this team
{{A numbered example pipeline specific to this project type and roster, e.g. Development: lead → architect (design) → developer (implement) → tester + reviewer (parallel) → lead (integrate & report). Creative: lead → researcher → writer + designer (parallel) → editor → lead.}}
````

---

## Step 5 — Confirm

When the files are written:

1. Print the created tree, for example with `find .herdrteam work memory | sort`.
2. Show the final roster table, with each role, its group or tab, and its CLI, followed by the layout diagram.
3. Tell the user how to start: open Herdr, open a new tab just for `lead`, and start the `lead` agent in that tab's pane with `herdr agent start lead --kind <kind> --pane <pane_id>` (or run the lead CLI directly in a Herdr pane), and tell it to *"Follow the Onboard section of .herdrteam/roles/lead/AGENTS.md."* `lead` then brings up and onboards the whole team in its group tabs, with no task needed, reports who is ready, and waits for the first request.
4. Tell the user how to stop: say *"team, leave"* to `lead`. It runs the Leave steps in `.herdrteam/WORKFLOW.md` for every role: each role saves its work, then `lead` quits its CLI and closes its pane and group tab. Last, `lead` runs its own Leave, reports to the user, and closes its own pane.

Finally, ask the user: *"Onboard the team now?"*
- **Yes, and you are `lead`** (inside Herdr, `HERDR_ENV=1`): run the team onboarding in `.herdrteam/WORKFLOW.md`, then report team status and wait. Don't assign any tasks.
- **Yes, but you are not running in Herdr:** explain how to start `lead` (item 3 above). `lead` onboards the team when it starts.
- **No:** stop here. Never start agents without the user's yes.
