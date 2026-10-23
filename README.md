# auspex-site

The static front end for **Auspex**, a live signal feed for finding Solana and Base tokens early. Live at **https://auspex-site.vercel.app**.

It is a single `index.html` with no build step. It reads a shared cache from Supabase using the public anon key. The cache is filled every 90s by the background worker, [drixindustries/auspex-worker](https://github.com/drixindustries/auspex-worker).

## What the page reads

| Panel | Source |
|---|---|
| Signal Feed, Raw feeds, Just Launched, Just Graduated, hot themes, SOL regime | `auspex_signal_cache` row `latest` (written by the worker) |
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

## Configuration

The Supabase project URL and **anon** key are set near the top of the `<script>` in `index.html`. They are designed to be public; RLS limits the anon key to read-only access (the tables above plus `auspex_bad_actors` and `auspex_good_devs`). **Never put the `service_role` key in this repo.**

The database schema, including the RLS policies, is in the worker repo: [`auspex-schema.sql`](https://github.com/drixindustries/auspex-worker/blob/main/auspex-schema.sql). It was reconstructed from the code (added in auspex-worker PR #1) and should be diffed against the live project before applying.

## Run locally

```bash
python3 -m http.server 8080   # then open http://localhost:8080
```

## Deploy

Vercel deploys `main` automatically. The page is only as fresh as the cache, so if the banner says the data is stale, check that the Railway worker is running and that its `SUPABASE_URL` is the same project as `SUPABASE_URL` in `index.html` (`qwykkcsyoghljgndnvaz`).
