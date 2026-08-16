# T09 — Team & player APIs

**Repo:** footy-fc-api · **Depends on:** T05, T08 · **Blocks:** T11 (FE), T29

## Description
The full team/player CRUD surface the frontend builds against. All routes behind
`requireAuth`. This ticket also establishes the **shared API contract file**.

## Endpoints
| Method | Path | Purpose |
|---|---|---|
| POST | `/teams` | Create team + 15 players **atomically** (one transaction). Body: `{ teamName, formation?, players: [{ name, heightCm, strongFoot, position, avatar }] × 15 }` |
| GET | `/teams/me` | Full team, players populated |
| PATCH | `/teams/me/lineup` | `{ formation, startingXI: [playerIds], bench: [playerIds] }` |
| PATCH | `/players/:id` | Cosmetics only: `{ name?, avatar? }` — server ignores/rejects anything else |
| DELETE | `/teams/me` | Delete team + players (confirm-token pattern: body must echo team name) |

## Tasks
- [ ] Zod schemas for every body; strip unknown keys (`.strict()`) so a client sending
      `attributes` gets a 400, not a silent ignore
- [ ] `POST /teams`: mongoose session/transaction — team + 15 players commit or roll
      back together; runs T07 derivation per player; enforces ≥1 GK among the 15 and
      unique kit numbers
- [ ] Ownership middleware: `/players/:id` verifies the player belongs to `req.user`'s team
- [ ] **Contract file:** `src/types/api.ts` exporting every request/response type.
      Publish to the FE repo (copy or private package — document the chosen mechanism
      in the repo README). Every future endpoint PR updates it.

## Acceptance criteria
- Create-team payload with a tampered `overall: 99` → 400
- Kill the DB mid-create (test hook) → zero orphan players persisted
- Lineup PATCH with a player from another user's team → 403
- `GET /teams/me` before creation → 404 with `{ code: 'NO_TEAM' }` (FE routes on this)
