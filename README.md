# Refine

ASR-aware text refinement and pronunciation coaching plugin for Claude Code. Designed for non-native English speakers who use voice input.

## What it does

**`/refine:ask <text>`** — Corrects accent and speech-to-text errors in your input, then executes the refined request in Claude Code. Handles phoneme confusion (v/b, l/r, th/s), word boundaries, ASR hallucinations, and grammar fixes while preserving technical terms and intent.

**`/refine:only <text>`** — Same refinement as `ask`, but copies the corrected text to your clipboard instead of executing it. Useful when you want to paste the refined text somewhere else (Slack, email, docs).

**`/refine:summary`** — Analyzes your refinement history to identify recurring error patterns and provides personalized pronunciation coaching with minimal pairs, practice drills, and progress tracking.

## Install

```bash
claude plugin add xicv/refine
```

## Usage

Speak or type your rough input:

```
/refine:ask I want to add a bery important balidation to ze login form
```

Output:

```
Refined: I want to add a very important validation to the login form
Key corrections: bery→very, balidation→validation, ze→the
```

Then the refined request executes automatically.

Refine without executing (clipboard only):

```
/refine:only I sink we should refactor ze autentication module
```

Output:

```
Refined: I think we should refactor the authentication module
Key corrections: sink→think, ze→the, autentication→authentication
Copied to clipboard.
```

Review your patterns over time:

```
/refine:summary
```

## How it works

- Uses session context (project type, recent conversation, technical domain) as the primary signal for disambiguation
- Applies accent-aware phoneme correction tables for common L1 interference patterns
- Logs every refinement to `~/.claude/refine-history.jsonl` for longitudinal analysis
- Summary coaching identifies trends, provides minimal pair drills, and tracks improvement

## License

MIT
