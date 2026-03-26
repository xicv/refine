---
name: only
description: >
  Use when user provides rough, voice-transcribed, or accent-affected text and
  wants the refined version copied to clipboard without executing it as a Claude
  Code request. Same refinement as /refine:ask but clipboard-only output.
argument-hint: "<raw text with potential ASR errors>"
---

# Refine Only (Clipboard)

Correct ASR and accent errors in user input, copy the refined text to clipboard, and stop. Do NOT execute the refined text as a Claude Code request.

## Process

Check the user's argument: `$ARGUMENTS`

### Step 1: Gather Context

Use the current session to disambiguate — this is the primary signal, more important than the phoneme tables:
- **Working directory and project type** — informs domain-specific terms
- **Recent conversation** — what language, framework, tools, files were discussed
- **Technical domain** — programming vs general question vs system admin, etc.

Session context resolves most ambiguity. For example, "carrot code" in a Claude Code session obviously means "Claude Code".

### Step 2: Refine the Text

Apply corrections preserving the speaker's original intent:

**A. Phoneme confusion (accent-aware):**

| Confusion | Examples | Accents |
|-----------|----------|---------|
| v/b | "bery"="very", "bian"="vain" | Spanish, South Asian |
| l/r | "light"/"right", "craw"="claw" | East Asian |
| th/s/z | "ze"="the", "sink"="think" | Most non-native |
| p/f | "pood"="food", "pix"="fix" | South Asian, Arabic |
| w/v | "wery"="very", "vest"="west" | Slavic, South Asian |
| n/ng | "sin"="sing", "ban"="bang" | East Asian |

**B. Word-level corrections:**
- Homophones via context (there/their/they're, to/too/two, its/it's)
- Word boundaries: "alot"="a lot", "to gether"="together"
- ASR hallucinations: remove phantom phrases, repeated fragments
- Number confusion: "fifteen"/"fifty" — use context
- Contractions: "could of"="could have"
- Filler removal: strip meaningless "uh", "um", "like", "you know"

**C. Grammar:** Fix agreement, tense, missing articles, prepositions.

**D. Preserve absolutely:** Technical terms, CLI commands, code snippets, proper nouns, domain jargon, original intent.

**E. Programming context:**
- "rost"/"rust" = "Rust", "pie thon" = "Python", "no JS" = "Node.js"
- "get hub" = "GitHub", "docker"/"taker" = "Docker"
- "claw"/"crowd"/"carrot code" = "Claude Code", "re-act" = "React"

### Step 3: Log the Refinement

After refining, append a JSONL entry to the history log so `/refine:summary` can analyze patterns later:

```bash
printf '%s\n' '{"ts":"'"$(date -u +%Y-%m-%dT%H:%M:%SZ)"'","original":"ORIGINAL_TEXT","refined":"REFINED_TEXT","corrections":"CORRECTIONS_LIST"}' >> ~/.claude/refine-history.jsonl
```

Replace ORIGINAL_TEXT, REFINED_TEXT, and CORRECTIONS_LIST with the actual values. Escape any double quotes in the text with backslash. If the file does not exist yet, the append creates it.

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
