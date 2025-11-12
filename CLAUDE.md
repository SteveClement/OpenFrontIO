# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenFront.io is a real-time strategy game focused on territorial control and alliance building. It's a fork/rewrite of WarFront.io built as a TypeScript monorepo containing:
- **Frontend client**: PIXI.js-based game renderer and UI
- **Backend server**: Node.js/Express WebSocket game server with cluster/worker architecture
- **Shared core**: Deterministic game logic used by both client and server
- **Map generator**: Go-based tool for generating map data

## Development Commands

### Running the Game
- `npm run dev` - Start both client and server with hot reload (recommended for development)
- `npm run start:client` - Run only the client (webpack dev server)
- `npm run start:server-dev` - Run only the server with development settings
- `npm run dev:staging` - Connect to staging API servers
- `npm run dev:prod` - Connect to production API servers (useful for replaying production games)

### Building
- `npm run build-dev` - Build client bundle in development mode
- `npm run build-prod` - Build client bundle in production mode
- `npm run tunnel` - Build production assets and start server for Cloudflare tunnels

### Testing
- `npm test` - Run all Jest tests
- `npm run test:coverage` - Run tests with coverage report
- `npm run perf` - Run performance benchmarks in `tests/perf/`
- Test individual file: `npm test -- path/to/test.test.ts`

### Code Quality
- `npm run format` - Format code with Prettier
- `npm run lint` - Lint code with ESLint
- `npm run lint:fix` - Lint and auto-fix issues

### Map Generation
- `npm run gen-maps` - Run Go map generator and format output (requires Go installed)

## Architecture

### Module Organization

**`src/core/`** - Shared deterministic game logic
- Must be executable on both client and server
- All changes here **require tests**
- No client or server-specific code allowed
- Key modules:
  - `game/` - Core game state (`GameImpl`, `PlayerImpl`, `AllianceImpl`, etc.)
  - `execution/` - Turn-based execution system (all `*Execution.ts` files)
  - `configuration/` - Config system with env-specific overrides
  - `pathfinding/` - A* pathfinding algorithms
  - `validations/` - Input validation logic
  - `Schemas.ts` - Zod schemas for all data structures
  - `GameRunner.ts` - Orchestrates game execution and updates

**`src/client/`** - Frontend application
- Entry point: `Main.ts`
- `graphics/` - PIXI.js rendering components
- `components/` - Lit-based UI components and modals
- `utilities/` - Client-only helper functions
- `Transport.ts` - WebSocket communication with server
- `ClientGameRunner.ts` - Client-side game runner wrapper

**`src/server/`** - Backend services
- Entry point: `Server.ts` (uses Node.js cluster module)
- `Master.ts` - Master process managing workers and load balancing
- `Worker.ts` - Worker processes handling game instances
- Uses WebSocket for real-time communication
- Integrates with external APIs for authentication, storage, etc.

**`tests/`** - Test suites
- Mirrors gameplay domains (alliances, wars, teams, etc.)
- `tests/testdata/` - Test fixtures and shared test data
- `tests/perf/` - Performance benchmarks
- Test naming: `FeatureName.test.ts` mirrors `FeatureName.ts`

**`resources/`** - Static assets
- Images, maps, localization files
- Referenced via webpack loaders

**`map-generator/`** - Go application
- Generates map JSON files from geographic data
- Run via `npm run gen-maps`

### Game Execution Model

The game uses a **deterministic turn-based execution system**:

1. Players submit **Intents** (actions) via WebSocket
2. Server batches intents into **Turns**
3. `GameRunner` processes turns sequentially
4. Each intent creates an **Execution** object (via `ExecutionManager`)
5. Executions modify game state and produce **GameUpdates**
6. Updates are serialized and sent to clients
7. Client `GameRunner` applies same updates to local state

Key files:
- `src/core/Schemas.ts` - Intent and Turn types
- `src/core/execution/ExecutionManager.ts` - Intent → Execution mapping
- `src/core/execution/*Execution.ts` - Individual execution handlers
- `src/core/GameRunner.ts` - Orchestrates execution loop
- `src/core/game/GameUpdates.ts` - Update types and serialization

### Configuration System

Multi-environment config system in `src/core/configuration/`:
- `Config.ts` - Base config interface
- `DefaultConfig.ts` - Default values
- `DevConfig.ts` - Local development overrides
- `PreprodConfig.ts` / `ProdConfig.ts` - Production environments
- `ConfigLoader.ts` - Loads config based on `GAME_ENV` env var

Access config via `getConfig()` or `getServerConfigFromServer()`

### WebSocket Protocol

Client-server communication via `ws` library:
- Client sends intents as JSON messages
- Server broadcasts game updates to all clients
- Worker processes handle individual game rooms
- Master process routes connections to workers

## Testing Guidelines

### Requirements
- **All `src/core/` changes MUST include tests** (per README policy)
- Use fixtures from `tests/testdata/` instead of creating inline mocks
- Align test file names with the system under test

### Running Tests
```bash
npm test                    # All tests
npm test -- Attack.test.ts  # Specific test file
npm run test:coverage       # With coverage report
```

### Test Organization
- Tests live in `tests/` directory (not co-located with source)
- Use Jest with `@swc/jest` for fast TypeScript transpilation
- Coverage thresholds enforced (see `jest.config.ts`)

## Code Style

### Language & Formatting
- TypeScript with ES2020 target
- ESM modules (`type: "module"` in package.json)
- 2-space indentation, semicolons required
- Prettier and ESLint enforced via pre-commit hooks (Husky + lint-staged)

### Naming Conventions
- **PascalCase**: Classes, interfaces, types (`PlayerImpl`, `GameMap`)
- **camelCase**: Functions, variables, properties (`createGame`, `playerID`)
- **UPPER_SNAKE_CASE**: Constants and env vars (`GAME_ENV`, `API_DOMAIN`)

### Import Guidelines
- Keep shared logic in `src/core/` with named exports
- Avoid circular imports between modules
- Use TypeScript path aliases defined in `tsconfig.json`

## Contribution Notes

### Testing Changes
- Run full test suite before committing: `npm test`
- For core logic changes, check coverage: `npm run test:coverage`
- Test in browser for UI changes: `npm run dev`

### Commit Style
Follow existing Git conventions:
- Concise, imperative subjects
- Optional scope prefix (`fix:`, `feat:`)
- Reference PR number in commit message (#XXXX)
- Example: `Let nations send retaliation warships! (#2376)`

### Pull Request Requirements
- PRs should be focused on a single feature/bug
- Link to tracking issue if applicable
- Describe gameplay impact and testing performed
- Include screenshots for UI changes
- All core gameplay changes require tests

### Code Quality Checks
Pre-commit hooks automatically run:
- ESLint with auto-fix
- Prettier formatting
Ensure these pass before pushing.

## Environment Variables

Copy `example.env` to `.env` for local secrets. Key variables:
- `GAME_ENV` - Environment selector (dev/staging/prod)
- `API_DOMAIN` - API server domain for external services
- See `example.env` for complete list

## Build & Deployment

### Local Development
Use `npm run dev` for fastest development loop (concurrent client/server with hot reload)

### Production Build
```bash
npm run build-prod  # Build webpack bundle
npm run start:server  # Start production server
```

### Docker
- `Dockerfile` - Production container definition
- `build.sh` / `build-deploy.sh` - Build scripts
- `deploy.sh` - Deployment automation
- `nginx.conf` - Reverse proxy configuration

### Server Architecture
- Master process spawns worker processes via Node.js `cluster` module
- Each worker runs an Express server with WebSocket support
- Nginx reverse proxy routes traffic to workers
- Cloudflare tunnels for external access (non-dev environments)

## Replaying Production Games

To debug production issues:
1. Get game ID from production
2. Find `gitCommit` value via `https://api.openfront.io/game/[gameId]`
3. Check out that commit: `git checkout <commit>`
4. Connect to production API: `npm run dev:prod`
5. Load the game replay in browser

Note: Unfinished games cannot be replayed on localhost.
