# cli-output

Shared CLI presentation layer for the dotfiles CLI toolchain: domain-neutral views plus
Rich, plain and JSON renderers.

**Charter — presentation only.** This package renders what a caller hands it. It does **not**
own the runtime ↔ CLI invocation contract (that lives in the
[`contracts`](https://github.com/RandomGenericUsername/contracts) repo as
`cli-invocation-contract`), does not know any other repo's file layout, and does not depend on a
sibling package. Its only runtime dependency is `rich`.

## Public surface

```python
from cli_output import (
    # views (domain)
    ConfigInfoView, CustomView, ErrorView, ListView, MessageView,
    ProgressEvent, RawView, ResultView,
    # port
    Renderer,
    # adapters
    JsonRenderer, PlainRenderer, RichRenderer,
    # selection
    OutputFormat, create_renderer,
)
```

- `Renderer` is a `Protocol`: `message`, `result`, `list`, `error`, `config`, `raw`, `custom`,
  plus the `status()` and `progress()` context managers.
- `create_renderer(format)` maps an `OutputFormat` (`RICH`, `PLAIN`, `JSON`) to its adapter.
- Views live in `src/cli_output/domain/`; the port in `src/cli_output/ports/`; the three adapters
  in `src/cli_output/adapters/output/`. Layering is enforced by the tests in `tests/`.

## Install

From a checkout (development):

```bash
make sync          # uv sync --frozen
make gates         # test + lint + hermetic + lock-check + docs-check
```

As a consumer, pin an immutable tag — never a path and never a branch:

```toml
# pyproject.toml
dependencies = ["cli-output>=0.1.0"]

[tool.uv.sources]
cli-output = { git = "https://github.com/RandomGenericUsername/cli-output", tag = "v0.1.0" }
```

```bash
uv add "cli-output @ git+https://github.com/RandomGenericUsername/cli-output@v0.1.0"
```

A published tag is never moved or deleted; a fix ships as a new tag.

## What `dotfiles-v3` pins

`dotfiles-v3` consumes this repo as a pinned artifact (`v0.1.0` at first release) from:

- `src/runtime/pyproject.toml`
- `src/provisioning/pyproject.toml`
- each CLI package that renders output (`csg`, `weg`, `itr`)

No consumer may use `path = ...` or `editable = true` for this package.

## Release

Version in `pyproject.toml` + `CHANGELOG.md` entry → annotated tag `vX.Y.Z` → GitHub Actions
builds the sdist and wheel and attaches them to the GitHub Release. PyPI is not used.

## Repo harness

- `AGENTS.md` (with `CLAUDE.md` symlinked to it) is the operating contract: evidence rule,
  worktree discipline, planning-vs-implementation commits.
- `tickets/` is the work ledger (`INDEX.md`, `LIFECYCLE.md`, `_template/`).
- `openspec/` holds the specs and the change log (`config.yaml`, `specs/`, `changes/`).
- `scripts/worktree-guard.sh create <id> <slug>` makes `.worktrees/<id>/` on
  `ticket/<id>-<slug>`; never work on `main`.
- `.agents/skills/` + `skills-lock.json` and `_bmad/` are hash-pinned copies of the `mu` harness.

## License

MIT — see `LICENSE`.
