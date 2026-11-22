# Disdained Dozen Sports Network

The league site for **The Disdained Dozen** (Sleeper dynasty, 12 teams, full PPR, IDP), league id `1354220860705107969`.
Twelve teams, zero respect. Live at **https://disdained-dozen.vercel.app**.

Built from the same code as the Ordinary Gentleman Sports Network, as a separate repo so neither site can affect the other.

## What's here

```
public/                  everything the website serves
  index.html             the site (built; don't edit by hand)
  live.js                live layer: fresh standings + live scores from Sleeper
  data/league-data.json  the league data the page is built from
  favicon.svg
scripts/
  hub_template.txt       the page template
  refresh.py             pulls Sleeper, FantasyCalc and Dynasty Daddy, builds league-data.json,
                         and computes the IDP values (see below)
  build_site.py          template + data -> public/index.html (adds the live layer)
  news.py                news items for trades, $10+ waiver adds and renames
  recap_prep.py          gathers the facts for last week's recap once the week is over
  apply_writing.py       checks the writer's recap draft and fills every number in from the facts
.github/workflows/refresh.yml       runs everything every morning and commits
.github/workflows/recap-check.yml   manual check of a week's recap facts
vercel.json              hosting settings, including the /sleeper pass-through to Sleeper's API
```

## How it runs (no one has to do anything)

| When | What | Where |
|---|---|---|
| Someone opens the site | `live.js` pulls current records, standings and scores from Sleeper; the ticker refreshes every minute during games | the visitor's browser |
| Every morning, about 5 AM Central | `refresh.py` pulls Sleeper, FantasyCalc and Dynasty Daddy; `news.py` writes news items; `recap_prep.py` gathers last week's recap facts once the week is over (Tuesday); page rebuilt and committed | GitHub Actions |
| Every morning, about 8 AM Central | **DDSN Writer** (Claude scheduled task, runs in the cloud): writes the weekly recap when facts are waiting, and write-ups for new trades | Claude cloud |
| Every commit | Vercel redeploys the site | Vercel |

If Sleeper is down in the morning, the refresh keeps yesterday's data and tries again the next day.

The Week 10 recap is the **Season in Review** (Weeks 1 to 10). Weekly recaps start with Week 11.

## League rules

From the commissioner, stored in `public/data/league-data.json` under `rules` and shown on the Rules page: $225 buy-in; FAAB waivers capped at $100 a season; $5 per trade; $2 minimum for adds and uncontested waivers. Rookie draft: non-playoff teams by regular-season record, worst first; playoff teams in the order they are eliminated, champion last (`draft_order` in `refresh.py`, and the same rule in the page's playoff simulator). Tiebreaks the site assumes: fewer points for on equal records; worse seed first among teams eliminated in the same round.

## IDP values

FantasyCalc and Dynasty Daddy only value offense, so `refresh.py` estimates DL, LB and DB values (`idp_model`, `idp_values`):

1. **Position level:** how far the startable players at each defensive spot (top 12) score above a waiver-level player (25th), divided by the same gap for WRs (top 24 against the 49th), in points per game under this league's scoring.
2. **Player order:** points per game this season (60%), last season (25%) and the season before (15%), times an age factor.
3. **Value:** the Nth-best defender at a position gets the value of WR (2N-1) times the position ratio.

These are rough and the site says so on every value page and in "How this site works".

## Going live (one-time setup)

1. In this repo: **Settings → Actions → General → Workflow permissions → Read and write permissions**.
2. In [vercel.com](https://vercel.com): **Add New → Project**, pick this repo, deploy with the default settings. (Done: https://disdained-dozen.vercel.app)
3. Optional: run the workflow once from **Actions → Daily data refresh → Run workflow**.

## Local preview

```
python3 scripts/build_site.py
python3 -m http.server 8000 -d public
```
