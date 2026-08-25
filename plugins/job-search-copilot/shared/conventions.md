# Shared conventions

Read by every skill in this plugin before it touches project memory or the
tracker artifact. Keeping these consistent is what lets the scan, gmail-check,
log, and setup skills all read/write the same data without colliding.

## Project memory files (this project's persistent memory, not the user's global memory)

Use `project_memory_read` / `project_memory_write` (the remote-devices
project-memory tools). These are scoped to the current Cowork Project and
survive across sessions, including the fresh sessions that scheduled tasks
spin up — this is where all durable job-search state lives. If these tools
are not available in a given session (check by calling `project_memory_read`
with no arguments — if it errors or is absent from the tool list), fall back
to a plain project file called `job-search-config.md` written with `Write`/
`Edit` in the working directory instead, and note in your reply that project
memory wasn't available so config lives in that file. Never invent a third
location.

Files, each with YAML frontmatter (`name`, `description`, `type: project`):

- **`profile.md`** — the person's target profile: role titles/levels, seniority,
  industries of interest, locations/remote preference, must-have companies,
  deal-breakers, resume summary (2-3 lines, not the full resume text).
- **`search-config.md`** — the configurable scan inputs:
  - `job_boards`: list of `{name, search_url_or_query, notes}` — e.g. LinkedIn
    Jobs, Indeed, a specific saved search URL the user gave you.
  - `companies`: the full target-company list with `{name, career_page_url,
    priority: high|medium|low}`. This list can be long (Ivo's ran to 300+).
  - `scan_cadence`: how often job-board scans and company-page scans should
    run (default: job boards every 2 days, company pages in rotating batches
    every 3 days).
  - `rotation_index`: where the last company-page scan left off (an integer
    offset into the `companies` list, high-priority items first) — this is
    what makes batched rotation scans possible without re-checking everyone
    every run. `job-search-scan` reads and updates this every run.
- **`gmail-status.md`** — whether Gmail is connected, and the timestamp of the
  last successful gmail-check run (used so `job-search-gmail-check` can search
  only messages newer than the last run).
- **`network.md`** — condensed LinkedIn connections, if the user did the CSV
  import: a list of `{connection_name, headline, company}` limited to the
  fields needed for matching. Do not store the full export (it can contain
  message history, ad-targeting data, etc. that this plugin has no use for)
  — only Company, Name, Position from `Connections.csv`.
- **`tracker-data.md`** — the tracker's actual data, and its single source of
  truth. A YAML-fronted markdown file whose body is one fenced ` ```json `
  block in the shape given below. Every skill that changes tracker data
  (`job-search-setup`, `job-search-scan`, `job-search-gmail-check`,
  `job-search-log`, `job-search-interview-prep`) reads this file, changes the
  JSON in plain code logic, and writes the whole file back with
  `project_memory_write`/`memory_str_replace` — never the published webpage.
  This is deliberate: project memory has proven reliable from scheduled-task
  sessions, while the published webpage depends on network access those
  sessions don't always have configured (see "The tracker webpage" below) —
  so the data must never live only in the webpage.
- **`tracker.md`** — just a pointer to the *view*: the published webpage's
  URL, its favicon (so every sync reuses the same one without needing to
  fetch the page to check), and the date it was last successfully synced.
  Not the data. If this file doesn't exist yet, no webpage has been
  published yet.

`MEMORY.md` (the project memory index) should end up with one line per file
above, per the standard index format.

## Tracker data (source of truth)

JSON shape stored in `tracker-data.md`:

```json
{
  "profileSummary": "Sr. Engineering Manager targeting EM/Director roles, SF Bay Area + remote",
  "lastUpdated": "2026-08-25",
  "applications": [
    {
      "id": "app-0001",
      "company": "Acme Corp",
      "role": "Engineering Manager",
      "status": "Applied",
      "dateApplied": "2026-08-20",
      "source": "LinkedIn Jobs",
      "location": "Remote",
      "url": "https://...",
      "priority": "high",
      "contact": "",
      "notes": ""
    }
  ],
  "leads": [
    {
      "id": "lead-0001",
      "company": "Acme Corp",
      "role": "Director of Engineering",
      "url": "https://...",
      "source": "career page scan",
      "dateFound": "2026-08-24",
      "fitNotes": "Matches target seniority and remote preference",
      "knowSomeone": "Jane Doe — Senior PM"
    }
  ],
  "watchlist": [
    {
      "company": "Beta Inc",
      "why": "Admires their product, no open roles yet",
      "priority": "medium",
      "notes": ""
    }
  ]
}
```

`status` values for applications: `Not Applied`, `Applied`, `Recruiter Screen`,
`Hiring Manager Screen`, `Onsite`, `Offer`, `Rejected`, `Withdrawn`, `Ghosted`.
`priority` values: `high`, `medium`, `low`. IDs are `app-####` / `lead-####`,
zero-padded, incrementing — never reuse or renumber existing IDs.

## The tracker webpage (best-effort view)

The published `Artifact` webpage is a rendering of `tracker-data.md` — never
edit it directly, and never treat it as a data source. Any skill that just
changed `tracker-data.md` should, as its last step, run this **sync
procedure**: read `tracker-data.md`, build the full HTML page from that JSON
(same structure every time — this is a render, not an edit), and publish it
with `Artifact` (first time: no `url` yet, creates one, write it to
`tracker.md`; after that: pass the existing `url` from `tracker.md`, which
redeploys in place). On success, update the "last synced" date in
`tracker.md`.

**If the sync fails** (most likely cause: the session's network access
can't reach the artifact-hosting domain — this happens in scheduled-task
sessions whose environment hasn't been configured for it, see
`job-search-setup` step 7): this is now low-stakes, not data loss — the data
is already safely written to `tracker-data.md` regardless of whether the
sync succeeded. Don't retry repeatedly. Mention it only briefly, if at all
— e.g. "(the tracker page may take a bit to catch up — nothing's lost)" —
rather than treating it as an error to resolve. Never skip writing
`tracker-data.md` because a sync might fail later; the two are independent.

**Before writing the page HTML, load the `artifact-design` skill** (required
by the Artifact tool for any new artifact content). Reuse the `favicon`
recorded in `tracker.md` on every sync after the first — never fetch the
existing page just to check it, that reintroduces the same network
dependency this design avoids. On the very first sync, pick one (e.g. 🧭)
and record it in `tracker.md` alongside the URL.

Page sections, top to bottom: header with the person's name/profile summary
and last-updated date, a tab or section switcher for **Applications / Leads /
Watchlist**, each rendered as a sortable-by-eye table (status as a colored
pill for Applications), then a footer: one small, muted line — "Updates
automatically every few days — nothing to do here. (You may see new chats
appear under 'Scheduled' in your sidebar — that's just these checks running;
this page is always the current summary.)" Keep the visual design in the
dark, card-based style that's easy to scan on a phone — see
`skills/job-search-interview-prep/references/cheatsheet-template.html` in
this plugin for the color/spacing language to reuse (translated from cards
to table rows). This is part of the page structure, not a one-time addition
— since every sync rebuilds the full page from `tracker-data.md`, the
footer must be included on every sync, not just the first publish.

## Deduping new finds

Before adding anything to `leads` or `applications`, compare company+role
(case-insensitive, trimmed) against both `applications` and `leads` already
in `tracker-data.md`. Skip anything already present rather than adding a
duplicate.

## Talking to the user vs. writing data

Scheduled runs happen in a fresh session the user isn't watching in real
time. End every scan/check skill with a short plain-language summary in your
final reply (what ran, what's new, what needs attention) — that summary is
what the user sees when they open the run later or get a notification. Don't
rely on them seeing intermediate tool calls.

Scheduled tasks on this platform don't always fire precisely on their cron
schedule while the app is closed — they can run late, or batched, once the
app is next opened. Don't promise the person "this runs while you're away"
in absolute terms; it's closer to "this catches up whenever you're next in
the app." Setup should set that expectation once and skills shouldn't need
to repeat it.
