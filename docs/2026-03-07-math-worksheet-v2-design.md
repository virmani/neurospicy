# Math Worksheet Skill v2 — Design

**Date:** 2026-03-07
**Status:** Approved
**Supersedes:** 2026-03-07-math-worksheet-design.md

## Problem Statement

v1 used absolute CSS positioning for a "scattered" layout. This caused overlapping boxes, looked forced, and didn't match the reference worksheets the child actually likes. v2 replaces the layout engine entirely and adds non-math content and deeper theme integration.

## Reference Worksheets

User provided two reference worksheets (MathWorksheets.com style) with these characteristics:
- Row-based mosaic layout: rows of boxes with varying widths, no overlap possible
- Large answer spaces in problem boxes
- Word root factoid banner at the bottom of each page
- Mixed problem sizes within rows (wide + narrow side by side)
- Structured but visually varied

## Changes from v1

| Area | v1 | v2 |
|------|----|----|
| Layout | Absolute positioning (overlaps) | CSS flex row-mosaic (no overlaps) |
| Non-math content | None | 2 items per page (word root, fun fact, ELA) |
| Theme | 4 corner emoji only | Header + row tints + borders + image cell |
| Images | None | WebSearch → base64 embed, SVG/emoji fallback |

---

## Layout System

Replace absolute positioning entirely with **CSS flex row-mosaic**.

Each page is a vertical stack of rows. Each row uses `display: flex` with problem boxes as flex children. Box widths are set as percentages of the row width.

### Row Patterns

Claude selects row patterns to create visual variety. Never use the same pattern twice in a row.

| Pattern | Flex children widths |
|---------|---------------------|
| Three equal | 33% / 33% / 33% |
| Wide + narrow | 66% / 33% |
| Narrow + wide | 33% / 66% |
| Two equal | 50% / 50% |
| Wide + two narrow | 50% / 25% / 25% |
| Narrow + center + narrow | 25% / 50% / 25% |
| Full width | 100% |
| Four equal | 25% / 25% / 25% / 25% |

### Box Heights

- Vertical addition/subtraction: min-height 120px (generous write space)
- Horizontal / mental math: min-height 70px
- Word problems: min-height 140px (large answer area)
- Image cell: min-height 130px
- ELA question: min-height 90px
- Factoid banner: min-height 50px, full-width

### Row Borders

Each box has a border on all sides. Adjacent boxes share borders (use negative margins or border-collapse equivalent). Row boxes have `border: 2px solid` using theme border color.

---

## Non-Math Content

Each page includes exactly **2 non-math items** interleaved with math problems.

### Placement Rules
- Word root factoid: always the **last row**, full-width banner
- Fun fact or ELA question: placed mid-page as a regular cell within a row
- Never two consecutive non-math rows

### Types

**A) Word Root Factoid** (bottom banner, every page)
Format: `word root [root] can mean [meaning] — [example1], [example2], [example3]`
Style: rounded pill/banner, light background, bold root word
Grade-appropriate roots for grades 2-5 (e.g., spect, act, bene, port, dict, aud)

**B) Theme Fun Fact** (mid-page cell)
Format: small emoji + bold "Did you know?" + 1-2 sentence fact related to the theme
Example (space theme): "🚀 Did you know? The International Space Station travels at 17,500 mph!"

**C) ELA Question** (mid-page cell)
Types (rotate, don't repeat same type twice):
- Spelling: "Circle the correct spelling: [wrong] / [correct] / [wrong]"
- Sentence: "Write a sentence using the word: **[grade-appropriate word]**" + write line
- Unscramble: "Unscramble: T-R-A-V-E-L = ___"
- Grammar: "Fix the sentence: [sentence with one error]" + write line

---

## Theme Integration

### 1. Header
- Background: primary theme color
- Title: themed (e.g., "🪀 Chinese Yo-Yo Math", "🚀 Space Math")
- Text: white or light theme-complementary color
- Name/Date fields: white underlines

### 2. Row Backgrounds
- Alternate between white and a very light tint (8% opacity) of the primary theme color
- Creates visual row separation without being distracting

### 3. Box Borders
- All box borders use theme color palette (3 colors cycling)
- No default rainbow cycle — consistent with theme

### 4. Image Cell
One dedicated image cell per page placed within a row alongside problems.

**Image sourcing (fallback chain):**
1. **WebSearch** — search for `[theme] clipart kid transparent site:openclipart.org` or `[theme] cartoon illustration site:pixabay.com`
2. **WebFetch** — download the image if a suitable URL is found
3. **Base64 embed** — encode as data URI, set as `<img src="data:...">` within the cell
4. **SVG fallback** — if fetch fails, render a simple inline SVG illustration (Claude draws it)
5. **Emoji fallback** — if SVG too complex, render a large centered emoji (5rem) in the cell

**Image cell styling:**
- `object-fit: contain`, centered
- Light theme-colored background in the cell
- No border on image itself

### Built-in Theme Palettes

| Theme | Primary | Secondary | Accent | Emoji |
|-------|---------|-----------|--------|-------|
| `chinese yo-yo` | #DC2626 | #D97706 | #059669 | 🪀🧧🌟🎪 |
| `space` | #1E3A8A | #22D3EE | #94A3B8 | 🚀🌟🪐⭐ |
| `unicorns` | #7C3AED | #EC4899 | #6EE7B7 | 🦄✨🌈⭐ |
| `dinosaurs` | #16A34A | #F97316 | #92400E | 🦕🦖🌿🥚 |
| `ocean` | #0EA5E9 | #14B8A6 | #FDE68A | 🐠🌊🐚🦀 |
| `superheroes` | #DC2626 | #2563EB | #D97706 | ⚡💥🦸🌟 |

For unknown themes: derive 3 colors from obvious associations, pick 3-4 relevant emoji.

---

## HTML Output

### CSS Structure

```css
body { font-family: Arial, sans-serif; margin: 0; padding: 20px; background: #f0f0f0; }

.page {
  width: 8.5in;
  min-height: 11in;
  margin: 0 auto 40px auto;
  padding: 0.75in;
  box-sizing: border-box;
  background: white;
  box-shadow: 0 2px 8px rgba(0,0,0,0.15);
}

.header { display: flex; justify-content: space-between; align-items: center;
  margin-bottom: 0; padding: 10px 18px; border-radius: 10px 10px 0 0;
  background: [primary]; color: white; }

.worksheet-body { border: 2px solid [primary]; border-top: none; border-radius: 0 0 8px 8px; }

.row { display: flex; border-bottom: 2px solid [primary]; }
.row:last-child { border-bottom: none; }
.row.tinted { background: rgba([primary-rgb], 0.05); }

.cell { padding: 14px 16px; border-right: 2px solid [primary]; box-sizing: border-box; }
.cell:last-child { border-right: none; }

.problem-num { font-size: 10px; color: #aaa; font-weight: bold; letter-spacing: 0.5px; margin-bottom: 6px; }
.math { font-family: 'Courier New', monospace; font-size: 22px; font-weight: bold; line-height: 1.5; }
.answer-box { border-top: 2px solid #333; min-height: 30px; margin-top: 6px; }
.answer-blank { display: inline-block; border-bottom: 2px solid #333; width: 60px; margin-left: 6px; vertical-align: bottom; }

.word-problem { font-size: 14px; line-height: 1.6; }
.write-line { border-bottom: 1.5px solid #555; min-height: 28px; margin-top: 10px; }

.factoid-banner { background: rgba([primary-rgb], 0.08); padding: 10px 18px;
  border-radius: 20px; margin: 8px; text-align: center; font-size: 13px; }

.image-cell { display: flex; align-items: center; justify-content: center;
  background: rgba([primary-rgb], 0.06); }
.image-cell img { max-width: 90%; max-height: 110px; object-fit: contain; }

@media print {
  @page { size: letter; margin: 0; }
  body { background: white; margin: 0; padding: 0; }
  .page { width: 8.5in; height: 11in; min-height: unset; margin: 0;
    padding: 0.75in; box-shadow: none; page-break-after: always; }
}
```

### Example Row HTML

```html
<!-- Row: 2/3 word problem + 1/3 vertical addition -->
<div class="row">
  <div class="cell" style="width:66%">
    <div class="problem-num">#3</div>
    <div class="word-problem">
      Mrs. Chen had 532 red ribbons and 247 gold ribbons for the festival.
      How many ribbons did she have in all?
      <div class="write-line"></div>
    </div>
  </div>
  <div class="cell" style="width:33%">
    <div class="problem-num">#4</div>
    <div class="math">
      <div style="text-align:right">  347</div>
      <div style="text-align:right">+ 285</div>
      <div class="answer-box"></div>
    </div>
  </div>
</div>

<!-- Row: full-width factoid banner -->
<div class="row">
  <div class="cell" style="width:100%; padding: 8px 14px;">
    <div class="factoid-banner">
      word root <strong>port</strong> can mean <strong>to carry</strong> — portable, transport, import
    </div>
  </div>
</div>
```

---

## Output Instructions

1. **Always** output the complete HTML as an artifact (rendered in-chat)
2. **If running in Claude Code** (Write tool available): also write to current directory as `worksheet-YYYY-MM-DD.html`
3. After output say: "Print tip: open the artifact, press Cmd+P / Ctrl+P, set paper size to US Letter, and turn off headers/footers in print settings."

---

## Unresolved Questions

None.
