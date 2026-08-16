# T29 — `POST /matches` result endpoint + anti-abuse

**Repo:** footy-fc-api · **Depends on:** T09, T28 · **Blocks:** T30, T31 (FE)

## Description
The single write path for match results. Client submits raw stat sheets; server
validates plausibility, computes ratings/deltas via T28, and applies everything
atomically.

## Endpoint
`POST /matches` (auth) — body:
```
{ startedAt, endedAt, difficulty: 'easy'|'medium'|'hard',
  score: { user: number, ai: number },
  playerSheets: [ per-player T28 input, keyed by playerId ]  // XI + used subs (11–15)
}
```
Response: `201 { matchId, ratings: [{ playerId, rating, overallDelta, newOverall }], motmPlayerId, record }`

## Tasks
- [ ] `Match` model: persists the full submission + computed outputs (audit trail)
- [ ] Zod validation + **plausibility bounds** (reject outside):
      - per-player: goals ≤ 8, shots ≤ 25, passes attempted ≤ 200, tackles ≤ 20,
        saves ≤ 15, distance ≤ 15km, completions ≤ attempts
      - team: score.user === Σ player goals; sheets only contain players from the
        caller's team; 11 ≤ sheet count ≤ 15; exactly one GK sheet
      - duration: `endedAt − startedAt` within 0.8–3× expected match length
        (2 × 4-min halves + stoppage); `startedAt` not in the future, not older than 24h
- [ ] Rate limit: max 1 submission / 5 min / user; duplicate-window check
      (identical sheet hash within 10 min → 409)
- [ ] Transaction: create Match, update each player's `overall` + `attributes` +
      `statHistory`, update team `record` — all-or-nothing
- [ ] MOTM = highest rating (ties → most goals, then assists)
- [ ] Update shared contract types

## Acceptance criteria
- Tampered payload (40 goals, or a rival team's playerId) → 400/403, nothing persisted
- Legit match updates 11–15 players + team record in one transaction; induced failure
  mid-write leaves DB untouched
- Response deltas exactly equal DB changes (integration test)
