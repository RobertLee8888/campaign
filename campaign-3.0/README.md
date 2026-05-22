# Campaign 3.0

Iteration on `campaign-2.0/` after Feedback Round 1 (2026-05-22)
+ Round 2 in-conversation revisions the same day.

The 2.0 page was the pain-point-led rebuild. 3.0 absorbs feedback
from `#proj-kolplaybook`: drops the "KOL" framing, leans into
**FinTwit Alpha League** as the brand, and rewrites the narrative
around the leaderboard.

## Narrative spine (3.0, post-round-2)

```
Section 1 — Opening (hook, no CTA)
  H1: You follow @___ on X. Do you know their track record?
  Sub: We backtested every public call from FinTwit's most-followed
       creators. A few actually beat the market.

Section 2 — Stage (backtest claim + creator cards rail)
  H1: We backtested 2.4M+ public calls from FinTwit's most-followed
      creators. A few actually beat the market. Here's the ranking.
  CTA: Test any FinTwit creator (+ live "calls tracked today" pill,
       + 3 perms checks)

Section 3 — Real-time leaderboard lead-in (grey band, normal section)
  Eyebrow: REAL-TIME LEADERBOARD
  3-line H2:
    · We built a real-time leaderboard.
    · Every public call, backtested.        (teal)
    · Every account ranked by real returns.
  Body + Learn more + trust badges + 1,590 calls tracked today pill

Section 5 — THE FINTWIT ALPHA LEAGUE  (white band, independent module)
  Heading: THE FINTWIT ALPHA LEAGUE
  Sub:     See FinTwit accounts whose calls beat the market.
           Updated every hour.
  Action:  View full leaderboard
  Table:   # · Creator · Best ROI · Median · Win · Quality · Risk
           (KOL Score column removed — users found it subjective)

Section 6 — Chat with FinTwit (grey, CTA = Open playbook chat)
  Each creator's playbook, distilled into an AI agent.

Section 7 — Request a creator (white, CTA = Submit handle)
  Don't see the FinTwit creator you follow? Drop a handle.

Footer — Methodology · FAQ · Disclaimer
```

## What changed vs 2.0

**Terminology.** Global "KOL" → "FinTwit creator" / "FinTwit account".
Brand word for the leaderboard: **FinTwit Alpha League**. "KOL audit"
framing dropped entirely.

**Nav.**
- Tagline: `LIVE FINTWIT PLAYBOOK ENGINE`
- Nav tabs collapsed to two: `Leaderboard` + `Chat with Fintwit`
- Primary CTA: `Initialize Scan` → `Test any FinTwit creator`
- Ghost CTA: `For KOLs` → `For FinTwit creators`, opens the
  onboarding modal directly (no scroll-to-bottom + mailto detour)

**Hero (Section 1).** New question H1 `You follow @___ on X. Do you
know their track record?` + sub. No CTA on this screen.

**Stage (Section 2).** Rewritten H1 (backtest claim → "Here's the
ranking."). Primary CTA `Test any FinTwit creator` aligned with nav.
Live calls-tracked-today pill + 3 perms checks under the CTA.

**Section 3 leaderboard lead-in.** Round 1 wrote this as a separate
giant centered hero panel — round 2 collapsed it back into a regular
left-aligned `.section.alt` (grey band) so the page rhythm matches
the rest. 3-line H2 + body + trust badges + live pill.

**Section 5 leaderboard.** Heading `THE FINTWIT ALPHA LEAGUE` in the
diagonal layout (head top-right, table bottom-left). KOL Score column
removed (table + sort tabs + JS). Default sort: Best ROI.

**Methodology modal.** Refers to "rank" instead of "score"; explains
the 5 objective metrics (Best ROI / Median / Win / Quality / Risk).

**Removed in round 2 (was in round 1):**
- Section 3.1 "Initialize Scan" inline 3-step card with dual CTAs
- Section 4 three-number stats strip (2.4M+ / 3,000+ / 1,590)
- The standalone giant-centered-headline treatment for Section 3
- The `paste-handles` modal markup is left in place (unused now)

## Style

Identical to 2.0 — `#f5f5f7` page baseline, `#0a0a10` dark cards,
`#49A3A6` brand teal, Space Grotesk display / Inter body / JetBrains
Mono labels, Phosphor Icons.

## Files

- `index.html` — single-file landing page (HTML + CSS + JS inline)
- `logo.svg`   — brand mark, carried over from 2.0

## Run locally

```
python3 -m http.server
```

Open `http://localhost:8000` in a browser.

## Open items (not addressed in this round)

- P1 — merge Section 1 + Section 2 into a single tightened hero
  ("let the creator cards speak for themselves")
- P1 — Chat with Fintwit (Section 6) full redesign
- P2 — Mobile adaptation (currently inherits 2.0 media queries)
- P2 — Real /scan/* flow after the user clicks through

## Sources

- Slack `#proj-kolplaybook` — feedback round 1 thread + harryz 13:23
  meeting agenda + harryz 23:49 framing thoughts
- Notion: KOL Campaign Landing Page Feedback Round 1
- File: `campaign-2.0-feedback-round-1.md` (2026-05-22)
- Round 2: in-conversation user feedback on the same date
