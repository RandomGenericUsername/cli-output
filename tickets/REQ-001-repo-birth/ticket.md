---
id: REQ-001
title: "Birth the cli-output repo (mu-strict) and cut v0.1.0"
kind: requirement
status: done
lane: 3
links:
  openspec: repo-birth
  bmad: none
worktree: none
branch: ticket/req-001-repo-birth
---

# Birth the cli-output repo (mu-strict) and cut v0.1.0

## Requirement

Slice **S1** of the CLI-repos split SHALL create `RandomGenericUsername/cli-output` as a public
repo whose root is the imported `src/shared/cli-output` package, preserve that package's history
with `git subtree split`, install the `mu` harness byte-for-byte, and cut the repo's first tag
(`v0.1.0`) only after the gates, packaging and CI exist.

The package remains **presentation-only**: this slice changes no view, port or adapter behaviour,
adds no dependency, and adds neither the invocation contract nor a sibling-repo import (spine
AD-4). `dotfiles-v3` is not modified — the consumer rewire and the in-tree deletion are the next
step of the slice in that repo.

## Acceptance

- [x] The repo exists, is **public**, and its `main` carries the four imported commits from the
  subtree split.
  Verify: `gh api repos/RandomGenericUsername/cli-output --jq .private` → `false`;
  `git log --oneline | wc -l` → ≥ 4.
- [x] The harness is byte-identical to `mu` at every copied path.
  Verify: `diff` per path (see the evidence table) → empty, exit 0.
- [x] `AGENTS.md` carries the evidence rule and this repo's charter; `CLAUDE.md` is a symlink.
  Verify: `rg -c "Evidence:|Verify:" AGENTS.md` → ≥ 2; `test -L CLAUDE.md` → exit 0.
- [x] `tickets/INDEX.md` lists this ticket with the same status as the frontmatter, and the
  OpenSpec change validates strictly.
  Verify: `openspec validate repo-birth --strict` → exit 0.
- [x] A fresh clone passes `make sync && make test && make gates && make hermetic` and builds both
  artifacts.
  Verify: `uv sync --frozen && uv run pytest -q && make gates && make build` → exit 0;
  `ls dist/cli_output-0.1.0-*` → wheel + sdist.
- [x] `.github/workflows/gates.yml` exists and its job set equals `GATES` (plus declared CI-only
  jobs); the first run on `main` is green.
  Verify: `make ci-gates-parity` → `PASS ...`; `gh run list -R RandomGenericUsername/cli-output
  --limit 1 --json status,conclusion` → `completed` / `success`.
- [x] Annotated tag `v0.1.0` exists on the released commit, is anonymously fetchable, and its
  GitHub Release carries the wheel + sdist.
  Verify: `GIT_TERMINAL_PROMPT=0 git -c credential.helper= ls-remote
  https://github.com/RandomGenericUsername/cli-output 'refs/tags/v0.1.0'` → one line;
  `gh release view v0.1.0 --json assets --jq '.assets[].name'` → both artifacts.
- [x] `git status --porcelain` is empty in both repos; `dotfiles-v3` `master` is at the same SHA
  as before the slice.
  Verify: `git status --porcelain` → empty; `git -C /home/inumaki/Development/dotfiles-v3 rev-parse
  master` → `56d7d85e…`.

## Out of scope

- Rewiring `dotfiles-v3` (`src/runtime`, `src/provisioning`, `csg`/`weg`/`itr` `pyproject.toml`)
  to the tag, regenerating their locks, deleting `src/shared/cli-output`, and updating the two
  architecture-test forbidden sets. Those are the remaining S1 steps in that repo.
- Any package behaviour change, dependency change, or contract (argv/env/artifact) content.
- PyPI publication; the channel is the git tag plus GitHub Release artifacts.

## Closeout

Closed 2026-10-19 on `ticket/req-001-closeout-close-ticket` (worktree
`.worktrees/req-001-closeout/`), fast-forwarded onto `main`. The slice work is on
`ticket/req-001-repo-birth` (`501d2b2` planning, `ddae372` implementation) and the tag `v0.1.0`
points at `ddae372`; this closeout only ticks the record, writes this section, and moves
`tickets/INDEX.md` to Done.

**Worktree exception (birth only).** The guard creates a branch from a clean `main`, and this
repo had no `main` before the first scaffold. The first two commits were therefore made on
`ticket/req-001-repo-birth` in the clone, per the S1 instruction, and the guard was installed by
those commits. This closeout is the first item that uses the guard normally:
`bash scripts/worktree-guard.sh create req-001-closeout close-ticket` → `worktree=.worktrees/req-001-closeout/`,
`branch=ticket/req-001-closeout-close-ticket`, exit 0. Every later item uses it.

Evidence (commands run from the repo root unless a path is named):

| Claim | Command | Result |
| --- | --- | --- |
| Repo is public | `gh api repos/RandomGenericUsername/cli-output --jq .private` | `false`, exit 0 |
| History preserved by subtree split | `git -C /home/inumaki/Development/dotfiles-v3 subtree split -P src/shared/cli-output -b split/cli-output` then `git rev-parse split/cli-output` | `2fe5ff749c153b19402f9a2fc6664790cf1a7311`, exit 0; `git log --oneline split/cli-output \| wc -l` → `4` |
| `main` starts at the split | `git log --oneline \| tail -1` | `8998cb3 feat(shared): add cli-output shared rendering package` |
| Harness byte-identical | `diff -r /home/inumaki/Development/mu/<path> <path>` for `scripts/worktree-guard.sh`, `skills-lock.json`, `_bmad`, `.agents/skills`, `.opencode`, `tickets/_template`, `tickets/LIFECYCLE.md`, `tickets/_fixtures` | empty output, exit 0 for all eight |
| Guard is executable and clean-tree safe | `bash scripts/worktree-guard.sh check-clean` | exit 0 |
| Guard creates a worktree/branch | `bash scripts/worktree-guard.sh create req-001-closeout close-ticket` | `worktree=.worktrees/req-001-closeout/`, `branch=ticket/req-001-closeout-close-ticket`, exit 0 |
| AGENTS carries the contract | `rg -c "Evidence:\|Verify:" AGENTS.md`; `test -L CLAUDE.md` | `6`; symlink, exit 0 |
| Tests | `uv run pytest -q` | `63 passed in 0.19s`, exit 0 |
| Gates (local clone) | `make gates` | test `63 passed`, lint `All checks passed!`, hermetic `63 passed`, `uv lock --check` resolved 13 packages, docs-check silent → exit 0 |
| Build | `make build` | `dist/cli_output-0.1.0-py3-none-any.whl` + `dist/cli_output-0.1.0.tar.gz`, exit 0 |
| Specs valid | `openspec validate --specs --strict` | `✓ spec/cli-output-presentation`, `Totals: 1 passed, 0 failed (1 items)`, exit 0 |
| Change valid | `openspec validate repo-birth --strict` | `Change 'repo-birth' is valid`, exit 0 |
| Workflow ↔ Makefile parity | `make ci-gates-parity` | `PASS gates=['docs-check', 'hermetic', 'lint', 'lock-check', 'test'] jobs=['ci-gates-parity', 'docs-check', 'hermetic', 'lint', 'lock-check', 'release', 'test'] exempt=['ci-gates-parity', 'release']`, exit 0 |
| Fresh clone at the tag is green | `git clone --branch v0.1.0 --depth 1 <url> /tmp/opencode/cli-output-tag && make sync && make gates && make build` | exit 0 for all; same 63 tests; both artifacts in `dist/` |
| First `main` CI run is green | `gh run view 39350772036 --json status,conclusion,jobs` | `success`; jobs `test`, `lint`, `hermetic`, `lock-check`, `docs-check`, `ci-gates-parity` all `success`; `release` `skipped` (not a tag) |
| Tag CI run is green including release | `gh run view 39350824846 --json status,conclusion,jobs` | `success`; all six gate jobs `success` and `release` `success` |
| Tag is annotated and points at the tested commit | `git rev-parse v0.1.0`; `git rev-parse v0.1.0^{commit}` | tag `91489b73b54e6cd09dc54e360431a3ebb915fa05`; commit `ddae372786fd6053730133e82cf542a17e9a5ae5` |
| Tag is anonymously fetchable, one line | `GIT_TERMINAL_PROMPT=0 git -c credential.helper= ls-remote https://github.com/RandomGenericUsername/cli-output 'refs/tags/v0.1.0'` | `91489b7…	refs/tags/v0.1.0` — exactly one line, exit 0 |
| Release carries both artifacts | `gh release view v0.1.0 --json assets --jq '.assets[].name'` | `cli_output-0.1.0-py3-none-any.whl`, `cli_output-0.1.0.tar.gz` |
| `dotfiles-v3` untouched | `git -C /home/inumaki/Development/dotfiles-v3 rev-parse master`; `git status --porcelain \| wc -l` | `56d7d85ea4e7c2fcad8358c067ef1a7a17e47345`; `0` |

Note: `skills-lock.json`'s `computedHash` values do not equal a plain sha256 of `SKILL.md` — the
same is true in `mu` (verified by running the same recomputation in both repos). The requirement
is that the harness is copied, so the evidence is byte identity plus the lock file being
byte-identical, not a re-derivation of `bmod`'s hash scheme.

Slice commits (oldest first, all on `main`):

| Commit | Subject |
| --- | --- |
| `501d2b2` | `plan(REQ-001): open repo-birth ticket, OpenSpec change, and release docs` |
| `ddae372` | `feat(REQ-001): mu-strict harness, fresh-clone gates, and release CI` |
