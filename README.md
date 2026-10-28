# CI-MPC 517

Simulation-first constrained-interaction MPC experiments for a gripper in
MEAM 517. The project uses [Drake](https://drake.mit.edu/) for multibody
modeling, simulation, and optimization, with NumPy for numerical work.

## Prerequisites

- CPython 3.13 (the repository pins it in `.python-version`).
- A [Drake-supported platform](https://drake.mit.edu/installation.html): the
  initial team targets are Ubuntu 24.04 or macOS Tahoe on Apple Silicon.
- On macOS, use Homebrew Python, **not** Apple system Python.
- Do not use Conda for this project; Drake does not regularly test that setup.
- [uv](https://docs.astral.sh/uv/getting-started/installation/).

Intel macOS is not a supported Drake-wheel platform. The dependency lock keeps
Drake for Linux and Apple Silicon macOS, while Intel-macOS contributors can
still run the package, formatting, linting, types, and non-Drake tests.

## Getting started

```bash
git clone <repository-url>
cd ci_mpc_517
uv sync --locked --all-groups
uv run pre-commit install
```

On macOS, force the supported Homebrew interpreter when creating the
environment, then verify it:

```bash
uv sync --python "$(brew --prefix python@3.13)/bin/python3.13" --locked --all-groups
uv run python -c "import sys; print(sys.executable)"
```

On a supported Drake platform, confirm the installation:

```bash
uv run python -c "import pydrake; print(pydrake.__file__)"
```

`drake` is installed from its stable PyPI wheel and pinned by `uv.lock`. Review
and commit lockfile changes whenever dependencies change.

## Daily checks

```bash
uv run ruff format --check .
uv run ruff check .
uv run pyright
uv run pytest -v --cov=ci_mpc_517
```

Run every configured local check with:

```bash
uv run pre-commit run --all-files
```

The same checks run in GitHub Actions on every push and pull request.

## Scope

This repository intentionally starts with Python simulation tooling. acados,
real-time code generation, C++ integration, CMake, and bindings are out of
scope until the project demonstrates a need for them.
