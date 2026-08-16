# T08 — Team model

**Repo:** footy-fc-api · **Depends on:** T07 · **Blocks:** T09

## Description
The `Team` model: one per user, exactly 15 players, formation + starting XI + bench.

## Schema
```
userId: ref User (unique index — one team per user)
teamName: string (3–24 chars)
players: [ref Player]        // exactly 15
formation: '4-3-3' | '4-4-2' | '4-2-3-1' | '3-5-2' | '5-3-2' (default 4-3-3)
startingXI: [ref Player]     // exactly 11, subset of players
bench: [ref Player]          // exactly 4, the remaining players
record: { wins, draws, losses, goalsFor, goalsAgainst }
createdAt
```

## Tasks
- [ ] Schema-level validators:
      - `players.length === 15`; `startingXI.length === 11`; `bench.length === 4`
      - XI ∪ bench === players (no dupes, no outsiders)
      - XI contains **exactly one** player whose `position === 'GK'`
      - formation slot-count sanity: XI positions must be mappable onto the chosen
        formation's slots (define slot table per formation; allow off-position with
        no penalty in v1, but slots must all be filled)
- [ ] Default XI/bench auto-pick on creation (best overall per formation slot)
- [ ] Unique index on `userId`; index on `teamName`

## Acceptance criteria
- Creating a team with 14 or 16 players → validation error
- XI without a GK, or with 2 GKs, → validation error
- Second team for the same user → 409
