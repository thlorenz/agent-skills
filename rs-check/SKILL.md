---
name: rs-check
description: Formats, lints, and tests Rust projects with cargo. Runs cargo fmt, cargo clippy, and cargo nextest with optional error fixing. Use for Rust code quality checks and testing.
---

# Rust Check Skill

Automated code formatting, linting, and testing for Rust projects.

## Overview

Runs three sequential checks on Rust code:
1. **Format** - Ensures code style consistency
2. **Lint** - Checks for warnings and anti-patterns with clippy
3. **Test** - Runs tests with nextest for parallel execution

## Usage

When this skill loads, immediately run format, lint, and test with default mode (fix-lint). Do not ask the user for input.

### Options (optional, user may specify)

- **fix-lint**: Fix cargo clippy errors automatically (default mode)
- **fix-tests**: Attempt to fix test failures automatically
- **no-fix**: Don't fix any errors, only log them

If no option is specified, use `fix-lint` (fixes lint errors but not tests).

## Workflow

### 1. Format

- If `rustfmt-nightly.toml` exists in root: `cargo +nightly fmt -- --config-path rustfmt-nightly.toml`
- Otherwise: `cargo fmt`

### 2. Lint

- `cargo clippy --all-targets -- -D warnings`
- With `fix-lint`: `cargo clippy --fix --allow-dirty --allow-staged`
- With `no-fix`: Reports warnings only

### 3. Test

- `cargo nextest run -j16` (16 parallel jobs)
- For single packages: `cargo nextest run -p <package-name>`
- With `fix-tests`: Attempts common fixes (e.g., updating dependencies, clearing cache)
- With `no-fix`: Reports failures only

## Notes

- Format always runs first regardless of options
- Lint and test failures are logged but don't halt the skill
- Use `fix-tests` carefully as automatic fixes may not resolve root causes
- For package-specific testing, manually invoke: `cargo nextest run -p <package-name>`

## Non Rust

The checks above are tailored to Rust projects. For other projects substitute the commands with
appropriate equivalents.

You should determine this once before writing the plan and then use the same substitues
throughout the plan.

### TypeScript/NextJS/JavaScript

Look for yarn commands in package.json first and use them if found, otherwise fall back to
standard commands:

1. **Format**:: `yarn fmt` or `yarn format` or `prettier --write .`
2. **Lint**: `yarn lint` or `eslint . --fix`
3. **Test**: `yarn test` or do nothing if not defined
