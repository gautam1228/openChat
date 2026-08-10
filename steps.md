# Prettier, ESLint, Husky, and lint-staged setup

This document records each step and command used to configure Prettier with ESLint (`eslint-config-prettier`), plus pre-commit hooks via Husky and lint-staged. The package manager for this repo is **bun**.

---

## 1. Prettier (already present)

Prettier was already added to the project. The following files define its behavior:

### `.prettierrc`

```json
{
    "singleQuote": false,
    "trailingComma": "all",
    "arrowParens": "always",
    "bracketSpacing": true,
    "semi": true,
    "tabWidth": 4
}
```

### `.prettierignore`

Ignores build artifacts, dependencies, lock files, env files, and Next.js output directories (`node_modules`, `.next`, `out`, lock files, `.env*`, etc.).

### `package.json` scripts (added in step 4)

```json
"format": "prettier --write .",
"format:check": "prettier --check ."
```

### Format the codebase

```bash
bun run format
```

### Check formatting without writing

```bash
bun run format:check
```

---

## 2. ESLint + `eslint-config-prettier`

`eslint-config-prettier` turns off ESLint rules that conflict with or duplicate Prettier formatting.

### Install dependency

```bash
bun add -d eslint-config-prettier
```

### Update `eslint.config.mjs`

Import the flat-config export and add it **last** in the config array so it overrides conflicting rules from `eslint-config-next`:

```js
import eslintConfigPrettier from "eslint-config-prettier/flat";

const eslintConfig = defineConfig([
    ...nextVitals,
    ...nextTs,
    globalIgnores([...]),
    eslintConfigPrettier, // must be last
]);
```

### Verify no conflicting rules

```bash
bunx eslint-config-prettier eslint.config.mjs
```

Expected output: `No rules that are unnecessary or conflict with Prettier were found.`

### Run ESLint

```bash
bun run lint
```

### Auto-fix ESLint issues

```bash
bun run lint:fix
```

---

## 3. Husky (pre-commit hooks)

Husky installs Git hooks so checks run automatically before commits.

### Install dependency

Husky was installed together with other dev dependencies:

```bash
bun add -d husky lint-staged eslint-config-prettier
```

(If installing Husky alone: `bun add -d husky`)

### Initialize Husky

```bash
bunx husky init
```

This command:

- Creates the `.husky/` directory
- Adds a `prepare` script to `package.json`: `"prepare": "husky"`
- Creates `.husky/pre-commit` (default content is replaced in step 5)

The `prepare` script runs after `bun install`, keeping Git hooks wired up for all contributors.

---

## 4. lint-staged

lint-staged runs linters/formatters only on **staged** files during the pre-commit hook.

### Install dependency

Included in the same install as above:

```bash
bun add -d lint-staged
```

### Configure in `package.json`

```json
"lint-staged": {
    "*.{js,jsx,ts,tsx,mjs,cjs}": [
        "eslint --fix",
        "prettier --write"
    ],
    "*.{json,css,md,mdx,yml,yaml}": [
        "prettier --write"
    ]
}
```

---

## 5. Pre-commit hook

### Update `.husky/pre-commit`

Replace the default hook content with:

```bash
bunx lint-staged
```

On every commit, staged files are ESLint-fixed and Prettier-formatted before the commit completes.

---

## 6. Final verification

Run all checks manually:

```bash
bun run lint
bun run format:check
bunx eslint-config-prettier eslint.config.mjs
```

Format the repo once so existing files match Prettier rules:

```bash
bun run format
```

---

## Summary of dependencies added

| Package                 | Purpose                                      |
| ----------------------- | -------------------------------------------- |
| `prettier`              | Code formatter (already present)             |
| `eslint-config-prettier`| Disable ESLint rules that conflict with Prettier |
| `husky`                 | Git hooks                                    |
| `lint-staged`           | Run linters on staged files only             |

---

## Quick reference

| Command                  | Description                          |
| ------------------------ | ------------------------------------ |
| `bun run lint`           | Run ESLint on the project            |
| `bun run lint:fix`       | Run ESLint with auto-fix             |
| `bun run format`         | Format all files with Prettier       |
| `bun run format:check`   | Check Prettier formatting (CI-friendly) |
| `bunx lint-staged`       | Run lint-staged manually (same as pre-commit) |
