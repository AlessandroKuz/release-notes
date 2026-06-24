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

### Files modified:
- pyproject.toml
- CHANGELOG.md

### Next steps — review and run:
  git add -A
  git commit -m "release: v1.2.3"
  git tag -a v1.2.3 -m "v1.2.3"
  git push && git push --tags

  just build && just up          # docker deploy
  # or: ./deploy.sh              # VPS deploy
```

### 6. Show deploy commands
Provide deploy commands detected from project context:
- If `docker-compose.yml` + `deploy.sh` exist: show both options
- If only `Dockerfile`: show `docker build` + `docker push`
- If neither: show `git push --tags` as minimal step

## Notes
- Do NOT auto-execute git commands — user reviews first
- Do NOT auto-push or auto-deploy
- ALWAYS read current pyproject.toml version before bumping
- Preserve exact formatting of pyproject.toml (quoting, spacing)
- Use the current date in ISO format
- Commit messages are already written in English — keep them as-is for release notes
