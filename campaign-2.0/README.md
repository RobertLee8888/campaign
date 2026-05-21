# Campaign 2.0

Redesigned landing page for ALVA's KOL audit campaign.

The v1 page (`RobertLee8888/campaign`) was a product-driven dashboard. v2 is rebuilt around a single user pain question and answers it section by section.

## Narrative spine

Every section opens with a question the user actually has at that point in the funnel — the section title is the question, the subhead is the answer.

```
Hero       Q: 他们看上去每天都在赚钱。那我跟着，为什么在亏？
              A: 每条 call 都被回测，账号被打分。

Receipts   Q: 想看他们的真账本吗？
              A: 5 张 audited cards · win / loss / timeline 全部展开
              → CTA: 查看你关注的 KOL 评分 (Connect X)
              → side: How we score KOLs (methodology modal)

Leaderboard Q: 想找更多扛得住审计的 KOL 吗？
              A: 3,000+ 索引 / 86 全量回测 / 6 维实时排行榜

Agents     Q: 想学他们到底怎么赚的吗？
              A: KOL operating habits → AI agent, backtest 持续在跑

For KOLs   Q: 你自己也是 X 上的市场声音吗？
              A: Real signal stands up to audit. Come prove yours.

Request    Q: 没找到你关注的 KOL？（窄条）
              A: 留 handle, 7 天内回测 + 通知

Footer     Methodology · FAQ · Disclaimer
```

## What changed vs v1

- Hero pain question replaces "Alva KOL Leaderboard" product title
- KOL claim banner moved from Hero to dedicated For-KOLs section
- 4 stat cards in hero removed (replaced by single proof line + live ticker strip)
- Orbit avatar animation removed (replaced by horizontal "audited live" strip)
- New Receipts section: 5 dark cards on dark band, every card shows wins **and** misses
- Score is account-level only (per-call scoring explicitly removed from copy)
- Leaderboard simplified to full table only (no duplicate Top 3 cards)
- AI persona upgraded from "quotes public tweets" to "AI agent of operating system" — persona chips show actual style tags (Macro regime / Stops at -8% / etc.)
- For KOLs reframed from "$10,000 claim banner" to "Real signal stands up to audit. Come prove yours."
- Request section collapsed to a narrow strip above the footer
- Methodology demoted from full section to a modal — backing only, not part of the story

## Style reuse

- Light page baseline `#f5f5f7`, dark cards `#0a0a10`
- Brand teal `#49A3A6` (from `logo.svg`)
- Type stack: Space Grotesk display / Inter body / JetBrains Mono labels
- Phosphor Icons (regular / bold / fill)
- Same rounded card language, same shadow scale

## Files

- `index.html` — single-file landing page (HTML + CSS + JS inline, no build step)
- `logo.svg` — brand mark, carried over from v1

## Run locally

```
python3 -m http.server
```

Open `http://localhost:8000` in a browser.

## Push to GitHub

This directory is ready to be pushed as a new repo. From this folder:

```bash
git init
git add .
git commit -m "Campaign 2.0 · pain-point-led rebuild"
gh repo create campaign-2.0 --public --source=. --push
```

(Or create the repo on github.com first, then `git remote add origin … && git push -u origin main`.)
