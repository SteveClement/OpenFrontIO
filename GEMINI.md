# Project Overview

This is a real-time strategy game called OpenFront.io. It is a fork/rewrite of WarFront.io. The game is focused on territorial control and alliance building.

The project is a monorepo containing:
- A frontend game client built with TypeScript and Pixi.js.
- A backend game server built with Node.js, Express, and TypeScript.
- A map generator written in Go.

The client and server share core game logic and type definitions from the `src/core` directory.

# Building and Running

## Prerequisites

- [npm](https://www.npmjs.com/) (v10.9.2 or higher)
- [Go](https://go.dev/)

## Installation

```bash
npm i
```

## Running the Game

### Development Mode

Run both the client and server in development mode with live reloading:

```bash
npm run dev
```

### Client Only

To run just the client with hot reloading:

```bash
npm run start:client
```

### Server Only

To run just the server with development settings:

```bash
npm run start:server-dev
```

### Map Generation

To generate maps:

```bash
npm run gen-maps
```

## Testing

Run unit tests:

```bash
npm test
```

# Development Conventions

## Code Style

The project uses Prettier for code formatting and ESLint for linting.

- **Format code**: `npm run format`
- **Lint code**: `npm run lint`
- **Lint and fix code**: `npm run lint:fix`

## Git

The project uses `husky` and `lint-staged` to run linters on pre-commit.

## Testing

All code changes in `src/core` must be tested.
