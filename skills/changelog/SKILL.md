---
name: changelog
description: Generate release notes (a changelog entry) in English for any git repository, grouped by change type from Conventional Commits. Use when asked to write, generate or update a changelog or release notes for a tag, a version, a branch or a commit range.
argument-hint: "[submodule-path] [range, e.g. v1.3.0..v1.4.0 | version] [--write]"
---

## Goal

Produce clear, reviewer-friendly release notes in English, grouped by change type.

## Scope

- Applies to repository releases (tags) and to major feature branches that need aggregated notes.
- Works in any repository; uses Conventional Commits style inputs where available.

## Arguments

Arguments are optional and may come in any order:

- **Submodule path or name** (e.g. `packages/api`) — the repository to generate the changelog for.
- **Range** — `A..B`, a single tag, or a version. A single tag/version means "from the previous tag to it".
- **`--write`** — save the result into the target repository's `CHANGELOG.md`; without it, only print the entry.

Anything not given is inferred as described below; ask the user only when the inference is genuinely ambiguous.

## Choosing the target repository

The changelog is usually made for a **submodule** of a monorepo, not for the outer repository.

1. Resolve the target:
   - If an argument matches a path or name in `git submodule status` / `.gitmodules`, use that submodule.
   - Otherwise, if the current directory is inside a submodule (`git rev-parse --show-superproject-working-tree` is non-empty), use the current repository.
   - Otherwise, if `.gitmodules` exists, list the submodules (with `git submodule foreach --quiet 'echo $displaypath'`) and ask which one(s) to cover. Several can be processed in one go, each producing its own entry.
   - Otherwise, use the current repository.
2. Run **every** git command against the target with `git -C <path> ...`, and never mix in the outer repository's history. Tags, remote and `CHANGELOG.md` all belong to the submodule.
3. If the submodule is not initialized (empty directory), tell the user to run `git submodule update --init <path>` instead of guessing.
4. The superproject's pointer bump (e.g. `Update submodule X`) is not a change of the submodule; ignore it.

## Collecting commits

Let `T` be the target path (`.` when it is the current repository).

1. Determine the range. If the user gave one, use it. Otherwise use the latest tag up to `HEAD`:
   ```bash
   git -C T describe --tags --abbrev=0                 # latest tag (the previous release)
   git -C T log <tag>..HEAD --no-merges --format='%h%x09%s%n%b%x00'
   ```
   If `HEAD` itself is the tagged release, use the tag before it (`git -C T describe --tags --abbrev=0 HEAD^`).
   If there is no tag yet, use the whole branch (`git -C T log --no-merges`).
2. Version and date: take them from the tag/argument; for unreleased work use `Unreleased` and today's date.
3. Detect the remote for comparison links: `git -C T remote get-url origin` (convert SSH URLs to `https://github.com/<org>/<repo>`). Note that each submodule has its own remote.
4. PR/issue numbers: pick them from `(#123)` in subjects and from `Merge pull request #123` merge commits (use `git -C T log --merges` for that purpose only).
5. If `T/CHANGELOG.md` exists, follow its existing style (heading level, dash style, date format) and prepend the new entry below the title; otherwise use the template below. Only touch the file when `--write` was passed or the user confirms.

## Multiple submodules

When several submodules are requested, output one block per submodule, headed by its name (`# <submodule>`), each with its own version, range and compare link. Do not merge entries across submodules.

## Sections and order

List sections only if non-empty, in this order:

1. Breaking Changes
2. Features
3. Fixes
4. Performance
5. Documentation
6. Refactoring
7. Chore / Maintenance
8. Dependency Updates

## Entry format

- Use a single bullet per change. Keep it concise and start with a verb in past tense.
- If a PR number exists, append it as (#123). If an issue number exists, append it as closes #123 when applicable.
- If there is a BREAKING CHANGE footer in a commit, surface it under Breaking Changes with a one-line impact note.

### Examples

Features
- added healthcheck and documented PGDATA initialization
- introduced localtunnel Dockerfile and usage guide

Fixes
- corrected default port in README and sample env
- ensured Windows PowerShell commands in scripts work on WSL

Breaking Changes
- changed default volume layout; requires re-creating the data volume for existing users.

Dependency Updates
- bumped typescript to 5.5.x and eslint to 9.x

## How to compile the changelog

1. Collect commits since the previous tag (or from the start of the branch if first release).
2. Group by type and scope based on commit messages (Conventional Commits: feat, fix, perf, docs, refactor, chore, deps).
3. Exclude version-bump commits (e.g. "bump version", "release vX.X.X") from the changelog.
4. Promote commits with BREAKING CHANGE footers (or `!` after the type) into "Breaking Changes" with a concise impact statement.
5. De-duplicate entries when multiple commits contribute to the same user-visible change; keep the highest-level description.
6. Ensure language is clear and consistent; avoid internal-only jargon.

## Version header format

- Use the tag or release name as the heading, e.g. `v1.4.0 — 2025-09-30`.
- Optionally include a comparison link: `https://github.com/<org>/<repo>/compare/v1.3.0...v1.4.0`

## Template

```markdown
## vX.Y.Z — YYYY-MM-DD

### Breaking Changes
- impact note

### Features
- summary

### Fixes
- summary

### Performance
- summary

### Documentation
- summary

### Refactoring
- summary

### Chore / Maintenance
- summary

### Dependency Updates
- summary
```

## Quality checklist

- Entries are in English and concise.
- No empty section headers.
- Any breaking impacts are clearly called out.
- Comparison link (if applicable) is valid.