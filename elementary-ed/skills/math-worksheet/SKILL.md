---
name: math-worksheet
description: Use when generating math worksheets for kids. Supports multiple problem types, grade levels, optional themes, and outputs print-ready HTML for US letter paper.
---

# Math Worksheet Generator

Generate an engaging, print-ready math worksheet for kids. Problems are arranged in a row-mosaic layout — rows of boxes with varying widths, no overlapping, with non-math content and deep theme integration.

## Parse the Request

Extract from the user's invocation message:
- **Problem types** — comma-separated list (required)
- `grade: N` — grade level (default: 3)
- `theme: X` — visual theme like "unicorns", "space", "dinosaurs" (optional)
- `problems: N` — total problem count (optional; Claude decides if omitted)
- `pages: N` — page count (optional; Claude decides based on problem count if omitted)
- **Per-type difficulty overrides** — natural language within type descriptions (e.g., "multiplication facts for 2-5", "addition 100-500")

## Generate Problems

### Grade-Level Defaults (US Common Core)

**Grade 2:**
- Addition: 2-digit + 2-digit (10-99), some carrying
- Subtraction: 2-digit − 2-digit, positive difference
- Mental math: multiples of 10, doubles

**Grade 3 (default):**
- 3-digit addition: addends 100-999
- 3-digit subtraction: minuend 200-999, positive difference
- Multiplication facts: single-digit × single-digit (1-10)
- Division facts: dividends up to 100, divisors 1-10
- Mental math: multiples of 10 and 100, round numbers

**Grade 4:**
- Multi-digit addition/subtraction: up to 4 digits
- Multiplication: 2-digit × 1-digit, 2-digit × 2-digit
- Division: 3-digit ÷ 1-digit
- Fractions: simple fractions, equivalent fractions

**Grade 5:**
- Multi-digit multiplication: up to 3-digit × 2-digit
- Long division: 4-digit ÷ 2-digit
- Fractions: add/subtract unlike fractions, multiply fractions
- Decimals: add/subtract/multiply decimals

### Applying Overrides

Natural language overrides in the type description take priority over grade defaults:
- "multiplication facts for 2-5" → factors limited to 2, 3, 4, 5
- "addition 100-500" → addends in range 100-500
- "subtraction no regrouping" → ensure no borrowing required
- "easy multiplication" → single-digit × single-digit regardless of grade

### Problem Distribution

- Distribute problems roughly equally across types (round naturally)
- If `problems: N` not specified: target 10-14 problems for 1 page
- If `pages: N` not specified: compute required pages as `ceil(problems_count / 14)` — use 1 page for up to ~14 problems, 2 pages for 15-28, etc. Distribute evenly across pages; exact per-page count doesn't need to be equal.
- Shuffle all problems so no two consecutive problems are the same type
- If only one problem type is given, no shuffle constraint applies.

### Problem Formats

**Addition/Subtraction** — vertical column format:
```
  247
+ 138
-----
     (blank answer space)
```

**Multiplication/Division facts** — horizontal:
```
6 × 7 = ___
42 ÷ 6 = ___
```

**Mental math** — horizontal:
```
300 + 400 = ___
```

**Fractions** — horizontal or stacked as appropriate.

**Word problems** — 2-3 sentences with a blank write line for the answer.

## Non-Math Content

Each page includes exactly **2 non-math items** interleaved with math problems.

### Placement Rules
- **Word root factoid**: always the last row of every page — full-width banner
- **Fun fact or ELA question**: placed mid-page — either as a full-width row OR as a partial-width cell (50-66%) sharing a row with a math problem. Choose whichever fits the surrounding layout better.
- Try to space out non-math items (don't place them in back-to-back rows if avoidable)

### Types

**A) Word Root Factoid** (bottom banner, every page)

Format: `word root [root] can mean [meaning] — [example1], [example2], [example3]`

Style: rounded pill/banner with light theme-colored background, bold root word. Use grade-appropriate roots:
- Grade 2-3: act (do/drive), port (carry), dict (say), aud (hear), vis (see)
- Grade 4-5: spect (look), bene (good), mal (bad), tract (pull), struct (build)

Example: `word root port can mean to carry — portable, transport, import`

**B) Theme Fun Fact** (mid-page cell)

Format: `[emoji] Did you know? [1-2 sentence fact related to the theme]`

Example (space): `🚀 Did you know? The International Space Station travels at 17,500 mph — that's 5 miles every second!`

**C) ELA Question** (mid-page cell)

Rotate through these types (don't use same type twice per worksheet):
- Spelling: `Circle the correct spelling: [wrong] / [correct] / [wrong]` — all three options must be rendered with identical styling; never underline, bold, or visually distinguish the correct answer
- Sentence writing: `Write a sentence using the word: [grade-appropriate word]` + write line
- Unscramble: `Unscramble this word: [scrambled letters] = ___`
- Grammar fix: `Fix the sentence: [sentence with one error]` + write line

## Image Sourcing

Each page has one dedicated image cell. **Always use an inline SVG illustration** — draw simple theme-relevant shapes (2-4 colors, basic geometry, kid-friendly). If the theme is genuinely too abstract to illustrate simply, fall back to a large centered emoji (`<div class="emoji-fallback">[emoji]</div>`, `font-size: 5rem`). Do not search the web for images.

Image cell: `display:flex`, centered, `min-height:130px`, background `rgba(var(--primary-rgb), 0.06)`.

## Layout System

Use **CSS flex row-mosaic** layout. Each page is a vertical stack of rows. Each row uses `display: flex`. Box widths are percentages of row width. No absolute positioning — overlaps are impossible.

### Row Patterns

Vary row patterns across the page — prefer different patterns in adjacent rows, but this is a soft preference:

| Pattern name | Cell widths |
|---|---|
| Three equal | 33% / 33% / 33% |
| Wide + narrow | 66% / 33% |
| Narrow + wide | 33% / 66% |
| Two equal | 50% / 50% |
| Wide + two narrow | 50% / 25% / 25% |
| Center feature | 25% / 50% / 25% |
| Full width | 100% |
| Four equal | 25% / 25% / 25% / 25% |

### Minimum Cell Heights by Content Type

- Vertical addition/subtraction: `min-height: 110px`
- Horizontal / mental math: `min-height: 60px`
- Word problem: `min-height: 95px`
- Image cell: `min-height: 115px`
- ELA question: `min-height: 85px`
- Area diagram: `min-height: 110px`
- Word root factoid banner: `min-height: 44px`


### Word Problem Length

Keep word problems to **≤ 65 characters** so they wrap to at most 2 lines in a 33%-wide cell. Longer problems in narrow cells produce 4+ lines and silently blow past the row's min-height, causing page overflow.

✅ "5 rows of 6 seats each. 24 are occupied. How many empty?" (57 chars)
❌ "There are 5 rows of seats with 6 seats in each row. If 24 seats are occupied, how many seats are empty?" (103 chars)

### Row Visual Rhythm

Alternate row background between white and a very light primary color tint (5% opacity). Add class `tinted` to every other row to achieve this.

**Vary row patterns across the page** — avoid using the same pattern in back-to-back rows when easy to do so, but don't replan the layout to fix this.

## Theme Integration

### Theme Palettes

| Theme | Primary | Primary RGB | Emoji |
|---|---|---|---|
| `chinese yo-yo` | #DC2626 | 220,38,38 | 🪀🧧🌟🎪 |
| `space` | #1E3A8A | 30,58,138 | 🚀🌟🪐⭐ |
| `unicorns` | #7C3AED | 124,58,237 | 🦄✨🌈⭐ |
| `dinosaurs` | #16A34A | 22,163,74 | 🦕🦖🌿🥚 |
| `ocean` | #0EA5E9 | 14,165,233 | 🐠🌊🐚🦀 |
| `superheroes` | #DC2626 | 220,38,38 | ⚡💥🦸🌟 |

For unknown themes: pick a color based on obvious associations, derive its RGB. **No theme specified:** primary `#0D9488`, RGB `13,148,136`.

### Applying Theme

Set `--primary` and `--primary-rgb` as inline CSS custom properties on each `.page` div:

```html
<div class="page" style="--primary:#9C27B0;--primary-rgb:156,39,176;">
```

The shared stylesheet references `var(--primary)` and `rgba(var(--primary-rgb), N)` — no per-page CSS classes needed.

## Layout Planning Step

**Before writing any HTML**, output a compact layout plan for each page. One line per row:

```
Row N [tinted]: [pattern] | [cell: width% content-type #problem-num] ...
```

Example plan:
```
Page 1 — Sewing (#9C27B0)
Row 1: three-equal | 33% mult #1 · 33% seq #2 · 33% mult #3
Row 2 tinted: wide+narrow | 66% word #4 · 33% add #5
Row 3: three-equal | 33% image · 33% seq #6 · 33% sub #7
Row 4 tinted: two-equal | 50% ELA · 50% mental #8
Row 5: three-equal | 33% seq #9 · 33% mult #10 · 33% seq #11
Row 6 tinted: two-equal | 50% add #12 · 50% mental #13
Row 7: full-width | 100% word #14
Row 8 tinted: full-width | 100% word-root-banner
```

After drafting the plan, **verify the page budget before writing any HTML**:

- Available height per page: `11in − 2×0.6in padding − ~46px header ≈ 894px`
- Sum all row min-heights + border overhead (`N rows × 2px + 2px`)
- Safe target: **≤ 750px** (leaves headroom for text wrap in narrow cells)

If the sum exceeds 750px: **replan** — reduce rows per page or add a page. Do not proceed to HTML with an over-budget layout. Once the budget passes, **do not replan again**.

**Redistribute remaining space.** After the budget check passes, calculate headroom:

```
headroom = 894px − (total_min_height + N_rows × 2px border)
```

If headroom > 60px, add `floor(headroom / n_problem_rows)` to the `min-height` of each row that contains at least one math problem (vertical add/sub, word problem, horizontal fact). This fills the page more fully and gives students room to show their work. Non-math rows (image, ELA, fun-fact, banner) keep their original min-heights.

**Save the layout plan.** After finalizing (post-redistribution), write the complete plan text to `layout_plan.txt` in the same directory as the HTML file. This lets evals verify replanning behavior. Format: one line per row, same compact format as the plan above.

## HTML Output

Generate a single, fully self-contained HTML file.

### CSS

Use CSS custom properties for theming — the stylesheet is fully static. Emit exactly this `<style>` block (no substitutions needed; theme values go on each `.page` div instead):

```css
body{font-family:Arial,sans-serif;margin:0;padding:20px;background:#f0f0f0}
.page{width:8.5in;min-height:11in;margin:0 auto 40px;padding:.6in;box-sizing:border-box;background:white;box-shadow:0 2px 8px rgba(0,0,0,.15)}
.header{display:flex;justify-content:space-between;align-items:center;padding:8px 16px;border-radius:10px 10px 0 0;background:var(--primary);color:white;-webkit-print-color-adjust:exact;print-color-adjust:exact}
.header h1{font-size:18px;margin:0;font-weight:bold}.header-fields{display:flex;gap:20px;font-size:13px;align-items:center}
.field-line{border-bottom:2px solid white;display:inline-block;width:100px;margin-left:6px}
.worksheet-body{border:2px solid var(--primary);border-top:none;border-radius:0 0 8px 8px;overflow:hidden}
.row{display:flex;border-bottom:2px solid var(--primary)}.row:last-child{border-bottom:none}
.row.tinted{background:rgba(var(--primary-rgb),.05);-webkit-print-color-adjust:exact;print-color-adjust:exact}
.cell{padding:9px 11px;border-right:2px solid var(--primary);box-sizing:border-box;overflow:hidden}.cell:last-child{border-right:none}
.problem-num{font-size:9px;color:#aaa;font-weight:bold;letter-spacing:.5px;margin-bottom:3px}
.math{font-family:'Courier New',monospace;font-size:20px;font-weight:bold;line-height:1.4}
.answer-box{border-top:2px solid #333;min-height:26px;margin-top:5px}
.answer-blank{display:inline-block;border-bottom:2px solid #333;width:55px;margin-left:5px;vertical-align:bottom}
.word-problem{font-size:12.5px;line-height:1.5}.write-line{border-bottom:1.5px solid #555;min-height:20px;margin-top:7px}
.ela,.fun-fact{font-size:12px;line-height:1.5}
.factoid-banner{background:rgba(var(--primary-rgb),.08);padding:8px 16px;border-radius:20px;margin:4px;text-align:center;font-size:12px}
.image-cell{display:flex;align-items:center;justify-content:center;background:rgba(var(--primary-rgb),.06)}
.image-cell .emoji-fallback{font-size:5rem;line-height:1}
/* CRITICAL: area SVGs scale to fill cell width — without max-height a 66%-wide cell
   inflates a viewBox="0 0 230 105" SVG to ~200px tall, blowing up the row and causing
   page overflow. Always cap area diagram SVGs. */
.area-diagram svg{max-width:100%;max-height:82px;width:auto;height:auto}
.area-label{font-size:11px;color:#444;font-weight:bold;text-align:center;margin-top:2px}
@media print{@page{size:letter;margin:0}body{margin:0;padding:0;background:white}.page{height:11in;min-height:unset;margin:0;box-shadow:none;padding:.6in;page-break-after:always;overflow:hidden}}
```

### Page Structure

```html
<div class="page" style="--primary:#HEX;--primary-rgb:R,G,B;">
  <div class="header">
    <h1>[EMOJI] [Title]</h1>
    <div class="header-fields">
      <span>Name:<span class="field-line"></span></span>
      <span>Date:<span class="field-line"></span></span>
    </div>
  </div>
  <div class="worksheet-body"><!-- rows --></div>
</div>
```

For multi-page worksheets: repeat the full `.page` div with its own `style="--primary:..."`. Each page has its own header and worksheet-body.

### Row HTML Reference

Each row: `<div class="row [tinted]">` → cells: `<div class="cell" style="width:N%;min-height:Npx;">`.

Content patterns:
- **Vertical add/sub**: `<div class="math"><div style="text-align:right">&nbsp;NNN</div><div style="text-align:right">±NNN</div><div class="answer-box"></div></div>`
- **Horizontal fact**: `<div class="math">A × B = <span class="answer-blank"></span></div>`
- **Word problem**: `<div class="word-problem">[text]<div class="write-line"></div></div>`
- **Mental math**: multi-line steps, end with `Answer: <span class="answer-blank"></span>`
- **Image cell**: add class `image-cell` to cell; put inline `<svg>` or `<div class="emoji-fallback">[emoji]</div>` inside
- **Area diagram**: wrap SVG + label in `<div class="area-diagram">`. The `.area-diagram svg` CSS rule caps height at 82px — do NOT add inline `width`/`height` attributes or `style` overrides that remove this cap.
- **ELA**: `<div class="ela">[content]</div>`
- **Fun fact**: `<div class="fun-fact">[emoji] <strong>Did you know?</strong> [text]</div>`
- **Word root banner**: `<div class="factoid-banner">word root <strong>X</strong> can mean <strong>Y</strong> — ex1, ex2, ex3</div>`

Each cell starts with `<div class="problem-num">#N</div>` except image, ELA, fun-fact, and banner cells.

### Example Row

```html
<div class="row tinted">
  <div class="cell" style="width:66%;min-height:140px;">
    <div class="problem-num">#3</div>
    <div class="word-problem">Mrs. Chen had 532 red ribbons and 247 gold ribbons. How many in all?<div class="write-line"></div></div>
  </div>
  <div class="cell" style="width:33%;min-height:140px;">
    <div class="problem-num">#4</div>
    <div class="math"><div style="text-align:right">&nbsp;347</div><div style="text-align:right">+285</div><div class="answer-box"></div></div>
  </div>
</div>
```

## Output Instructions

1. **Always** output the complete HTML as an artifact (rendered in-chat)
2. **If running in Claude Code** (the `Write` tool is available): write the file using the `Write` tool to the current working directory as `worksheet-YYYY-MM-DD.html`. Do NOT use bash/cat commands to write the file.
3. **If running in Claude Desktop or Claude.ai**: output only the artifact — no file writing tools are available.
4. After output say: "Print tip: open the artifact, press Cmd+P / Ctrl+P, set paper size to US Letter, and turn off headers/footers in print settings."
