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

Then paste the three file blocks below the prompt. Cowork will create the files in its memory directory. The exact path varies by environment (commonly `/mnt/workspace/agent-state/projects/.../memory/`), but you don't need to know it — Cowork resolves the correct location automatically when you ask it to write to memory. After it confirms, the pattern is live for every future session in that workspace.

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
  - **Incomplete:** any task the user asked for that didn't finish (interrupted, errored out, ran out of context). List each with: what was asked, what inputs are available, what's missing. If the task depends on files in `working/`, note that these may not persist across sessions and flag which inputs would need to be regenerated. Omit the section if nothing is incomplete.
  - **Next:** what the user is likely to ask for next, or open threads
- Keep it tight — under ~40 lines. This is a handoff note, not an archive.
- If the task was trivial (a quick lookup, a one-line answer), skip the update — only write when state actually changed.
- Do this silently as part of wrapping up; don't announce "I'm updating memory now."
- **Memory cleanup:** if more than 10 `project_*.md` files exist, archive or remove the oldest stale ones to keep the memory directory lean and MEMORY.md under its line limit.
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
Last session, Cowork was mid-rebuild when context ran out. At wrap-up it wrote `Incomplete: rebuild of consultant deck — source script at working/foo.js (may need regenerating if working/ was cleared), methodology fix still needed at slides 5-8`. You open a fresh session. Cowork sees the incomplete item, checks whether the inputs still exist, and either re-runs it directly or tells you what needs to be recreated first.

## Scope and Caveats

- **Workspace-scoped:** Memory persists per workspace. If you open a different workspace in Cowork, it has its own memory. The pattern doesn't follow you across workspaces unless you install it in each one (which is what this file is for).
- **Cowork follows the rule because it's in feedback memory:** This isn't a hardcoded feature — it works because you've told Cowork (via the feedback memory) to do this at task end and session start. Cowork loads those instructions every session and applies them.
- **You can edit the rules:** If you want a different template, different triggers, or different re-entry behaviour, just ask Cowork to update the relevant feedback memory. The pattern is yours to tune.
- **Trivial tasks are skipped:** The rule explicitly says don't write a state memory for quick lookups or one-line answers — only when state actually changed. This stops the memory from filling with noise.
- **Concurrent sessions:** If two sessions run against the same workspace at the same time, both could overwrite the same `project_*.md` file. Avoid running parallel sessions on the same project, or name the project memories differently per session.

## One More Tip

When you start a brand new project in this workspace, the first time you give Cowork a substantive task, it'll create `project_<name>.md` for that project. If you want to seed the project name yourself, just say so: "Use `project_marketing_site.md` for this work." Otherwise Cowork picks a sensible name from context.

---

## Optional — Stop-hook backstop

The pattern above relies on Cowork following its own feedback memory. That works almost all the time, but a Stop hook makes it bulletproof: at the moment Cowork tries to end a turn, the hook compares the freshness of `output/` against your project memory files and — if output is fresher — blocks the stop with a reminder to write the wrap-up.

It's optional. Skip this section if you're happy with the soft-rule version.

### How to install

In a fresh Cowork session, paste:

> Please install the Stop-hook backstop for the cowork memory pattern. Create the script and settings files below verbatim. Both live under `/mnt/user-config/.claude/` so they persist across sessions.

Then paste the two file blocks below.

### File A — `/mnt/user-config/.claude/hooks/wrap-up-check.py`

```python
#!/usr/bin/env python3
"""
Stop-hook backstop for the cowork session-continuity memory pattern.

If files under /mnt/workspace/output/ have been modified more recently than any
project_*.md memory file, block stop and remind the model to write the
end-of-task wrap-up. Designed to be conservative: stays quiet when nothing
substantive happened, when memory is already current, or when re-entered
(stop_hook_active).
"""
import json
import sys
import time
from pathlib import Path

OUTPUT_DIR = Path("/mnt/workspace/output")
MEMORY_ROOT = Path("/mnt/workspace/agent-state/projects")
RECENT_WINDOW_SECONDS = 3600   # only nag if output was modified in the last hour
GAP_TOLERANCE_SECONDS = 60     # ignore tiny clock-skew gaps
DEBOUNCE_FILE = Path("/tmp/cowork_wrapup_hook_last_fire")
DEBOUNCE_SECONDS = 120         # don't fire again within 2 minutes of last block


def newest_mtime(paths):
    times = []
    for p in paths:
        try:
            times.append(p.stat().st_mtime)
        except OSError:
            continue
    return max(times) if times else 0.0


def main():
    try:
        payload = json.loads(sys.stdin.read() or "{}")
    except Exception:
        sys.exit(0)

    # Re-entry guard (two layers):
    # 1. Hook-runner flag: if the runner sets stop_hook_active on re-entry,
    #    exit cleanly to avoid an infinite block loop.
    # 2. Timestamp debounce: if the hook fired within the last 2 minutes,
    #    skip — this covers hook runners that don't inject stop_hook_active.
    if payload.get("stop_hook_active"):
        sys.exit(0)

    if DEBOUNCE_FILE.exists():
        try:
            last_fire = DEBOUNCE_FILE.stat().st_mtime
            if time.time() - last_fire < DEBOUNCE_SECONDS:
                sys.exit(0)
        except OSError:
            pass

    if not OUTPUT_DIR.exists():
        sys.exit(0)

    output_files = [p for p in OUTPUT_DIR.rglob("*") if p.is_file()]
    if not output_files:
        sys.exit(0)

    newest_output = newest_mtime(output_files)
    if time.time() - newest_output > RECENT_WINDOW_SECONDS:
        sys.exit(0)

    project_memories = (
        list(MEMORY_ROOT.rglob("memory/project_*.md"))
        if MEMORY_ROOT.exists() else []
    )
    newest_memory = newest_mtime(project_memories)

    if newest_output - newest_memory <= GAP_TOLERANCE_SECONDS:
        sys.exit(0)

    # Write debounce marker so the hook won't fire again for 2 minutes
    try:
        DEBOUNCE_FILE.touch()
    except OSError:
        pass

    reason = (
        "Wrap-up backstop: files under output/ have been modified more recently "
        "than any project_*.md memory file. Per the standing end-of-task rule, "
        "update (or create) the relevant project memory now so future sessions "
        "can resume cold. If this turn was trivial and no state actually "
        "changed, briefly acknowledge that and stop — the backstop will not "
        "fire again on re-entry."
    )
    print(json.dumps({"decision": "block", "reason": reason}))
    sys.exit(0)


if __name__ == "__main__":
    main()
```

### File B — `/mnt/user-config/.claude/settings.json`

If `settings.json` already exists in user-config, merge the `hooks.Stop` entry into it instead of overwriting.

```json
{
  "hooks": {
    "Stop": [
      {
        "type": "command",
        "command": "python3 /mnt/user-config/.claude/hooks/wrap-up-check.py"
      }
    ]
  }
}
```

> **Note:** The schema above uses the flat hook format (`Stop → [{ type, command }]`). If your environment uses nested hook definitions (`Stop → [{ hooks: [{ type, command }] }]`), adjust accordingly. When in doubt, ask Cowork to inspect your existing `settings.json` and merge the entry for you.

### How it behaves

- **Fires when:** something in `output/` was modified in the last hour AND is newer than every `project_*.md` memory file by more than 60 seconds.
- **Stays quiet when:** `output/` is empty, no recent edits, memory is already up to date, the hook is being re-entered after a previous block (`stop_hook_active` flag), or the hook already fired within the last 2 minutes (timestamp debounce via `/tmp` marker).
- **Re-entry protection:** Two layers — the `stop_hook_active` payload flag (if your hook runner supports it) and a file-based debounce timer (works universally). This prevents infinite block loops regardless of your environment.
- **Tunables:** `RECENT_WINDOW_SECONDS` (default 3600), `GAP_TOLERANCE_SECONDS` (default 60), and `DEBOUNCE_SECONDS` (default 120) at the top of the script.

### How to disable

Either rename `/mnt/user-config/.claude/settings.json` or remove the `Stop` entry from it. The script alone, with no registration, does nothing.

### Caveats

- Paths are hard-coded to `/mnt/workspace/output/` and `/mnt/workspace/agent-state/projects/`. If your environment uses different paths, edit the two constants at the top of the script.
- The hook is heuristic, not semantic — it only knows mtimes. If you save a project memory but then touch an unrelated file in `output/`, it'll fire again. The fix is usually to either ignore it (re-entry is silent) or update the memory.
- Memory itself isn't portable across machines — the hook just enforces the pattern. You still need to install File 1, File 2, and the `MEMORY.md` entries on each new machine.
