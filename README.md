# Copilot Agent Submission Specifications

Structured intake forms for crafting high-quality prompts across each Microsoft 365 Copilot capability area.

## What's in here

| Folder | Contents |
|---|---|
| `specs/` | One markdown spec per capability area (purpose, activities, required + optional fields, example, expected output) |
| `templates/` | Copy-paste prompt templates with bracket placeholders |
| `docs/` | Master Word document containing all 19 specs in one file, plus the session continuity pattern |
| `examples/` | (Optional) worked examples and sample outputs |
| `.github/` | Issue templates for proposing new areas or refining existing ones |

## The Universal Prompt Pattern

Every well-formed Copilot prompt follows a five-slot structure. The more slots you fill, the sharper the output.

```
[ACTION] + [SUBJECT/TARGET] + [SCOPE & FILTERS] + [OUTPUT FORMAT] + [TONE / CONSTRAINTS]
```

- **Action** — the verb (search, draft, send, summarize, schedule, create…)
- **Subject / Target** — what or who the action operates on
- **Scope & Filters** — time window, location, people, topic boundaries
- **Output format** — bullets, table, document, chart, email, etc.
- **Tone / Constraints** — formal/casual, length, must-haves, avoid-lists

## Capability index

| # | Area | Spec | Template |
|---|---|---|---|
| 01 | Email (Outlook) | [spec](specs/01-email.md) | [template](templates/01-email.md) |
| 02 | Calendar — Scheduling | [spec](specs/02-calendar-scheduling.md) | [template](templates/02-calendar-scheduling.md) |
| 03 | Teams | [spec](specs/03-teams.md) | [template](templates/03-teams.md) |
| 04 | Meeting Intelligence | [spec](specs/04-meeting-intelligence.md) | [template](templates/04-meeting-intelligence.md) |
| 05 | Files (SharePoint / OneDrive) | [spec](specs/05-files.md) | [template](templates/05-files.md) |
| 06 | Document Creation — Word | [spec](specs/06-document-word.md) | [template](templates/06-document-word.md) |
| 07 | Document Creation — Excel | [spec](specs/07-document-excel.md) | [template](templates/07-document-excel.md) |
| 08 | Document Creation — PowerPoint | [spec](specs/08-document-powerpoint.md) | [template](templates/08-document-powerpoint.md) |
| 09 | Document Creation — PDF | [spec](specs/09-document-pdf.md) | [template](templates/09-document-pdf.md) |
| 10 | Daily Briefing | [spec](specs/10-daily-briefing.md) | [template](templates/10-daily-briefing.md) |
| 11 | Stakeholder Communications | [spec](specs/11-stakeholder-comms.md) | [template](templates/11-stakeholder-comms.md) |
| 12 | Calendar Management | [spec](specs/12-calendar-management.md) | [template](templates/12-calendar-management.md) |
| 13 | People & Org | [spec](specs/13-people-org.md) | [template](templates/13-people-org.md) |
| 14 | Deep Research | [spec](specs/14-deep-research.md) | [template](templates/14-deep-research.md) |
| 15 | Power BI | [spec](specs/15-power-bi.md) | [template](templates/15-power-bi.md) |
| 16 | Image Search | [spec](specs/16-image-search.md) | [template](templates/16-image-search.md) |
| 17 | Visual Rendering (Adaptive Cards) | [spec](specs/17-visual-rendering.md) | [template](templates/17-visual-rendering.md) |
| 18 | Recurring / Scheduled Tasks | [spec](specs/18-recurring-tasks.md) | [template](templates/18-recurring-tasks.md) |
| 19 | Skills & Customization | [spec](specs/19-skills-customization.md) | [template](templates/19-skills-customization.md) |

## Session Continuity Pattern (Start Here if You Do Multi-Session Work)

> **This is the single most important thing in this repo if you work on projects that span more than one session.**

Copilot sessions don't share memory by default. That means every time you come back, you re-brief from scratch — what was built, where it lives, what's still open, what decisions were made. Over a multi-week project, this burns real time and leads to dropped tasks.

The **[Cowork Session Continuity Pattern](docs/cowork-memory-pattern.md)** fixes this with two self-perpetuating memory rules:

| Rule | What it does |
|---|---|
| **End-of-task wrap-up** | At the end of every task, Copilot silently writes a project state file — status, file locations, decisions, incomplete work, and likely next steps |
| **Start-of-session resume** | On the next session, Copilot reads that state file, picks up where you left off, and automatically re-runs any interrupted tasks (or flags what's missing) |

**Once installed, it runs forever** — no reminders, no re-briefing, no dropped work. It also includes:

- A **memory cleanup rule** to prevent stale project files from accumulating
- A **working-file persistence warning** so interrupted tasks flag which inputs may need regenerating
- An **optional stop-hook backstop** (a Python script + hook config) that blocks session exit if the wrap-up hasn't been written — making the pattern bulletproof, not just best-effort
- **Dual re-entry protection** (payload flag + timestamp debounce) to prevent infinite loops in the hook

### Quick install

In a fresh Cowork session, paste:

> *"Please set up the following memory pattern so that future sessions can resume cold."*

Then paste the contents of **[cowork-memory-pattern.md](docs/cowork-memory-pattern.md)** — it contains the exact files and instructions. Takes 30 seconds.

### Why this matters

Without it, every session starts cold. With it, you can close your laptop on Friday, open it Monday, and say "add a slide on Q3 pipeline" — and Copilot already knows the deck, the style, the data sources, and the open items.

## How to use

1. **Pick the area** that matches the work you need to do.
2. **Open the spec** to see the required and optional fields.
3. **Copy the matching template** and fill in the brackets.
4. **Send the completed prompt** to Copilot.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to propose new capability areas or refine existing ones.

## License

MIT — see [LICENSE](LICENSE).
