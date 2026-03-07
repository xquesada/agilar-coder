# Node.js / TypeScript Stack

## Tech Stack

- **Runtime:** Node.js
- **Language:** TypeScript (or JavaScript)
- **Package manager:** npm (or pnpm / yarn)

## Test Commands

```bash
# Unit tests
npm test

# Specific test file
npm test -- --testPathPattern="path/to/test"

# Watch mode (development)
npm test -- --watch
```

## Lint Commands

```bash
# ESLint
npx eslint . --ext .ts,.tsx,.js,.jsx

# Prettier (check)
npx prettier --check .

# Prettier (fix)
npx prettier --write .

# TypeScript type check
npx tsc --noEmit
```

## Build Commands

```bash
npm run build
```

## Recommended Configuration

### warnings-as-errors

```json
// .eslintrc or eslint.config.js
{
  "rules": {
    "no-unused-vars": "error",
    "no-undef": "error"
  }
}
```

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

### Test framework

Jest or Vitest recommended. Both support TypeScript with minimal configuration.

### BDD framework

Cucumber.js with `@cucumber/cucumber`. Feature files in `features/`, step definitions in `features/steps/`.

### Pre-commit hook

```json
// package.json
{
  "scripts": {
    "precommit": "npm run lint && npm test"
  }
}
```

Or use `husky` + `lint-staged` for faster pre-commit checks on changed files only.
