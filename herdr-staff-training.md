# Herdr Staff Training

You are training one or more roles of an existing Herdr team. You will interview the user in six short steps, then edit the trained roles' instruction files, write a playbook into each trained role's `skills/` folder, and run a tabletop check that decides whether the role counts as trained.

**You only read and edit files. Never run any `herdr` command.** The `herdr` CLI belongs to `lead` alone (see the team-wide rules in `.herdrteam/AGENTS.md`). This file works with the team stopped.

Work through the steps in order. **Ask one step at a time and stop to wait for the user's answer before continuing.** Do not edit any file until Step 4.

The template ships every role's `skills/` folder empty. Filling it is what this procedure does: a training run ends with a real playbook in the trained role's folder, not just edited prose.

## How the steps run

Step 1 picks the roles and runs once. **Steps 2, 3 and 5 then run once per role, one role at a time, in the order the user named them**: role A through Steps 2 → 3 → 5, then role B through 2 → 3 → 5, and so on. Step 4 runs **once, at the end**, after every named role has had its edit set approved and has passed its Step 5 check. Nothing is written to disk before Step 4: until then you are proposing, not changing.

If a role fails Step 5 twice, stop and report it rather than looping; the other roles still go through Step 4.

---

## Preconditions

Check that `.herdrteam/roles/` exists in the project root and contains at least one role folder. If it doesn't, the team isn't set up yet: tell the user to follow `herdr-team-setup.md` first, and stop.

Ignore any `_archived/` folder under `.herdrteam/roles/` — those are roles that were removed from the team and are not trainable.

For each role you are asked to train, check that `.herdrteam/roles/<role>/AGENTS.md` exists. If it doesn't, report it and skip that role; don't create role files here.

---

## Step 1 — Which roles to train

List the folders in `.herdrteam/roles/`. For each one, read the first line of its `AGENTS.md` and its Mission, and write a one-line summary. Present the list, for example:

```
writer     — drafts copy, scripts, and long-form content
designer   — visual direction, layouts, asset specs, prompts
editor     — clarity, tone, and the final quality pass
```

Then ask:

> Which roles do you want to train? You can name one or several. (`lead` can be trained, but its orchestrator rules live in `.herdrteam/WORKFLOW.md`, which this procedure does not rewrite.)

Wait for the answer. Record the named roles and their order; they are processed one at a time through Steps 2, 3 and 5.

---

## Step 2 — Diagnose and suggest

For the current role, read:

- its `AGENTS.md` in full
- every file in `.herdrteam/roles/<role>/logs/`
- handoffs addressed to it in `.herdrteam/handoffs/` (any status)

**If there is real evidence** (at least one log or handoff), report what you actually found, as concrete lines: where the role overstepped its **Not your job** list, where it stalled with `status: question`, where it produced nothing against its **Outputs / definition of done**. Cite the log or handoff for each line.

**If there is no evidence** — the logs folder holds only `.gitkeep` and no handoff names this role — say exactly that, in one line, and say that everything below therefore rests on the role file alone. Do not invent findings to fill this section. A first training run on a new team normally has no evidence, and that is a normal state, not a failure.

Then propose, as a set of proposals for the user to approve in Step 3:

- specific edits per section — **Mission**, **Responsibilities**, **Not your job**, **Inputs**, **Outputs / definition of done** — each one quoting the current text it replaces and citing what justified the change, or marked as coming from the role file alone
- one candidate playbook: a procedure the role repeats or would stumble on, given as a title, a one-line purpose, and a 4–8 step outline. Full prose comes later, in Step 4.

---

## Step 3 — Approve the change set

Ask:

> Approve these changes for `<role>`? You can **approve**, **edit**, **add** (your own instruction or playbook point), or **remove** suggestions.

Wait for the answer. Apply the user's changes and show the final change set for this role — the exact section rewrites and the playbook's title, purpose and outline — and confirm it before moving to the next role's Step 2.

Do not write anything to disk in this step. The change set is applied in Step 4, after every role has been through Steps 2, 3 and 5.

---

## Step 4 — Apply

Run this once, after every named role has an approved change set and a passing Step 5 check.

Before writing, check the target files. If a role's `AGENTS.md` was edited outside this procedure since you read it, show the difference and ask before overwriting. Never delete a file, and never overwrite a `skills/` or `memory/decisions/` file that already exists: if a playbook or decision file has the same name, add a suffix and tell the user.

For each trained role:

1. Apply the approved edits to `.herdrteam/roles/<role>/AGENTS.md` as targeted section replacements, not a full rewrite. Leave **Onboard** and **Leave** byte-for-byte unchanged — the team's startup and shutdown depend on their literal strings, including `ready: <role>`, `not ready: <role> — missing <path>`, `left: <role>`, and the sentence that onboarding only reads files. Leave the section headings and their order as the template wrote them.
2. Write the approved playbook to `.herdrteam/roles/<role>/skills/<slug>.md` — a short, checkable procedure, not an essay. Slug rules: lowercase `[a-z0-9-]`, derived from the title, unique in that folder.
3. Record the training in `memory/decisions/` as `YYYYMMDD-<slug>.md` — context, what changed and why, who decided — following the rules in `memory/README.md`, and add one line to its index. Include the Step 5 result here; do not open a second decision file for it.

---

## Step 5 — The tabletop check

This is a check of the **files**, not of the role. The team is stopped, so nothing runs and nothing is observed. Say so to the user in one line before you start; do not present this as a live test. Verifying that a role actually behaves this way needs a real handoff through `lead`, which is out of scope here.

What you are checking is whether the trained files are self-consistent and specific enough that an agent following them literally could not do anything else. That is decidable from the text.

1. **Write one exercise task** in handoff shape — a **Task** plus at least two checkable **Acceptance criteria** — that sits inside this role's **Mission** and touches none of its **Not your job** entries. Save it to `.herdrteam/roles/<role>/context/training-<YYYYMMDD>.md` together with the results below, so the check leaves evidence instead of vanishing from the chat.
2. **Simulate the Onboard section literally.** Go through the role's **Onboard** steps in written order and write down, as a numbered list, the concrete actions each step forces: which files it must open, what it must reply, and where it must stop. Quote the Onboard line that forces each action. You are transcribing what the text compels, not narrating what a helpful agent might choose to do. (The template's own Onboard step 2 puts it this way: "Check `.herdrteam/roles/{{role}}/skills/` for the playbooks you have." That is the kind of line to quote.)
3. **Simulate the first work step.** From the state Onboard leaves the role in, write the first three actions an agent taking the exercise task would take under the trained **Responsibilities**, **Inputs** and **Outputs**, plus the playbook.
4. **Check each action against the files.** An action passes only if a specific line in the role's `AGENTS.md`, the playbook, or the exercise's acceptance criteria makes it the action that text forces. Record the line for each.

- **Pass:** every action in both simulations is traceable to a line, the exercise task stays inside Mission and clear of **Not your job**, and no action is one the role's own rules would forbid.
- **Fail:** any action you cannot trace to a line, any action another role's **Not your job** claims, or any acceptance criterion the trained files do not tell the role how to satisfy. Name the failing line, return that role to Step 2 with it, and leave the other roles' results as they are. Two failures on the same role: stop and report it to the user instead of iterating.

Record the outcome — pass or fail, the exercise task, and the numbered traces — in the `memory/decisions/` file from Step 4. A role counts as trained only on a pass.

---

## Step 6 — Report

Tell the user, per role: what changed in `AGENTS.md`, which playbook was written and where, and the tabletop result. Then say what happens next: the role's **Onboard** step 2 checks its `skills/` folder, so the playbook is in front of the role at its next session; the first real handoff to a trained role is where the training meets live work, and that is the first time any of this is actually tested.

---

## What this does not do

- It does not start, stop, or inspect a running team, and it runs no `herdr` command. Everything here is file reads and file edits.
- It does not rewrite `.herdrteam/WORKFLOW.md`, the handoff convention, or `lead`'s orchestration procedures. Training `lead` means training its role file only.
- The Step 5 check reads the trained files; it does not observe the role doing work. A pass means the instructions are consistent and specific, not that the role performs well.
- It does not create roles, delete roles, or touch `memory/` categories other than `decisions/`.
