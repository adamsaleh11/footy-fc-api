# T28 — Match rating + overall-delta algorithm

**Repo:** footy-fc-api · **Depends on:** T07 · **Blocks:** T29

## Description
Pure, deterministic service that turns a validated per-player match stat sheet into a
0–10 match rating and an overall delta. Server-side only; the FE merely displays results.
Start this as soon as T09 ships — it has no FE dependency and the FE match engine takes
weeks.

## Input (per player)
```
{ position, minutesPlayed, goals, assists, shotsOnTarget, shots,
  passesCompleted, passesAttempted, keyPasses, dribblesCompleted,
  tacklesWon, tacklesLost, interceptions, blocks, foulsCommitted,
  duelsWon, duelsLost, saves, goalsConceded, cleanSheet, distanceKm }
```

## Algorithm
1. **Base rating 6.0**, adjusted by position-weighted event scores:
   - ST/W: goals (+1.0 each), assists (+0.7), shots on target (+0.1), dribbles (+0.05)
   - CM/CAM/CDM: assists, key passes (+0.15), pass accuracy vs 80% baseline (±0.5 max),
     tackles/interceptions for CDM
   - CB/FB: duel win % (±0.6), tackles won (+0.15), interceptions (+0.1),
     clean sheet (+0.8), each goal conceded (−0.3), fouls (−0.1)
   - GK: saves (+0.25 each), clean sheet (+1.0), goals conceded (−0.4), save % curve
   - All positions: goals always count (CB scoring = +1.0 too); cap rating to [0, 10]
2. **Delta:** `delta = k × (rating − 6.5)` mapped into **[−1.0, +1.0]**
3. **Diminishing returns:** multiply positive deltas by a decay factor as overall rises
   (e.g. ×1.0 at ≤70, ×0.5 at 80, ×0.2 at 88, ×0 at the soft cap **90**). Negative
   deltas are not decayed (falling is always possible). Floor overall at 40.
4. Sub-60-minute appearances scale delta by `minutes/60`.
5. Deterministic: same sheet in → same numbers out. No RNG anywhere.

## Tasks
- [ ] `services/rating.ts`: `computeRating(sheet, position): number` and
      `computeDelta(rating, currentOverall, minutes): number` — pure functions
- [ ] All weights/curves as exported named constants (T35 tunes them; target average
      ≈ +0.1/match for decent play)
- [ ] Attribute growth split: delta applies to `overall` and is distributed to the
      attribute block position-weighted (reuse T07 weight table), so a CB grows
      defending faster than finishing

## Acceptance criteria
- Unit tests: hat-trick ST → rating ≥ 9, delta ≈ +1.0 (below decay range);
  CB conceding 3 with lost duels → rating ≤ 5, negative delta;
  identical sheets → identical outputs (snapshot)
- Delta at overall 88 is < ¼ of the same performance's delta at overall 65
- Overall can never exceed 90 or drop below 40
