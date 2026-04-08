---
name: only
description: >
  Use when user provides rough, voice-transcribed, or accent-affected text and
  wants the refined version copied to clipboard without executing it as a Claude
  Code request. Same refinement as /refine:ask but clipboard-only output.
argument-hint: "<raw text with potential ASR errors>"
---

# Refine Only (Clipboard)

Correct ASR, accent, grammar, and style errors in user input, copy the refined text to clipboard, and stop. Do NOT execute the refined text as a Claude Code request.

## Process

Check the user's argument: `$ARGUMENTS`

### Step 1: Gather Context

Use the current session to disambiguate — this is the primary signal, more important than the phoneme tables:
- **Working directory and project type** — informs domain-specific terms
- **Recent conversation** — what language, framework, tools, files were discussed
- **Technical domain** — programming vs general question vs system admin, etc.

Session context resolves most ambiguity. For example, "carrot code" in a Claude Code session obviously means "Claude Code".

### Step 2: Refine the Text

Apply corrections preserving the speaker's original intent. Categorize every correction into one of the categories below — this is critical for `/refine:summary` analysis.

**A. Phoneme confusion (accent-aware) — category: `pronunciation`**

| Confusion | Subcategory | Examples | Common L1 |
|-----------|-------------|----------|-----------|
| v/b | `v_b` | "bery"="very", "bian"="vain" | Spanish, South Asian, Japanese |
| l/r | `l_r` | "light"/"right", "craw"="claw" | Chinese, Japanese, Korean |
| th/s/z | `th_s` | "ze"="the", "sink"="think" | Most non-native |
| p/f | `p_f` | "pood"="food", "pix"="fix" | South Asian, Arabic, Korean |
| w/v | `w_v` | "wery"="very", "vest"="west" | Slavic, South Asian |
| n/ng | `n_ng` | "sin"="sing", "ban"="bang" | East Asian |
| vowel length | `vowel` | "ship"/"sheep", "bit"/"beat" | Chinese, Japanese, Spanish |

**B. Grammar errors — category: `grammar`**

| Pattern | Subcategory | Examples |
|---------|-------------|----------|
| Missing article | `missing_article` | "I need file" → "I need a file" |
| Wrong article | `wrong_article` | "I saw a moon" → "I saw the moon" |
| Unnecessary article | `extra_article` | "The life is beautiful" → "Life is beautiful" |
| Wrong preposition | `wrong_preposition` | "depend of" → "depend on" |
| Missing preposition | `missing_preposition` | "listen the music" → "listen to the music" |
| Subject-verb agreement | `sv_agreement` | "it don't work" → "it doesn't work" |
| Tense error | `tense` | "I have seen it yesterday" → "I saw it yesterday" |
| Word order | `word_order` | "always I go" → "I always go" |
| Plural/singular | `plural` | "many informations" → "much information" |

**C. Collocation errors — category: `collocation`**

| Pattern | Subcategory | Examples |
|---------|-------------|----------|
| Verb + noun | `verb_noun` | "do a mistake" → "make a mistake" |
| Adj + noun | `adj_noun` | "strong rain" → "heavy rain" |
| Verb + preposition | `verb_prep` | "consist in" → "consist of" |
| Wrong intensifier | `intensifier` | "I very like" → "I really like" |

**D. Word-level corrections — category: `word_boundary`**
- Homophones via context (there/their/they're, to/too/two, its/it's)
- Word boundaries: "alot"="a lot", "to gether"="together"
- Number confusion: "fifteen"/"fifty" — use context
- Contractions: "could of"="could have"

**E. ASR artifacts — category: `asr`**
- Remove phantom phrases, repeated fragments
- Filler removal: strip meaningless "uh", "um", "like", "you know"
- ASR hallucinations: "carrot code"="Claude Code"

**F. Preserve absolutely:** Technical terms, CLI commands, code snippets, proper nouns, domain jargon, original intent.

**G. Programming context:**
- "rost"/"rust" = "Rust", "pie thon" = "Python", "no JS" = "Node.js"
- "get hub" = "GitHub", "docker"/"taker" = "Docker"
- "claw"/"crowd"/"carrot code" = "Claude Code", "re-act" = "React"

### Step 3: Log the Refinement

After refining, append a structured JSONL entry to the history log. Each correction MUST include its category and subcategory so `/refine:summary` can generate accurate charts and coaching.

Build a JSON object with this structure and append it:

```
{
  "ts": "<ISO 8601 UTC timestamp>",
  "original": "<original text>",
  "refined": "<refined text>",
  "corrections": [
    {
      "from": "<original word/phrase>",
      "to": "<corrected word/phrase>",
      "category": "<pronunciation|grammar|collocation|word_boundary|asr>",
      "subcategory": "<specific subcategory from tables above>"
    }
  ]
}
```

Use a bash command to append the single-line JSON to the log file:

```bash
printf '%s\n' '<SINGLE_LINE_JSON>' >> ~/.claude/refine-history.jsonl
```

**IMPORTANT:**
- The corrections field MUST be a JSON array of objects, not a plain string. Each correction object must have `from`, `to`, `category`, and `subcategory` fields.
- Escape all double quotes inside string values with backslash.
- If text contains single quotes (e.g., "don't", "it's"), use a heredoc or `$'...'` quoting to avoid shell escaping issues. Example: `printf '%s\n' "$json_line" >> ~/.claude/refine-history.jsonl` where `json_line` is constructed with proper escaping.
- Keep the entire JSON on a single line.

If no corrections were needed, still log the entry with an empty corrections array.

### Step 4: Copy to Clipboard

Copy the refined text to the system clipboard using the appropriate command for the user's platform:

```bash
printf '%s' 'REFINED_TEXT' | pbcopy
```

Replace REFINED_TEXT with the actual refined text. If `pbcopy` is not available (non-macOS), fall back to `xclip -selection clipboard` or `wl-copy`.

### Step 5: Show Result

Display the refinement and confirm clipboard copy:

```
Refined: <corrected text>
Key corrections: <comma-separated list of changes, only if non-trivial>
Copied to clipboard.
```

If no corrections were needed, still copy the original text to clipboard and confirm:

```
No corrections needed. Copied to clipboard.
```

**IMPORTANT: Do NOT execute the refined text. Do NOT treat it as a Claude Code request. The task is complete after copying to clipboard.**
