# T30 — Stats & history read endpoints

**Repo:** footy-fc-api · **Depends on:** T29 · **Blocks:** T32 (FE), T33

## Description
Read APIs powering the progression/history UI. All behind `requireAuth`, all scoped to
the caller's team.

## Endpoints
| Method | Path | Returns |
|---|---|---|
| GET | `/players/:id/history` | `statHistory` (paginated, newest first) + career totals (goals, assists, apps, avg rating) + last-5 form |
| GET | `/teams/me/stats` | record, top scorers/assisters (top 5), average squad overall + 10-match trend series |
| GET | `/matches` | caller's match list (paginated): date, score, difficulty, MOTM |
| GET | `/matches/:id` | full stored match detail (sheets + computed ratings) |

## Tasks
- [ ] Aggregation pipelines for career totals + squad-overall trend (compute from
      `statHistory` deltas, don't store denormalized copies yet)
- [ ] Pagination: `?limit=` (default 20, max 50) + `?cursor=`
- [ ] Ownership checks on every route (404 for other users' resources, not 403 —
      don't leak existence)
- [ ] Update shared contract types

## Acceptance criteria
- After 3 seeded matches, career totals and form match hand-computed values
- Requesting another user's player/match → 404
- p95 < 150ms on a seeded 100-match team (index check)
