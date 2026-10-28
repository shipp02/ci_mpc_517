# Python Development Setup Design

## Intent

Create a low-friction, reproducible Python foundation for a three-person,
simulation-first CI-MPC gripper project. The team will use Drake as the
simulation and optimization platform they already know; this setup should make
experiments easy to run while catching common errors before code is merged.

## Goals

- Manage Python versions, project dependencies, and a committed lockfile with
  `uv`.
- Provide a conventional installable package layout for reusable modeling,
  controller, and simulation code.
- Standardize formatting and linting with Ruff.
- Type-check project code with Pyright.
- Test with pytest and report coverage in CI.
- Run the same quality checks locally through pre-commit and in GitHub Actions.
- Document a straightforward Drake installation and verification path.

## Non-goals

- Real-time deployment, code generation, acados, or controller benchmarking.
- C++ code, bindings, CMake, or a mixed-language build.
- Docker/devcontainers or full OS-level reproducibility.
- Selecting or implementing the gripper plant, MPC formulation, or experiment
  configuration system.

## Constraints and assumptions

- The project is simulation-only for now and primarily Python based.
- Drake is the selected controls/simulation platform.
- Drake publishes binary wheels to PyPI, so the stable `drake` wheel can be a
  regular runtime dependency managed and locked by `uv`. Contributors must use
  a Drake-supported CPython and operating-system combination; Conda and Apple
  system Python are not supported installation targets.
- The repository will pin Python 3.13, which Drake officially supports on
  Ubuntu 24.04 and macOS Tahoe (arm64) at the time of setup. Contributors on
  another platform must use a supported Drake configuration or a future
  container-based setup.
- **Compatibility amendment (2026-10-28):** The team is using Intel macOS, so
  the repository instead pins Python 3.12 and Drake 1.34.0, the newest PyPI
  wheel available for that environment. Revisit this baseline when the team
  moves to Apple Silicon or a Linux-only setup.
- CI will run on GitHub Actions using Linux and validate formatting, linting,
  types, tests, and package installation. It will not initially execute
  graphics-dependent or expensive simulation suites.

## Approaches considered

1. **Lean research baseline (selected):** `uv`, a package layout, Ruff,
   Pyright, pytest, pre-commit, and GitHub Actions. This gives the team fast,
   consistent feedback with little operational overhead.
2. **Reproducible simulation baseline:** the selected tools plus a Docker or
   devcontainer environment. This gives stronger cross-machine consistency but
   is disproportionate for the expected simulation-only class project.
3. **Research-to-deployment scaffold:** the selected tools plus CMake, C++,
   bindings, and experiment-management infrastructure. It would prepare for a
   possible future direction but creates unused maintenance surface now.

The team selected approach 1 because it matches the current scope and retains
an uncomplicated path to add containers or C++ later.

## Proposed repository structure

```
src/ci_mpc_517/       # reusable Python modules
tests/                # pytest test suite
scripts/              # small explicit entry points for simulations/experiments
docs/                 # onboarding and technical notes
.github/workflows/    # continuous integration
pyproject.toml        # package metadata, dependencies, and tool configuration
uv.lock               # committed resolved dependency graph
.pre-commit-config.yaml
README.md
```

## Tooling design

- `pyproject.toml` is the single source for package metadata, Python version,
  runtime/development dependency groups, Ruff, Pyright, pytest, and coverage
  configuration.
- `uv` creates and synchronizes the virtual environment and maintains
  `uv.lock`. Contributors install developer tooling with a documented `uv`
  command; lockfile changes are reviewed like source changes.
- Ruff enforces formatting and a focused set of correctness/import/modern-Python
  lint rules. It is the sole formatter to avoid tool overlap.
- Pyright checks code under `src/`. Initial settings will accommodate the
  incomplete stubs frequently found in numerical/scientific dependencies while
  still treating errors in project annotations as failures.
- pytest discovers `tests/`; small deterministic unit tests run by default.
  Drake-based integration/simulation tests can later be marked separately once
  the project has them.
- pre-commit invokes whitespace/file-hygiene checks plus Ruff and Pyright so
  contributors get feedback before pushing.
- GitHub Actions installs the pinned Python/uv environment, runs the project
  checks, and records coverage. Workflow commands will be the documented local
  commands wherever practical.

## Drake dependency policy

Drake stays the primary modeling, simulation, visualization, and
optimization framework. `uv` will install its stable PyPI wheel as a locked
runtime dependency. The onboarding documentation will specify Drake's
supported operating-system and CPython requirements, including Homebrew Python
on macOS, and a minimal import/version validation command. A future
container-based environment remains an option if platform consistency becomes
a problem.

## Definition of done

- A new contributor can follow the README to install prerequisites, synchronize
  dependencies, install git hooks, and run a smoke test.
- `uv` lockfile and dependency groups are present and committed.
- Local and CI commands successfully run formatter/linter, type checker, and
  tests.
- CI runs on pull requests and pushes.
- The repository contains a small tested importable package rather than only
  loose scripts.

## Builder decisions still open

- Keep initial quality gates pragmatic for scientific Python; ratchet typing
  strictness as the package grows.
