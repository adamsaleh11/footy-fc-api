# T05 — Auth endpoints + JWT middleware

**Repo:** footy-fc-api · **Depends on:** T04 · **Blocks:** T06 (FE), T09

## Description
Register/login issuing 7-day JWTs, plus the auth middleware every protected route uses.

## Endpoints
| Method | Path | Body | Response |
|---|---|---|---|
| POST | `/auth/register` | `{ email, username, password }` | `201 { token, user }` |
| POST | `/auth/login` | `{ email, password }` | `200 { token, user }` |
| GET  | `/auth/me` | — (Bearer) | `200 { user }` |

## Tasks
- [ ] Zod schemas for both bodies (password ≥ 8 chars); validation middleware returns
      400 with field-level errors
- [ ] JWT: payload `{ sub: userId }`, 7d expiry, HS256 with `JWT_SECRET`
- [ ] Login failure (unknown email OR bad password) → identical 401 message
      (no account enumeration)
- [ ] `requireAuth` middleware: parses `Authorization: Bearer`, verifies, loads user,
      attaches `req.user`; 401 on missing/expired/garbage token
- [ ] Rate limit `/auth/*`: 10 req/min/IP (`express-rate-limit`)
- [ ] Update shared API types file (see T09 contract rule) with request/response shapes

## Acceptance criteria
- Supertest flow: register → login → `GET /auth/me` returns the user
- Wrong password → 401; expired/malformed token on protected route → 401
- 11th login attempt in a minute → 429
