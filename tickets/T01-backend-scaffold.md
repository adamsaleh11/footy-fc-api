# T01 — Backend scaffold (Express + TS + tooling)

**Repo:** footy-fc-api · **Depends on:** — · **Blocks:** T02

## Description
Stand up the backend project skeleton so every later ticket has a consistent place to
put code: TypeScript Express server, dev tooling, folder conventions, env handling.

## Tasks
- [ ] `npm init -y`; install `express`, `cors`, `helmet`, `dotenv`, `zod`
- [ ] Dev deps: `typescript`, `ts-node-dev`, `@types/express`, `@types/cors`,
      `eslint` + `@typescript-eslint`, `prettier`, `vitest`, `supertest`
- [ ] `tsconfig.json`: `strict: true`, `esModuleInterop`, `outDir: dist`, path alias `@/* → src/*`
- [ ] Folder structure:
      ```
      src/
        config/      # env parsing (zod-validated), constants
        routes/      # express routers only — no logic
        controllers/ # req/res handling, calls services
        services/    # business logic, pure where possible
        models/      # mongoose schemas
        middleware/  # auth, validation, error handler, rate limit
        utils/
        index.ts     # bootstrap
      ```
- [ ] `src/config/env.ts`: zod-parse `PORT`, `MONGO_URI`, `JWT_SECRET`, `CORS_ORIGIN`;
      crash on boot with a readable message if any is missing/malformed
- [ ] Global middleware: `helmet`, `cors` (origin from env), `express.json({ limit: '1mb' })`
- [ ] Central error handler middleware: known `AppError` → status + JSON;
      unknown → 500 + logged stack, generic body
- [ ] `GET /health` → `{ status: "ok" }`
- [ ] Scripts: `dev` (ts-node-dev), `build` (tsc), `start` (node dist), `test` (vitest), `lint`
- [ ] `.env.example` with all keys; `.gitignore` (node_modules, dist, .env)

## Acceptance criteria
- `npm run dev` boots; `GET /health` returns 200 `{ status: "ok" }`
- Missing `JWT_SECRET` in `.env` → process exits with clear error, not a runtime crash later
- `npm run lint` and `npm test` pass (one smoke test hitting `/health` via supertest)
