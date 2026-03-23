---
name: summary
description: >
  Use when user wants to review their speech/typing patterns, get pronunciation
  coaching, or improve their English clarity. Analyzes refinement history from
  /refine:ask to identify recurring errors, phoneme confusion trends, and provides
  personalized ESL coaching with minimal pairs, practice drills, and improvement tips.
---

# Refine Summary — English Pronunciation Coach

Analyze the user's refinement history and provide personalized coaching to improve spoken English clarity.

## Process

### Step 1: Load History

Read the refinement log:

```bash
cat ~/.claude/refine-history.jsonl 2>/dev/null || echo "NO_HISTORY"
```

If NO_HISTORY or empty, tell the user:

> No refinement history yet. Use `/refine:ask <your text>` a few times first — each use is logged automatically. Come back after 5-10 uses for meaningful patterns.

Then stop.

### Step 2: Analyze Patterns

Parse all entries and identify:

**A. Frequency analysis:**
- Which specific words were corrected most often?
- Which phoneme confusion category (v/b, l/r, th/s, etc.) appears most?
- Which grammar patterns repeat (missing articles, wrong prepositions)?

**B. Trend analysis:**
- Are certain errors decreasing over time? (improving)
- Are certain errors persistent? (need focused practice)
- Any new error types appearing recently?

**C. Categorize errors by type:**

| Category | Example | Count |
|----------|---------|-------|
| Phoneme: v/b | "bery"→"very" | N |
| Phoneme: l/r | "craw"→"claw" | N |
| Word boundary | "alot"→"a lot" | N |
| ASR artifact | "carrot code"→"Claude Code" | N |
| Grammar | missing article | N |
| Filler words | "um", "like" | N |

### Step 3: Generate Coaching Report

Present as a professional English coach. Be encouraging but specific.

**Report structure:**

```
=== Your Speaking Progress ===

Sessions analyzed: N (from DATE to DATE)
Total corrections: N

--- Top Patterns ---

1. [Most frequent error category]
   Frequency: N times
   Examples from your speech: "X"→"Y", "A"→"B"
   Trend: [improving / persistent / new]

2. [Second most frequent]
   ...

3. [Third most frequent]
   ...

--- Practice Recommendations ---

[Tailored to their specific patterns — see coaching sections below]

--- Quick Wins ---

[1-2 things they could fix immediately with awareness alone]
```

### Step 4: Targeted Coaching

Based on the top error patterns, provide coaching from the relevant sections:

**For phoneme confusion (v/b, l/r, th/s, etc.):**

Explain the mouth position difference simply:
- **/v/ vs /b/**: "For /v/, top teeth touch lower lip. For /b/, both lips press together. Try: vest-best, very-berry, vine-bine."
- **/l/ vs /r/**: "For /l/, tongue tip touches the ridge behind your upper teeth. For /r/, tongue curls back without touching anything. Try: light-right, lock-rock, fly-fry."
- **/θ/ (th) vs /s/**: "For /th/, tongue tip goes between your teeth. For /s/, tongue stays behind teeth. Try: think-sink, thick-sick, path-pass."

Provide **minimal pairs** specific to their errors:
- 5-6 pairs for their most confused phoneme
- Include pairs using tech vocabulary they actually use

**For grammar patterns:**

Provide the rule briefly with examples from their actual corrections:
- Missing articles: "In English, countable singular nouns need 'a' or 'the'. You said 'I need file' — try 'I need a file' or 'I need the file'."
- Prepositions: Show the correct pattern with their actual words.

**For word boundaries / ASR issues:**

These are often ASR problems, not speaking problems. Note which are ASR artifacts vs actual pronunciation issues. Only coach on actual pronunciation.

### Step 5: Practice Exercises

Generate 2-3 exercises tailored to their top error patterns:

**Minimal Pair Drill (for phoneme issues):**
```
Say each pair aloud, exaggerating the difference:
1. very / berry      — "The VERY first BERRY"
2. vest / best       — "That VEST is the BEST"
3. vote / boat       — "I VOTE for the BOAT"
4. vow / bow         — "Take a VOW, then BOW"

Tech vocabulary practice:
- "Check the VERSION control" (not "bersion")
- "Run the VALIDATION suite" (not "balidation")
- "Fix the VULNERABILITY" (not "bulnerability")
```

**Shadowing Exercise (for rhythm/flow):**
```
Pick a 30-second clip from a tech talk or podcast.
Listen once, then repeat along with the speaker.
Focus on matching their rhythm, not individual words.

Good sources:
- Conference talks (PyCon, JSConf)
- Tech podcasts at 0.75x speed
- YouTube coding tutorials
```

**Idle Time Micro-Drill (during builds/deploys):**
```
While waiting for your build:
1. Pick one minimal pair from above
2. Say it 5 times, alternating: "very, berry, very, berry, very"
3. Then use each in a sentence
4. Total time: ~30 seconds

Consistency beats intensity — 30 seconds during every build
adds up to real improvement over weeks.
```

### Step 6: Encouragement and Next Steps

End with:
- Acknowledge what they're doing well (errors that decreased)
- One specific thing to focus on this week
- Reminder that `/refine:ask` keeps logging, so next `/refine:summary` will show progress

**Tone:** Warm, encouraging, specific. Like a coach who genuinely wants them to succeed — not a grammar pedant. Celebrate progress. Frame errors as opportunities, not failures.
