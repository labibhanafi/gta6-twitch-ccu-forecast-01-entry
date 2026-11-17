[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/c/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/labibhanafi/gta6-twitch-ccu-forecast-01-entry/rank.svg)](https://make.fun/c/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/labibhanafi/gta6-twitch-ccu-forecast-01-entry/reward.svg)](https://make.fun/c/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

# gta6-twitch-ccu-forecast-01-entry

Forecast GTA VI's launch-day Twitch viewers hour by hour, for make.fun's
[Forecast GTA VI's launch-day Twitch viewers hour by hour](https://make.fun/c/gta6-twitch-ccu-forecast-01)
($250, free entry). `main.py` (Python 3.13, standard library only) reads one
case and writes, for each of hours 2–24 of the release day (UTC), five
quantiles (5/25/50/75/95%) of that hour's Twitch concurrent viewers in the
Grand Theft Auto VI category, scored by pinball loss on the log scale.

## Method

The forecaster is fitted at run time from the case's own comparables (38
past launches in a practice case, all 42 in the final), so nothing about the
answer is baked in:

- **The day's shape depends on the hour-1 print.** A launch whose first
  hour is small — most markets still locked, or a single simultaneous
  global unlock later in the day — grows 10–1000× by the evening
  (Hollow Knight: Silksong peaked 2,121× its hour 1). A launch whose first
  hour is already huge — rolling midnight unlocks across Europe, the UK,
  New Zealand — stays within roughly 0.5–1.3× of it all day, dipping
  midday and recovering in the evening (ELDEN RING peaked at 1.0×).
- **Median: a quadratic fit** of log(y_h / y_1) on log(hour-1 print), per
  hour. A linear fit extrapolates the small-launch explosion into a
  collapse for record-size launches (it predicts ~0.2× by the evening for
  a 1.2M hour 1); the quadratic bends back to the flat big-launch shape.
  The predictor is winsorized to the training range, so a bigger-than-ever
  hour 1 is forecast like the biggest launch on record, never worse.
- **Spread: empirical, not assumed normal.** The 8 launches nearest in
  log(hour-1 print) give the per-hour quantiles of the log-ratio; they are
  re-centred on the quadratic median and widened by 1.15.
- **Fallback:** with fewer than 6 usable comparable curves, the house
  persistence baseline (every hour at the hour-1 print, spread widening
  like a random walk).

## Validation (leave-one-out, shipped code)

Each of the 37 comparables with a full release-day curve was forecast from
the other 36 and scored exactly as the contest scores (mean pinball loss,
log scale, lower is better). There is no local grade for this competition,
so the practice score arrives from the server after submission.

| | Overall (37) | Rolling-midnight console* | Simultaneous-global | House baseline (overall) |
|---|---|---|---|---|
| This entry | **0.393** | **0.292** | 0.546 | 0.930 |
| House baseline | 0.930 | 0.406 | 1.755 | — |

\* The GTA VI reference class: console launches with rolling midnight-local
unlocks (Ghost of Yōtei, Resident Evil Requiem, Final Fantasy VII Rebirth,
The Last of Us Part II, Call of Duty, Monster Hunter Wilds, …). The four
practice launches are the same class. Coverage of the 90% band is 0.81
overall; typical miss 77% (the house baseline: 0.93 loss overall).

Practice-case spot checks (median hour-19 ratio to hour 1): Wolverine
23,357 → 5.06×; Red Dead Redemption 2 116,550 → 1.65×; God of War
Ragnarök 269,517 → 1.11×; Spider-Man 2 119,087 → 1.63×.

## Run

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

Deterministic: pure functions of the case file, fixed sort orders, no
clock, no randomness. Four practice cases run in ~0.2 s total, far inside
the 8 s per-case cap; outputs are byte-identical across runs.
