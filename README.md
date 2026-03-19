# clean-commits

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that organizes local git changes into clean, logical conventional commits grouped by concern.

## What it does

When you have a pile of local changes across many files, `/clean-commits` analyzes the diffs, groups them into logical commits (config, CI, refactoring, features, docs, etc.), and creates them in the right order — respecting your project's conventional commit types, scopes, and pre-commit hooks.

### Key features

- **Discovers project conventions** — reads commit scopes and types from `conventional-commit-scopes.cjs`, `.cz-config.cjs`, or `commitlint.config.ts/.js/.cjs`
- **Groups by concern** — config, deps, CI, formatting, refactoring, fixes, features, tests, docs
- **Strategic commit ordering** — config/tooling first so pre-commit hooks work for subsequent commits
- **Hunk-level staging** — splits mixed-purpose files into separate commits using patch-based staging
- **Pre-commit hook handling** — diagnoses and fixes hook failures, never skips them
- **Clean conventional commits** — proper `type(scope): description` format with bodies when needed

## Installation

### From GitHub (recommended)

Add to your Claude Code settings:

```bash
claude plugins add dsebastien/claude-code-clean-commits
```

Or add manually to `~/.claude/settings.json`:

```json
{
    "enabledPlugins": {
        "clean-commits@dsebastien/claude-code-clean-commits": true
    }
}
```

### Local development

```bash
claude --plugin-dir /path/to/claude-code-clean-commits
```

## Usage

In any git repository with uncommitted changes:

```
/clean-commits
```

The skill will:

1. Read your project's conventional commit configuration (scopes, types)
2. Analyze all local changes (staged, unstaged, untracked)
3. Group changes into logical commits by concern
4. Create commits in the right order with proper messages

### Example output

```
Discovered scopes: app, providers, auth, tray, ui, settings, cli, ci, deps, docs, config

Creating 5 commits:

| # | Type | Message |
|---|------|---------|
| 1 | chore(config) | add dev tooling, release pipeline, and update deps |
| 2 | ci(ci) | rewrite release workflow with composite actions |
| 3 | refactor(providers) | apply cargo fmt and use Rust 2024 let chains |
| 4 | refactor(app) | apply cargo fmt and use let chains in Tauri app |
| 5 | docs(docs) | add DEVELOPMENT.md, CLAUDE.md, and update README |
```

### Supported conventional commit configs

The plugin automatically detects scopes from:

| Config file | Format |
|-------------|--------|
| `conventional-commit-scopes.cjs` | `module.exports = ['app', 'ui', ...]` |
| `.cz-config.cjs` | `scopes: [{ name: 'app' }, ...]` |
| `commitlint.config.ts/js/cjs` | `'scope-enum': [2, 'always', ['app', ...]]` |

Falls back to standard conventional commits if no config is found.

## License

MIT
