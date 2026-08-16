# T33 — Backend deployment (Render/Railway)

**Repo:** footy-fc-api · **Depends on:** T30 · **Blocks:** T34 (FE)

## Description
Ship the API to production and lock it down.

## Tasks
- [ ] Deploy to Render or Railway from the GitHub repo (build: `npm run build`,
      start: `npm start`); set `PORT`, `MONGO_URI`, `JWT_SECRET` (freshly generated,
      not the dev secret), `CORS_ORIGIN` (placeholder until T34 gives the Vercel URL)
- [ ] Atlas: replace `0.0.0.0/0` network access with the host's egress IPs
      (or keep 0.0.0.0/0 only if the host has no static egress — document the decision)
- [ ] Production hardening checklist: `NODE_ENV=production`, helmet on, rate limits on,
      error handler never leaks stacks, request logging (morgan or pino) with no
      PII/token logging
- [ ] `/health` used as the platform health check
- [ ] Smoke script (`scripts/smoke.sh`): register → create team → PATCH lineup →
      POST a valid match → GET history, against the prod URL

## Acceptance criteria
- Smoke script passes against the production URL
- Requests from a non-allowed origin are CORS-rejected once T34 sets the final origin
- Dev JWT tokens are invalid in prod (different secret)
