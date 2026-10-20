# cli-output-presentation Specification

## Purpose

`cli-output` is the shared presentation layer for the dotfiles CLI toolchain. Callers build a
domain-neutral view and a renderer prints it in Rich, plain or JSON form. This spec fixes the
public surface and the boundary that keeps the package presentation-only (spine AD-1, AD-2,
AD-4).

## Requirements

### Requirement: the package exports one public surface

`cli_output` SHALL export exactly the names in `__all__`: the eight views `MessageView`,
`ResultView`, `ListView`, `ErrorView`, `ConfigInfoView`, `RawView`, `CustomView`,
`ProgressEvent`; the `Renderer` protocol; the three adapters `RichRenderer`, `PlainRenderer`,
`JsonRenderer`; and `OutputFormat` + `create_renderer`.

Source: `src/cli_output/__init__.py:1-33`.

#### Scenario: importing the documented names succeeds

- **WHEN** `from cli_output import *` is executed in a fresh interpreter
- **THEN** every name in `__all__` resolves and `__all__` has 14 entries

### Requirement: one Renderer port covers common CLI output

`Renderer` SHALL be a runtime-checkable `Protocol` with `message`, `result`, `list`, `error`,
`config`, `raw`, `custom`, plus `status(message)` and `progress(total, message)` returning
context managers; the progress context manager SHALL yield an object with `advance(n=1)`.

Source: `src/cli_output/ports/renderer.py:17-32`.

#### Scenario: a complete adapter satisfies the protocol

- **WHEN** an object implementing those nine methods is checked with
  `isinstance(obj, Renderer)`
- **THEN** the check is true

#### Scenario: an incomplete adapter is rejected

- **WHEN** an object missing `error` is checked with `isinstance(obj, Renderer)`
- **THEN** the check is false

### Requirement: format selection is total over OutputFormat

`create_renderer(fmt, console=None)` SHALL return `JsonRenderer` for `OutputFormat.JSON`,
`PlainRenderer` for `OutputFormat.PLAIN`, and `RichRenderer` for `OutputFormat.RICH` (using the
supplied `Console` when given); any other value SHALL raise `ValueError`.

Source: `src/cli_output/adapters/factory.py:12-19`.

#### Scenario: each enum member maps to its adapter

- **WHEN** `create_renderer` is called with each of the three `OutputFormat` members
- **THEN** the returned types are `JsonRenderer`, `PlainRenderer`, `RichRenderer`

#### Scenario: an unsupported value fails loudly

- **WHEN** `create_renderer("yaml")` is called
- **THEN** it raises `ValueError` naming the value

### Requirement: the package is presentation-only

The package SHALL depend on `rich` only; it SHALL NOT import another repo's package, read a path
in another repo, or contain argv/env/artifact contract data or a conformance kit.

Source: spine AD-4; `pyproject.toml` `dependencies`.

#### Scenario: no sibling import

- **WHEN** `rg -n "^(from|import) (config_assembler_engine|oci_runtime|cli_invocation_contract)" src`
  runs at the repo root
- **THEN** it exits 1 (no matches)

#### Scenario: one runtime dependency

- **WHEN** `pyproject.toml` `[project].dependencies` is read
- **THEN** it is exactly `["rich>=13.0"]`

### Requirement: one package, repo root is the package root

The repo SHALL hold one `pyproject.toml` at its root and one import package at
`src/cli_output/`; tests SHALL live in `tests/` and there SHALL be no nested project root or
second OpenSpec root under `src/`.

Source: spine AD-2.

#### Scenario: layout on a fresh clone

- **WHEN** `test -f pyproject.toml -a -d src/cli_output -a -d tests` runs at the repo root
- **THEN** it exits 0, and `find src -name pyproject.toml` prints nothing below the root

#### Scenario: the suite runs from the root

- **WHEN** `make test` runs on a fresh clone
- **THEN** exit 0 and the reported test count covers every module in `tests/`
