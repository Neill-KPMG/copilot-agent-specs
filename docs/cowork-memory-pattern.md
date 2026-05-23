# Cowork Session Continuity Pattern

A workflow for getting Cowork (Copilot in this environment) to resume multi-session projects cold — no recap required — and to pick up unfinished work automatically.

## The Problem

Cowork sessions don't share working memory by default. If you spend a week building something across multiple sessions:

- You end up re-briefing every time you come back.
- Output files show *what* exists, but not *why* it's there or what's still open.
- If a task gets interrupted (context runs out, error, you close the tab), it can be silently dropped.

## The Pattern

Two memory files set up a closed loop:

1. **End-of-task rule** — at the end of every task, Cowork writes a per-project state memory: what was done, where files live, decisions made, what didn't finish.
2. **Start-of-session rule** — when Cowork loads that memory on next session, if anything's listed as incomplete, it either re-runs it (if inputs are available) or flags it up front.

The pattern is self-perpetuating — once installed, every future session uses it without you having to remind Cowork.

## How to Install It

In a fresh Cowork session, paste the following message:

> Please set up the following memory pattern so that future sessions can resume cold. Create three files in the auto-memory directory and update MEMORY.md to index them. The three files are below — write them verbatim.

Then paste the three file blocks below the prompt. Cowork will create the files in its memory directory (something like `/mnt/workspace/agent-state/projects/.../memory/`). After it confirms, the pattern is live for every future session in that workspace.

---

## File 1 — `feedback_task_wrap_up.md`

```markdown
---
name: Write a project-state memory at the end of every task
description: Standing instruction to update a per-project state memory whenever a task wraps up, so future sessions can resume cold without recap
type: feedback
---

At the end of every task the user gives you, update (or create) a `project_<name>.md` memory capturing the current state of that project — without being asked. Then update MEMORY.md if the file is new.

**Why:** The user works across multi-session projects and may return days later. Output files show *what* exists but not *why* or *what's still open*. Without a state memory, the user has to re-brief from scratch every session.

**How to apply:**
- Trigger: whenever a discrete task finishes — a deliverable shipped, a decision made, a correction applied. Not every single tool call; once per task.
- Pick the project name from context (e.g. `project_<short_name>.md`). One state memory per project, not per task — overwrite the existing one.
- Use this template:
  - **Status:** one line on where we are
  - **Files:** key paths in `output/` (source scripts in `working/` if still load-bearing)
  - **Recent decisions:** anything that would surprise a fresh-me — corrections, vocab choices, deliberate omissions
  - **Incomplete:** any task the user asked for that didn't finish (interrupted, errored out, ran out of context). List each with: what was asked, what inputs are available, what's missing. Omit the section if nothing is incomplete.
  - **Next:** what the user is likely to ask for next, or open threads
- Keep it tight — under ~40 lines. This is a handoff note, not an archive.
- If the task was trivial (a quick lookup, a one-line answer), skip the update — only write when state actually changed.
- Do this silently as part of wrapping up; don't announce "I'm updating memory now."
```

---

## File 2 — `feedback_resume_incomplete.md`

```markdown
---
name: Resume incomplete work at session start
description: When a project memory's Incomplete section lists unfinished tasks, either re-run them automatically (if inputs are sufficient) or flag them to the user at the start of the session
type: feedback
---

At the start of any session, if the project memory loaded via MEMORY.md has an **Incomplete** section listing unfinished tasks, act on it before waiting for new instructions.

**Why:** The user expects continuity — if they asked for something and it didn't finish (interrupted, errored, context ran out), they want it resumed, not silently dropped. They'd rather you just re-run it than ask "what should I do today?"

**How to apply:**
- **If all inputs are available** (source files, prior decisions, clear spec in the memory) → re-run the work without asking. Announce briefly ("Picking up the X that didn't finish last session") then do it.
- **If inputs are missing or ambiguous** → flag it at the top of your first response: "Last session this didn't complete: [task]. I need [missing input] to finish it." Don't bury the flag mid-message and don't assume the user remembers.
- **Multiple incomplete items:** list them all up front, then proceed with the ones you can run and ask about the rest.
- Once an incomplete item finishes, remove it from the project memory's Incomplete section as part of the normal wrap-up.
- If the user explicitly says "drop it" or moves on, also remove it.
```

---

## File 3 — `MEMORY.md` (additions)

Add these two lines to `MEMORY.md`. If `MEMORY.md` doesn't exist yet, create it with just these lines:

```markdown
- [Write a project-state memory at the end of every task](feedback_task_wrap_up.md) — standing rule: update `project_<name>.md` when a task wraps up so future sessions resume cold
- [Resume incomplete work at session start](feedback_resume_incomplete.md) — if a project memory lists unfinished tasks, re-run them when inputs are available, flag them when inputs are missing
```

---

## How It Works in Practice

**Session 1 — first task on a new project:**
You ask Cowork to build a deck. It builds the deck. At the end, it silently creates `project_<your_project>.md` with status, file paths, decisions, and likely next steps. You close the session.

**Session 2 — coming back two weeks later:**
You open a fresh Cowork session. `MEMORY.md` auto-loads, which pulls in your project state memory. Cowork already knows what was built, where it lives, what was decided. You can just say "add a slide on X" and it picks up.

**Session 3 — task got interrupted last time:**
Last session, Cowork was mid-rebuild when context ran out. At wrap-up it wrote `Incomplete: rebuild of consultant deck — source script at working/foo.js, methodology fix still needed at slides 5-8`. You open a fresh session. Cowork sees the incomplete item, sees the inputs are all there, and just re-runs it — telling you up front that's what it's doing.

## Scope and Caveats

- **Workspace-scoped:** Memory persists per workspace. If you open a different workspace in Cowork, it has its own memory. The pattern doesn't follow you across workspaces unless you install it in each one (which is what this file is for).
- **Cowork follows the rule because it's in feedback memory:** This isn't a hardcoded feature — it works because you've told Cowork (via the feedback memory) to do this at task end and session start. Cowork loads those instructions every session and applies them.
- **You can edit the rules:** If you want a different template, different triggers, or different re-entry behaviour, just ask Cowork to update the relevant feedback memory. The pattern is yours to tune.
- **Trivial tasks are skipped:** The rule explicitly says don't write a state memory for quick lookups or one-line answers — only when state actually changed. This stops the memory from filling with noise.

## One More Tip

When you start a brand new project in this workspace, the first time you give Cowork a substantive task, it'll create `project_<name>.md` for that project. If you want to seed the project name yourself, just say so: "Use `project_marketing_site.md` for this work." Otherwise Cowork picks a sensible name from context.
