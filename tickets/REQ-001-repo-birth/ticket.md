---
id: REQ-001
title: "Birth the cli-output repo (mu-strict) and cut v0.1.0"
kind: requirement
status: in-progress
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

- [ ] The repo exists, is **public**, and its `main` carries the four imported commits from the
  subtree split.
  Verify: `gh api repos/RandomGenericUsername/cli-output --jq .private` → `false`;
  `git log --oneline | wc -l` → ≥ 4.
- [ ] The harness is byte-identical to `mu` at every copied path.
  Verify: `diff` per path (see the evidence table) → empty, exit 0.
- [ ] `AGENTS.md` carries the evidence rule and this repo's charter; `CLAUDE.md` is a symlink.
  Verify: `rg -c "Evidence:|Verify:" AGENTS.md` → ≥ 2; `test -L CLAUDE.md` → exit 0.
- [ ] `tickets/INDEX.md` lists this ticket with the same status as the frontmatter, and the
  OpenSpec change validates strictly.
  Verify: `openspec validate repo-birth --strict` → exit 0.
- [ ] A fresh clone passes `make sync && make test && make gates && make hermetic` and builds both artifacts.
  Verify: `uv sync --frozen && uv run pytest -q && make gates && make build` → exit 0;
  `ls dist/cli_output-0.1.0-*` → wheel + sdist.
- [ ] `.github/workflows/gates.yml` exists and its job set equals `GATES` (plus declared CI-only
  jobs); the first run on `main` is green.
  Verify: `make ci-gates-parity` → `PASS ...`; `gh run list -R RandomGenericUsername/cli-output
  --limit 1 --json status,conclusion` → `completed` / `success`.
- [ ] Annotated tag `v0.1.0` exists on the released commit, is anonymously fetchable, and its
  GitHub Release carries the wheel + sdist.
  Verify: `GIT_TERMINAL_PROMPT=0 git -c credential.helper= ls-remote
  https://github.com/RandomGenericUsername/cli-output 'refs/tags/v0.1.0'` → one line;
  `gh release view v0.1.0 --json assets --jq '.assets[].name'` → both artifacts.
- [ ] `git status --porcelain` is empty in both repos; `dotfiles-v3` `master` is at the same SHA
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

(recorded at closeout: worktree exception, commands, observed results)
