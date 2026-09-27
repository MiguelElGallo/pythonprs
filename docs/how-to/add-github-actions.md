# Add the checks to an existing project

Use this guide to add all five quality jobs to a Python repository that you can push to. Start with a clean working branch, a working test suite, and a project compatible with Python 3.12. If you want a first practice run, use the [local tutorial](../tutorials/first-check.md).

## 1. Copy the runner, examples, tests, and workflow

Get the files from [the starter repository](https://github.com/MiguelElGallo/pythonprs). Copy these paths into your project:

| Path | Purpose |
| --- | --- |
| `.github/workflows/python-quality.yml` | The five GitHub Actions jobs |
| `scripts/__init__.py`, `scripts/quality.py`, `scripts/check_docstrings.py` | Shared command runner and documentation check |
| `.python-version` | Python 3.12 selection |
| `examples/good.py` | Passing example used by the gate tests |
| `tests/__init__.py`, `tests/test_docstrings.py`, `tests/test_quality_gates.py` | Tests of the checks themselves |
| All files in `tests/fixtures/` | Deliberately bad examples stored as text |

If a path already exists, merge its contents deliberately. Keep your application tests. A project that already has a `scripts` package must reconcile module names and imports; the workflow invokes `scripts.quality`.

## 2. Merge configuration and dependencies

Open [the starter's pyproject.toml](https://github.com/MiguelElGallo/pythonprs/blob/main/pyproject.toml) alongside your own.

Add the pinned quality tools from `[dependency-groups].dev`: complexipy, Interrogate, Pyrefly, pytest, and Ruff. ty is an additional development check; Zensical builds documentation. Include those two if you want those features.

Merge these sections, keeping your own application metadata, dependencies, and build settings:

```toml
[tool.pythonprs]
[tool.complexipy]
[tool.ruff]
[tool.ruff.lint]
[tool.ruff.lint.mccabe]
[tool.pyrefly]
[tool.pyrefly.errors]
[tool.interrogate]
[tool.pytest.ini_options]
```

This list names the sections to merge; it is not a complete configuration to paste. The [configuration reference](../reference/configuration.md) explains their values. Resolve settings that already exist, including Ruff rules and pytest test directories.

The starter has `[tool.uv] package = false` because it consists of scripts. Preserve your project's packaging choice. An application whose tests import its installed package may need that package installed by `uv sync`.

Review `exclude-dirs`. Every `.py` file outside those directories is checked, including tests, private functions, hidden directories, and existing code. Modules and classes also need nonempty docstrings. Add your application's dependencies so Pyrefly can resolve its imports.

## 3. Create your project's lock and run the checks

With [uv installed](https://docs.astral.sh/uv/getting-started/installation/), run from your project's root:

```sh
uv lock
uv sync --locked
uv run --no-sync python -m scripts.quality all
```

Use your own generated `uv.lock`; the starter's lock describes the starter's dependencies. Commit the lock and `.python-version` with the configuration.

Fix failures before requiring checks on your default branch. Use the [failure guide](fix-failed-checks.md) to work on one job at a time.

The supplied gate tests assume the starter's limits: cognitive complexity 15, McCabe complexity 10, and summaries of 8 words and 40 characters. If you change those limits or add lint rules, update the relevant fixtures and assertions to test your approved policy. Keep both passing and failing boundary cases.

## 4. Push and verify a pull request

Commit the copied files and merged configuration, push your working branch, and open a pull request into your project's default branch. In **Checks**, verify that all five `Quality (...)` jobs appear and pass. The workflow runs on pushes and pull requests; manual runs are also available.

For a repository using a merge queue, add `merge_group:` beside `pull_request:` in the workflow's `on:` section. GitHub needs runs for the temporary merge group as well. [GitHub merge-queue check requirements](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/troubleshooting-required-status-checks#status-checks-with-github-actions-and-a-merge-queue).

Finally, [require all five checks](require-checks.md) on the branch you merge into.
