---
name: summary
description: >
  Use when user wants to review their speech/typing patterns, get pronunciation
  coaching, or improve their English clarity. Analyzes refinement history from
  /refine:ask to identify recurring errors across pronunciation, grammar,
  collocations, and word boundaries. Default: quick text summary. Pass --html
  for a full interactive dashboard.
argument-hint: "[--html]"
---

# Refine Summary — English Language Coach

Analyze the user's refinement history and show a coaching summary.

**Mode:** Check `$ARGUMENTS` for `--html`.
- If `--html` is present → generate the full HTML dashboard (Step 4B)
- Otherwise → show a text summary in the terminal (Step 4A, default)

## Step 1: Load History

```bash
cat ~/.claude/refine-history.jsonl 2>/dev/null || echo "NO_HISTORY"
```

If NO_HISTORY or empty:

> No refinement history yet. Use `/refine:ask <your text>` a few times first — each use is logged automatically. Come back after 5-10 uses for meaningful patterns.

Then stop.

## Step 2: Parse and Normalize

Parse all JSONL entries. Handle both formats:

- **New format:** `corrections` is an array of `{from, to, category, subcategory}` objects
- **Legacy format:** `corrections` is a plain string like `"bery→very, balidation→validation"` — infer categories from patterns

For legacy entries, classify each correction:
- **pronunciation**: phoneme confusions (crowd→cloud, bonding→binding, Mac→Map, v/b, l/r, th/s swaps)
- **grammar**: articles, tenses, prepositions, agreement, word form, word order
- **collocation**: wrong word combinations (memory→history, do a mistake→make a mistake)
- **word_boundary**: compounds split/joined wrong (lockout→logout, POSTMAP→POST map)
- **asr**: severe garbles where ASR hallucinated entirely different words (barrack chicken→Bearer token, cryo petrophysics→Cloud API)

## Step 3: Analyze Patterns

Compute:

**A. Metrics:** total sessions, date range, total corrections, consecutive-day streak, avg corrections/session

**B. Category counts:** pronunciation, grammar, collocation, word_boundary, asr

**C. Top subcategories** by frequency (top 8)

**D. Trends:** Compare first half vs second half → classify each category as improving/persistent/worsening

**E. Top repeated corrections** (most frequent from→to pairs)

## Step 4A: Text Summary (default)

Output this directly in the terminal — no files, no HTML:

```
══════════════════════════════════════
  English Coaching Summary
  {first_date} → {last_date}
══════════════════════════════════════

Sessions: {N}  |  Corrections: {N}  |  Streak: {N} days  |  Avg: {N.N}/session

── Category Breakdown ──────────────────

  {icon} Pronunciation   {count}  {bar}  {pct}%  {trend_arrow}
  {icon} Grammar         {count}  {bar}  {pct}%  {trend_arrow}
  {icon} Word Boundary   {count}  {bar}  {pct}%  {trend_arrow}
  {icon} Collocations    {count}  {bar}  {pct}%  {trend_arrow}
  {icon} ASR Artifacts   {count}  {bar}  {pct}%  (tool noise)

── Top Error Patterns ──────────────────

  1. {from}→{to}  ×{count}  [{category}]
  2. {from}→{to}  ×{count}  [{category}]
  3. {from}→{to}  ×{count}  [{category}]
  4. {from}→{to}  ×{count}  [{category}]
  5. {from}→{to}  ×{count}  [{category}]

── Coaching Tips ───────────────────────

  Based on your #1 pattern ({top_pattern}):

  {2-3 lines of specific, actionable coaching advice}
  {minimal pair examples if pronunciation}
  {grammar rule if grammar}

── Quick Wins ──────────────────────────

  • {one specific micro-drill for their top issue}
  • {one resource link relevant to their top issue}
  • Keep using /refine:ask — tracking is the first step

══════════════════════════════════════
  Run /refine:summary --html for the full interactive dashboard
══════════════════════════════════════
```

**Formatting rules:**
- Use Unicode box-drawing characters for structure
- `{bar}` = inline bar using `█` and `░` characters, 20 chars wide, proportional to category count
- `{trend_arrow}` = `↑ improving` (green), `→ persistent` (yellow), `↓ worsening` (red)
- `{icon}` = category icon: pronunciation=🗣, grammar=📝, word_boundary=🔗, collocation=🧩, asr=🤖
- Keep total output under 40 lines
- ASR artifacts: note they reflect tool noise, not the user's actual errors — exclude from coaching

**Tone:** Warm, encouraging. Frame errors as growth opportunities. Celebrate improvements.

Then stop. Do NOT generate HTML in this mode.

## Step 4B: HTML Dashboard (--html flag)

Generate a complete, self-contained HTML file at `/tmp/refine-report.html` and open it.

The HTML must include Chart.js via CDN, embedded CSS, embedded data as JavaScript, responsive design.

### HTML Sections

**1. Header** — Title "English Coaching Dashboard", date range subtitle, gradient background (blue→indigo)

**2. Metric Cards** (4-card grid) — Sessions, Corrections, Streak, Most improved category

**3. Charts** (2×2 grid):

| Chart | Type | Details |
|-------|------|---------|
| Corrections Over Time | Line | X=dates, Y=count per session. Lines per category + total. Colors: pronunciation=#ef4444, grammar=#f59e0b, collocation=#8b5cf6, word_boundary=#3b82f6, asr=#6b7280, total=#475569. tension: 0.3 |
| Error Profile | Radar | 5 axes for categories, semi-transparent blue fill |
| Error Distribution | Doughnut | Segments per category with %, legend on right |
| Top Error Patterns | Horizontal bar | Top 8 subcategories, colored by parent category |

Chart.js settings: `responsive: true`, `maintainAspectRatio: false`, containers `height: 300px`, legend at bottom for line, right for doughnut.

**4. Pronunciation Coaching** (if pronunciation errors exist) — For each phoneme confusion:
- Phoneme pair heading with IPA (e.g., "/aʊ/ vs /aː/" for crowd/cloud)
- Mouth position guide (1-2 sentences each sound)
- Minimal pairs table (6-8 pairs, include tech terms)
- Practice tip + reference links (Rachel's English, Forvo, YouGlish, Sounds of Speech Iowa)

**5. Grammar Coaching** (if grammar errors exist) — For each pattern:
- Rule heading + explanation
- User's actual before/after examples (2-3)
- Mnemonic + 3-4 fill-in-the-blank exercises
- Reference links (Grammarly, Cambridge Grammar, BBC Learning English)

**6. Word Boundary Coaching** (if word_boundary errors exist) — Homophones table, compound word rules, user's examples, quick tests

**7. ASR Note** — Brief note that ASR errors reflect tool limitations, not speech. Exclude from coaching.

**8. Practice Resources** — Curated list: pronunciation drills, grammar checklist, shadowing exercises, general tips

**9. Footer** — Generation date, encouragement, "keep using /refine:ask"

### CSS

```
pronunciation: #ef4444    grammar: #f59e0b    collocation: #8b5cf6
word_boundary: #3b82f6    asr: #6b7280       improving: #10b981
```
Cards: shadow + 12px radius. Font: `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif`. IPA: `'Lucida Sans Unicode', 'DejaVu Sans', Arial`. Responsive grid, print-friendly.

### Write and Open

```bash
cat > /tmp/refine-report.html << 'HTMLEOF'
<!-- full HTML here -->
HTMLEOF
open /tmp/refine-report.html
```

Then show the same text summary from Step 4A as well (so the terminal isn't empty).
