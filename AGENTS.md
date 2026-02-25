# Agentic Instructions

**IMPORTANT: AI agents must strictly follow the instructions in this document.**

## Project Overview

This is a custom GitHub Action that transforms strings using the ROT-13 cipher. It demonstrates best practices for building TypeScript-based GitHub Actions with:

- Bun runtime and tooling
- Clean architecture with dependency injection
- Property-based testing (fast-check)
- Mutation testing (Stryker)

## Task Automation

Use `task <name>` for all common operations. Run `task -a` to list all available tasks.

| Command | Description |
| --------- | ------------- |
| `task install` | Install dependencies |
| `task build` | Build GitHub Action bundle |
| `task clean` | Clean and rebuild artifacts |
| `task test` | Run full test suite |
| `task test:unit` | Run unit tests |
| `task test:watch` | Run tests in watch mode |
| `task test:mutation` | Run mutation tests |
| `task test:build` | Run post-build test |
| `task lint` | Lint the codebase |
| `task format` | Format code with Biome |
| `task docs` | Update documentation |

## Project Structure

```plain
bin/index.ts       → GitHub Actions entry point
src/action.ts      → Main action class with dependency injection
src/rot13.ts       → ROT-13 transformation logic
src/input.ts       → Input validation using Zod
src/types.ts       → TypeScript interfaces
tests/             → Unit and property-based tests
dist/index.js      → Bundled distribution (committed to Git)
```

## GitHub Actions Constraints

When developing GitHub Actions:

1. **Single bundle requirement**: Actions must be a single JS file. Use `task build` to bundle everything.
2. **Node.js runtime**: Actions run on Node.js 24, not Bun. Build output must be Node-compatible.
3. **Use `@actions` packages**: Use `@actions/core` for GitHub Actions API interactions.
4. **Commit `dist/`**: The build output must be committed to the repository.
5. **No build step in CI**: GitHub Actions cannot run install or build when executing your action.

## Build Process

The `task build` command creates a single JavaScript bundle:

- Entry point: `bin/index.ts`
- Output: `dist/index.js` (always committed to Git)
- Target: Node.js 24 runtime
- Format: ESM
- Minification: Enabled (via `--production` flag)

## Dependency Injection Pattern

The action uses interfaces to decouple from GitHub Actions toolkit:

```typescript
interface Core {
  getInput(name: string): string;
  setOutput(name: string, value: unknown): void;
  setFailed(message: string): void;
  info(message: string): void;
}
```

This enables:

- Testing without GitHub Actions environment
- Fast tests with test doubles (see `tests/utils.ts`)
- Clear separation between business logic and side effects

## Development Workflow (TDD)

Follow the TDD cycle:

1. Write a new unit test in `tests/` using test doubles
2. Run `task test:unit` and ensure the new test **fails**
3. Make changes to source code until the test **passes**
4. Refactor while keeping the test passing
5. Run `task test` for the full suite

## Git Hooks

### Pre-commit

```sh
task -p lint build
git add dist README.md
```

The `dist/index.js` bundle must always be committed with source changes.

### Pre-push

```sh
task test
```

Prevents pushing broken code.

## Testing Strategy

### Unit Tests

- Use test doubles pattern (FakeCore) instead of mocking
- Located in `tests/action.test.ts`

### Property-Based Tests

- Located in `tests/rot13.test.ts`
- Verify mathematical properties:
  - Length preservation
  - Idempotency (ROT-13 applied twice returns original)
  - Case preservation
  - Non-alphabetic character preservation

### Mutation Testing

- 100% mutation score threshold required
- Run with `task test:mutation`

## Debugging CI Failures

1. Check that `dist/index.js` is up to date
2. Run `task build` and commit if needed
3. Verify bundle works locally: `bun dist/index.js`
4. Check CI logs: `gh run list` then `gh run view <ID> --log-failed`

## Code Style

- Use Biome for formatting and linting
- Follow conventional commits format
- Keep functions small and focused
- Prefer explicit types over inference for public APIs
