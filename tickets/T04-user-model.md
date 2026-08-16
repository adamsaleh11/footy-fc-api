# T04 — User model

**Repo:** footy-fc-api · **Depends on:** T02 · **Blocks:** T05

## Description
Mongoose `User` model with secure password storage.

## Tasks
- [ ] Schema: `email` (unique, lowercase, indexed, validated), `username`
      (unique, 3–20 chars, alphanumeric + underscore), `passwordHash`, `createdAt`
- [ ] `passwordHash` has `select: false` — never returned by default queries
- [ ] Pre-save hook: bcrypt (cost 10) hash when password modified; virtual `password`
      setter or explicit `setPassword()` so plaintext never sits on the doc
- [ ] Instance method `comparePassword(plain): Promise<boolean>`
- [ ] `toJSON` transform strips `passwordHash` and `__v`

## Acceptance criteria
- Unit tests: create user → stored hash ≠ plaintext; `comparePassword` true/false paths
- Duplicate email or username → mongoose unique error surfaced as 409 by error handler
- Fetching a user via any normal query never includes `passwordHash`
