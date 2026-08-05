# WS Display — Ruler Verification Tracker

Single-page static site (`index.html`, no build step) hosted on GitHub Pages:
https://mikeb-art.github.io/rulers/

## What it does
Tracks media-compensation ruler checks for 7 heat presses across CA / MX / PA
(CA Monti-1, CA Monti-2, MX Monti-1, MX Monti-3, MX Monti-4, PA Monti-1, PA Monti-2),
8 materials each — 4 monthly (Stretch, Black and White, Backlit, MultiStretch) and
4 quarterly (Mesh, Tent, SilverBack, SmoothWall). Green = current, amber = due soon,
red = overdue.

## Data source
Google Sheet **"Ruler Measurements (Responses)"**
ID: `1TPT9JRnmKSdCxjSql-SGl21mUyk2ZcCnUoa3lubcAyc` (mostly Google Form responses,
plus hand-edited rows). Read client-side via the Sheets API after Google sign-in
(wsdisplay.com accounts only, OAuth client in CONFIG). One tab per machine;
columns match `HEADER_RE`: `(CA|MX|PA) Monti-N <Material> (Monthly|Quarterly) (Horizontal|Vertical)`.

## Status logic
- `SCHEDULE` pins the 2026 upload Wednesdays (monthly = every 4 weeks; quarterly =
  Mar 18, Jun 10, Sep 2, Dec 23). **Needs updating each new year.**
- An upload counts for a scheduled date if it lands up to `EARLY_DAYS` (5) before it.
- "Due soon" = next scheduled date within `LEAD_DAYS` (10).
- Never-uploaded machine×material combos count as overdue.
- Colors on deviation values use inches (actual − expected): <+0.5" red, +0.5–1.5" green, >+1.5" orange.

## Sheet quirks (learned the hard way — keep the parser tolerant)
- Some tabs (PA Monti-1, PA Monti-2) have **duplicate header columns** for the same
  press/material/direction; data lands in only one of them, and which one varies.
  The parser keeps ALL candidate columns and per-row uses whichever cell has a measurement.
- Measurement typos are common: missing slash between quoted inches (`100"96.25"`,
  `93"89.5"`), fractions (`100"/98 5/8"`), junk tokens (`NA`, `pending`, `complete`,
  `.`, `-`, `0`, `1`), and free-text "via chat" rows with bare numbers that must NOT
  be parsed as measurements.
- Date column is "Date" or "Timestamp"; formats include M/D/YYYY h:mm, YYYYMMDD,
  and sheet serial numbers (fetched with `dateTimeRenderOption=SERIAL_NUMBER`).
- PA tabs label pairs "(Expected / Actual)"; others "(Ruler / Actual)". Same meaning.

## History
Built 2026-07-30. 2026-08-05: fixed dropped rows (duplicate header columns +
missing-slash typo) that made PA Monti-1 Stretch show overdue despite a same-day upload.

## Sibling dashboards (same pattern, linked in the sidebar)
- https://mikeb-art.github.io/barbieri-doc-monitoring/
- https://mikeb-art.github.io/machine-maintenance/
- https://mikeb-art.github.io/temp-humidity/
