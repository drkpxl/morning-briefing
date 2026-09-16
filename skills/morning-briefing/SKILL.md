---
name: morning-briefing
description: "Use when running the daily printable morning briefing."
version: 2.1.0
author: Steven Hubert (drkpxl)
license: MIT
platforms: [macos, linux]
metadata:
  hermes:
    tags: [briefing, morning, print, newspaper, cron, weather, calendar, news, newsletter]
    related_skills: [hermes-cron-automation]
---

# Morning Briefing Skill

A personal daily newspaper: every morning the agent gathers weather, calendar, air quality, curated news, and newsletter summaries, renders a single 8.5×11 newsprint page, and prints it. Sources degrade gracefully — each section renders or renders its own error text. The skill is a rubric, not a pipeline: the agent discovers what tools exist at runtime. It does not do breaking-news alerts or full inbox triage.

## When to Use

- Scheduled 6 AM cron run, or "run my briefing" / "print my morning briefing" / "generate today's briefing"
- Onboarding: "set up morning briefing" / "configure briefing skill"

## Prerequisites

- **WeasyPrint** (`pip install weasyprint`) + pango/glib (`brew install pango glib` on macOS) — HTML→PDF rendering and page-count checks
- **qrcode[pil]** and **Pillow** (`pip install "qrcode[pil]" Pillow`) — QR codes and radar image processing
- **A CUPS printer** reachable via `lp` (`lpstat -p` lists names)
- Optional, each section degrades gracefully without it: Home Assistant entities (weather, calendars, AQI), Gmail OAuth for newsletters, xAI OAuth for X search, tinyair MCP for AQI fallback, Tailscale for overflow pages
- System Python 3.9 lacks the syntax some helper scripts need — run Python steps with the agent venv interpreter

## How to Run

Say "run my morning briefing" or trigger the cron job. The full procedure is in **Procedure** below; onboarding a new user follows `references/onboarding.md`.

**Cron job setup** (the run is a fresh agent session, so pin everything):

- Schedule `0 6 * * *`; toolsets `homeassistant`, `file`, `terminal`, `web`, `browser`
- **Pin the model**: `hermes cron edit <job_id> --model <model> --provider <provider>` — an unpinned job inherits the global default and may land on a model that can't follow the skill
- Deliver `local` — the printed page is the deliverable

## Quick Reference

Config lives at `~/.hermes/scripts/morning-briefing-config.json` (created by onboarding; read it first each run):

| Setting | Purpose |
|---|---|
| weather_entity | HA weather entity ID |
| calendar_entities | List of HA calendar entity IDs — check ALL of them |
| air_quality_entity | HA AQI sensor (or tinyair MCP fallback) |
| printer_name | CUPS printer name |
| location | Display name + news geo-filter |
| news_topics | Interest profile: topic, keywords, interest_description, sources, subreddits |
| newsletter_senders | Sender DOMAINS (addresses change — search by domain) |
| notify_entity | HA notify entity for failure alerts |

**Design system**: `references/design-system.md` holds the complete CSS stylesheet — copy it verbatim into the `<style>` block. The agent controls content, never design. Layout spec: `references/layout-spec.md`. Template skeleton: `templates/briefing.html`.

## Procedure

### 1. Read config

`read_file` the config. If missing, run onboarding (`references/onboarding.md`). Get today's date via `terminal` — use it everywhere.

### 2. Gather data

Each source is independent; one self-healing retry, then render error text in that section. Never skip a section silently.

**Weather** — `ha_get_state` on the weather entity: temp, condition, humidity, wind, pressure. Forecast (today's high/low, precip) via the NWS API `https://api.weather.gov/gridpoints/<office>/<x>,<y>/forecast` with `web_extract` when the HA forecast service is unavailable.

**Calendar** — `ha_get_state` for every configured calendar entity. Today's events only, sorted by start time. Empty renders "No events scheduled today."

**Air quality** — `ha_get_state` on the AQI sensor; tinyair MCP tools as fallback.

**News** — HARD RULE: every story must be published or first reported within the last 36 hours. No verifiable date, no story — no exceptions regardless of relevance. Include the story date in every source attribution.

Reddit RSS is the primary source (timestamped by construction): write a throwaway script to `/tmp/reddit_rss.py` that fetches each configured subreddit (`https://www.reddit.com/r/<sub>/new/.rss?limit=25` with a `User-Agent` header — headerless requests get 429/403), parses Atom (`a:entry`, `a:title`, `a:updated`, `a:link`), and filters to the last 36 hours. Fetch sequentially, save each feed to its own file, retry 429s after 30-60s.

Then per topic: `x_search` with explicit `from_date`/`to_date` (last 36 hours) and `web_search` with today's date in the query. Score against the interest profile; discard noise (pass-purchase chatter, gear advice, memes). A topic with nothing recent gets no stories — a thin news day is honest; stale news is not.

**Newsletters** — search Gmail by sender DOMAIN (senders change delivery addresses): run `google_api.py` from the google-workspace skill via `terminal` — `gmail search "from:<domain> newer_than:1d"`, then `gmail get <id>` for the body. If nothing matches, retry with `newer_than:1d label:CATEGORY_UPDATES` before declaring none found. Summarize sections matching the interest profile; note the rest in one line.

### 3. Process images

Throwaway scripts written to `/tmp/` with `write_file`, run via `terminal`:

- **Radar**: fetch `https://radar.weather.gov/ridge/standard/<station>_0.gif`, convert RGB, crop ~33% centered on the user's area (storms arrive from the mountains — shift west on the Front Range), resize 140px wide, emit a base64 data URI
- **QR codes**: one per story + newsletter, `qrcode` lib at `box_size=3, border=1`, base64 data URIs

### 4. Write the HTML

Assemble the page with all data, QR codes, and radar as base64 data URIs; `write_file` to `/tmp/briefing.html`. Copy the CSS from `references/design-system.md` verbatim. Key invariants: real HTML `<table>` elements (never flexbox — the PDF renderer doesn't support it); weather + calendar share one section (calendar nested under the weather columns, radar in the right cell spanning both); 2-column masonry news grid with QR on each card; lead story full width; joke of the day at the bottom, family-friendly; black and white except the color radar; serif body (Iowan Old Style/Palatino), sans-serif section headers.

### 5. Render and check page count

Write `/tmp/render_pdf.py` (sets `DYLD_LIBRARY_PATH=/opt/homebrew/lib` on macOS before importing weasyprint, renders `/tmp/briefing.html` → `/tmp/briefing.pdf`, prints page count) and run it. If >1 page: trim the lowest-ranked news, rewrite the HTML, re-render — max 3 iterations.

### 6. Print

`terminal`: `lp -d <printer_name> -o media=letter /tmp/briefing.pdf`. Verify exit 0 and a request ID. If the printer is unreachable, notify the user's preferred channel with a link to the rendered HTML.

### 7. Cleanup and confirm

Leave `/tmp/briefing.html` and `/tmp/briefing.pdf` for debugging. Final response: one-line confirmation ending with a period.

## Pitfalls

1. **Inline Python is blocked in cron mode.** Never `python3 -c` or `execute_code` in a cron run — always `write_file` a script to `/tmp/` first, then run it via `terminal`. This is the #1 cause of cron failures.
2. **Unpinned cron models fail.** An unpinned job rode the global default onto a weaker model that couldn't follow the skill and looped on blocked commands. Pin every job.
3. **WeasyPrint needs DYLD_LIBRARY_PATH on macOS** (`/opt/homebrew/lib`), set before import, or pango/glib won't load.
4. **No flexbox.** Real HTML `<table>` elements only — WeasyPrint's flexbox support is unreliable.
5. **Page count check is mandatory.** Render and check every time; never assume it fits.
6. **QR codes minimum 45px** (lead 50px) — smaller won't scan when printed. Base64 data URIs, not file paths.
7. **Newsletter senders change addresses.** Store and search by DOMAIN; retry with `CATEGORY_UPDATES` before declaring none found.
8. **Stale news backfills from memory.** Every story needs a verified date within 36 hours; the printed attribution date is the receipt. Reddit RSS makes this verifiable by construction.
9. **Gmail helper needs Python 3.10+** — run `google_api.py` with the agent venv interpreter.
10. **GLM models: end the cron response with a period** to avoid a false-positive truncation heuristic.
11. **No permanent scripts in the skill** — everything is throwaway in `/tmp/`. The skill is a rubric, not a codebase.

## Verification

- [ ] Config read; today's date fetched via `terminal`
- [ ] Every data source attempted; failures rendered as error text in their sections
- [ ] Every news story dated within 36 hours; dates printed in source attributions
- [ ] Newsletter search by domain, with CATEGORY_UPDATES fallback
- [ ] HTML written with `write_file`; CSS copied verbatim from the design system
- [ ] PDF rendered; page count verified 1
- [ ] `lp` returned exit 0 with a request ID
- [ ] Final response is one line ending with a period