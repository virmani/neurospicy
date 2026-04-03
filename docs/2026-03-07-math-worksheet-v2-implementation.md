# Math Worksheet Skill v2 Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Rewrite `skills/math-worksheet/SKILL.md` to replace the broken absolute-positioning layout with a CSS flex row-mosaic, add non-math content (word root factoids, fun facts, ELA questions), and integrate theme deeply into header, row backgrounds, borders, and a dedicated image cell.

**Architecture:** Single file rewrite — `skills/math-worksheet/SKILL.md`. No new files, no scripts. The skill prompt is the implementation: it instructs Claude how to generate the HTML. All changes are to the skill's instructions, CSS templates, and HTML examples within that one file.

**Tech Stack:** Markdown skill file, HTML, CSS flex layout, WebSearch/WebFetch for image sourcing (with emoji/SVG fallback).

---

### Task 1: Replace SKILL.md with complete v2 rewrite

**Files:**
- Modify (full rewrite): `skills/math-worksheet/SKILL.md`

This is a complete replacement. Read the file first, then overwrite with the full v2 content below.

**Step 1: Read current file**

```bash
cat skills/math-worksheet/SKILL.md | head -5
```
Expected: sees the v1 frontmatter

**Step 2: Write the new SKILL.md**

Replace the entire file with this exact content:

````markdown
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
- If `problems: N` not specified: target 12-14 problems for 1 page
- If `pages: N` not specified: compute required pages as `ceil(problems_count / 14)` — use 1 page for 14 or fewer problems, 2 pages for 15-28, etc.
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
- **Fun fact or ELA question**: placed mid-page as a regular cell within a row
- Never place two non-math items in adjacent rows

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
- Spelling: `Circle the correct spelling: [wrong] / [correct] / [wrong]`
- Sentence writing: `Write a sentence using the word: [grade-appropriate word]` + write line
- Unscramble: `Unscramble this word: [scrambled letters] = ___`
- Grammar fix: `Fix the sentence: [sentence with one error]` + write line

## Image Sourcing

Each page has one dedicated image cell showing a theme-related illustration.

**Fallback chain — try each step in order, stop when one succeeds:**

1. **WebSearch** — search for: `[theme] clipart kids illustration transparent`
   - Look for results from pixabay.com, openclipart.org, or wikimedia.org
   - Pick a result that looks like a simple, colorful, kid-friendly illustration

2. **WebFetch** — fetch the direct image URL found in step 1
   - If successful, use the URL directly as `<img src="[url]">` in the image cell
   - Note in an HTML comment: `<!-- image: [url] -->`

3. **SVG fallback** — if WebSearch/WebFetch fails or returns nothing suitable, draw a simple inline SVG illustration relevant to the theme (e.g., a rocket for space, a dinosaur outline for dinosaurs, a yo-yo for chinese yo-yo)

4. **Emoji fallback** — if SVG is too complex, place a large centered emoji (font-size: 5rem) in the image cell

**Image cell requirements:**
- `object-fit: contain`, max-height: 110px, centered horizontally and vertically
- Light theme-colored background (5% opacity primary color)
- No additional border on the image itself

## Layout System

Use **CSS flex row-mosaic** layout. Each page is a vertical stack of rows. Each row uses `display: flex`. Box widths are percentages of row width. No absolute positioning — overlaps are impossible.

### Row Patterns

Vary row patterns across the page — never use the same pattern twice consecutively:

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

- Vertical addition/subtraction: `min-height: 120px`
- Horizontal / mental math: `min-height: 70px`
- Word problem: `min-height: 140px`
- Image cell: `min-height: 130px`
- ELA question: `min-height: 90px`
- Word root factoid banner: `min-height: 50px`

### Row Visual Rhythm

Alternate row background between white and a very light primary color tint (5% opacity). Add class `tinted` to every other row to achieve this.

## Theme Integration

### Theme Palettes

| Theme | Primary | Secondary | Accent | Emoji |
|---|---|---|---|---|
| `chinese yo-yo` | #DC2626 | #D97706 | #059669 | 🪀🧧🌟🎪 |
| `space` | #1E3A8A | #22D3EE | #94A3B8 | 🚀🌟🪐⭐ |
| `unicorns` | #7C3AED | #EC4899 | #6EE7B7 | 🦄✨🌈⭐ |
| `dinosaurs` | #16A34A | #F97316 | #92400E | 🦕🦖🌿🥚 |
| `ocean` | #0EA5E9 | #14B8A6 | #FDE68A | 🐠🌊🐚🦀 |
| `superheroes` | #DC2626 | #2563EB | #D97706 | ⚡💥🦸🌟 |

For unknown themes: pick 3 colors based on obvious associations, choose 3-4 relevant emoji.

**No theme specified:** use a default palette of teal (#0D9488), purple (#7C3AED), orange (#EA580C) with a light gray header.

### Applying Theme

1. **Header** — background: primary color, text: white, Name/Date underlines: white
2. **Worksheet body border** — primary color, 2px solid, wraps all rows
3. **Row borders** — primary color, 2px solid between rows and between cells
4. **Tinted rows** — every other row gets `background: rgba([primary as r,g,b], 0.05)`
5. **Image cell background** — `rgba([primary as r,g,b], 0.06)`
6. **Word root factoid banner** — `rgba([primary as r,g,b], 0.08)` background

## HTML Output

Generate a single, fully self-contained HTML file (exception: externally-fetched image URLs if image search succeeds).

### Required CSS

```css
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 20px;
  background: #f0f0f0;
}
.page {
  width: 8.5in;
  min-height: 11in;
  margin: 0 auto 40px auto;
  padding: 0.75in;
  box-sizing: border-box;
  background: white;
  box-shadow: 0 2px 8px rgba(0,0,0,0.15);
}
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0;
  padding: 10px 18px;
  border-radius: 10px 10px 0 0;
  background: [PRIMARY];
  color: white;
}
.header h1 { font-size: 20px; margin: 0; font-weight: bold; }
.header-fields { display: flex; gap: 24px; font-size: 14px; align-items: center; }
.field-line { border-bottom: 2px solid white; display: inline-block; width: 110px; margin-left: 6px; }
.worksheet-body {
  border: 2px solid [PRIMARY];
  border-top: none;
  border-radius: 0 0 8px 8px;
  overflow: hidden;
}
.row {
  display: flex;
  border-bottom: 2px solid [PRIMARY];
}
.row:last-child { border-bottom: none; }
.row.tinted { background: rgba([PRIMARY_RGB], 0.05); }
.cell {
  padding: 14px 16px;
  border-right: 2px solid [PRIMARY];
  box-sizing: border-box;
  overflow: hidden;
}
.cell:last-child { border-right: none; }
.problem-num {
  font-size: 10px;
  color: #aaa;
  font-weight: bold;
  letter-spacing: 0.5px;
  margin-bottom: 6px;
}
.math {
  font-family: 'Courier New', Courier, monospace;
  font-size: 22px;
  font-weight: bold;
  line-height: 1.5;
}
.answer-box {
  border-top: 2px solid #333;
  min-height: 30px;
  margin-top: 6px;
}
.answer-blank {
  display: inline-block;
  border-bottom: 2px solid #333;
  width: 60px;
  margin-left: 6px;
  vertical-align: bottom;
}
.word-problem { font-size: 14px; line-height: 1.7; }
.write-line { border-bottom: 1.5px solid #555; min-height: 28px; margin-top: 10px; }
.ela { font-size: 14px; line-height: 1.7; }
.fun-fact { font-size: 13px; line-height: 1.6; color: #222; }
.factoid-banner {
  background: rgba([PRIMARY_RGB], 0.08);
  padding: 10px 18px;
  border-radius: 20px;
  margin: 6px;
  text-align: center;
  font-size: 13px;
}
.image-cell {
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba([PRIMARY_RGB], 0.06);
}
.image-cell img {
  max-width: 90%;
  max-height: 110px;
  object-fit: contain;
}
.image-cell .emoji-fallback {
  font-size: 5rem;
  line-height: 1;
}

@media print {
  @page { size: letter; margin: 0; }
  body { background: white; margin: 0; padding: 0; }
  .page {
    width: 8.5in;
    height: 11in;
    min-height: unset;
    margin: 0;
    padding: 0.75in;
    box-shadow: none;
    page-break-after: always;
  }
}
```

Replace `[PRIMARY]` with the hex color, and `[PRIMARY_RGB]` with comma-separated r,g,b values (e.g., `220,38,38` for #DC2626).

### Page Structure

```html
<div class="page">
  <div class="header">
    <h1>[EMOJI] [Themed Title]</h1>
    <div class="header-fields">
      <span>Name:<span class="field-line"></span></span>
      <span>Date:<span class="field-line"></span></span>
    </div>
  </div>
  <div class="worksheet-body">
    [ROWS]
  </div>
</div>
```

For multi-page worksheets: repeat the full `<div class="page">` structure. Each page has its own header and worksheet-body. Distribute problems evenly across pages.

### Row HTML Examples

**Row: 2/3 word problem + 1/3 vertical addition (tinted)**
```html
<div class="row tinted">
  <div class="cell" style="width:66%;min-height:140px;">
    <div class="problem-num">#3</div>
    <div class="word-problem">
      Mrs. Chen had 532 red ribbons and 247 gold ribbons for the festival.
      How many ribbons did she have in all?
      <div class="write-line"></div>
    </div>
  </div>
  <div class="cell" style="width:33%;min-height:140px;">
    <div class="problem-num">#4</div>
    <div class="math">
      <div style="text-align:right">  347</div>
      <div style="text-align:right">+ 285</div>
      <div class="answer-box"></div>
    </div>
  </div>
</div>
```

**Row: 3 equal cells — horizontal problems**
```html
<div class="row">
  <div class="cell" style="width:33%;min-height:70px;">
    <div class="problem-num">#5</div>
    <div class="math">4 × 6 = <span class="answer-blank"></span></div>
  </div>
  <div class="cell" style="width:33%;min-height:70px;">
    <div class="problem-num">#6</div>
    <div class="math">700 − 300 = <span class="answer-blank"></span></div>
  </div>
  <div class="cell" style="width:33%;min-height:70px;">
    <div class="problem-num">#7</div>
    <div class="math">3 × 5 = <span class="answer-blank"></span></div>
  </div>
</div>
```

**Row: image cell + two vertical problems**
```html
<div class="row tinted">
  <div class="cell image-cell" style="width:33%;min-height:130px;">
    <!-- image: https://... -->
    <img src="https://..." alt="theme illustration">
    <!-- fallback if no image: -->
    <!-- <div class="emoji-fallback">🪀</div> -->
  </div>
  <div class="cell" style="width:33%;min-height:130px;">
    <div class="problem-num">#8</div>
    <div class="math">
      <div style="text-align:right">  631</div>
      <div style="text-align:right">+ 248</div>
      <div class="answer-box"></div>
    </div>
  </div>
  <div class="cell" style="width:33%;min-height:130px;">
    <div class="problem-num">#9</div>
    <div class="math">
      <div style="text-align:right">  874</div>
      <div style="text-align:right">− 356</div>
      <div class="answer-box"></div>
    </div>
  </div>
</div>
```

**Row: ELA question (50%) + horizontal problem (50%)**
```html
<div class="row">
  <div class="cell" style="width:50%;min-height:90px;">
    <div class="ela">
      <strong>Circle the correct spelling:</strong><br>
      recieve &nbsp;&nbsp; <u>receive</u> &nbsp;&nbsp; receve
    </div>
  </div>
  <div class="cell" style="width:50%;min-height:90px;">
    <div class="problem-num">#10</div>
    <div class="math">5 × 8 = <span class="answer-blank"></span></div>
  </div>
</div>
```

**Row: Fun fact (full width)**
```html
<div class="row tinted">
  <div class="cell" style="width:100%;min-height:70px;">
    <div class="fun-fact">
      🪀 <strong>Did you know?</strong> The diabolo (Chinese yo-yo) was invented over 4,000 years ago in China and was used in circus performances!
    </div>
  </div>
</div>
```

**Row: Word root factoid banner (always last row)**
```html
<div class="row">
  <div class="cell" style="width:100%;min-height:50px;">
    <div class="factoid-banner">
      word root <strong>port</strong> can mean <strong>to carry</strong> — portable, transport, import
    </div>
  </div>
</div>
```

## Output Instructions

1. **Always** output the complete HTML as an artifact (rendered in-chat)
2. **If running in Claude Code** (Write tool available): also write the file to the current working directory as `worksheet-YYYY-MM-DD.html` using today's date
3. After output say: "Print tip: open the artifact, press Cmd+P / Ctrl+P, set paper size to US Letter, and turn off headers/footers in print settings."
````

**Step 3: Verify the file was written**

```bash
wc -l skills/math-worksheet/SKILL.md
```
Expected: ~250+ lines

**Step 4: Commit**

```bash
git add skills/math-worksheet/SKILL.md
git commit -m "feat: rewrite math-worksheet skill v2 — row-mosaic layout, non-math content, deep theme"
```

---

### Task 2: Smoke test — generate a sample worksheet

**Step 1: Manually verify the skill file reads cleanly**

Read `skills/math-worksheet/SKILL.md` and confirm:
- [ ] Frontmatter present (`name:`, `description:`)
- [ ] `## Parse the Request` section with all 6 parameters
- [ ] `## Generate Problems` with grade 2-5 defaults
- [ ] `## Non-Math Content` with all 3 types (word root, fun fact, ELA)
- [ ] `## Image Sourcing` with 4-step fallback chain
- [ ] `## Layout System` with row patterns table and min-heights
- [ ] `## Theme Integration` with palette table and 6 application rules
- [ ] `## HTML Output` with Required CSS, Page Structure, and 6 row HTML examples
- [ ] `## Output Instructions` with 3 numbered rules

**Step 2: Generate a test worksheet HTML**

Follow the skill instructions to generate a worksheet for:
```
3-digit addition, multiplication facts for 3-6, mental math subtraction, theme: space
```

Write the output to `/Users/virmani/workspace/education/math/worksheet-v2-test.html`

**Step 3: Verify the generated HTML**

Check the generated file for:
- [ ] CSS uses `.row` + `.cell` with `display:flex` (NO `position:absolute`)
- [ ] Header has space-themed background color (#1E3A8A or similar)
- [ ] At least 4 different row patterns used (verify by inspecting row widths)
- [ ] Word root factoid is the last row, full-width
- [ ] At least one fun fact or ELA question mid-page
- [ ] Image cell present (with image URL, SVG, or emoji fallback)
- [ ] `@media print` block with `@page { size: letter; margin: 0; }`
- [ ] No `position: absolute` anywhere in the file (grep for it)

```bash
grep -c "position: absolute" /Users/virmani/workspace/education/math/worksheet-v2-test.html
```
Expected: `0`

```bash
grep -c "display: flex" /Users/virmani/workspace/education/math/worksheet-v2-test.html
```
Expected: `1` or more (in the CSS)

**Step 4: Fix any issues found, commit if changes made**

```bash
git add skills/math-worksheet/SKILL.md
git commit -m "fix: refine math-worksheet v2 skill based on smoke test"
```

---

### Task 3: Push branch

**Step 1: Verify clean state**

```bash
cd /Users/virmani/workspace/adulting && git status
```
Expected: nothing to commit (or only unrelated files)

**Step 2: Push**

```bash
git push origin ashish/math-worksheet-skill
```
