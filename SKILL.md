---
name: release-notes
description: >
  Generate release notes and update CHANGELOG with semantic versioning.
  Bumps version in pyproject.toml, generates structured release notes
  from git log, updates CHANGELOG.md in keepachangelog format.
  Trigger on: "create release", "bump version", "tag vX.Y.Z",
  "generate changelog", "release notes", "cut a release", "/release",
  "new version", semantic versioning, or any mention of tagging/releasing.
  Auto-triggers when user asks about versions, tags, or changelogs.
---

# Release Notes Generator

Generate a structured release with semantic versioning. Workflow:
read git log since last tag, categorize commits, bump version,
create/update CHANGELOG.md, output release summary + git/deploy commands.

## Workflow

### 1. Read git context

- Find the latest tag: `git describe --tags --abbrev=0` (fallback: initial state)
- Gather commits since that tag: `git log <tag>..HEAD --oneline --no-decorate`
- Parse each commit as `<type>(<scope>): <description>`
- Categorize by type:
  - `feat` or `feature` -> **Added**
  - `fix` -> **Fixed**
  - `update` -> **Updated**
  - `refactor` -> **Changed**
  - `perf` -> **Performance**
  - `chore`, `docs`, `style`, `build`, `ci`, `test`, `revert` -> **Other**
  - `!` suffix or `BREAKING CHANGE` footer -> **Breaking Changes** (cross-category)

### 2. Determine next version

Use the current version from `pyproject.toml` (field `version`).
Strip leading `v` for comparison, add it back for git tags.
Semver bump rules:

- Breaking Changes -> **major** (X+1.0.0)
- Any `feat` -> **minor** (0.X+1.0)
- Only fix/update/refactor/chore/docs -> **patch** (0.0.X+1)

If the user specifies an explicit version (e.g. "tag v2.0.0"), use that instead.

### 3. Update pyproject.toml

Read the current `version = "x.y.z"` line. Replace with the new version.
Keep all other content unchanged.

### 4. Generate CHANGELOG.md

If no CHANGELOG.md exists, create one with header `# Changelog`.
Format (keepachangelog.com, one section per version):

```markdown
# Changelog

## [v1.2.3] - YYYY-MM-DD

### Added
- feat(scope): description (#hash)

### Fixed
- fix(scope): description (#hash)

### Updated
- update(scope): description (#hash)

### Changed
- refactor(scope): description (#hash)

### Performance
- perf(scope): description (#hash)

### Other
- chore(scope): description (#hash)

---
```

- Only include sections that have commits (skip empty ones)
- Append short git hash in parentheses at end of each line
- Date format: ISO 8601 (YYYY-MM-DD)
- Prepend new version entry at top of file (newest first)
- If the last tag was the initial commit or no tag exists, mark as `## [v0.1.0]` (initial release)
- CHANGELOG and pyproject.toml are generated as unstaged artifacts — user reviews before tagging

### 4a. Generate RELEASE-v<NEXT>.md (disposable)

Generate alongside the CHANGELOG update. Same categorized commits as Section 4,
but in prose-friendly format (no hash suffixes). **Apply skip-empty-section rule**:
omit any category heading with zero commits.

End with two footer lines:
- `**Changelog:**` + `[CHANGELOG.md](CHANGELOG.md)`
- `**Full Changelog:**` + compare URL derived from `git remote get-url origin`
  (normalize SSH/HTTPS, strip `.git`. Skip if no remote, no previous tag,
  or non-GitHub.)

> **Warning:** Do NOT commit this file. It is a disposable artifact for `gh release create`.
> After publishing: **`rm RELEASE-v<NEXT>.md`** — delete immediately.

### 5. Output release summary

After all file changes, print:

```
## Release v1.2.3 ready

### Summary
- X commits since v1.2.2
- X Added | X Fixed | X Updated | X Changed | X Other

### What changed
Brief prose summary of major changes, 2-4 lines. Interpret most significant
commits (feat and fix first, update last), written as human-readable overview.
Example:
"This release introduces the contact form with HTMX submission
and project filtering. It also fixes the theme toggle flash on
first load and updates dependencies to Django 6.0.3."

### Suggested commit message

Derive `<type>` from dominant category in this release batch:
- Breaking changes present → `feat!` or `fix!`
- Any `feat` (no breaking) → `feat`
- Only `fix` → `fix`
- Only `refactor` → `refactor`
- Only `update` → `update`
- Only chore/docs/etc → `chore`

If `caveman-commit` or another dedicated commit-message skill is available
in the agent's skills, suggest invoking it for a compressed message.
Otherwise, use the standard Conventional Commits format:

  <type>(<scope>): <description>

Append `!` for breaking changes (e.g. `feat!:`). Scope is optional —
omit if none applies.

### Files modified (unstaged — review first):
- pyproject.toml
- CHANGELOG.md
- RELEASE-v1.2.3.md (⚠️ do NOT commit — disposable, delete after publish)

### Next steps — review and run:
  git diff                         # verify version + changelog

  git add -A
  # Use suggested message from the section above
  git commit --amend --no-edit     # bundle into last code commit (optional)
  # or: git commit -m "<type>(<scope>): <description>"

  git tag -a v1.2.3 -m "v1.2.3"
  git push && git push --tags

  gh release create v1.2.3 \
    --title "v1.2.3" \
    --notes-file RELEASE-v1.2.3.md

  just build && just up          # docker deploy
  # or: ./deploy.sh              # VPS deploy
```

### 6. Show deploy commands

Provide deploy commands detected from project context:

- If `docker-compose.yml` + `deploy.sh` exist: show both options
- If only `Dockerfile`: show `docker build` + `docker push`
- If neither: show `git push --tags` as minimal step
- If `gh` CLI is available: suggest `gh release create` as optional publishing step

## Notes

- Do NOT auto-execute git commands — user reviews first
- Do NOT auto-push or auto-deploy
- ALWAYS read current pyproject.toml version before bumping
- Preserve exact formatting of pyproject.toml (quoting, spacing)
- Use the current date in ISO format
- Commit messages are already written in English — keep them as-is for release notes
- No separate "release commit" generated — CHANGELOG and pyproject.toml are unstaged
  artifacts. Tag the last code commit directly. Use `git commit --amend --no-edit` to
  bundle into that commit if desired.
- No em dashes (—). Use other punctuation (: ; , .) in generated release text.
