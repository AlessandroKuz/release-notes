# release-notes

Agent skill for AI coding assistants. Generates structured semver releases from git history.

## Features

- Reads git log since last tag, categorizes commits by Conventional Commits type
- Bumps version in pyproject.toml (major/minor/patch)
- Creates/updates CHANGELOG.md in keepachangelog format
- Detects deploy strategy from project files

## Compatible agents

OpenCode, Claude Code, Cursor, Copilot, and any agent supporting SKILL.md format (YAML frontmatter + markdown body).

## Install

Drop `SKILL.md` into your agent's skills directory or reference it in the agent's config (e.g., `opencode.json` skills list).

## Trigger phrases

"create release", "bump version", "tag vX.Y.Z", "generate changelog", "release notes", "cut a release", "/release", "new version"

## Workflow

1. Read git context (latest tag + commits since)
2. Determine next version (semver from commit types or explicit user request)
3. Update pyproject.toml version
4. Generate CHANGELOG.md entry
5. Print release summary + deploy commands (review-only, no auto-exec)

## License

[MIT](LICENSE)
