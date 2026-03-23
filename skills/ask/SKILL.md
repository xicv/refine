---
name: ask
description: >
  Use when user provides rough, voice-transcribed, or accent-affected text as
  their request. Triggers on ASR artifacts like v/b, l/r, th/s confusion,
  misspellings, grammar errors, or garbled speech-to-text output that needs
  correction before processing as a Claude Code request.
argument-hint: "<raw text with potential ASR errors>"
---

# Refine and Ask

Correct ASR and accent errors in user input, then execute the refined text as a Claude Code request immediately.

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

### Step 4: Show Refinement and Execute Immediately

Show a brief refinement summary, then execute without asking for confirmation:

```
Refined: <corrected text>
Key corrections: <comma-separated list of changes, only if non-trivial>
```

Then immediately execute the refined text as if the user had typed it directly — answer the question, perform the task, write the code, or whatever the refined request asks for.

If no corrections were needed, skip the summary and just execute directly.
