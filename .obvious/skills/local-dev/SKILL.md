---
name: local-dev
description: Bring this conda-forge feedstock to a working local build environment and verify the recipe end-to-end
---

# local-dev — pygeohash-feedstock

Durable record of the 2026-09-05 onboarding run (sandbox `cmp_njPejd1k`,
snapshot `ib4aknjx5i44hpnx4hf2m`). Verified: recipe builds and the packaged
library works.

## Environment facts

- Docker is **unavailable** in the sandbox → canonical `python build-locally.py`
  fails at the docker step. Use the dockerless conda-build path below.
- micromamba 2.9.0 at `~/.local/bin`; conda env `forge` = python 3.11.16 +
  conda-build 26.7.1 + pip; conda env `pkgtest` = python 3.11 with the built
  pygeohash installed.

## Bring-up from a fresh machine

1. Install micromamba:
   `mkdir -p ~/.local/bin && cd /tmp && curl -Ls https://micro.mamba.pm/api/micromamba/linux-64/latest | tar -xj bin/micromamba && mv bin/micromamba ~/.local/bin/`
2. Create the build env:
   `micromamba create -y -n forge -c conda-forge python=3.11 conda-build pip`
3. Configure channels (docker flow normally does this via `setup_conda_rc`):
   `printf 'channels:\n  - conda-forge\nchannel_priority: strict\n' > ~/.condarc`
4. Build the recipe (~2 min):
   `micromamba run -n forge conda build recipe -m .ci_support/linux_.yaml -c conda-forge --output-folder build_artifacts`

## Verify the primary flow

Install the built artifact into a fresh env and exercise the library:

```
micromamba create -y -n pkgtest -c conda-forge python=3.11
micromamba install -y -n pkgtest -c file://$PWD/build_artifacts pygeohash
micromamba run -n pkgtest python -c "import pygeohash; print(pygeohash.encode(41.8781, -87.6298, 9))"
```

Expected: `dp3wjztvt`; `pygeohash.decode('dp3wjztvt')` → `(41.8781, -87.6298)`.

## Quick health check

`micromamba run -n pkgtest python -c "import pygeohash; assert pygeohash.encode(41.8781, -87.6298, 2) == 'dp'"`

## Gotchas learned this run

- `conda build` fails with `NoChannelsConfiguredError` unless `~/.condarc`
  exists or `-c conda-forge` is passed.
- `.ci_support/linux_.yaml` pins host python 2.7 — pip deprecation warnings
  during the build are expected, not failures.
- `.scripts/build_steps.sh` references a clobber file that does not exist in
  this repo; omit `--clobber-file` when invoking conda-build manually.
- The build leaves caches under the `forge` env's `conda-bld/`; `build_artifacts/`
  is gitignored.
- Generated files must change via `conda smithy rerender`, not by hand.
