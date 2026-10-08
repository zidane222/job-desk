# Job Desk

A personal job-search dashboard in one HTML file. It pulls live openings from Indeed, Dice, ZipRecruiter and LinkedIn job-alert emails, shows recruiter threads from Gmail, lists upcoming interviews from Google Calendar, and keeps an application pipeline that saves as you edit.

## What's on the page

| Section | Source | Refresh |
|---|---|---|
| Fresh openings | Indeed, Dice, ZipRecruiter (one call per saved search each), plus LinkedIn alert emails read from Gmail | every 30 min |
| Application pipeline | Artifact database (`pipeline` collection) | live |
| Recruiter inbox | Gmail `search_threads`, last 14 days | every 15 min |
| Upcoming interviews | Google Calendar `list_events`, next 3 weeks | every 15 min |
| Summary numbers | Derived from the sections above | live |

## How it runs

This page runs as a **Claude artifact**. It uses the claude.ai artifact runtime (`window.claude`) for two things:

- `claude.use("mcp")` calls your connected claude.ai connectors (Indeed, Dice, ZipRecruiter, Gmail, Google Calendar) with your own credentials. The page never sees tokens.
- `claude.use("db")` stores the pipeline and saved searches.

`capabilities.json` is the manifest the artifact is published with. It names every connector tool the page is allowed to call.

**Opening `index.html` outside Claude** (GitHub Pages, a local browser) shows the layout, but the live sections will say they can't run in this view. That's expected. To run it on your own server, replace the two `claude.use(...)` calls with your own backend: for example, job-board APIs for the feeds, the Gmail and Calendar APIs with OAuth, and any key-value store for the pipeline.

## Data model

`searches/{id}`: `{ query, location, order }`. Set `location` to `"remote"` for a remote-only search.

`pipeline/{id}`: `{ role, company, url, location, pay, stage, nextStep, nextDate (YYYY-MM-DD), source, notes, sample, createdAt }`

`stage` is one of `saved`, `applied`, `screening`, `interview`, `offer` or `closed`.

## Code map (inside `index.html`)

- **Indeed parsing:** `parseJobs()` turns the connector's markdown into job objects.
- **Multi-source feeds:** the `SRC` array holds each board's server, tool, input builder and parser. `watchSearch()` sets up one `watchTool` per board per search, and `mergeSearch()` removes duplicates by title and company.
- **LinkedIn:** `watchLinkedIn()` finds LinkedIn job emails in Gmail. `parseLinkedIn()` pulls title, company, location and job ID out of each email body.
- **Gmail:** `parseThreads()` and `renderMail()`. A thread is flagged REPLY NEEDED when you replied earlier and the latest message is from the recruiter.
- **Calendar:** `watchCal()` and `renderCal()`. Events are treated as interviews by keyword match (`INTRX`). Join links come from the conference data or the description.
- **Pipeline:** `renderPipeline()`, `card()`, plus the board's click handlers.

## Customizing

- **Default searches:** add or remove them on the page, or edit the `searches` collection.
- **Interview detection:** edit the `INTRX` regex.
- **Recruiter filtering:** edit the `AUTO` regex (senders treated as automated job alerts).
- **Refresh rates:** edit the `refetchInterval` values (the minimum is about 30 seconds).
- **Colors and fonts:** edit the CSS variables on `:root`, which has separate dark-theme values.

## Privacy

The file contains no personal data or credentials. All email, calendar and job data is fetched at view time through your connectors. Don't commit exports of your pipeline data if the repo is public.
