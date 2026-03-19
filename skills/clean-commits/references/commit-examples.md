# Commit Message Examples

Real-world examples of well-structured conventional commits, organized by type.

## Config/Tooling

```
chore(config): add commitizen, release-it, and pre-commit hooks

Add .cz-config.cjs, .release-it.js, and commitlint commit-msg hook
for conventional commit workflow. Expand pre-commit hook with tsc,
frontend tests, cargo fmt, and clippy checks.
```

```
chore(config): add workspace-scoped cargo scripts and dev tooling

Update package.json cargo commands from single-crate to --workspace.
Add .nvmrc, .vscode/extensions.json, and vitest passWithNoTests.
```

## Dependencies

```
fix(deps): update Tauri NPM packages to match Rust crate versions

Align @tauri-apps/api, plugin-log, and plugin-updater versions with
their Rust counterparts to fix version mismatch build errors.
```

```
chore(deps): update dependencies to latest versions
```

## CI/CD

```
ci(ci): rewrite release workflow with prepare stage and multi-platform builds

Rename release-cli.yml to release.yml. Add prepare job supporting both
release-event and manual workflow_dispatch triggers. Split Linux builds
into Ubuntu 22.04 (deb) and 24.04 (AppImage + RPM).
```

```
ci(ci): refactor workflows with reusable composite actions

Extract setup steps into .github/actions/ for consistent toolchain
setup across CI and release workflows.
```

## Refactoring

```
refactor(providers): apply cargo fmt and use Rust 2024 let chains

Reformat all provider crate code with cargo fmt and replace nested
if-let blocks with Rust 2024 let chains for cleaner control flow.
```

```
refactor(app): flatten match arm closures in tray manager
```

## Features

```
feat(providers): add support for Gemini API quota monitoring
```

```
feat(ui): add dark mode toggle with system preference detection
```

## Bug Fixes

```
fix(auth): resolve cookie extraction on Windows Edge browser

The cookie path was using Chrome's registry key instead of Edge's,
causing silent authentication failures.
```

## Documentation

```
docs(docs): add DEVELOPMENT.md and update README with fork attribution

Add comprehensive development guide covering scripts, hooks, versioning,
and CI/CD documentation. Update README with correct release URL.
```

## Style

```
style(providers): alphabetize module declarations in mod.rs
```

## Tests

```
test(providers): add regression tests for cookie extraction edge cases
```

## Performance

```
perf(providers): cache provider responses for 5 minutes
```
