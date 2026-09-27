# Commands

Run these commands from the repository root, the directory containing `pyproject.toml`. [uv](https://docs.astral.sh/uv/getting-started/installation/) manages the Python interpreter and installed tools.

## Install the locked environment

```sh
uv sync --locked
```

Installs the development tools and reads `.python-version`. `--locked` refuses a lock that no longer matches the project configuration. After an intentional dependency change, use `uv lock`, review `uv.lock`, then synchronize again.

CI pins uv to 0.12.17. For reproductions that need the same uv version, use the [version-specific installer URLs](https://docs.astral.sh/uv/getting-started/installation/#standalone-installer).

## Quality runner

A **gate** is a named group of checks, such as `ruff` or `docstrings`. The runner selects one gate or runs them all.

```text
uv run --no-sync python -m scripts.quality [GATE] [--root PATH]
```

| Argument | Default | Accepted values |
| --- | --- | --- |
| `GATE` | `all` | `all`, `complexity`, `ruff`, `types`, `docstrings`, `tests` |
| `--root PATH` | Current working directory | Root containing the target `pyproject.toml` and Python source |

`--no-sync` uses the environment already installed by `uv sync --locked`. It avoids changing dependencies during a check.

```sh
uv run --no-sync python -m scripts.quality all
uv run --no-sync python -m scripts.quality complexity
uv run --no-sync python -m scripts.quality ruff
uv run --no-sync python -m scripts.quality types
uv run --no-sync python -m scripts.quality docstrings
uv run --no-sync python -m scripts.quality tests
```

An alternate root lets the installed runner inspect a temporary or separate project:

```sh
uv run --no-sync python -m scripts.quality docstrings --root /path/to/practice-project
```

Replace that absolute path with the directory you want to inspect. Keep the terminal in the checkout containing `scripts.quality`; the target supplies the policy and source, while this checkout supplies the runner and tools.

`all` runs the gates in this order: complexity, Ruff, types, docstrings, tests. Normal tool failures do not stop later commands. A configuration or scan exception aborts the local run. GitHub's independent jobs run separately and do not cancel each other when one fails.

There is no option to check only changed files. Raw tool commands can behave differently because the runner supplies file scope and strictness flags.

## Exit status

| Status | Meaning |
| --- | --- |
| `0` | All selected commands passed |
| `1` | Normal quality violations or failed tests |
| `2` | Runner configuration/scan error, missing or unlaunchable executable, signal termination, or a tool's own status 2 |
| Other positive values | Preserved from a tool, for example pytest's status 5 when no tests are collected |

The runner returns the largest status from the commands it completed. Every nonzero result fails a GitHub job. The tests verify both the expected status and diagnostic for deliberate quality failures.

## Additional development commands

```sh
uv run --no-sync ruff format path/to/file.py
uv run --no-sync ty check
```

The formatter edits the named file; replace the example path. ty performs an additional type check and is not one of the five workflow jobs.

## Documentation commands

```sh
uv run --no-sync zensical serve
uv run --no-sync zensical build --clean --strict
```

`serve` previews this site at <http://localhost:8000/pythonprs/>. Stop it with `Ctrl+C`.

`build` writes generated HTML and assets to `site/`. `--clean` removes the build cache before rebuilding; `--strict` fails the build on warnings. The Pages workflow uses this build command. [Zensical build options](https://zensical.org/docs/usage/build/).

Related: [publishing the site](../how-to/publish-documentation.md), [fixing failed checks](../how-to/fix-failed-checks.md).
