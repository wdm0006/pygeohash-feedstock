# Codebase Map — pygeohash-feedstock

Depth-2 map. ⚙️ = conda-smithy generated; do not hand-edit (use `conda smithy rerender`).

| Path | What it is |
|---|---|
| `recipe/` | The product: `meta.yaml` (pygeohash 1.2.0, noarch, sha256-pinned PyPI source, pip build script) + upstream `LICENSE.txt` |
| `.scripts/` | ⚙️ CI build scripts run inside the docker image: `build_steps.sh` (conda build + test), `run_docker_build.sh` |
| `.ci_support/` | ⚙️ Build matrix: `linux_.yaml` (docker image `condaforge/linux-anvil-comp7`, python 2.7 variant, channel sources/targets) + README |
| `.azure-pipelines/`, `azure-pipelines.yml` | ⚙️ Azure Pipelines CI definition (linux build) |
| `.circleci/` | ⚙️ CircleCI config |
| `.github/` | `CODEOWNERS` + `workflows/conda-build.yml` (dispatch-only placeholder, `if: false`) |
| `build-locally.py` | ⚙️ Local build entrypoint: picks a `.ci_support/*.yaml` config, runs `.scripts/run_docker_build.sh` (Docker required) |
| `conda-forge.yml` | Feedstock policy: output validation on, branch `main`, pkg format 2 |
| `.recipe_maintainers.json` | Recipe maintainer list |
| `README.md` | Feedstock readme — package info, conda install instructions, conda-forge background, update workflow |
| `LICENSE.txt` | Feedstock license (BSD 3-Clause) |
| `.gitignore`, `.gitattributes` | Ignores `*.pyc`, `build_artifacts`; line-ending policy |
