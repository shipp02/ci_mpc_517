# Python Development Setup Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Establish a reproducible, checked Python/Drake workspace that a three-person simulation-controls team can clone, synchronize, and validate locally or in CI.

**Architecture:** A `uv`-managed installable package provides the stable home for reusable gripper, controller, and simulation modules. The project locks Drake and numerical runtime dependencies in `uv.lock`; local pre-commit hooks and one GitHub Actions workflow execute the same Ruff, Pyright, and pytest commands. Documentation makes the supported Drake/Python platform requirements explicit.

**Tech Stack:** Python 3.13, uv, Drake/PyDrake, NumPy, Ruff, Pyright, pytest, pytest-cov, pre-commit, GitHub Actions.

**Spec:** `docs/superpowers/specs/2026-10-28-python-development-setup-design.md`

## Global Constraints

- Pin the working interpreter in `.python-version` to Python 3.13.
- Declare `drake` and `numpy` as runtime dependencies and commit the `uv.lock` resolution.
- Use Drake's stable PyPI wheel; use no nightly index or acados dependency.
- Target only Drake-supported CPython / operating-system combinations and document that macOS users need Homebrew Python, not Apple system Python.
- Place reusable modules in `src/ci_mpc_517/`, tests in `tests/`, and executable experiments in `scripts/`.
- CI runs on Linux for push and pull-request events; it does not run graphics-dependent or expensive simulation suites.
- Use Ruff as the only formatter, and maintain pragmatic Pyright checking by suppressing only missing third-party type stubs.

## Review Focus

- A clean clone with no pre-existing virtual environment must synchronize exactly from `uv.lock`; Task 1 runs `uv sync --locked` after lock creation.
- The installed package must import without importing Drake at module import time; Task 1's smoke test imports `ci_mpc_517` only.
- A contributor on unsupported macOS Python or Conda must see a clear prerequisite warning before trying to synchronize; Task 3 documents these constraints and the exact validation command.
- Formatting must be checked rather than silently rewritten in CI; Task 2 invokes `ruff format --check` and validates the command in the workflow.
- Changes bypassing local hooks must still fail CI for lint, types, tests, or stale lockfiles; Task 3 runs every Task 2 check after `uv sync --locked`.

---

## File Structure

- `.python-version` — exact interpreter selected by the team.
- `.gitignore` — excludes `uv` environments, Python caches, coverage output, and editor/OS noise while retaining `uv.lock`.
- `pyproject.toml` — package metadata, runtime and development dependencies, build backend, and Ruff/Pyright/pytest/coverage configuration.
- `uv.lock` — resolved, committed dependency graph for all supported lock platforms.
- `src/ci_mpc_517/__init__.py` — minimal package boundary and public version constant.
- `tests/test_package.py` — installation/import smoke test.
- `.pre-commit-config.yaml` — file hygiene and project quality-hook definitions.
- `README.md` — installation, common commands, workflow, and Drake platform requirements.
- `.github/workflows/ci.yml` — locked-install continuous-integration job.
- `scripts/.gitkeep` — preserves the location for explicit simulation and experiment entry points until the first script exists.

### Task 1: Create the uv Project and Importable Package

**Files:**
- Create: `.python-version`
- Create: `.gitignore`
- Create: `pyproject.toml`
- Create: `uv.lock`
- Create: `src/ci_mpc_517/__init__.py`
- Create: `tests/test_package.py`

**Interfaces:**
- Produces: installable distribution `ci-mpc-517`; importable module `ci_mpc_517` exposing `__version__: str`.
- Produces: `uv run pytest`, `uv run ruff`, and `uv run pyright` commands for Tasks 2 and 3.

- [ ] **Step 1: Create `tests/test_package.py` with the package import smoke test**

```python
import ci_mpc_517


def test_package_has_version() -> None:
    assert ci_mpc_517.__version__ == "0.1.0"
```

- [ ] **Step 2: Run the smoke test before creating the package**

Run: `uv run pytest tests/test_package.py -v`

Expected: FAIL because `ci_mpc_517` is not importable.

- [ ] **Step 3: Add the Python-3.13 uv project configuration and package implementation**

Create `.python-version` containing `3.13`. Configure `pyproject.toml` with project name `ci-mpc-517`, version `0.1.0`, `requires-python = ">=3.13,<3.14"`, a `src` package build using Hatchling, runtime dependencies `drake` and `numpy`, and a `dev` dependency group containing `pre-commit`, `pyright`, `pytest`, `pytest-cov`, and `ruff`. Configure Ruff to target Python 3.13, Pyright in `standard` mode for `src`, with `reportMissingTypeStubs = "none"`, and pytest coverage for `ci_mpc_517`. Implement `src/ci_mpc_517/__init__.py` with `__version__: str = "0.1.0"`. Add a focused Python `.gitignore` that ignores `.venv/`, caches, coverage output, and local editor/OS files but not `uv.lock`.

- [ ] **Step 4: Lock and synchronize the declared environment**

Run: `uv lock && uv sync --locked --all-groups`

Expected: a committed `uv.lock` is created and the environment installs the package, Drake, NumPy, and development tools successfully.

- [ ] **Step 5: Run the package smoke test**

Run: `uv run pytest tests/test_package.py -v`

Expected: PASS with `test_package_has_version` passing.

- [ ] **Step 6: Commit the project baseline**

```bash
git add .python-version .gitignore pyproject.toml uv.lock src/ tests/
git commit -m "build: initialize uv Python project"
```

### Task 2: Add Local Quality Gates

**Files:**
- Create: `.pre-commit-config.yaml`
- Modify: `pyproject.toml`

**Interfaces:**
- Consumes: the `uv` commands and tools declared in Task 1.
- Produces: `pre-commit run --all-files` executes whitespace hygiene, formatter verification, linting, and type checking without changing source files.

- [ ] **Step 1: Add a deliberately unformatted temporary Python file and verify the formatter check fails**

Create a temporary file with `x=1`, then run:

Run: `uv run ruff format --check <temporary-file>`

Expected: FAIL and identify the file as requiring formatting; remove the temporary file after confirming the failure.

- [ ] **Step 2: Configure local pre-commit hooks and focused Ruff lint rules**

Create `.pre-commit-config.yaml` with standard YAML/end-of-file/trailing-whitespace checks plus local hooks that run `uv run ruff format --check`, `uv run ruff check`, and `uv run pyright`. Extend the `pyproject.toml` Ruff configuration with the selected correctness, import, modern-Python, and pytest rule families; exclude generated/build/cache directories. Do not configure a second formatter or a separate import sorter.

- [ ] **Step 3: Install hooks and run all local quality checks**

Run: `uv run pre-commit install && uv run pre-commit run --all-files && uv run ruff format --check . && uv run ruff check . && uv run pyright && uv run pytest -v --cov=ci_mpc_517`

Expected: every command exits successfully; coverage reports the package smoke test.

- [ ] **Step 4: Commit local quality gates**

```bash
git add pyproject.toml .pre-commit-config.yaml
git commit -m "build: add Python quality gates"
```

### Task 3: Document Onboarding and Add Locked CI

**Files:**
- Create: `README.md`
- Create: `.github/workflows/ci.yml`
- Create: `scripts/.gitkeep`

**Interfaces:**
- Consumes: the locked project and checks from Tasks 1–2.
- Produces: documented commands `uv sync --locked --all-groups`, `uv run pytest`, `uv run ruff check .`, `uv run ruff format --check .`, and `uv run pyright`; CI runs those checks on push and pull request.

- [ ] **Step 1: Write the onboarding README and placeholder scripts directory**

Document project purpose, supported platform prerequisites (CPython 3.13; a Drake-supported OS; Homebrew Python rather than Apple Python on macOS; no Conda), uv installation link, clone/sync/hook-install commands, daily check commands, and a Drake validation command: `uv run python -c "import pydrake; print(pydrake.__file__)"`. State that the stable PyPI `drake` wheel is locked by uv and that acados/C++ integration is intentionally out of scope. Add `scripts/.gitkeep` with a brief comment explaining its future experiment-entry-point role.

- [ ] **Step 2: Add GitHub Actions CI using a locked uv install**

Create `.github/workflows/ci.yml` triggered by `push` and `pull_request`, with one Ubuntu job. Pin `actions/checkout`, `actions/setup-python`, and `astral-sh/setup-uv` to full commit SHAs; configure Python from `.python-version`; enable uv caching; run `uv sync --locked --all-groups`; then run `uv run ruff format --check .`, `uv run ruff check .`, `uv run pyright`, and `uv run pytest -v --cov=ci_mpc_517`. Give the workflow only `contents: read` permission.

- [ ] **Step 3: Validate the documented clean-environment and CI commands locally**

Run: `uv sync --locked --all-groups && uv run python -c "import pydrake; print(pydrake.__file__)" && uv run ruff format --check . && uv run ruff check . && uv run pyright && uv run pytest -v --cov=ci_mpc_517`

Expected: synchronization uses the committed lock unchanged; PyDrake imports; formatter, linter, type checker, and test suite all exit successfully.

- [ ] **Step 4: Inspect the workflow syntax and repository status**

Run: `uv run pre-commit run --files README.md .github/workflows/ci.yml && git status --short`

Expected: pre-commit accepts the added text/YAML files and status shows only intended documentation, workflow, and placeholder additions.

- [ ] **Step 5: Commit onboarding and CI**

```bash
git add README.md .github/workflows/ci.yml scripts/.gitkeep
git commit -m "ci: add project validation workflow"
```

## Self-Review

1. **Spec coverage:** Task 1 implements uv, package layout, runtime/development dependencies, lockfile, and pytest; Task 2 implements Ruff, Pyright, and pre-commit; Task 3 implements CI, Drake onboarding, and the scripts/docs structure. No spec requirement is uncovered.
2. **Step scan:** Each step names the exact file or command and expected result. Task 2 intentionally uses validation-after-configuration rather than product TDD, per the approved request to skip TDD for environment scaffolding.
3. **Type consistency:** `ci_mpc_517.__version__: str` is produced in Task 1 and consumed only by its import test; all later tools target the `ci_mpc_517` package consistently.
4. **Review Focus:** The five highest-risk operational cases—clean clone synchronization, lightweight package import, unsupported Drake platform, non-mutating format checking, and CI enforcement—are each assigned a validating step in Tasks 1–3.
5. **Proportion:** The plan specifies repository decisions and verifications without prescribing tool-generated lockfile contents or implementation bodies beyond the minimal public package interface.
