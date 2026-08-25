# Job Search Copilot

A Cowork plugin that runs an ongoing job search: one place to track applications
and target companies, automatic scanning of job boards and company career pages,
Gmail reply tracking, LinkedIn "who do I know here" matching, and on-demand
interview prep.

## What it does

- **Tracker** — your applications, a leads queue from automated scans, and a
  watchlist of companies, kept in the project's own memory (so nothing is
  ever lost even if a scan can't reach the internet), and rendered as a
  private webpage you can bookmark and check from your phone.
- **Scheduled job board scans** — searches the job boards you choose (e.g.
  LinkedIn Jobs, Indeed) for roles matching your target profile and adds new
  matches to the tracker's Leads section.
- **Scheduled company career page scans** — works through your list of target
  companies in rotating batches, checking their career pages directly, so you
  hear about openings even at companies that don't post to job boards.
- **Gmail reply check** (optional) — scans for recruiter/company email threads
  tied to companies already in your tracker and flags ones that need a reply.
- **LinkedIn network match** (optional) — a one-time import of your LinkedIn
  connections so new leads get flagged with "you know someone here."
- **Interview cheatsheets** — on request, researches a company and role and
  builds a single-page HTML cheat sheet (company snapshot, likely questions,
  talking points, questions to ask, watch-outs).
- **Practice interviews** — on request, roleplays a mock interview based on a
  specific job posting, one question at a time, with feedback.

## Prerequisites

- A Claude Cowork-enabled plan — Cowork requires Pro, Max, Team, or
  Enterprise; it isn't available on the Free plan.
- Gmail reply tracking and LinkedIn network matching are optional and can be
  skipped or added later; everything else works without them — but both are
  very helpful, so it's worth setting them up if you can.

## Setup (one time)

1. Install this plugin.
2. Create a new Cowork Project (name it anything — "Job Search" is a good
   default) and open it.
3. In the chat, upload your resume and say **"set up my job search."** That
   runs the onboarding skill, which will:
   - ask a handful of questions about the roles, industries, and locations
     you're targeting
   - ask which job boards and (optionally) which specific companies to watch
   - publish your tracker webpage and give you the link
   - optionally walk you through a one-time LinkedIn data export for network
     matching
   - optionally prompt you to connect Gmail for reply tracking
   - set up the recurring scans
4. `job-search-setup` will also mention an **optional** one-time setting
   that keeps the tracker webpage instantly current after a scheduled scan,
   instead of catching up the next time you're chatting here (which happens
   either way — nothing about your data depends on this). Skip it if it
   sounds fiddly.
5. That's it. A note on how "scheduled" scans actually behave right now:
   this platform's scheduled tasks don't reliably fire to-the-minute while
   the app is closed — they tend to catch up whenever you next open Cowork,
   rather than running silently in the true background. So treat it as
   "checks for what's new whenever I check in" rather than "runs unattended
   on a strict clock," and open the project every few days for the freshest
   results. Come back any time to check the tracker, ask for an interview
   cheatsheet, or ask for a practice interview.

## Skills included

| Skill | Triggered by |
|---|---|
| `job-search-setup` | "set up my job search" (run once) |
| `job-search-scan` | runs on a schedule; can also be asked for on demand ("scan for new jobs now") |
| `job-search-gmail-check` | runs on a schedule if Gmail is connected; can be asked for on demand |
| `job-search-interview-prep` | "prep me for the interview at X", "make a cheat sheet for this posting" |
| `job-search-practice-interview` | "practice interview for X", "mock interview based on this job req" |
| `job-search-log` | "I applied to X", "add Y to my watchlist", "heard back from Z" |

See `shared/conventions.md` for the file/data conventions all the skills share.

## Sources

- [Organize your tasks with projects in Claude Cowork](https://support.claude.com/en/articles/14116274-organize-your-tasks-with-projects-in-claude-cowork)
- [Use plugins in Claude](https://support.claude.com/en/articles/13837440-use-plugins-in-claude)
