---
name: project-planner
description: Scaffold an interactive product roadmap (Gantt + tickets + system map + flow) into any project. Inspects the project (CLAUDE.md, README, recent commits, research/ docs), drafts a phase + ticket structure, and drops a customized planner.html into ./planner/. Use when user types /project-planner, asks for a "project planner", "roadmap", "Gantt chart", "ticket board", or wants to turn a plan into a visual sprint board. Replicates the WastePlace / Rhys Landscaping / CRM Cabin planner pattern. NOT for task management, sprint execution, or daily standup facilitation.
---

# project-planner

Scaffolds a self-contained interactive roadmap into any project. Single HTML file with Gantt chart, ticket table, status filters, system map, and business-flow narrative. State persists in browser localStorage. No backend required.

## Quick start

User in their project directory types `/project-planner`. Output: `./planner/planner.html` opens in the browser, fully customized to their project.

## Workflow

### Step 1 — Detect project state

Before asking the user anything, gather context from the current working directory:

- Read `CLAUDE.md`, `AGENT.md`, `README.md`, `package.json` (or equivalent) if present.
- List `research/`, `docs/`, `.claude/` directories.
- Run `git log --oneline -20` to see recent work (if it's a git repo).
- Note the directory basename — likely close to the project name.

This becomes your evidence base for drafting the planner.

### Step 2 — Lock the metadata

Use `AskUserQuestion` to collect the things you can't infer with confidence. Always offer your inferred default as the first option, marked `(Recommended)`. Questions to ask (combine into one AskUserQuestion call):

1. **Project name** — what gets shown in the header h1. If `package.json#name` or repo name is sane, propose that.
2. **Day 0 date** — calendar date that maps to project day 0. Default = today.
3. **Project duration** — short (<30 days) / medium (30-100) / long (100+). Maps to `DAY_COUNT` and `DEADLINE_DAY`.
4. **Owner cast** — the primary slot uses the fixed internal id `kado` (a template constraint — only display labels are configurable). Set its display name to the skill owner; name the other 2 collaborator slots. Default ids: `kado` + `collab` + `claude` + `external`.

If user runs `/project-planner --quick` or types something like "just use defaults", skip the AskUserQuestion and infer everything from context.

### Step 3 — Draft the phases

Default phase shape: **scaffold → foundation → build → launch** (4 phases). For most projects this works. Override only if the project has a domain-specific structure (e.g., a research project might use discover/analyze/synthesize/publish).

For each phase, decide:

- Label (shown in filter chip + Gantt swim lane)
- Day range (which days the phase spans)
- 1-3 representative tickets (you'll expand in Step 4)
- 1 milestone

### Step 4 — Draft the SEED_TASKS

This is the **highest-value step**. The planner is only useful if its tickets reflect the actual work. Drafting principles:

- **Target ~30-50 tickets total.** Fewer = useless. More = overwhelming. Aim for ~10/phase.
- **Each ticket has a `deliverable`** that names the artifact / verifiable outcome (not just "do thing X").
- **Tag the critical path** (`is_critical_path: true`) on the chain from M1 → M2 → … through the milestones.
- **Mark milestones** (`is_milestone: true, duration_hours: 0`) with the ★ glyph for visibility.
- **Flag the launch milestone** with `is_launch: true` — on exactly one milestone, the gate your deadline measures. Its diamond renders red and the launch-deadline banner keys off it (via `findLaunchMilestone()`). Set `-DeadlineDay` to that milestone's day index + 1 (see the DEADLINE_DAY convention in Common pitfalls).
- **Set `default_status: 'done'`** on anything that's already completed (with `default_notes` summarizing what shipped).
- **Set `default_status: 'in_progress'`** on anything actively running.
- **Use `dependencies: ['ID1','ID2']`** to draw arrows between tickets — Critical Path computation uses these.

Source tickets from: research/ docs (each major decision often = a ticket), CLAUDE.md / AGENT.md (each numbered step = a ticket), recent git commits (each major thread = a ticket cluster), the actual roadmap if one exists.

Don't write tickets the user can't act on. If you don't know what something means, ask.

### Step 5 — Run the scaffolder

```powershell
& "$env:USERPROFILE\.claude\skills\project-planner\scripts\scaffold-planner.ps1" `
    -ProjectPath "<cwd>" `
    -ProjectName "<name>" `
    -ProjectSlug "<slug>" `
    -ProjectTagline "<one-liner>" `
    -DayZeroDate "<YYYY-MM-DD>" `
    -DayCount <int> `
    -DeadlineDay <int> `
    -OwnerALabel "<label for primary, e.g. '<You> - build (primary)'>" `
    -OwnerBLabel "<label>" `
    -OwnerCLabel "<label>" `
    -OwnerDLabel "<label>" `
    -OwnerAName "<YourName>" `
    -OwnerBName "<name>" `
    -OwnerCName "<name>" `
    -OwnerDName "External" `
    -Phase1Label "<e.g. 'Scaffold (Wk 0)'>" `
    -Phase2Label "<e.g. 'Foundation (Wk 1-2)'>" `
    -Phase3Label "<e.g. 'Build (Wk 3-12)'>" `
    -Phase4Label "<e.g. 'Launch (Wk 13+)'>"
```

The script copies the template to `./planner/planner.html` and applies all the simple token substitutions.

### Step 6 — Replace SEED_TASKS, map, and flow

The scaffolder leaves these as scaffolds with `{{P1_T1_NAME}}` etc. placeholders. **You** replace them via `Edit` operations, using the drafts from Steps 3-4.

- `SEED_TASKS = [` … `]` block: replace the entire array with your ~30-50 tickets.
- `<h2>System map — {{PROJECT_NAME}}</h2>` section: replace the placeholder `map-section` with one `map-section` div per major subsystem, status-coded nodes inside.
- `<h2>Business logic flow — {{PROJECT_NAME}}</h2>` section: replace the placeholder steps with 5-8 `<h3>` + `<p>` blocks narrating the journey.

If a section doesn't make sense yet (e.g., no system architecture is clear), leave a brief placeholder like *"To be filled when architecture solidifies."*

### Step 7 — Verify

After the edits, sanity-check:

- No `{{TOKEN}}` strings remain (grep the file).
- `SEED_TASKS` array opens and closes cleanly with `];`.
- All `dependencies: ['IDx']` references point to real IDs in the array.
- The first task's `start_day: 0` and the last milestone's `start_day` ≤ `DAY_COUNT - 1`.
- No orphaned object literals outside the array (caused by partial edits — see Step 8 below).

### Step 8 — Open in browser

```powershell
Start-Process "./planner/planner.html"
```

End by telling the user: planner is at `./planner/planner.html`, state persists in browser localStorage under `<project_slug>_planner_v1_state`, click Export to back up.

## Common pitfalls

- **Replacing SEED_TASKS via a single Edit can leave orphan tasks** if the old_string doesn't capture the entire array. Match `const SEED_TASKS = [` through the closing `];` exactly, or use a python helper that does index-based slicing.
- **Owner identifiers in the JS are fixed** (`kado`, `dogra`, `claude`, `external`) — don't try to rename them. Only the **display labels** change per project. If a project genuinely needs a 5th owner, that's a template extension job, not a per-project customization.
- **Phase identifiers are also fixed** (`scaffold`, `foundation`, `build`, `launch`). Change phase **labels** freely, but if a project needs different phase concepts, several `const phases = ['scaffold','foundation','build','launch']` renderer arrays (grep that literal — currently ~L2166 / L2394 / L2675) and the `state.filters.phases` Set (~L1411) reference these identifiers — keep the IDs, rename only the labels.
- **localStorage key collisions:** if two projects use the same `LS_KEY`, opening both planners in the same browser will corrupt state. The scaffolder uses `{{PROJECT_SLUG}}_planner_v1_state` — keep the slug unique. The migration ledger key is likewise per-project (`{{PROJECT_SLUG}}_planner_migrations_applied`).
- **`STATE_MIGRATIONS` ships EMPTY — keep it that way for fresh planners.** The template emits `const STATE_MIGRATIONS = [];`. Do NOT paste another planner's migration array in: on first load `runPendingMigrations()` would force-set those foreign task IDs (e.g. `M1`/`M2`, which the default seeds use as milestones) to `done` with stale notes, silently corrupting a brand-new board. Fresh state already comes from `defaultStateForTasks()` applying each seed's `default_status`/`default_notes`. Only add a migration to patch state users have **already saved**, and only reference task IDs that exist in *this* planner's `SEED_TASKS`.
- **Gantt column count is dynamic — never hardcode it.** `renderGantt()` sets `gantt.style.gridTemplateColumns = '200px repeat(${DAY_COUNT}, minmax(0,1fr))'` so the axis always matches the data and all days fit with **no horizontal scroll**. The `.gantt` CSS `grid-template-columns` is only a pre-render fallback. Do NOT reintroduce a hardcoded `repeat(15, 1fr)` or a fixed `min-width` on `.gantt` — that was the original bug that garbled the chart and forced scrolling for any `DAY_COUNT ≠ 15`.
- **Day axis is 1-indexed for display, 0-indexed internally.** The header, phase ranges, today-label, and day badges all show `Day ${i+1}` (so the chart starts at **Day 1**), but `start_day`/`end_day`/`todayDayIndex()` stay **0-based** for positioning. When writing SEED_TASKS, keep using 0-based `start_day` (first day = `0`).
- **`DEADLINE_DAY` is the launch gate, not the project length.** CONVENTION: `DEADLINE_DAY = the launch milestone's 0-based day index + 1` (its 1-based "Day N" on the axis). e.g. a launch milestone at `end_day:30` → pass `-DeadlineDay 31`. The launch-deadline banner measures **only the launch milestone** (the seed flagged `is_launch: true`, resolved by `findLaunchMilestone()`), so tickets intentionally scheduled **after** launch (post-launch layering) never trip it. This replaces the old behavior, where the banner compared the full-project `cpDuration` against `DEADLINE_DAY` and false-fired on any plan with work past the deadline. **Off-by-one trap:** a launch on day index `N` needs `DEADLINE_DAY = N+1`, or the banner false-fires by one day. With the cascade **enabled** (opt-in; default off), the banner measures the launch milestone's **effective** end (`effEndOf`), so weekends/cascade that slip the launch past `DEADLINE_DAY` will fire it — a real overrun signal. With the cascade **off** (default), `effEndOf` falls back to the authored `end_day`.
- **Weekend/dependency cascade is OPT-IN — default OFF** (`const WEEKEND_CASCADE = false`, just above `computeWeekendSchedule`). By default bars render at their **authored** `start_day`/`end_day` and the axis tracks `DAY_COUNT` — predictable and proportionate. When enabled (`= true`), `computeWeekendSchedule()` re-lays each ticket as **working days** (skipping Sat/Sun) shifted after every dependency's effective end (cascade), writing `_effStart`/`_effEnd` that the gantt / critical-path / deadline-banner read via `effStartOf()`/`effEndOf()` (authored fields stay the persisted source — non-destructive). **Why it defaults off:** on a board with a long dependency chain the cascade pushes the launch far past the authored horizon — a 5-day plan can sprawl to 30+ columns, crushing short bars into one-day-looking slivers AND making each re-render heavy enough to hang the page. That's the "bars don't extend through all their days / the chart froze" regression. Only flip it on when the working-day calendar genuinely matters and the chain is short.
- **The gantt axis auto-grows.** `renderGantt` renders `gridDays = max(DAY_COUNT, latest effective end + 1)` columns so weekend/cascade extension never pushes tickets off the right edge. `-DayCount` is the **minimum** horizon, not a hard width.
- **Don't revert the bar-width calc.** `bar.style.width = \`calc(${span*100}% + ${span}px - 4px)\`` — the `+ ${span}px` compensates for each `.g-cell`'s 1px left border (a `%` width on the absolutely-positioned bar resolves against the cell's *padding* box, which excludes that border). Drop it and long bars fall ~1px per spanned column short of their end date (the original "bars don't carry through to their end date" bug).

- **Don't remove the `TICKET_DETAILS` markers.** The board's read-only Work-breakdown + Acceptance-criteria popover sections render from a `const TICKET_DETAILS = {…}` block bounded by `/* <<TICKET_DETAILS>> */ … /* <</TICKET_DETAILS>> */`. `ticket render` overwrites *between* those markers (matched literally, not by brace-counting — so nested `{}` in the data are safe). Strip the markers and `render` silently stops syncing the sections. `tasks[]`/`acceptance[]` are intentionally kept OUT of the `SEED_TASKS` objects (they contain nested braces, which would break the `init`/`render` per-object `{…}` regexes) — that's why they live in their own block. `render` escapes the `<` character to `\\u003c` in the JSON so ticket text can't break out of the `<script>`.

- **Scaffolder must stay pure ASCII.** `scaffold-planner.ps1` is parsed by Windows PowerShell 5.1, which reads BOM-less `.ps1` as Windows-1252 — a non-ASCII char (em-dash `—`, middot `·`, arrows) can decode into a curly quote that PowerShell treats as a string delimiter, throwing a cascade of parse errors far from the real line. Keep the script ASCII-only (use `-`, `>`, `*`). Verify with `[System.Management.Automation.Language.Parser]::ParseFile(path,[ref]$null,[ref]$errs)`.

## Enforcement — hard gate (built in by default)

The scaffolder installs an **enforcement bundle** (`templates/enforcement/`) so the planner is *enforced*, not
just displayed: agents update tickets as work is done, and "done" is **earned** against each ticket's acceptance
criteria — verified by a *different* agent — never self-declared. Skip with `-NoEnforcement` for a display-only board.

**Source of truth shifts to `planner/tickets.json`** (generated by `ticket init` from SEED_TASKS). `planner.html`
is the human view; `node planner/ticket.mjs render` syncs statuses back into it.

### Authoring requirement (Step 4 addendum)
Every ticket must also get, in `tickets.json`:
- **`tasks[]`** — the work-breakdown checklist the assignee ticks off.
- **`acceptance[]`** — **≥1 verifiable completion criterion**. Prefer a `verify` *command* per criterion
  (`npm test -- x`, `tsc --noEmit`, a grep, a curl) — deterministic checks are never argued. Subjective criteria
  are judged by the verifier agent (see `VERIFIER.md`).
- **`paths[]`** — globs so file edits auto-stamp the ticket `in_progress`.
- tag auth/payments/PII tickets **`security`** → those require a **human** verdict.

### The loop (see `TICKET_PROTOCOL.md`)
`start → check (tick tasks) → submit --evidence → [different agent] verify → verdict {pass|rework|block}`.
Workers **cannot** set `done`. `rework` reassigns to the author with the failed criteria as notes. Attempt cap
(3) → auto-escalates to a human.

### Graduated enforcement (best across the most scenarios)
- **Auto-stamp** (PostToolUse) + **verifier≠author** + **`--pass` re-runs deterministic checks** — always on.
- **Stop-hook** — **warn + record** by default; `node planner/ticket.mjs strict` makes it **hard-block**.
- **`ticket ci`** (`.githooks/pre-commit` + `.github/workflows/planner-gate.yml`) — the **hard floor** nothing merges past.
- **Workflows**: persist verdicts in the orchestrator (a `verify`/`verdict` stage by a *different* agent), not the worker.

### Step 7 addendum (after SEED_TASKS are authored)
```
node planner/ticket.mjs init      # seed tickets.json from the board
# fill tasks[]/acceptance[]/paths[] per ticket, then:
node planner/ticket.mjs render    # sync status + tasks/acceptance into planner.html
# merge .claude/planner-hooks.snippet.json into .claude/settings.json (or /update-config)
git config core.hooksPath .githooks   # enable the pre-commit gate
```

**The board displays the two lists.** `render` writes each ticket's `tasks[]` and
`acceptance[]` into a `TICKET_DETAILS` block in `planner.html`; the ticket popover renders
them **read-only** (Work breakdown checklist + Acceptance criteria with pass/fail/pending
badges + a verdict line). Mutation stays on the CLI — the board only shows. A display-only
board (`-NoEnforcement`, no `tickets.json`) leaves `TICKET_DETAILS = {}` and the sections stay
hidden. Re-run `render` after every verdict/check to keep the board current.

**Upgrading an existing (popover-lineage) board** to the new format:
`node scripts/upgrade-planner.mjs --apply [path…]` injects the CSS + popover sections +
`TICKET_DETAILS` block + `renderTicketDetail` by anchor (idempotent, writes `.bak-preupgrade`).
Defaults to the known board list; pass paths to override. It targets the `pop-deliverable`/
`openPopover` popover lineage only — older `task-modal` boards (no `pop-deliverable`) need
re-scaffolding instead. After upgrading a board that has a `tickets.json`, copy the current
`templates/enforcement/ticket.mjs` over the board's copy and run `render` to populate it.

## Stream Deck binding

See [STREAMDECK.md](STREAMDECK.md).

## References

- Template lives at `~/.claude/templates/project-planner/planner.html`.
- Scaffolder script: `scripts/scaffold-planner.ps1` (inside this skill).
- Example output: `D:/Homebrew Apps/CRM - Claude Based/planner/planner.html` was built with this pattern (manually) — use as reference for what a finished planner looks like.
- Original pattern source: `D:/Homebrew Apps/Rhys Landscaping/implementation/planner/` and `D:/Homebrew Apps/WastePlace/onboarding/wave_1_gantt_v2.html`.

## Common Anti-Patterns

### 1. Building Gantt charts before defining dependencies
**Symptom**: Building Gantt charts before defining dependencies
**Problem**: A timeline without mapped dependencies produces an optimistic schedule that collapses on the first blocked task.
**Solution**: Map all task dependencies first. Identify the critical path before assigning any dates.

### 2. Planning to 100% resource utilization
**Symptom**: Planning to 100% resource utilization
**Problem**: Teams planned at 100% capacity have zero buffer. A single sick day or scope change cascades into a missed milestone.
**Solution**: Plan to 70-80% capacity. Reserve 20-30% for unplanned work, context-switching, and scope changes.
