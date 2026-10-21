# repo-birth

## Why

`cli-output` is the first package extracted from `dotfiles-v3` in the CLI/split programme
(spine `SLICES.md` § S1). The package itself already exists and is green in-tree, but a
repository is not a valid extraction target until it is mu-strict from its first commit
(spine **AD-12**: "harness later" is exactly the failure the guide names).

Evidence: the package is presentation-only and green in-tree — `src/shared/cli-output` has a
`pyproject.toml` with `rich` as its only runtime dependency and 8 test modules.
Verify: `git -C /home/inumaki/Development/dotfiles-v3 ls-tree -r --name-only master -- src/shared/cli-output | grep -c '^tests/test_'`
→ `8`.

Evidence: the split must preserve history with `git subtree split`, not a fresh-root import
(spine OPEN-8, `SLICES.md` § Slice template step 2).
Verify: `git -C /home/inumaki/Development/dotfiles-v3 subtree split -P src/shared/cli-output -b split/cli-output`
→ exit 0, branch tip `2fe5ff7`.

Evidence: the harness must be copied from `mu`, not re-authored (AD-12).
Verify: `diff -r /home/inumaki/Development/mu/_bmad _bmad` → empty; same for `scripts/worktree-guard.sh`
and `skills-lock.json`.

## What Changes

- Create the public repo `RandomGenericUsername/cli-output`, default branch `main`, from the
  `git subtree split` result of `src/shared/cli-output` (four commits preserved).
- Install the mu-strict harness **byte-for-byte**: `scripts/worktree-guard.sh`, `skills-lock.json`,
  `_bmad/`, `.agents/skills/`, `.opencode/`, `tickets/_template/`, `tickets/LIFECYCLE.md`.
- Add the repo contract: `AGENTS.md` (evidence rule, charter = presentation only, ticket routing,
  never work on main, planning vs implementation, release ritual), `CLAUDE.md` → `AGENTS.md`.
- Add the work ledger: `tickets/INDEX.md` (counting rule header), `tickets/REQ-001-repo-birth/`.
- Add the OpenSpec root: `openspec/config.yaml` (evidence rules, repo context),
  `openspec/specs/cli-output-presentation/`, and this change.
- Add a fresh-clone Makefile (`sync`, `test`, `lint`, `format`, `hermetic`, `lock-check`,
  `docs-check`, `gates`, `gates-ci`, `build`, `install`, `clean`, `ci-gates-parity`),
  `.github/workflows/gates.yml` (one job per gate + `ci-gates-parity` + tag release),
  `scripts/ci-gates-parity.py`, MIT `LICENSE`, `README.md`, `CHANGELOG.md`, `.gitignore`.
- Keep packaging honest: `[tool.uv.sources]` empty (no sibling dependency), the committed
  `uv.lock` current, and `uv build` producing `cli_output-0.1.0` sdist + wheel.

## Capabilities

- **New Capabilities:** `repo-harness` — the repo contract, ticket ledger, OpenSpec root, gate set
  and release ritual that make this repo a valid extraction target.

- **Modified Capabilities:** none. `cli-output-presentation` is documented as the existing
  behaviour; no view, port or adapter behaviour changes.

## Non-goals (out of scope)

- **Not** rewiring `dotfiles-v3`: no consumer pin, no deletion of `src/shared/cli-output`, no
  lock regeneration there. That is the next slice step and a different repo/commit set.
- **Not** changing package behaviour: no new view, port, adapter, flag or dependency.
- **Not** adding the invocation contract: it lives in `contracts` (AD-4).
- **Not** publishing to PyPI (OPEN-3): the release channel is a git tag plus GitHub Release
  artifacts.

## Impact

- New: `AGENTS.md`, `CLAUDE.md` (symlink), `Makefile`, `LICENSE`, `CHANGELOG.md`, `.gitignore`,
  `.github/workflows/gates.yml`, `scripts/ci-gates-parity.py`, `openspec/`, `tickets/`,
  and the copied harness trees.
- Modified: `pyproject.toml` (readme/license metadata, sdist `only-include`).
- Unchanged: `src/cli_output/`, `tests/`, `uv.lock` dependency set (no dependency added or
  removed), and every file in `dotfiles-v3`.
