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
- **`tracker.md`** — a single line: the current tracker artifact's URL, plus
  the date it was last successfully updated. Every skill that touches the
  tracker reads this file first to get the URL, and updates the "last
  successfully updated" line after a successful publish.

`MEMORY.md` (the project memory index) should end up with one line per file
above, per the standard index format.

## The tracker artifact

Published once during setup with the `Artifact` tool, then updated in place
(`Artifact` with the same file path/url redeploys it) by `job-search-setup`,
`job-search-scan`, `job-search-gmail-check`, and `job-search-log`.

**Before writing or editing it, load the `artifact-design` skill** (required
by the Artifact tool for any new artifact content) and give it a two-emoji
`favicon` that stays the same across every update (pick one at first
publish, e.g. 🧭, and never change it).

Structural rule: keep all tracker data in one `<script type="application/json"
id="tracker-data">...</script>` block in the page, and render it into the
visible tables with a small inline `<script>` that runs on load. Every update
is: read the artifact (`Artifact` action `read`), parse that JSON block,
make the data change in plain code logic (not by editing rendered HTML),
write the full file back out with the updated JSON block, and republish with
`Artifact` to the same `url`. This keeps updates mechanical and avoids
corrupting hand-styled HTML.

JSON shape:

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

Page sections, top to bottom: header with the person's name/profile summary
and last-updated date, a tab or section switcher for **Applications / Leads /
Watchlist**, each rendered as a sortable-by-eye table (status as a colored
pill for Applications). Keep the visual design in the dark, card-based style
that's easy to scan on a phone — see `skills/job-search-interview-prep/
references/cheatsheet-template.html` in this plugin for the color/spacing
language to reuse (translated from cards to table rows).

## Deduping new finds

Before adding anything to `leads` or `applications`, compare company+role
(case-insensitive, trimmed) against both `applications` and `leads` already
in the tracker. Skip anything already present rather than adding a duplicate.

## Talking to the user vs. writing data

Scheduled runs happen in a fresh session the user isn't watching in real
time. End every scan/check skill with a short plain-language summary in your
final reply (what ran, what's new, what needs attention) — that summary is
what the user sees when they open the run later or get a notification. Don't
rely on them seeing intermediate tool calls.
