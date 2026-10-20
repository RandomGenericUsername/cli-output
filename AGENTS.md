# AGENTS.md — cli-output operating contract

Canonical agent instructions for this repo. Claude (`CLAUDE.md` symlinks here), OpenCode, and all
BMAD/OpenSpec skills must follow this.

## 1. Evidence rule (hard requirement)

No suggestion "in the air". Every technical claim, recommendation, or design decision must carry testable evidence — at least one of:

- **Doc evidence:** file path + line range, or commit SHA, or URL + quoted section.
- **Prototype evidence:** a script/command in-repo that runs and prints the validating output. State the exact command and observed result.
- **Test evidence:** `command run → result` (exit code + key output). Failing-as-expected counts if it validates the claim.

Format: any non-trivial claim gets an `Evidence:` line and a `Verify:` line (how to reproduce in one command). If evidence doesn't exist yet, say `Evidence: MISSING — run <X> to produce it` and do not present the claim as fact.

## 2. Charter — presentation only

- This repo owns one package: `cli-output` (import package `cli_output`). It is the shared
  **presentation layer** for the dotfiles CLI toolchain: domain-neutral views, one `Renderer`
  port, and the Rich / plain / JSON adapters.
- Public surface: `MessageView`, `ResultView`, `ListView`, `ErrorView`, `ConfigInfoView`,
  `RawView`, `CustomView`, `ProgressEvent`, `Renderer`, `RichRenderer`, `PlainRenderer`,
  `JsonRenderer`, `OutputFormat`, `create_renderer`.
  Evidence: `src/cli_output/__init__.py:1-33`.
- The only runtime dependency is `rich`. The package imports no sibling repo and reads no file
  path in another repo.
  Evidence: `pyproject.toml` `dependencies = ["rich>=13.0"]`; `rg -n "^from|^import" src/cli_output`.
- This repo does **not** own the runtime ↔ CLI invocation contract. That record and its
  conformance kit live in the `contracts` repo (`cli-invocation-contract`), per the split spine
  AD-4. Do not add argv/env/artifact literals, a loader, or a testkit here.
- Consumers (the runtime and the `csg` / `weg` / `itr` CLIs) consume this repo as an immutable
  git tag, never a path and never `editable = true` (spine AD-3).

## 3. How we work here

- Read files, run commands (`rg`, `git log`, `uv`, `openspec validate`), paste results. Prefer
  executed output over recalled text.
- When findings contradict an earlier claim, state the discrepancy and trust the evidence.
- Keep changes small, verifiable, committable. A suite that is green only on the author's host is
  not green: tests are hermetic (empty `HOME` / `XDG_*`, foreign cwd), enforced by
  `tests/conftest.py` plus the canary in `tests/test_hermetic_canary.py`.
  Verify: `make hermetic` → exit 0.

## 4. OpenSpec + BMAD routing (via tickets)

- All work starts as a ticket in `tickets/` (`_template/`, vocab + rollup in `INDEX.md`, flow in
  `LIFECYCLE.md`). Triage with `/triage` (lanes in `tickets/_fixtures/router-lanes.md`).
- Broad scope (package charter, public API evolution, dependency policy) → BMAD. Installed
  skills are exactly the seven in `skills-lock.json`: `bmad`, `bmad-architecture`, `bmad-build`,
  `bmad-product-brief`, `bmad-spec`, `bmod-core-tools`, `bmod-method` — say so when a step needs a
  missing skill instead of improvising.
- Bounded slices (one proposal + spec + design + tasks) → OpenSpec
  (`/opsx-propose`, `/opsx-apply`, `/opsx-archive`). Artifact rules in `openspec/config.yaml`
  restate this evidence rule per artifact.
- Never work on main: `scripts/worktree-guard.sh create <id> <slug>` scaffolds
  `ticket/<id>-<slug>` + `.worktrees/<id>/`, and aborts when the tree is dirty. Record
  worktree/branch in the ticket.

## 5. Planning vs implementation

- Planning edits (tickets, `openspec/` changes, docs) and implementation changes (code, packaging,
  harness) land in **separate commits**.
- Never mix planning edits with implementation in one commit or one turn.

## 6. Release ritual

- Green gates on the ticket branch → version bumped in `pyproject.toml` + `CHANGELOG.md` →
  annotated tag `vMAJOR.MINOR.PATCH` → CI builds sdist + wheel and attaches them to the GitHub
  Release → consumer pin bump in its own commit.
- A published tag is never moved or deleted; a fix is a new tag. PyPI is not a channel.
  Evidence: `CHANGELOG.md` `0.1.0`; `git tag -l`.

## 7. Gates

- `make gates` = `test lint hermetic lock-check docs-check`; `make build` produces the sdist and
  wheel. CI (`.github/workflows/gates.yml`) runs one job per gate target plus `ci-gates-parity`,
  which compares the workflow's job set against `GATES` in the `Makefile`.
  Verify: `make gates && make build` → exit 0; `make ci-gates-parity` → exit 0.
