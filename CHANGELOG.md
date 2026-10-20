# Changelog

Notable changes to the `cli-output` repo. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow the git tags
(`vMAJOR.MINOR.PATCH`, annotated, never moved).

## [Unreleased]

## [0.1.0] - 2026-10-19

First release. The package content is the in-tree `src/shared/cli-output` package (version
`0.1.0` there) imported with `git subtree split`, history preserved.

### Added

- `cli-output` 0.1.0 — the `cli_output` import package: domain views (`MessageView`,
  `ResultView`, `ListView`, `ErrorView`, `ConfigInfoView`, `RawView`, `CustomView`,
  `ProgressEvent`), the `Renderer` port, and the `RichRenderer`, `PlainRenderer` and
  `JsonRenderer` adapters selected by `create_renderer` / `OutputFormat`.
- Repo harness copied byte-for-byte from `mu`: `scripts/worktree-guard.sh`, `skills-lock.json`,
  `_bmad/`, `.agents/skills/`, `.opencode/`, `tickets/_template/`, `tickets/LIFECYCLE.md`.
- `Makefile` fresh-clone verbs (`sync`, `test`, `lint`, `format`, `hermetic`, `gates`, `build`,
  `install`, `clean`, `ci-gates-parity`), `.github/workflows/gates.yml`, and a hermetic test
  fixture (`tests/conftest.py`) with a canary.
- MIT `LICENSE`, `README.md`, `AGENTS.md`, `CLAUDE.md` symlink, ticket `REQ-001-repo-birth`, and
  the OpenSpec change `repo-birth`.

### Changed

- `pyproject.toml`: added release metadata (`readme`, license classifier) and
  `[tool.hatch.build.targets.sdist] only-include` so the harness trees stay out of the sdist.
  `[tool.uv.sources]` is empty — the package has no sibling dependency.
