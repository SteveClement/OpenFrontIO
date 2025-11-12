# Repository Guidelines

## Project Structure & Module Organization
- `src/client` hosts the PIXI-based front-end client, while `src/core` carries shared combat, economy, and config logic that both client and server import.
- `src/server` is the Node/TypeScript backend entry point; `startup.sh`, `build.sh`, and `Dockerfile` orchestrate deployment-ready builds.
- `tests` mirrors gameplay domains (alliances, wars, teams) plus `tests/client` and `tests/core`; fixtures live under `tests/testdata`.
- `resources/` stores static assets and localization files, and `map-generator/` (Go) creates map JSON via `npm run gen-maps`.
- Copy `example.env` to `.env` for local secrets; scripts read runtime values via `GAME_ENV` and `API_DOMAIN`.

## Build, Test, and Development Commands
- `npm run dev` launches concurrent client/server dev loops with hot reload.
- `npm run start:client` or `npm run start:server-dev` run each side independently; use `npm run dev:staging` or `npm run dev:prod` to point at remote APIs.
- `npm run build-dev` / `npm run build-prod` emit webpack bundles; `npm run tunnel` builds prod assets and boots the server for Cloudflare tunnels.
- `npm run gen-maps` executes the Go generator and formats the output.

## Coding Style & Naming Conventions
- TypeScript is standard with 2-space indentation and semicolons; server code uses modern ESM imports (`type: module`).
- Prefer PascalCase for classes/components (e.g., `PlayerImpl`), camelCase for functions/vars, and UPPER_SNAKE for env constants.
- ESLint (`npm run lint`) and Prettier (`npm run format`) enforce formatting; Husky + lint-staged auto-run them on staged files.
- Keep shared logic inside `src/core` and export named utilities to avoid circular imports.

## Testing Guidelines
- Primary tests are Jest suites ending in `.test.ts` inside `tests/`; align file names with the system under test (`Attack.test.ts` mirrors `Attack.ts`).
- Run `npm test` for the default suite, `npm run test:coverage` when touching server/core logic, and `npm run perf` for benchmark harnesses in `tests/perf`.
- Core gameplay changes are required to ship with tests (per README); add fixtures to `tests/testdata` instead of inventing inline mocks.
- Document new test helpers in `tests/global.d.ts` when they add globals.

## Commit & Pull Request Guidelines
- Follow the existing Git style: concise, imperative subjects optionally prefixed with a scope (`fix:`) and suffixed with the PR number (e.g., `Let nations send retaliation warships! (#2376)`).
- Each PR should link the tracking issue, describe the gameplay impact, list manual/automated test commands, and attach screenshots or replays for UI/gameplay shifts.
- Keep PRs focused; maintainers expect lint and unit tests to pass locally before requesting review.
