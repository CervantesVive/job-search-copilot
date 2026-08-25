---
name: job-search-setup
description: Onboards a new job search in this Cowork project — intakes a resume, interviews the user for their target roles/industries/locations/companies, publishes the tracker webpage, and schedules recurring scans. Triggers on "set up my job search", "start my job search", "get my job search tracker going", "job-search-copilot setup", or when a resume is uploaded in a project that has this plugin and no tracker yet.
---

# Job search setup

Runs once per person, at the start. Conversational, one question at a time —
the person using this may not be technical, so keep every message short,
plain-language, and free of jargon (no "cron", "artifact", "MCP" in anything
you say to them — just "your tracker page", "check every few days", etc.).

Read `${CLAUDE_PLUGIN_ROOT}/shared/conventions.md` before doing anything else
in this skill — it defines every file name and data shape referenced below.

## 0. Check for an existing setup

Call `project_memory_read` with no arguments. If `tracker.md` already exists,
read it, tell the person their tracker is already set up, give them the link
again, and ask if they want to update their target profile instead (roles,
locations, companies) rather than starting over. If they do, jump to step 2
and only touch `profile.md` / `search-config.md`, not the tracker artifact
structure. Otherwise stop here.

## 1. Resume intake

If a resume file is attached to their message, read it directly. If not, ask
for one first, in plain terms: *"Can you upload your resume? I'll use it to
figure out what roles and companies make sense to search for."* Wait for it.

From the resume, extract a short working summary — current title/level,
years of experience, core skills, 2-3 standout achievements or companies. You
will write a condensed version of this to `profile.md` in step 3, not the
full resume text.

## 2. Interview

Ask these one at a time, waiting for each answer before moving on. Skip a
question if the resume or a prior answer already makes it obvious, and say so
("Looks like you're targeting product management roles based on your resume
— sound right?") rather than asking blind.

1. **Target role(s) and level** — job titles and seniority to search for.
2. **Industries or company types** — specific industries, or "open to
   anything."
3. **Locations and work style** — cities/regions, and remote / hybrid /
   onsite preference.
4. **Specific companies** — any dream companies or ones they already have an
   eye on. These seed the tracker's watchlist. Fine to have none.
5. **Deal-breakers** — anything that's a hard no (e.g. no relocation, no 5x
   in-office). Optional.
6. **Job boards to search** — propose sensible defaults for their role/
   location (e.g. "LinkedIn Jobs and Indeed — I'll search those with your
   target titles and location"). Ask if they have others, or a saved search
   URL they already use, and want it included.
7. **How often to check** — offer a default and let them accept it rather
   than making them design a schedule: *"I'll check job boards every couple
   of days and work through company career pages in rotating batches every
   few days, so nothing gets missed but I'm not hammering the same sites
   constantly. Want me to check more or less often than that?"*
8. **LinkedIn network matching (optional)** — explain in plain terms: *"If
   you want, I can flag when a new lead is at a company where you already
   know someone. That needs a one-time export of your LinkedIn connections —
   go to LinkedIn → Settings & Privacy → Data privacy → Get a copy of your
   data, request just 'Connections', and LinkedIn will email you a link in
   a few minutes. Whenever it arrives, come back here, drag the
   Connections.csv file into the chat, and say 'here's my LinkedIn export.'
   Want to set that up now, later, or skip it?"* If they have the file
   already or get it during this conversation, process it per step 5 now;
   otherwise move on and handle it whenever it shows up in a later message
   (any skill in this plugin should recognize an uploaded `Connections.csv`
   and process it per step 5, then confirm to the user).
9. **Gmail reply tracking (optional)** — explain: *"I can also watch your
   email for recruiter replies and flag ones you haven't responded to yet.
   Want to connect Gmail for that?"* If yes, use `SuggestConnectors` with
   keywords `["gmail", "email"]` and let them complete the connection. If no
   or they skip, note it and move on — this can be turned on later just by
   asking.

## 3. Write project memory

Write `profile.md`, `search-config.md` (job_boards, companies list seeded
from step 2.4 at `priority: high` plus any others they mentioned, cadence
from 2.7, `rotation_index: 0`), and `gmail-status.md` (`connected: yes/no`)
per the shapes in `shared/conventions.md`. Each with proper frontmatter.
Update `MEMORY.md`'s index with one line per file.

## 4. Publish the tracker

Load the `artifact-design` skill, then build and publish the tracker page per
the JSON-data-block structure in `shared/conventions.md`. Seed `watchlist`
from any companies named in step 2.4 (empty `applications` and `leads` — the
scans will populate those). Use the person's first name and a one-line
profile summary in the header. Pick a favicon now and record it mentally —
every future update to this page must reuse the same one.

Write `tracker.md` with the resulting URL and today's date.

## 5. Process a LinkedIn export, if provided

If `Connections.csv` was uploaded (now or, per step 2.8, in some later
message), read it. LinkedIn's export has a header row (commonly `First Name,
Last Name, URL, Email Address, Company, Position, Connected On`, though
exact columns can vary — read the actual header row rather than assuming).
For each row, keep only `Company`, full name, and `Position`. Skip rows with
no company. Write the condensed list to `network.md`. Confirm to the person
how many connections were imported and that new leads will now get flagged
when they're at a company where this shows a contact.

## 6. Schedule the recurring scans

Use `create_trigger` (never a local/in-session cron tool — those don't
survive between sessions). Create:

- **Job board scan** — cadence from step 2.7 (default: every 2 days).
  Prompt: *"Run the job-search-scan skill's job-board mode for this
  project's job search (job-search-copilot plugin). Read
  shared/conventions.md and search-config.md first, then search the
  configured job boards, dedupe against the tracker, add new matches to
  the Leads section, and end with a short plain-language summary of what's
  new."*
- **Company career page scan** — cadence from step 2.7 (default: every 3
  days). Prompt: *"Run the job-search-scan skill's company-page mode for
  this project's job search (job-search-copilot plugin). Read
  shared/conventions.md and search-config.md first, work through the next
  batch of companies starting at the stored rotation_index, update
  rotation_index when done, dedupe against the tracker, add new matches to
  the Leads section, and end with a short plain-language summary."*
- **Gmail reply check**, only if Gmail was connected in step 2.9 — daily.
  Prompt: *"Run the job-search-gmail-check skill for this project's job
  search (job-search-copilot plugin) and end with a short plain-language
  summary of anything needing a reply."*

Set `notifications: {"push": true}` on each so the person gets pinged when a
run finds something, and `initiation: "human_request"` since this was set up
at their explicit ask. Leave `environment_id` unset so each trigger runs in
this project's environment.

## 7. Wrap-up message

End with one short message covering: the tracker link, what's scheduled and
roughly how often, and three things they can just say any time — *"prep me
for an interview at [company]"*, *"practice interview for this posting"* (a
job link or pasted description), and *"I applied to [company]"* or *"add
[company] to my watchlist"* to update the tracker by hand. Keep it to a few
sentences — they now have a working link to check instead of a wall of text.
