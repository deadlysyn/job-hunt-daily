# job-hunt-daily — Release Notes

![Job Hunt Daily](https://github.com/deadlysyn/job-hunt-daily/blob/5cb306d0e115c90a2c325e33452c3684b0ec2639/assets/Main%401x.png)

A Claude skill that runs a personalized job search: scans job boards, scores openings against your resume and preferences, and gives you a short ranked report with direct apply links.

## What it does

- **Scores every opening on 5 weighted criteria:** skills match (rewards a strong-but-stretching fit, not just a perfect one), employer quality (reviews, layoffs, funding), freshness and competition, and compensation + benefits (with a real bonus if dependent health coverage is explicitly offered).
- **Filters hard on your requirements first:** remote status, employment type, pay floor, and any dealbreakers you set — before anything gets scored.
- **Verifies its top 5, not just lists them.** It opens each finalist's actual posting (preferring the employer's own career page over aggregators), checks employer reviews from at least two sources, and runs a quick scam/fraud check on any third-party link.
- **Shows its confidence.** Every scored role gets a High/Medium/Low confidence tag based on whether the link, employer rating, and pay were verified or estimated — so a good-looking score never hides shaky data.
- **Remembers what you've already seen.** Each report skips roles from your last run, and a permanent decisions log ("passed," "applied," "not interested") means a role you've rejected won't quietly resurface later.
- **Gives you next steps, not just a list:** a one-line "apply strengths" note per role on which resume points to lead with.

## Setup (first run)

The first time you use it, there's no personal data yet — it will:
1. Ask you to upload your resume.
2. Pull your skills, titles, and experience out of it automatically.
3. Ask a short round of questions for anything the resume can't answer (salary range, benefits priorities, dealbreakers, remote/location needs, on-call tolerance).
4. Offer to adjust the default scoring weights, though most people keep the defaults.

Nothing runs before that's done, so there's no risk of it searching against blank preferences.

## Opt-in features (off by default)

These make the skill better but aren't required — it works fine without either:

- **Report history via a Claude Doc.** If you use Claude Docs, the skill can keep an automatic running log of past reports, so it can skip previously-seen roles without you pasting the last report every time. If you don't use Docs, it just asks you to paste your last report instead — a bit more manual, but zero setup.
- **LinkedIn alerts via Gmail.** If you connect a Gmail connector, the skill can read LinkedIn job-alert emails from your inbox to pull in applicant counts (a data point it otherwise has no way to see). **This access is deliberately narrow:**
  - Inbox only — it never touches Sent, Drafts, Trash, Spam, or other labels.
  - Never touches Calendar, Contacts, Drive, or anything else the connector might expose.
  - Strictly read-only — it never sends, replies, labels, archives, or deletes anything.
  It will never prompt you to connect this — it's there only if you choose to turn it on. Without it, you can just forward alerts manually, or skip applicant-count data entirely.

## Also uses (if connected)

Official Dice and Indeed job-search connectors, if you have them connected, to cross-check postings and pull employer ratings. Neither is required — the skill searches the open web otherwise.

## What it won't do

- It won't try to scrape or bypass login walls on LinkedIn or Glassdoor — it treats those as off-limits and works around them with public search results instead.
- It won't silently loosen a filter (like your pay floor) to pad out a thin result list — it'll tell you plainly if fewer than 3 roles qualify and suggest what to relax.
- It won't reuse a stale employer rating past 30 days without refreshing it.

## Running it on a schedule

The skill now supports two modes so it can run on a schedule without redoing full verification every day:

- **Full mode** — the complete workflow: broad multi-title, multi-source scan, both job-search connectors, the target-company watchlist, layoff/funding checks, cache refreshes, and full verification and scoring of the top 5. This is the right mode for a weekly run.
- **Light mode** — a cheap daily pass that only checks postings from the last ~48 hours and the target-company watchlist, reuses the most recent full run's cached scores, and only spends real verification effort if something new looks genuinely competitive. Most days it should report "nothing beat your current top 5" quickly and cheaply.

Scheduling itself is a native Claude feature (scheduled tasks), not something the skill sets up on its own — you create the schedule yourself in Claude's UI, in a conversation or from the Scheduled Tasks page. The built-in cadence options are hourly, daily, weekly, on weekdays, or manual — there's no native "one day heavy, rest of the week light" option, so the clean way to get that is **two separate scheduled tasks**:

1. **Weekly, your chosen day and time** — prompt: *"Run my job-hunt-daily skill in full mode."*
2. **Weekdays, same time** — prompt: *"Run my job-hunt-daily skill in light mode. If today is [the day Task 1 runs], skip — the full run already covers it."* (Native "weekdays" scheduling can't exclude a single day, so this line makes the light task a clean no-op on the day the full run already happened.)

When creating either task, set **approval mode to fully automatic** if you don't want to review each run before it proceeds, and set the **time zone explicitly** (e.g. America/New_York) rather than relying on a default, especially across daylight saving changes.

**A scheduled task only has access to whatever's connected at your account level** (Settings → Connectors, or claude.ai/settings/connectors) — not just what was connected in whatever chat you're currently using. Check that page to confirm Dice, Indeed, or Gmail are connected there if you want a scheduled run to use them.

Results land on Claude's "Scheduled" page for review, the same as any other scheduled task.

## Optional: email delivery
On top of keeping results in Scheduled Tasks / chat (the default), you can ask the skill to also email each report to an address of your choice. This is set up during onboarding (or any time after, just ask). It's best-effort: sending requires a connected, send-capable mail tool at run time, and if one isn't available in a given session, the skill says so and the report still lands in Scheduled Tasks / chat as normal.

## Region support (US, Canada, EU/UK, and beyond)

Onboarding now asks where you're searching from, and the skill uses that to pick the right job sites automatically:

- **United States:** Indeed, Dice, LinkedIn, Glassdoor, Wellfound, Built In, plus the usual remote-first boards and company career pages.
- **Canada:** Indeed.ca, Job Bank (the Government of Canada's official listing service), Eluta.ca, LinkedIn, Glassdoor.ca, plus the same remote-first boards and company pages. Dice is skipped by default, since it's a US-focused board.
- **European Union / UK:** EURES (the official EU/EEA job board), your country's regional Indeed domain, LinkedIn, Glassdoor, plus a well-known local board when you name a specific country (for example StepStone or Xing for Germany/Austria/Switzerland, Otta for the UK).
- **Anywhere else:** LinkedIn, Glassdoor, remote-first boards, and company career pages as a solid baseline — tell it about a strong local board for your country and it'll add it.

If your home region isn't the US, you can also choose to **include US-based roles** alongside your region's results — useful if you're open to (and authorized for) working for a US employer. Remote-first boards (We Work Remotely, Remotive, RemoteOK, Wellfound) and company career pages aren't region-locked, so they're searched for everyone regardless of home region.

Pay comparisons already adapt to your currency (not just USD), and the remote/location gates check whichever locations you're actually eligible for, rather than assuming a single country.

## Visual status tracking

![Kanban Mode](https://github.com/deadlysyn/job-hunt-daily/blob/831fe1199d5bc7b1668b01d323b20914cddf532b/assets/job-hunt-daily.png)

Clicking an apply link isn't tracked by anything — the skill now makes status explicit instead of leaving it invisible:

- **A Kanban board is the primary way to see and change status.** Four columns — Not Reviewed, Applied, Interviewing, Passed — with one card per role (title, company, score, a direct link to the posting, and status buttons). It's a real, persistent artifact: click a status button any time, from any device, and it's saved immediately, no need to go through chat.
- **Chat shorthand still works** as a shortcut — reply "applied: 1, 3" or "pass: GitLab" after any report — and updates land on the board too, so the two never drift apart.
- **A text log is kept as a backup.** If the board is ever unreachable, the skill can still track decisions from the log alone, and can recreate the board from it.
- The board's link is created once during setup (or on request) and reused every run — the skill upserts new roles into it after each report, without ever overwriting a status you've already set.

## Scoring preferences (optional)

Two new optional preferences, asked about during setup:

- **Title preference:** if two title categories both make it past your filters (for example, SRE/DevOps roles and Platform Engineer roles included via a duties-match exception), you can tell the skill to favor one over the other. It's a soft nudge — a strong role in the deprioritized category can still rank well, it just loses a small amount of ground in close calls.
- **Skill-category weighting:** for infra/ops-focused roles, you can tell the skill to treat a whole category — like programming languages — as supporting evidence rather than a primary requirement. A posting that lists a language as "required" won't drag the score down if you're missing it, and having it won't inflate the score either; it just gets mentioned as a nice-to-have note when relevant.

## Known limitations



- LinkedIn applicant counts are only visible if you connect Gmail or forward alerts — otherwise freshness scoring relies on posting age alone.
- Small companies often have too few reviews for a reliable employer-quality score; the skill flags this rather than guessing.
- Scheduling requires two separate scheduled tasks (see above) since there's no single native cadence for "one heavy day, light the rest of the week."
