# Onboarding — Morning Briefing Skill

The onboarding is a conversation, not a wizard. The agent discovers what's available, presents findings, and the user adjusts in natural language.

## Step 1: Auto-Discovery

Probe the environment for available data sources and infrastructure. Do this silently — present results, not the probing process.

### Check for Home Assistant
- Call `ha_list_entities(domain='weather')` — if results, HA is available
- Call `ha_list_entities(domain='calendar')` — find calendar entities
- Call `ha_list_entities(domain='sensor')` — filter for AQI/air quality sensors
- Record: weather entity ID(s), calendar entity ID(s), AQI sensor entity ID(s)

### Check for Google Calendar connector
- Call `manage_connections(action='status')` and look for gmail/google-workspace
- If connected, note it as a calendar source

### Check for Gmail connector (for newsletters)
- Same status check — if Gmail is connected, newsletter scanning is possible

### Check for tinyair MCP
- Call `tool_search(queries=['air quality'])` — if tinyair tools appear, note them

### Check for xAI OAuth (for X/Twitter search)
- Call `tool_search(queries=['x search twitter'])` — if x_search appears, X search is available

### Check for printer
- Run `lpstat -p` via terminal — list available CUPS printers
- If no printers found, note that the skill will need to use file output + notification fallback

### Check for Tailscale
- Run `tailscale status` via terminal — if active, record the hostname
- Run `tailscale serve status` — see what ports are already served

### Check for Cloudflare Tunnel
- Run `which cloudflared` and check for config — if present, note as alternative for overflow hosting

## Step 2: Present Findings

Present a summary of what was found:

```
Here's what I found for your morning briefing setup:

✅ Weather: <entity_id> (HA, NWS data)
✅ Calendar: <entity_id> (HA)
✅ Air Quality: <entity_id> (HA, AirNow)
✅ Printer: <printer_name> (CUPS)
✅ Tailscale: <hostname>
✅ X Search: available (xAI OAuth)
❌ Gmail: not connected (newsletter section will be skipped)

Does this look right? Anything you'd like to change?
```

Let the user confirm or adjust. Accept natural language corrections.

## Step 3: Interest Profile Interview

Ask the user what topics they want in their news briefing. Then deep-dive each topic.

```
What topics do you want covered in your morning news briefing?

Recommended: Start with 3-5 broad topics (e.g., "local news", "AI", "ski industry", "mountain biking").
```

For each topic, ask a follow-up to narrow the interest profile:

```
For [topic], what specifically interests you? What should I include vs. ignore?

Example for "ski industry": "Resort acquisitions, pass changes, labor/seasonal workforce, weather impact on operations. Not brand campaigns or gear reviews."
```

Store the interest profile as structured data in the config file:
```json
{
  "news_topics": [
    {
      "topic": "Ski Industry",
      "keywords": ["resort operator", "season pass", "ski area"],
      "interest_description": "Resort acquisitions, pass changes, labor/seasonal workforce, weather impact on operations. Not brand campaigns or gear reviews.",
      "sources": ["x_search", "web_search"]
    },
    {
      "topic": "AI",
      "keywords": ["AI", "LLM", "machine learning"],
      "interest_description": "Model releases, safety research, regulatory moves, industry impact. Not marketing blog posts or tutorial content.",
      "sources": ["x_search", "web_search"]
    }
  ]
}
```

## Step 4: Newsletter Discovery (if Gmail connected)

If the Gmail connector is available:

```
Would you like me to scan your inbox for newsletters to include in the briefing?

I'll look for recurring email senders with newsletter-like content and show you what I find. You can pick which ones to include.
```

If the user says yes, scan the inbox for patterns:
- Sender frequency (recurring daily/weekly emails)
- Unsubscribe links present
- Content structure (articles, links, summaries)

Present candidates:

```
I found these potential newsletters in your inbox:

1. Morning Brew (morningbrew@...) — daily, tech/business news
2. [other]

Which would you like included in your morning briefing?
```

Store selected senders in the config.

## Step 5: Overflow Hosting

```
For overflow content (news that doesn't fit on the printed page), I can host a web page you can reach via QR code.

I detected Tailscale running on <hostname>. I can serve overflow pages there, or if you prefer, I can use your Cloudflare tunnel.

Which would you prefer? (Tailscale is more secure, Cloudflare works from anywhere)
```

Set up a `tailscale serve` port for the overflow directory.

## Step 6: Write Config

Write the final config to `~/.hermes/scripts/morning-briefing-config.json`:

```json
{
  "weather_entity": "<ha_weather_entity>",
  "calendar_entities": ["<ha_calendar_entity>"],
  "air_quality_entity": "<ha_aqi_sensor>",
  "printer_name": "<cups_printer_name>",
  "location": "<city, state>",
  "news_topics": [...],
  "newsletter_senders": [...],
  "overflow_host": "<tailscale_hostname>",
  "overflow_port": "18091",
  "overflow_dir": "~/.hermes/www/briefing-overflow/",
  "notify_entity": "<ha_notify_entity>"
}
```

## Step 7: Test Run

Generate a sample briefing immediately:

```
Let me generate a test briefing right now so you can see how it looks.
```

Run the full procedure (gather data, render, check page count, print). Show the user the rendered HTML. Ask for feedback on layout and content.

## Step 8: Set Up Cron Job

If the user is satisfied with the test run, set up the daily cron job:

- Schedule: `0 6 * * *` (6 AM local)
- Toolsets: `homeassistant`, `file`, `terminal`, `web`, `browser`
- Model: pin explicitly to the user's preferred model
- Deliver: `local`
- Prompt references this skill by name

Test with `cronjob(action='run')` before reporting done.