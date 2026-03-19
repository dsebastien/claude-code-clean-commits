---
name: clean-commits
description: Organizes all local git changes into clean, logical conventional commits grouped by concern. This skill should be used when there are uncommitted local changes that need to be committed following the project's conventional commit format. Dynamically reads commit scopes and types from config files, groups changes by purpose, and creates commits in dependency order using hunk-level staging when needed.
disable-model-invocation: true
allowed-tools: Read, Edit, Bash, Glob, Grep
---

# Clean Commits

Analyze all local git changes, group them into logical conventional commits by concern, and create them in the right order — respecting the project's commit types, scopes, and pre-commit hooks. Use hunk-level staging when a single file contains changes belonging to different commit groups.

## Critical Rules

1. **Never use `--no-verify`** — pre-commit hooks must run and pass for every commit
2. **Never stage with `git add .` or `git add -A`** — always stage specific files or hunks
3. **Read before grouping** — inspect actual diffs, not just filenames; understand what changed and why
4. **Config/tooling commits first** — changes to build config, hooks, or CI must be committed before code that depends on them
5. **One concern per commit** — each commit should be reviewable in isolation
6. **Use HEREDOC for commit messages** — ensures correct formatting with multi-line bodies
7. **Verify clean tree at end** — run `git status` after all commits to confirm nothing was missed
8. **Prefer hunk staging over whole-file staging** when a file mixes concerns — split with patch files

## Workflow

### Step 1: Discover Project Conventions

Read the project's conventional commit configuration to extract the allowed **types** and **scopes**. Check these sources in priority order — stop at the first one that provides scopes:

#### Source 1: Dedicated scopes file

```bash
# Check for conventional-commit-scopes.cjs (exports a plain string array)
```

Format: `module.exports = ['app', 'providers', 'auth', 'ui', ...]`

Read the file directly — the exported array IS the scope list.

#### Source 2: Commitizen config

```bash
# Check for .cz-config.cjs
```

Format: `scopes: [{ name: 'app' }, { name: 'ui' }, ...]`

Extract the `name` values from the `scopes` array. Also check the `types` array for allowed commit types. Note: some repos import scopes from `conventional-commit-scopes.cjs` via `require()` — in that case, read the referenced file instead.

#### Source 3: Commitlint config

```bash
# Check for commitlint.config.ts, commitlint.config.js, or commitlint.config.cjs
```

Format: `'scope-enum': [2, 'always', ['app', 'ui', 'deps', ...]]`

Extract the string array from the `scope-enum` rule (third element of the tuple). Note: some configs import scopes via `require('./conventional-commit-scopes.cjs')` — in that case, read the referenced file instead.

#### Fallback

If no config is found, use standard conventional commit types (feat, fix, chore, refactor, docs, test, perf, build, ci, style, revert) with no scope restriction.

**After extraction, print the discovered scopes** to confirm before proceeding:
```
Discovered scopes: app, providers, auth, tray, ui, settings, cli, ci, deps, docs, config
```

Use ONLY these scopes in commit messages. If a change doesn't fit any configured scope, omit the scope rather than inventing one (unless `allowCustomScopes: true` is set in `.cz-config.cjs`).

### Step 2: Analyze All Changes

Run these commands in parallel to understand the full picture:

```bash
git status                    # overview of staged, unstaged, untracked
git diff --stat               # summary of unstaged changes
git diff --cached --stat      # summary of staged changes
git log --oneline -5          # recent commit style reference
```

Then inspect actual diffs to understand the nature of changes. For large changesets, use `git diff --stat` to prioritize, then read diffs selectively.

**Key question for each changed file:** What is the *purpose* of this change — new feature, bug fix, formatting, config, CI, docs, deps, refactoring?

**Key question for mixed-purpose files:** Does this file contain hunks belonging to different commit groups? If yes, mark it for hunk-level staging.

### Step 3: Group Changes by Concern

Assign each changed file (or hunk) to a logical commit group. Common groupings:

| Group | Type | Scope | Examples |
|-------|------|-------|----------|
| Build/tooling config | `chore` | `config` | package.json scripts, tsconfig, eslint, vite, vitest, .gitignore, hooks |
| Dependencies | `chore` or `fix` | `deps` | package.json version bumps, lock files |
| CI/CD | `ci` | `ci` | .github/workflows/, .github/actions/ |
| Code formatting | `style` | varies | cargo fmt, prettier-only changes |
| Refactoring | `refactor` | varies | structural improvements, let chains, deduplication |
| Bug fixes | `fix` | varies | behavioral corrections |
| New features | `feat` | varies | new functionality |
| Tests | `test` | varies | new or updated test files |
| Documentation | `docs` | `docs` | README, DEVELOPMENT.md, CLAUDE.md, AGENTS.md |

**Grouping principles:**
- When a file has hunks serving different purposes, split at hunk level (see Step 5)
- Keep lock files (bun.lock, Cargo.lock) with the commit that changed their source (package.json, Cargo.toml)
- Group by logical concern, not by directory
- When changes are purely formatting (cargo fmt, prettier), group by language/toolchain

### Step 4: Determine Commit Order

Order commits to avoid pre-commit hook failures and maintain logical progression:

1. **Config/tooling** — hook changes, build config, test config (fixes hooks for subsequent commits)
2. **Dependencies** — version bumps, new packages
3. **CI/CD** — workflow changes
4. **Formatting/style** — pure formatting commits
5. **Refactoring** — structural improvements
6. **Bug fixes** — behavioral corrections
7. **Features** — new functionality
8. **Tests** — new or updated tests
9. **Documentation** — docs, README, guides

If a commit would fail the pre-commit hook because a later commit fixes the issue, reorder to commit the fix first.

### Step 5: Stage and Create Commits

For each group, in the determined order:

1. **Reset staging area** if needed: `git reset HEAD` (only if prior staging exists)
2. **Stage files or hunks** (see staging strategies below)
3. **Commit with HEREDOC format**:

```bash
git commit -m "$(cat <<'EOF'
type(scope): concise imperative description

Optional body explaining why, not what. Keep to 1-2 sentences
when the diff isn't self-explanatory.

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

#### Staging Strategy: Whole Files

When all changes in a file belong to the same commit group, stage the entire file:

```bash
git add file1.rs file2.ts file3.json
```

#### Staging Strategy: Hunk-Level (for mixed-purpose files)

When a file contains changes belonging to different commit groups, use patch-based staging to split them precisely. This avoids including unrelated changes in a commit.

**Method: Extract and apply specific hunks**

```bash
# 1. Extract the full diff for the file
git diff path/to/file.ts > /tmp/full.patch

# 2. Edit the patch to keep only the hunks for this commit group
#    Remove unwanted @@ hunk sections, keeping the file header intact

# 3. Stage only those hunks
git apply --cached /tmp/partial.patch

# 4. Verify what was staged vs what remains
git diff --cached -- path/to/file.ts   # staged hunks
git diff -- path/to/file.ts            # remaining hunks
```

**When to use hunk staging:**
- A file has both formatting changes AND logic changes
- package.json has script additions AND dependency version bumps for different commits
- A source file has a bug fix AND an unrelated refactor
- Config files with changes serving different purposes

**When NOT to use hunk staging (prefer whole-file):**
- All changes in the file serve the same purpose
- The file is new (untracked) or deleted
- Hunks are interleaved and can't be cleanly separated
- The overhead of splitting outweighs the benefit (small files with 1-2 hunks)

**Practical shortcut:** If only a few lines need exclusion, consider staging the whole file and noting the mixed nature in the commit body rather than splitting a trivial diff.

#### Commit Message Rules

- Subject line: `type(scope): imperative description` (max ~72 chars)
- Use present tense imperative: "add", "fix", "update", not "added", "fixes", "updated"
- Scope is optional but preferred when a clear scope exists
- Body only when the "why" isn't obvious from the diff
- Always include Co-Authored-By trailer

### Step 6: Handle Pre-Commit Hook Failures

If a commit fails due to the pre-commit hook:

1. **Read the error output** — identify which check failed (tsc, eslint, clippy, tests, etc.)
2. **Diagnose the root cause** — is it a real error in the staged code, or a working-tree issue?
3. **Fix the issue** — edit the file, then re-stage and create a **new** commit (never amend)
4. **If the fix belongs in a different commit group** — reorder: commit the fix group first
5. **Never skip the hook** — if the hook fails, the code has a problem that needs fixing

### Step 7: Verify

After all commits are created:

```bash
git status                    # must show clean working tree
git log --oneline -N          # show the N new commits for review
```

Report the final commit list to the user with a summary table.

## Edge Cases

- **Mixed formatting + logic changes in one file**: Use hunk staging to separate them. If hunks are interleaved, keep the file in the logic-change commit and note the formatting in the body.
- **Rename + modify**: Use `git add` for both the old and new paths so git detects the rename. Group with the commit that explains why the rename happened.
- **Single-file changes**: A single commit is fine — don't force artificial splits.
- **Already-staged files**: Run `git reset HEAD` first to start from a clean staging area.
- **Lock files with mixed sources**: When bun.lock or Cargo.lock reflects changes from multiple commits, include it with the first commit that modifies its source file.

## Commit Message Examples

See [references/commit-examples.md](references/commit-examples.md) for real-world examples.
