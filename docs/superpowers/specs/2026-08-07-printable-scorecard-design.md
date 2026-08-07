# Printable Scorecard Design

## Purpose

Produce a physical, foldable scorecard for Roby Mill Pub Golf (Sat 15th August, Blackpool) that Conor can print at home to check, then send to an online print shop to have ~10 copies professionally printed (3 players, spares) before the event.

## Format

- Flat sheet: A5 landscape (210mm x 148mm), pre-scored down the middle.
- Folded size: A6 (105mm x 148mm) — a single fold produces 4 panels: front cover, inside-left, inside-right, back cover.
- Target print vendor: [Quadrant Print — A6 Folded Cards](https://www.quadrantprint.co.uk/a6-folded-cards.php). 350gsm smooth white card, 10 cards for £31, next-day delivery if ordered by 1pm (accepts JPEG/TIFF/PDF upload). This is a recommendation, not an integration — Conor places the order himself.

## Content per panel

**Front cover** (105 x 148mm)
- "Roby Mill Pub Golf" title
- "Saturday 15th August"
- "12 Holes · Par 35" tagline
- Navy/white/coral styling consistent with the live site

**Inside spread** (inside-left + inside-right read as one continuous 210 x 148mm table across the fold)
- One row per hole (12 rows) + header row + totals row
- Columns: hole #, pub name, drink, par, and one score-entry box per player (3 players)
- Same hole order/pars/drinks as the live site (Ma Kellys → Cowboy & Co. → Walkabout → Nellie Deans → Bees Knees → Retro Lounge → Rose & Crown → Scruffy Murphys → Layton Rakes → Shenanigans → Coyote Ugly → Knobby's Karaoke)

**Back cover** (105 x 148mm)
- Penalty Strokes table (same order/values as the site: Piss between pubs +1, Spilled drink +2, Broken glass +3, Dry hole +3, Throwing up +3)
- 3 player name fields
- Final total-score box per player

## Technical approach

- New standalone file, `scorecard-print.html`, alongside `index.html` in the repo root. Not linked from the main site nav (Conor opens it directly/locally) — purely a print artifact.
- Sized precisely with CSS `@page { size: 210mm 148mm; margin: 0; }` and panels laid out with `mm` units so the browser's print/"Save as PDF" output matches the physical trim size exactly.
- Reuses the site's existing fonts (Playfair Display / Inter / IBM Plex Mono) and color palette (navy `--green`/`--green-deep`, coral `--gold`, light-blue `--cream`/`--cream-deep`) for visual consistency with `index.html`.
- Four panels built as print pages/sections in reading order (front cover, inside-left, inside-right, back cover) with `page-break-after` between the outside pages and the inside spread, so printing produces a 2-page PDF: page 1 = outside (back cover + front cover side by side, in the correct order for folding), page 2 = inside spread (inside-left + inside-right side by side).
- Score boxes are empty bordered cells (this is a printed artifact filled in by hand with a pen during the crawl) — no interactivity, no JS.
- Deliverable is the HTML file plus instructions for Conor to open it in Chrome, use Print → Save as PDF (paper size A5, landscape, no margins/scaling) to produce the upload-ready PDF for Quadrant Print.

## Out of scope

- Placing the print order (Conor does this himself).
- Any interactivity, score totaling, or persistence — it's a static print layout.
- Linking the scorecard page from the main site navigation.
- Bleed/crop marks — Quadrant Print's digital short-run product doesn't require them for this trim size; if their upload tool flags an issue, Conor will follow their on-site guidance.

## Verification

- Open `scorecard-print.html` in a browser and use print preview to confirm: exact A5 landscape page size, no clipped content, fold falls in a sensible place on the inside spread (between columns, not through them), all 12 holes + penalties + player fields are legible at actual print size.
