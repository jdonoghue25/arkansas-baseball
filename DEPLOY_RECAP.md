# Arkansas Baseball App — Deploy Recap

## Live URLs

- **Website:** https://arkansas-baseball-6n4ugma71-jdonoghue25-2785s-projects.vercel.app
- **GitHub:** https://github.com/jdonoghue25/arkansas-baseball

---

## How the Auto-Update Pipeline Works

```
Mac cron (8am / 12pm / 6pm CT)
  → scrape_arkansas.py   (fetches site, parses 57 games, updates SQLite, writes JSON)
  → push_update.sh       (copies JSON → git commit → git push → vercel --prod)
  → Live website updates automatically
```

Each run takes ~15 seconds total. The website reflects fresh data within seconds of each cron fire.

---

## How to Manually Trigger an Update

```bash
# Run scraper + export JSON
cd ~/Projects/scripts
.venv/bin/python scrape_arkansas.py --export

# Push JSON to GitHub and redeploy Vercel
bash ~/Projects/scripts/push_update.sh
```

Or do both in one command:
```bash
.venv/bin/python /Users/ai_harper/Projects/scripts/scrape_arkansas.py --export && bash ~/Projects/scripts/push_update.sh
```

---

## Cron Schedule

Runs at **8:00am, 12:00pm, and 6:00pm CT** every day:

```
0 8,12,18 * * * /Users/ai_harper/Projects/scripts/.venv/bin/python /Users/ai_harper/Projects/scripts/scrape_arkansas.py --export >> /Users/ai_harper/Projects/scripts/scraper_log.txt 2>&1 && /bin/bash /Users/ai_harper/Projects/scripts/push_update.sh >> /Users/ai_harper/Projects/scripts/scraper_log.txt 2>&1
```

View the log: `tail -f ~/Projects/scripts/scraper_log.txt`

---

## File Locations

| File | Path |
|------|------|
| Website HTML | `~/Projects/html-tools/arkansas-baseball/index.html` |
| JSON data | `~/Projects/html-tools/arkansas-baseball/arkansas_schedule.json` |
| Scraper | `~/Projects/scripts/scrape_arkansas.py` |
| Push script | `~/Projects/scripts/push_update.sh` |
| SQLite DB | `~/Projects/scripts/college_baseball.db` |
| Cron log | `~/Projects/scripts/scraper_log.txt` |

---

## Issues Found and Fixed

1. **Vercel GitHub auto-deploy link failed** — Vercel couldn't link to GitHub during CLI deploy (requires connecting accounts in the Vercel dashboard). Fixed by adding `vercel --prod --yes` to `push_update.sh` so every cron run triggers an explicit redeploy.

2. **JSON missing `last_updated` field** — The original scraper didn't include a timestamp in the JSON export. Added `last_updated` field so the website can display "Last updated: YYYY-MM-DD HH:MM CT" at the bottom.

---

## What the Website Shows

- Overall / SEC / Home / Away record + Run differential + Current streak
- "Next Game" banner with pulsing dot (Ole Miss, May 1)
- Full 2026 schedule (57 games) with filter tabs: All / Wins / Losses / Upcoming / Home / Away / SEC Only
- Each game card: date, opponent with ranking badge, Home/Away/Neutral tag, SEC badge, city, score/result or upcoming time
- Tap any completed game with a recap → opens recap article in new tab
- Dark theme, Cardinal red (#CC0000), Bebas Neue headers, mobile-first
