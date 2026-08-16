# Footy FC — Master Ticket Sequence

One global, dependency-ordered sequence across both repos. Backend tickets live in
`footy-fc-api/tickets/`, frontend tickets in `footy-fc-frontend/tickets/`. Ticket numbers
are **global** — a ticket's number tells you where it sits in the overall build order,
regardless of which repo it belongs to.

**How the two devs work in parallel:** each dev works down their own repo's tickets in
number order. The 🔗 rows are integration points — the FE ticket cannot be *finished*
(only stubbed/mocked) until the BE ticket it links to is deployed to the shared dev API.

| # | Repo | Ticket | Depends on |
|-----|------|--------|------------|
| T01 | BE | Backend scaffold (Express + TS + tooling) | — |
| T02 | BE | MongoDB Atlas + Mongoose connection | T01 |
| T03 | FE | Frontend scaffold (Vite + React + TS + Phaser) | — |
| T04 | BE | User model | T02 |
| T05 | BE | Auth endpoints + JWT middleware | T04 |
| T06 | FE | Auth UI flow 🔗 T05 | T03, T05 |
| T07 | BE | Player model + attribute derivation service | T02 |
| T08 | BE | Team model | T07 |
| T09 | BE | Team & player APIs | T05, T08 |
| T10 | FE | API client layer + shared types | T03 |
| T11 | FE | Team builder wizard 🔗 T09 | T06, T09, T10 |
| T12 | FE | Squad management screen 🔗 T09 | T11 |
| T13 | FE | Phaser-in-React shell + scene flow | T10 |
| T14 | FE | Pitch, camera, HUD | T13 |
| T15 | FE | Ball physics (spin, height, first touch) | T14 |
| T16 | FE | Player locomotion + stamina model | T15 |
| T17 | FE | Input system — advanced two-hand keyboard + gamepad | T16 |
| T18 | FE | Player switching | T17 |
| T19 | FE | Advanced dribbling + skill moves | T17 |
| T20 | FE | Advanced passing system | T19 |
| T21 | FE | Advanced shooting + timed finishing | T20 |
| T22 | FE | Defending: jockey, tackles, interceptions, fouls | T21 |
| T23 | FE | Goalkeeper AI | T21 |
| T24 | FE | Teammate AI (attack + defense shape) | T22 |
| T25 | FE | Opponent team AI + difficulty tiers | T24 |
| T26 | FE | Match rules & flow (restarts, halves, subs) | T25 |
| T27 | FE | In-match stat tracking | T26 |
| T28 | BE | Match rating + overall-delta algorithm | T07 |
| T29 | BE | `POST /matches` result endpoint + anti-abuse | T09, T28 |
| T30 | BE | Stats & history read endpoints | T29 |
| T31 | FE | Post-match summary screen 🔗 T29 | T27, T29 |
| T32 | FE | Progression history UI 🔗 T30 | T31, T30 |
| T33 | BE | Backend deployment (Render/Railway) | T30 |
| T34 | FE | Frontend deployment (Vercel) 🔗 T33 | T32, T33 |
| T35 | FE | Balancing & playtest pass (joint w/ BE for deltas) | T34 |

## Critical path notes

- **BE dev order:** T01 → T02 → T04 → T05 → T07 → T08 → T09 → T28 → T29 → T30 → T33.
  T28 (rating algorithm) has no FE dependency — start it as soon as T09 ships; the FE
  match engine (T13–T27) takes far longer, so BE will be waiting otherwise.
- **FE dev order:** T03 → T06* → T10 → T11* → T12 → T13 … T27 → T31* → T32* → T34 → T35.
  Starred tickets can begin against mocked responses (shapes defined in T10) and are
  closed when wired to the live dev API.
- **Contract-first rule:** every BE ticket that adds/changes an endpoint must update the
  shared request/response type definitions (see T09/T10) *in the same PR*. The FE never
  guesses shapes.
- **Server authority rule:** attributes, overalls, ratings, and deltas are only ever
  computed server-side. The client sends raw match events; it never sends computed stats.

## Design pillars (apply to every gameplay ticket)

1. **FC27-grade mechanics** — every ball action has: input nuance (tap/hold/modifier),
   stat-driven error model, weak-foot + body-orientation penalties, and a skilled-input
   reward (timed finishing, precision modifiers, skill-move chains).
2. **Two hands on the keyboard** — left hand moves/modifies (WASD + Shift/Ctrl/Q/E/Space),
   right hand executes (J/K/L/I/U/O + arrows). No mechanic is reachable one-handed.
3. **Stats matter visibly** — a 60-rated and an 85-rated player must feel different in
   the first 10 seconds of controlling them.
4. **Difficulty = smarter, not stronger** — AI tiers change reaction time and decision
   quality, never player stats.
