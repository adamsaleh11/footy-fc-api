# T07 — Player model + attribute derivation service

**Repo:** footy-fc-api · **Depends on:** T02 · **Blocks:** T08, T28

## Description
The `Player` model plus the **server-side** service that deterministically derives a
starting attribute block from height + position. This derivation is the single source
of truth — the client only ever sends creation inputs (name, height, foot, position,
avatar), never attributes.

## Schema
```
name: string (1–24 chars)
heightCm: number (160–200)
strongFoot: 'L' | 'R'
weakFootRating: number (1–5, derived: default 3, GK 2, CAM/W 4 — tunable table)
position: GK | CB | LB | RB | CDM | CM | CAM | LW | RW | ST
avatar: { skinTone, hairStyle, hairColor, kitNumber (1–99, unique within team) }
overall: number            // server-computed, starts ~65
attributes: {
  pace, acceleration, shooting, finishing, shotPower, longShots,
  passing, shortPassing, longPassing, vision, curve,
  dribbling, agility, balance, ballControl,
  defending, standingTackle, slideTackle, interceptions, marking,
  physical, strength, stamina, jumping, heading,
  // GK only:
  gkDiving, gkReflexes, gkHandling, gkPositioning, gkKicking
}
statHistory: [{ matchId, date, rating, goals, assists, passAcc, tackles,
                interceptions, saves, cleanSheet, overallDelta }]
teamId: ref Team
```

## Derivation rules (`services/attributeDerivation.ts` — pure function)
- Base template per position (e.g. ST: high finishing/shotPower/pace; CB: high
  defending/strength/heading; CAM: passing/vision/dribbling; GK: gk* block, outfield
  stats floored ~40)
- Height modifiers (linear from 160→200cm): `+strength +jumping +heading`,
  `−agility −balance −acceleration`. GK: `+gkReflexes reach` with height
- Strong foot: no attribute change; `weakFootRating` drives in-game error (FE reads it)
- Normalize so every new player's `overall` (position-weighted attribute average)
  lands at 65 ± 1 — height shifts the *shape*, not the total budget
- Deterministic: same inputs → identical attribute block (no RNG)

## Tasks
- [ ] Schema + indexes (`teamId`)
- [ ] Derivation service with the position templates + height curve as **named exported
      constants** (T35 balancing will tune them)
- [ ] `overall` computed via position weight table (also exported — reused by T28)
- [ ] Validation: reject out-of-range height, bad position, kit number collisions

## Acceptance criteria
- Unit tests: 200cm CB > 165cm CB in strength/heading, lower in agility; GK gets gk*
  block; every position/height combo yields overall 64–66
- Derivation is pure + deterministic (snapshot test)
- No API path allows writing `attributes`/`overall` directly
