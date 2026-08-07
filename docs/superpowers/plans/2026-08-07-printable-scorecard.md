# Printable Scorecard Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build `scorecard-print.html`, a static, print-ready A5-landscape scorecard (folds once to A6) for Roby Mill Pub Golf, ready to export as a 2-page PDF and upload to Quadrant Print.

**Architecture:** A single self-contained HTML file with no JavaScript. Two `.sheet` divs, each exactly `210mm x 148mm`, represent the two printed sides (outside = back cover + front cover; inside = the continuous 12-hole scorecard spread). `@page { size: 210mm 148mm; margin: 0 }` plus `break-after: page` between the sheets makes a browser's native Print → Save as PDF produce a correctly-sized 2-page PDF with no extra tooling.

**Tech Stack:** Plain HTML + CSS (mm units), Google Fonts (Playfair Display / Inter / IBM Plex Mono) — same stack as `index.html`. No build step, no JS, no test framework in this repo.

## Global Constraints

- Flat sheet size: A5 landscape, `210mm x 148mm`. Folds once down the middle to A6 (`105mm x 148mm`).
- File is a print artifact only — do not link it from `index.html` navigation.
- No JavaScript, no interactivity — score boxes are empty cells filled in by hand.
- Reuse `index.html`'s exact color palette: `--cream:#F2F6FB; --cream-deep:#E1EBF7; --white:#FFFFFF; --green:#1D4E89; --green-deep:#0B2545; --gold:#E8622C; --text:#14213D; --muted:#5B6B85; --line:#C9D6E8;`
- Reuse `index.html`'s font stack: Playfair Display (headings), Inter (body), IBM Plex Mono (labels/numbers), loaded from the same Google Fonts URL.
- No automated test framework exists in this repo. Verification is done by loading the file in the browser (`mcp__Claude_Browser__navigate`) and checking rendered dimensions/overflow via `mcp__Claude_Browser__javascript_tool`, plus a visual screenshot check.
- Hole order, pars, and drinks must match the live site exactly (see Task 3) — this is the same 12-stop route already shipped in `index.html`.

---

### Task 1: Page scaffold and print sizing

**Files:**
- Create: `scorecard-print.html`

**Interfaces:**
- Consumes: nothing (new file).
- Produces: two top-level `.sheet` elements (`#sheet-outside`, `#sheet-inside`), each `210mm x 148mm`. `#sheet-outside` contains `.panel.back-cover` and `.panel.front-cover` (each `105mm x 148mm`, left-to-right). `#sheet-inside` contains a single `.inside-content` div (`210mm x 148mm`, `padding: 4mm 7.5mm`). Tasks 2 and 3 fill these containers — their class names and dimensions are fixed by this task and must not change.

- [ ] **Step 1: Create `scorecard-print.html` with the full page skeleton**

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Roby Mill Pub Golf — Printable Scorecard</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@600;700;800&family=Inter:wght@400;500;600;700&family=IBM+Plex+Mono:wght@500;600&display=swap" rel="stylesheet">
<style>
  :root{
    --cream:#F2F6FB;
    --cream-deep:#E1EBF7;
    --white:#FFFFFF;
    --green:#1D4E89;
    --green-deep:#0B2545;
    --gold:#E8622C;
    --text:#14213D;
    --muted:#5B6B85;
    --line:#C9D6E8;
  }
  *{box-sizing:border-box;}
  html,body{margin:0;padding:0;background:#CCCCCC;font-family:'Inter',sans-serif;color:var(--text);}
  @page{ size:210mm 148mm; margin:0; }

  .sheet{
    width:210mm;
    height:148mm;
    display:flex;
    background:var(--white);
    page-break-after:always;
    break-after:page;
    margin:0 auto 4mm;
    box-shadow:0 0 0 0.3mm var(--line);
    overflow:hidden;
  }
  .sheet:last-child{ page-break-after:auto; break-after:auto; margin-bottom:0; }

  .panel{
    width:105mm;
    height:148mm;
    box-sizing:border-box;
    overflow:hidden;
  }

  .inside-content{
    width:210mm;
    height:148mm;
    box-sizing:border-box;
    padding:4mm 7.5mm;
    overflow:hidden;
  }

  @media print{
    body{ background:var(--white); }
    .sheet{ box-shadow:none; margin:0; }
  }
</style>
</head>
<body>

<div class="sheet" id="sheet-outside">
  <div class="panel back-cover"></div>
  <div class="panel front-cover"></div>
</div>

<div class="sheet" id="sheet-inside">
  <div class="inside-content"></div>
</div>

</body>
</html>
```

- [ ] **Step 2: Verify page structure and exact print dimensions**

Run (via `mcp__Claude_Browser__navigate` to `file:///Users/conorhiggins/pubgolf/scorecard-print.html`, then `mcp__Claude_Browser__javascript_tool`):

```js
(() => {
  const sheets = document.querySelectorAll('.sheet');
  const outside = document.getElementById('sheet-outside').getBoundingClientRect();
  const inside = document.querySelector('.inside-content').getBoundingClientRect();
  const back = document.querySelector('.back-cover').getBoundingClientRect();
  const front = document.querySelector('.front-cover').getBoundingClientRect();
  return {
    sheetCount: sheets.length,
    outsideW: Math.round(outside.width), outsideH: Math.round(outside.height),
    insideW: Math.round(inside.width), insideH: Math.round(inside.height),
    backW: Math.round(back.width), frontW: Math.round(front.width)
  };
})()
```

Expected (96 CSS px/in, 1mm = 3.7795px, tolerance ±2px): `sheetCount: 2`, `outsideW: 794`, `outsideH: 559`, `insideW: 794`, `insideH: 559`, `backW: 397`, `frontW: 397`.

- [ ] **Step 3: Commit**

```bash
git add scorecard-print.html
git commit -m "Add print scorecard page scaffold sized to A5 landscape"
```

---

### Task 2: Front cover and back cover content

**Files:**
- Modify: `scorecard-print.html` (fill `.panel.front-cover` and `.panel.back-cover`, both currently empty from Task 1)

**Interfaces:**
- Consumes: `.panel.front-cover` and `.panel.back-cover` empty containers (`105mm x 148mm` each) produced by Task 1.
- Produces: fully populated front/back cover markup. Task 4's full-card screenshot depends on this being visually complete (no empty panels).

- [ ] **Step 1: Fill in the front cover**

Replace `<div class="panel front-cover"></div>` with:

```html
<div class="panel front-cover" style="display:flex;flex-direction:column;align-items:center;justify-content:center;text-align:center;padding:10mm;background:var(--cream);">
  <div style="font-family:'IBM Plex Mono',monospace;letter-spacing:0.16em;text-transform:uppercase;font-size:7pt;color:var(--green);margin-bottom:6mm;">Saturday 15th August</div>
  <h1 style="font-family:'Playfair Display',serif;font-weight:800;font-size:20pt;line-height:1.05;margin:0;color:var(--green-deep);">Roby Mill<br>Pub Golf</h1>
  <hr style="width:14mm;height:0.6mm;background:var(--gold);border:none;margin:6mm auto;">
  <div style="font-size:9pt;color:var(--muted);font-weight:500;">12 Holes &middot; <strong style="color:var(--green-deep);">Par 35</strong></div>
</div>
```

- [ ] **Step 2: Fill in the back cover**

Replace `<div class="panel back-cover"></div>` with:

```html
<div class="panel back-cover" style="padding:8mm;background:var(--white);display:flex;flex-direction:column;">
  <div style="font-family:'Playfair Display',serif;font-weight:700;font-size:12pt;color:var(--green-deep);margin-bottom:2mm;">Penalty Strokes</div>
  <table style="width:100%;border-collapse:collapse;font-size:7.5pt;margin-bottom:6mm;">
    <tr><td style="padding:1.2mm 0;border-bottom:0.2mm solid var(--line);">Piss between pubs</td><td style="padding:1.2mm 0;border-bottom:0.2mm solid var(--line);text-align:right;font-family:'IBM Plex Mono',monospace;color:var(--green);font-weight:600;">+1</td></tr>
    <tr><td style="padding:1.2mm 0;border-bottom:0.2mm solid var(--line);">Spilled drink</td><td style="padding:1.2mm 0;border-bottom:0.2mm solid var(--line);text-align:right;font-family:'IBM Plex Mono',monospace;color:var(--green);font-weight:600;">+2</td></tr>
    <tr><td style="padding:1.2mm 0;border-bottom:0.2mm solid var(--line);">Broken glass</td><td style="padding:1.2mm 0;border-bottom:0.2mm solid var(--line);text-align:right;font-family:'IBM Plex Mono',monospace;color:var(--green);font-weight:600;">+3</td></tr>
    <tr><td style="padding:1.2mm 0;border-bottom:0.2mm solid var(--line);">Dry hole</td><td style="padding:1.2mm 0;border-bottom:0.2mm solid var(--line);text-align:right;font-family:'IBM Plex Mono',monospace;color:var(--green);font-weight:600;">+3</td></tr>
    <tr><td style="padding:1.2mm 0;">Throwing up</td><td style="padding:1.2mm 0;text-align:right;font-family:'IBM Plex Mono',monospace;color:var(--green);font-weight:600;">+3</td></tr>
  </table>
  <div style="font-family:'Playfair Display',serif;font-weight:700;font-size:12pt;color:var(--green-deep);margin-bottom:2mm;">Players</div>
  <table style="width:100%;border-collapse:collapse;font-size:8pt;">
    <tr><td style="padding:2.5mm 0;border-bottom:0.3mm solid var(--text);width:5mm;">1.</td><td style="padding:2.5mm 0;border-bottom:0.3mm solid var(--text);">&nbsp;</td></tr>
    <tr><td style="padding:2.5mm 0;border-bottom:0.3mm solid var(--text);">2.</td><td style="padding:2.5mm 0;border-bottom:0.3mm solid var(--text);">&nbsp;</td></tr>
    <tr><td style="padding:2.5mm 0;border-bottom:0.3mm solid var(--text);">3.</td><td style="padding:2.5mm 0;border-bottom:0.3mm solid var(--text);">&nbsp;</td></tr>
  </table>
  <div style="margin-top:auto;display:flex;justify-content:space-between;font-family:'IBM Plex Mono',monospace;font-size:6.5pt;color:var(--muted);border-top:0.3mm solid var(--line);padding-top:3mm;">
    <span>TOTAL P1 ____</span><span>P2 ____</span><span>P3 ____</span>
  </div>
</div>
```

- [ ] **Step 3: Verify no overflow and visually check**

Run via `javascript_tool`:

```js
(() => {
  const front = document.querySelector('.front-cover');
  const back = document.querySelector('.back-cover');
  return {
    frontOverflow: front.scrollHeight > front.clientHeight,
    backOverflow: back.scrollHeight > back.clientHeight
  };
})()
```

Expected: `{ frontOverflow: false, backOverflow: false }`. Then take a screenshot (`mcp__Claude_Browser__computer` action `screenshot`) and confirm the front cover is centered/legible and the back cover's penalty table, player rows, and total line all fit without clipping.

- [ ] **Step 4: Commit**

```bash
git add scorecard-print.html
git commit -m "Add front and back cover content to print scorecard"
```

---

### Task 3: Inside spread scorecard table

**Files:**
- Modify: `scorecard-print.html` (fill `.inside-content`, currently empty from Task 1)

**Interfaces:**
- Consumes: `.inside-content` empty container (`210mm x 148mm`, `padding: 4mm 7.5mm`) produced by Task 1.
- Produces: the completed 12-hole `.score-table`. Task 4's full-card screenshot depends on this being visually complete.

- [ ] **Step 1: Add the score-table CSS**

Add inside the existing `<style>` block in `scorecard-print.html`:

```css
  .score-table{ width:195mm; border-collapse:collapse; table-layout:fixed; }
  .score-table thead tr{ height:8mm; }
  .score-table tbody tr{ height:10.17mm; }
  .score-table tfoot tr{ height:10mm; }
  .score-table thead th{
    font-family:'IBM Plex Mono',monospace;
    font-size:6.5pt;
    text-transform:uppercase;
    letter-spacing:0.04em;
    color:var(--white);
    background:var(--green-deep);
    text-align:left;
    padding:0 1.5mm;
  }
  .score-table thead th.center{ text-align:center; }
  .score-table tbody td{
    font-size:7pt;
    color:var(--text);
    border-bottom:0.2mm solid var(--line);
    padding:0 1.5mm;
    vertical-align:middle;
  }
  .score-table tbody td.hole-num{
    font-family:'IBM Plex Mono',monospace;
    font-weight:600;
    text-align:center;
    color:var(--green);
  }
  .score-table tbody td.par{
    font-family:'IBM Plex Mono',monospace;
    text-align:center;
    color:var(--muted);
  }
  .score-table tbody td.score-box{
    border-left:0.2mm solid var(--line);
  }
  .score-table tfoot td{
    font-family:'IBM Plex Mono',monospace;
    font-weight:700;
    font-size:7.5pt;
    color:var(--green-deep);
    border-top:0.4mm solid var(--text);
    padding:0 1.5mm;
  }
```

- [ ] **Step 2: Fill in `.inside-content` with the 12-hole table**

Replace `<div class="inside-content"></div>` with:

```html
<div class="inside-content">
  <table class="score-table">
    <colgroup>
      <col style="width:9.5mm;">
      <col style="width:68mm;">
      <col style="width:20mm;">
      <col style="width:32.5mm;">
      <col style="width:32.5mm;">
      <col style="width:32.5mm;">
    </colgroup>
    <thead>
      <tr>
        <th class="center">#</th>
        <th>Pub &amp; Drink</th>
        <th class="center">Par</th>
        <th class="center">P1</th>
        <th class="center">P2</th>
        <th class="center">P3</th>
      </tr>
    </thead>
    <tbody>
      <tr><td class="hole-num">1</td><td>Ma Kellys — Pint of lager</td><td class="par">5</td><td class="score-box"></td><td class="score-box"></td><td class="score-box"></td></tr>
      <tr><td class="hole-num">2</td><td>Cowboy &amp; Co. — Whisky &amp; mixer</td><td class="par">3</td><td class="score-box"></td><td class="score-box"></td><td class="score-box"></td></tr>
      <tr><td class="hole-num">3</td><td>Walkabout — Cocktail of choice</td><td class="par">3</td><td class="score-box"></td><td class="score-box"></td><td class="score-box"></td></tr>
      <tr><td class="hole-num">4</td><td>Nellie Deans — Baby Guinness shot</td><td class="par">1</td><td class="score-box"></td><td class="score-box"></td><td class="score-box"></td></tr>
      <tr><td class="hole-num">5</td><td>Bees Knees — Cocktail off the menu</td><td class="par">3</td><td class="score-box"></td><td class="score-box"></td><td class="score-box"></td></tr>
      <tr><td class="hole-num">6</td><td>The Retro Lounge — 2-4-1 cocktail</td><td class="par">3</td><td class="score-box"></td><td class="score-box"></td><td class="score-box"></td></tr>
      <tr><td class="hole-num">7</td><td>Rose &amp; Crown — Pint of lager</td><td class="par">4</td><td class="score-box"></td><td class="score-box"></td><td class="score-box"></td></tr>
      <tr><td class="hole-num">8</td><td>Scruffy Murphys — Full pour Guinness</td><td class="par">5</td><td class="score-box"></td><td class="score-box"></td><td class="score-box"></td></tr>
      <tr><td class="hole-num">9</td><td>The Layton Rakes — Double spirit + mixer</td><td class="par">3</td><td class="score-box"></td><td class="score-box"></td><td class="score-box"></td></tr>
      <tr><td class="hole-num">10</td><td>Shenanigans — Irish whiskey shot</td><td class="par">1</td><td class="score-box"></td><td class="score-box"></td><td class="score-box"></td></tr>
      <tr><td class="hole-num">11</td><td>Coyote Ugly — Tequila shot</td><td class="par">1</td><td class="score-box"></td><td class="score-box"></td><td class="score-box"></td></tr>
      <tr><td class="hole-num">12</td><td>Knobby's Karaoke — Final song</td><td class="par">3</td><td class="score-box"></td><td class="score-box"></td><td class="score-box"></td></tr>
    </tbody>
    <tfoot>
      <tr>
        <td colspan="2">TOTAL PAR</td>
        <td class="par" style="color:var(--green-deep);">35</td>
        <td class="score-box"></td>
        <td class="score-box"></td>
        <td class="score-box"></td>
      </tr>
    </tfoot>
  </table>
</div>
```

- [ ] **Step 3: Verify table width, fold alignment, and no overflow**

Run via `javascript_tool`:

```js
(() => {
  const table = document.querySelector('.score-table');
  const wrap = document.querySelector('.inside-content');
  const cols = [...document.querySelectorAll('.score-table colgroup col')].map(c => parseFloat(c.style.width));
  const foldOffsetMm = cols[0] + cols[1] + cols[2];
  return {
    tableWidthPx: Math.round(table.getBoundingClientRect().width),
    contentOverflow: wrap.scrollHeight > wrap.clientHeight,
    rowCount: document.querySelectorAll('.score-table tbody tr').length,
    foldOffsetMm
  };
})()
```

Expected: `tableWidthPx: 737` (195mm, tolerance ±2px), `contentOverflow: false`, `rowCount: 12`, `foldOffsetMm: 97.5` (this is the physical fold line at 105mm from the sheet's left edge, minus the 7.5mm left padding — it must land exactly on the boundary between the Par and P1 columns, not inside a column).

Then screenshot and visually confirm all 12 rows are legible and the header/total rows are styled distinctly (dark navy header, bold total row).

- [ ] **Step 4: Commit**

```bash
git add scorecard-print.html
git commit -m "Add 12-hole scorecard table to inside spread"
```

---

### Task 4: Full-card verification

**Files:**
- None (verification only, no code changes expected)

**Interfaces:**
- Consumes: the complete `scorecard-print.html` produced by Tasks 1-3.
- Produces: nothing further downstream — this is the final gate before the file is considered done.

- [ ] **Step 1: Screenshot both sheets**

Navigate to `file:///Users/conorhiggins/pubgolf/scorecard-print.html` with `mcp__Claude_Browser__navigate`, then use `mcp__Claude_Browser__computer` (`screenshot`) to capture the page. Since both sheets are stacked vertically with a visible gap (`margin-bottom:4mm` from Task 1, hidden in `@media print`), scroll if needed to see both the outside sheet (back cover + front cover) and the inside sheet (12-hole table).

- [ ] **Step 2: Confirm against the spec checklist**

Manually confirm from the screenshots:
- Front cover: "Roby Mill Pub Golf", "Saturday 15th August", "12 Holes · Par 35" all present and centered.
- Back cover: 5-row penalty table (Piss between pubs +1, Spilled drink +2, Broken glass +3, Dry hole +3, Throwing up +3, in that order), 3 numbered player-name lines, TOTAL P1/P2/P3 line.
- Inside spread: 12 holes in order (Ma Kellys → Cowboy & Co. → Walkabout → Nellie Deans → Bees Knees → Retro Lounge → Rose & Crown → Scruffy Murphys → Layton Rakes → Shenanigans → Coyote Ugly → Knobby's Karaoke), correct pars, empty score boxes for 3 players, TOTAL PAR row showing 35.
- No text is clipped or overflowing any panel.

- [ ] **Step 3: Report the print/export steps to Conor (no code change)**

Tell Conor: open `scorecard-print.html` in Chrome, `Cmd+P`, set Paper size to A5, Layout to Landscape, Margins to None, Scale to 100%, destination "Save as PDF" — this produces the 2-page PDF to upload to [Quadrant Print's A6 Folded Cards](https://www.quadrantprint.co.uk/a6-folded-cards.php) (10 for £31, next-day if ordered by 1pm).

No commit for this task — it's verification of work already committed in Tasks 1-3.
