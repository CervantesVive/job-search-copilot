---
name: job-search-log
description: Makes a quick manual update to the tracker outside of a scheduled scan — a new application, a status change, a new watchlist company, or a note. Triggers on phrases like "I applied to X", "add X to my watchlist", "I heard back from X", "update X to onsite/offer/rejected", "remove X from my watchlist", or "note on X: ...".
---

# Manual tracker update

Read `${CLAUDE_PLUGIN_ROOT}/shared/conventions.md` and `tracker.md`. If no
tracker exists yet, tell the user to run setup first ("set up my job
search") and stop.

Figure out the intent from the message:

- **New application** ("I applied to X [for Y role]") — if a matching `leads`
  entry already exists (same company+role), promote it: move it into
  `applications` with `status: "Applied"` and today's date, keep its
  `fitNotes`/`knowSomeone` as `notes`/`contact`. If there's no existing lead,
  create a new `applications` entry directly — ask only for what's missing
  and matters (role, source/link) in one short message; don't demand every
  field, leave anything unknown blank rather than blocking.
- **Status change** ("heard back from X", "X moved to onsite", "got an offer
  from X", "rejected by X") — find the matching `applications` entry (ask
  which one if the company has multiple open roles in the tracker) and
  update `status` plus a dated line in `notes`.
- **New watchlist company** ("add X to my watchlist", "keep an eye on X") —
  add to `watchlist` with whatever reason they gave in `why`.
- **Remove from watchlist** — remove the matching entry.
- **Freeform note** ("note on X: ...") — append a dated line to that entry's
  `notes`, wherever it currently lives (applications/leads/watchlist).

Load `artifact-design` only if not already loaded this session. Read the
current tracker artifact, make the one change in the JSON data block, bump
`lastUpdated`, keep the same favicon, and republish to the same URL from
`tracker.md`.

Confirm with one short line — what changed, nothing more. This should feel
instant, not like a form.
