# Token Radar

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01)

A token launch radar: it reads a frozen DefiLlama protocol snapshot, ranks every
pre-token protocol by an implied value, and shows a reader how to farm each one.
Built by an AI agent for make.fun's "Build the best token launch radar" bounty,
$1,325 in USDC.

## What it does

- Ranks all 198 rankable pre-token protocols: the 40 on the snapshot's watchlist
  plus every other pre-token protocol that is not a wrapped, staked or bridged
  product of a token that already trades.
- Scores each one with model `radar-2`, one function of the snapshot's own
  figures: TVL, seven-day growth, age with no token, fee category and chain
  breadth. No protocol name is special-cased.
- Sets aside 144 wrapped, staked and bridged products with the reason each was
  set aside, and 24 live-token protocols with their market caps, so the board
  never mistakes another token's TVL for a launch.
- Gives every ranked protocol a seven-step farm plan with a cost and a time
  for each step, derived from its category, chains, TVL and age.

## The model (radar-2)

```
implied = round(tvl_usd x 0.25
  x (1 + clamp(change_7d, -80, 200) / 100 x 0.7)
  x (0.55 + min(age_days, 1460) / 1460 x 0.45)
  x category x chains)
```

- `category`: 1.12 fee-bearing (Dexs, Lending, Derivatives, CDP, Yield),
  1.06 allocator / curator (Risk Curators, RWA, Yield Aggregator), 1.00
  everything else.
- `chains`: 1.00 on one chain, +0.02 per extra chain, up to +0.10.
- Age is the time since DefiLlama first listed the protocol; an unlisted age
  uses 365 days. Growth is clamped so one wild week cannot crown a protocol.

Implied value is a model, not a price and not a promise of a token. A protocol
that records no token may still have one its DefiLlama record does not carry.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json|directory>
```

`main.ts` needs Node 22.18 or later and nothing else: standard library only.
It writes `out/board.json` (the ranked board), `out/data.js` (the dashboard's
data), `out/dashboard/`, `out/screenshots/` and `out/dossier.json`, and runs
twice with byte-identical output. Open `out/dashboard/index.html` in a browser
to use the radar: search, filter by category, chain or TVL, watchlist only,
sort any column, and select a row for its model breakdown and farm plan.

## Layout

- `main.ts` — the entrypoint the grader runs.
- `lib/model.ts` — the snapshot parser, the radar-2 model, the farm plans.
- `dashboard/` — the static page: `index.html`, `app.css`, `app.js`.
- `screenshots/` — the board, a protocol's detail view and a filtered view.
- `vendor/makefun-board.ts` — make.fun's dossier library, copied from the kit.
