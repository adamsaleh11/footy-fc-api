# T02 — MongoDB Atlas + Mongoose connection

**Repo:** footy-fc-api · **Depends on:** T01 · **Blocks:** T04, T07

## Description
Provision the Atlas cluster and wire Mongoose into server boot with fail-fast behavior
and health reporting. (Merges original backlog T3 + T4.)

## Tasks
- [ ] Atlas: new project → **M0 free cluster**, region closest to intended backend host
      (e.g. AWS us-east-1)
- [ ] Database Access: DB user with password auth, `readWriteAnyDatabase` (dev)
- [ ] Network Access: `0.0.0.0/0` for dev — **leave a TODO to tighten in T33**
- [ ] Connection string (Drivers → Node.js), DB name `soccergame`, stored as `MONGO_URI`
- [ ] Install `mongoose`; `src/config/db.ts`:
      - connect on boot **before** listening; retry 5× with backoff, then exit(1)
      - log connected host/db name (never the credential string)
      - listeners for `disconnected` / `reconnected`
- [ ] Extend `/health` → `{ status: "ok", db: "connected" | "disconnected" }`
- [ ] Vitest setup: `mongodb-memory-server` for model/unit tests so CI never needs Atlas

## Acceptance criteria
- Server boot logs successful Atlas connection; `/health` shows `db: "connected"`
- Bad `MONGO_URI` → retries then clean exit with readable error
- Test suite runs green against in-memory Mongo with no network access
