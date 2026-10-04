# pygeohash-feedstock — Agent Guide

Repo: `wdm0006/pygeohash-feedstock` — conda-forge feedstock for the `pygeohash` library.

## What this repo is

A conda-forge **feedstock**: a packaging repo whose product is a conda recipe
(`recipe/meta.yaml`) plus conda-smithy-generated CI scaffolding. It builds and
publishes `pygeohash` (geohash encode/decode for lat/lon) to the conda-forge
channel. There is no application, server, or service here — no ports, no
databases, no env vars required.

## Stack

- **Type**: conda packaging repo (conda-forge feedstock)
- **Packaged library**: `pygeohash` 1.2.0 (AGPL-3.0) from a sha256-pinned PyPI sdist
- **Package manager**: conda/mamba — `conda-build`; micromamba for local envs
- **Recipe**: `noarch: python`, host python pinned `2.7.* *_cpython` by the CI variant
- **CI**: conda-smithy rendered — Azure Pipelines + CircleCI; GitHub workflow is dispatch-only/disabled
- **Rerender tool**: `conda smithy rerender` regenerates `.scripts/`, `.ci_support/`, CI yml

## Commands

| Task | Command | Notes |
|---|---|---|
| Build locally (canonical) | `python build-locally.py [linux_]` | Requires Docker (`condaforge/linux-anvil-comp7`); Docker is unavailable in this sandbox |
| Build without Docker (verified) | `micromamba run -n forge conda build recipe -m .ci_support/linux_.yaml -c conda-forge --output-folder build_artifacts` | Full build+test, ~2 min |
| Render recipe only | `micromamba run -n forge conda render recipe -m .ci_support/linux_.yaml` | Validates meta.yaml without building |
| Rerender feedstock | `conda smithy rerender` | Regenerates CI/config files — never hand-edit generated files |
| Smoke-test built package | `micromamba run -n pkgtest python -c "import pygeohash; print(pygeohash.encode(41.8781, -87.6298, 9))"` | Expected `dp3wjztvt` |

## Codebase map

See [codebase-map.md](codebase-map.md).

## Local Verification Summary

Validated 2026-09-05 on sandbox `cmp_njPejd1k` (fresh checkout, dockerless path):

- `dev_stack_healthy: true`
- **Recipe build**: PASS — `conda build recipe -m .ci_support/linux_.yaml` produced
  `build_artifacts/noarch/pygeohash-1.2.0-pyh030149a_1.conda`; conda-build's test phase ended
  `TEST END` (~2 min).
- **Source integrity**: PASS — downloaded sdist sha256
  `750e51643e743eabd065a84fc8c1912c5843b648143137919e4a9776366d921e` matches the pin in `recipe/meta.yaml`.
- **Recipe test gate**: PASS — `imports: pygeohash` executed by conda-build in a test env.
- **Primary user flow**: PASS — built `.conda` artifact installed into a fresh env from the local
  channel; `encode(41.8781, -87.6298, 9)` → `dp3wjztvt`, `decode` → `(41.8781, -87.6298)` exact roundtrip.
- **Recipe metadata**: PASS — `conda render` fully renders the recipe with the `linux_` variant.
- **Config syntax**: PASS — `conda-forge.yml`, `.ci_support/linux_.yaml`, `.github/workflows/conda-build.yml`,
  `.circleci/config.yml`, `.recipe_maintainers.json` all parse.
- **Lint**: PASS — `python -m py_compile build-locally.py` (repo configures no linter; generated files are not linted).
- **Typecheck**: N/A — no type checking configured (packaging repo; only generated Python).
- **Unit tests**: N/A — repo ships no test suite; the recipe `test:` section is the test gate (passed above).

## Sandbox snapshot

- **Snapshot ID**: `ib4aknjx5i44hpnx4hf2m`
- **Built at**: 2026-09-05T21:20:42Z
- **Restores**: micromamba 2.9.0 (`~/.local/bin`), conda env `forge` (python 3.11.16, conda-build 26.7.1),
  conda env `pkgtest` with the built `pygeohash` installed, `~/.condarc` with conda-forge channels,
  build artifact under gitignored `build_artifacts/`.

## Gotchas

- Docker is unavailable in this sandbox — use the dockerless build path above; `build-locally.py`
  itself only fails at the `run_docker_build.sh` step.
- conda-build raises `NoChannelsConfiguredError` without `~/.condarc` (the docker flow normally
  writes it via `setup_conda_rc`); pass `-c conda-forge` or write the condarc first.
- The CI variant pins host python 2.7 — pip deprecation warnings during build are expected, not failures.
- `.scripts/build_steps.sh` references a clobber file that does not exist here; omit `--clobber-file`
  when building manually.
- `build_artifacts/` and `*.pyc` are gitignored — build outputs never belong in commits.
- Changes to generated files (`.scripts/`, `.ci_support/`, CI yml) must go through `conda smithy rerender`.
