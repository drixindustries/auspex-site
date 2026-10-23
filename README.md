# auspex-site

The static front end for **Auspex**, a live signal feed for finding Solana and Base tokens early. Live at **https://auspex-site.vercel.app**.

It is a single `index.html` with no build step. It reads a shared cache from Supabase using the public anon key. The cache is filled every 90s by the background worker, [drixindustries/auspex-worker](https://github.com/drixindustries/auspex-worker).

## What the page reads

| Panel | Source |
|---|---|
| Signal Feed, Raw feeds, Just Launched, Just Graduated, hot themes, SOL regime | `auspex_signal_cache` row `latest` (written by the worker) |
| Signal Performance, "since signal" chips | `auspex_signal_cache` row `latest`: `signalPerformance` (aggregates) and `signalPerf` on each tracked row. Both are computed by the worker from `auspex_signal_outcomes`. |
| Paper Trading | `auspex_paper_trades` |
| Hall of Fame | `auspex_hall_of_fame` |
| CA scanner | DexScreener `tokens/v1` and RugCheck report, called from the browser |
| Connect a feed | any JSON URL that allows browser access (CORS) |

A sticky banner warns when the cache is more than 5 minutes old or can't be read.

## Creator badges

These badges come from the worker's payload. Hover any badge to see the matching source, the reason and the listing date. The rules are in the worker README, under "Creator reputation".

| Badge | Style | Meaning |
|---|---|---|
| ⛔ SANCTIONED | solid red | Creator is on the OFAC SDN list (Solana, or ETH/Base). |
| 🚨 RUG DEV · CONFIRMED | red fill | Auspex's own list: one of the wallet's tokens was rugged (per RugCheck), or 2 or more of its graduations collapsed. |
| ⚠️ SUSPECTED RUG DEV | red outline | Auspex's own list: one graduation collapsed. |
| ⚠️ HEURISTIC LIST · UNVERIFIED | dashed amber | Third-party automated list (AllenHark). Unverified, with known false positives. |
| 🏆 PROVEN DEV / ⭐ PROMISING DEV | gold | 2 or more / 1 earlier tokens that graduated, held and never rugged. The tooltip lists them. Struck through when the wallet is also on a bad list. |
| ☠ RUGGED, 🕸 INSIDERS DETECTED (N), 🔁 SERIAL LAUNCHER: N TOKENS | various | From RugCheck reports. Many launches is not proof of a rug. |

A row with a bad match gets a red border, or a dashed amber border if the only match is heuristic. Bad flags always take priority over good ones.

## Signal performance

The **📊 Signal Performance** panel shows what happened after Auspex surfaced a signal. It covers the last 7 days.

- **Breakdowns:** type (top signal, launch, graduation), source, badge, score band and chain.
- **Columns:**
  - N;
  - hit +50% and 2×: reached that before falling -50%, counted over resolved signals, with hits/resolved shown next to the rate;
  - rug rate;
  - median return at 5m, 15m, 1h, 6h and 24h;
  - median max-up.
- Click a header to sort; empty cells always sort last.
- Rows with fewer than 10 signals are dimmed.
- On phones the table scrolls sideways with the group column pinned.

Tracked rows in the Signal Feed, Just Launched and Just Graduated show a chip such as **+34% since signal · 12m**. It turns red, with ☠, if RugCheck marked the token rugged or its liquidity collapsed. Hover the chip for the checkpoints, max up and down, and the entry basis. Definitions and caps are in the worker README, under "Signal performance".

## Configuration

The Supabase project URL and **anon** key are set near the top of the `<script>` in `index.html`. They are designed to be public; RLS limits the anon key to read-only access (the tables above plus `auspex_bad_actors`, `auspex_good_devs` and `auspex_signal_outcomes`). **Never put the `service_role` key in this repo.**

The database schema, including the RLS policies, is in the worker repo: [`auspex-schema.sql`](https://github.com/drixindustries/auspex-worker/blob/main/auspex-schema.sql). It was reconstructed from the code (added in auspex-worker PR #1) and should be diffed against the live project before applying.

## Run locally

```bash
python3 -m http.server 8080   # then open http://localhost:8080
```

## Deploy

Vercel deploys `main` automatically. The page is only as fresh as the cache, so if the banner says the data is stale, check that the Railway worker is running and that its `SUPABASE_URL` is the same project as `SUPABASE_URL` in `index.html` (`qwykkcsyoghljgndnvaz`).
