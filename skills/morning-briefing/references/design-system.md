# Design System — Morning Briefing

The agent MUST use this CSS stylesheet verbatim in every briefing HTML. Copy it into the `<style>` block of `/tmp/briefing.html`. Do not modify sizes, fonts, colors, or layout rules. Only fill in the content.

## Complete Stylesheet

```css
@page { size: letter; margin: 0.3in 0.4in; }
* { box-sizing: border-box; margin: 0; padding: 0; }
body {
  font-family: 'Iowan Old Style', 'Palatino Linotype', 'Book Antiqua', Palatino, Georgia, serif;
  font-size: 8.5pt; color: #000; line-height: 1.3;
}

/* Masthead */
.masthead {
  border-bottom: 3px double #000; padding-bottom: 4px; margin-bottom: 6px; text-align: center;
}
.masthead h1 { font-size: 24pt; font-weight: 900; letter-spacing: -1px; }
.masthead .dateline {
  font-size: 7pt; margin-top: 2px; font-variant: small-caps; letter-spacing: 1.5px;
}

/* Sections */
.section { margin-bottom: 5px; }
.section-title {
  font-size: 6.5pt; font-weight: 700; text-transform: uppercase;
  letter-spacing: 2px; border-bottom: 1px solid #000; padding-bottom: 1px;
  margin-bottom: 3px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
}

/* Weather + Calendar — table layout (NOT flexbox) */
table.layout { width: 100%; border-collapse: collapse; }
table.weather-cols { width: 100%; border-collapse: collapse; }
td { vertical-align: top; }
td.w-main { width: 27%; border-right: 1px solid #000; padding-right: 6px; }
td.w-current { width: 36%; border-right: 1px solid #000; padding-right: 6px; font-size: 7.5pt; }
td.w-forecast { width: 37%; font-size: 7.5pt; }
td.w-main .temp { font-size: 20pt; font-weight: 700; line-height: 1; }
td.w-main .cond { font-size: 7pt; margin-top: 1px; }
.forecast-line .day { font-weight: 700; font-size: 6.5pt; text-transform: uppercase; }
td.wc-left { width: 68%; border-right: 1px solid #000; padding-right: 6px; }
td.wc-right { width: 32%; padding-left: 6px; text-align: center; vertical-align: middle; }
td.wc-right img { width: 140px; height: auto; }
td.wc-right .radar-label { font-size: 6pt; color: #555; margin-top: 1px; font-style: italic; }

/* Calendar nested below weather inside left cell */
.cal-nested { border-top: 1px dotted #000; padding-top: 3px; margin-top: 4px; font-size: 7.5pt; }
.cal-nested-title {
  font-weight: 700; font-size: 6.5pt; text-transform: uppercase;
  letter-spacing: 2px; margin-bottom: 2px;
  font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
}
table.cal-event { width: 100%; border-collapse: collapse; font-size: 8pt; }
td.cal-time { font-weight: 700; width: 50px; font-size: 7.5pt; }
td.cal-text .cal-empty { color: #555; font-style: italic; }

/* Lead story — full width above the grid */
.lead-story { margin-bottom: 5px; padding-bottom: 4px; border-bottom: 2px solid #000; }
.lead-story .topic {
  font-weight: 700; font-size: 6.5pt; text-transform: uppercase;
  letter-spacing: 2px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
}
.lead-story .headline { font-size: 11pt; font-weight: 900; line-height: 1.1; margin: 1px 0 2px; }
table.lead-body { width: 100%; border-collapse: collapse; }
td.lead-summary { font-size: 8.5pt; line-height: 1.3; }
td.lead-qr { width: 55px; text-align: center; }
td.lead-qr img { width: 50px; height: 50px; }
td.lead-qr .qr-label { font-size: 5.5pt; color: #555; }
.lead-story .source { font-size: 6.5pt; color: #555; font-style: italic; margin-top: 2px; }

/* News masonry grid — 2 columns with QR codes on right */
.news-grid { column-count: 2; column-gap: 10px; column-rule: 1px solid #000; }
.news-card { break-inside: avoid; margin-bottom: 4px; padding-bottom: 3px; border-bottom: 1px dotted #000; }
.news-card:last-child { border-bottom: none; }
table.news-body { width: 100%; border-collapse: collapse; }
td.news-text { vertical-align: top; }
td.news-qr { width: 50px; vertical-align: top; text-align: center; }
.news-card .topic {
  font-weight: 700; font-size: 6pt; text-transform: uppercase;
  letter-spacing: 1.5px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
  margin-bottom: 1px; border-bottom: 0.5px solid #000; padding-bottom: 1px;
}
.news-card .headline { font-size: 8.5pt; font-weight: 700; line-height: 1.1; margin-bottom: 1px; }
.news-card .summary { font-size: 8pt; line-height: 1.25; }
.news-card .source { font-size: 6pt; color: #555; margin-top: 1px; font-style: italic; }
td.news-qr img { width: 45px; height: 45px; }
td.news-qr .qr-label { font-size: 5pt; color: #555; }

/* Newsletter section */
.newsletter-section { margin-top: 5px; padding: 4px 0; border-top: 1px solid #000; }
.newsletter-title {
  font-weight: 700; font-size: 6.5pt; text-transform: uppercase;
  letter-spacing: 2px; margin-bottom: 3px;
  font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
}
table.nl-body { width: 100%; border-collapse: collapse; }
td.nl-text { vertical-align: top; font-size: 8pt; line-height: 1.25; }
td.nl-text .nl-name {
  font-weight: 700; font-size: 6.5pt; text-transform: uppercase;
  letter-spacing: 1.5px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
}
td.nl-text .nl-headline { font-size: 8.5pt; font-weight: 700; margin: 1px 0; }
td.nl-text .nl-summary { font-size: 8pt; }
td.nl-text .nl-also { font-size: 7pt; color: #555; font-style: italic; margin-top: 2px; }
td.nl-qr { width: 50px; vertical-align: top; text-align: center; }
td.nl-qr img { width: 45px; height: 45px; }
td.nl-qr .qr-label { font-size: 5pt; color: #555; }

/* Joke */
.joke {
  text-align: center; font-size: 8pt; font-style: italic;
  margin-top: 4px; padding: 3px 15px; border-top: 1px solid #000;
}
.joke .joke-label {
  font-size: 6pt; text-transform: uppercase; letter-spacing: 2px;
  font-style: normal; font-weight: 700; margin-bottom: 1px;
  font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
}

/* Footer */
.footer {
  font-size: 6pt; color: #888; text-align: center;
  margin-top: 2px; border-top: 1px solid #000; padding-top: 2px; font-style: italic;
}
```

## Layout Rules

These are hard constraints. The agent must follow them exactly.

### Structure (top to bottom)

1. **Masthead** — centered title "The Morning Briefing" + dateline (date, location, volume/issue)
2. **Weather & Calendar** — single section, table layout with 3 weather columns + calendar nested below + radar in right cell
3. **News & Signal** — lead story (full width) + 2-column masonry grid
4. **Newsletter Digest** — after news, separated by solid rule
5. **Joke of the Day** — centered, italic, above footer
6. **Footer** — centered, small, italic

### Typography

| Element | Font | Size | Weight |
|---------|------|------|--------|
| Body | Iowan Old Style / Palatino (serif) | 8.5pt | normal |
| Title | Iowan Old Style / Palatino (serif) | 24pt | 900 |
| Section headers | Helvetica Neue (sans-serif) | 6.5pt | 700, uppercase |
| Topic labels | Helvetica Neue (sans-serif) | 6pt | 700, uppercase |
| Lead headline | Iowan Old Style (serif) | 11pt | 900 |
| Card headline | Iowan Old Style (serif) | 8.5pt | 700 |
| Dateline | Iowan Old Style (serif) | 7pt | small-caps |
| Source attribution | Iowan Old Style (serif) | 6pt | italic |
| Footer | Iowan Old Style (serif) | 6pt | italic |

### Color

- **All black and white** — no color anywhere except the NWS radar image
- Text: #000
- Secondary text: #555
- Borders: #000 (solid for sections, dotted between cards)
- No colored badges, no colored AQI, no colored backgrounds

### QR Code Sizes (minimum, do not shrink)

| Location | Size |
|----------|------|
| Lead story | 50px × 50px |
| Grid cards | 45px × 45px |
| Newsletter | 45px × 45px |

### Radar Image

- Color (not grayscale)
- NWS GIF source, cropped ~33% centered on user's area
- Resized to 140px wide, embedded as base64 data URI
- Right cell of weather table, vertically centered

### Layout Technology

- **Use real HTML `<table>` elements** for all multi-column layouts
- **Do NOT use CSS flexbox** — WeasyPrint does not support it reliably
- **Do NOT use CSS `display: table`** — real `<table>` elements are more reliable
- Use `column-count: 2` for the news masonry grid (this works in WeasyPrint)