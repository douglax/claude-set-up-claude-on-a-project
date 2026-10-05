# CLAUDE.md

A small Express REST API (users + health check) backed by an in-memory store, used as the starter project for a Claude Code course.

## Commands

- `npm run dev` — start the API with `node --watch` on http://localhost:3000 (port from `PORT` env var)
- `npm test` — run all tests with the built-in `node:test` runner
- `node --test --test-name-pattern="404" tests/users.test.js` — run a single test by name
- `npm run lint` — ESLint (`eslint:recommended`)

CI (`.github/workflows/ci.yml`) runs `npm run lint` then `npm test` on Node 22; both must pass.

## Conventions

- CommonJS only: use `require` / `module.exports`, not `import` / `export` (ESLint is set to `sourceType: "script"`).
- Tests use `node:test` + `node:assert` + `supertest`, not Jest or Mocha. Import `app` from `../server` and call `request(app)` — never start a real server in tests.
- Routes never touch data directly; all reads/writes go through functions exported from `db/store.js`.
- Error responses are JSON shaped `{ error: "<message>" }` with the matching status (400 for invalid input, 404 for missing resources).
- Convert `req.params.id` with `Number(...)` before lookup — store IDs are numbers.
- `.env` is never loaded by the app (no dotenv) and must not be read or committed; `.env.example` documents config.

## Architecture

- `server.js` builds the Express app, mounts each router under its path (`/users`, `/health`), and exports `app`. It only calls `listen()` when run directly (`require.main === module`), which is what lets tests import it.
- `routes/` holds one `express.Router()` file per resource. Adding a resource = new file in `routes/` + `app.use(...)` in `server.js`.
- `db/store.js` is an in-memory array standing in for a database; state resets on restart and is shared across tests in the same process (e.g. a `POST /users` test leaves the user in the list).
