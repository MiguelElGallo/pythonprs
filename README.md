# pythonprs

Blocking Python quality checks for GitHub pull requests, with passing examples and deliberately failing examples that test the checks themselves.

Four gates check the code. A fifth gate runs the regression tests. The same commands run locally and in GitHub Actions.

| Gate | Tool | Passing requirement |
| --- | --- | --- |
| `complexity` | [complexipy](https://github.com/rohaquinlop/complexipy) | Each function's cognitive complexity is **15 or lower**. |
| `ruff` | [Ruff C901](https://docs.astral.sh/ruff/rules/complex-structure/) and essential lint rules | Each function's McCabe complexity is **10 or lower**; lint and formatting checks also pass. |
| `types` | [Pyrefly](https://github.com/facebook/pyrefly) | Default type checks and required parameter/return annotations pass. |
| `docstrings` | [Interrogate](https://interrogate.readthedocs.io/en/latest/) and a Python AST checker | Every definition has its own nonempty docstring; every function summary has at least **8 words and 40 non-whitespace characters**. |
| `tests` | pytest | Good examples pass and bad examples produce the expected blocking diagnostics. |

The complexity limits are inclusive: cognitive complexity 15 passes and 16 fails; McCabe complexity 10 passes and 11 fails. The two tools measure different aspects of complexity. The chosen limits follow their upstream defaults. [complexipy threshold](https://complexipy.com/usage-guide/#setting-complexity-threshold), [Ruff threshold](https://docs.astral.sh/ruff/settings/#lint_mccabe_max-complexity).

## Run locally

Use Python 3.12 and [uv](https://docs.astral.sh/uv/getting-started/installation/). CI pins uv to 0.12.17. From the repository root:

```sh
uv sync --locked
uv run --no-sync python -m scripts.quality all
```

Run one gate while fixing a particular problem:

```sh
uv run --no-sync python -m scripts.quality complexity
uv run --no-sync python -m scripts.quality ruff
uv run --no-sync python -m scripts.quality types
uv run --no-sync python -m scripts.quality docstrings
uv run --no-sync python -m scripts.quality tests
```

A nonzero exit status blocks the check. The runner continues through the selected gates after normal tool failures; invalid configuration or a failed file scan aborts the local run. The independent CI jobs still run separately. Ruff checks formatting without rewriting files. To apply formatting locally, run `uv run --no-sync ruff format .` and review the changes.

[pyproject.toml](pyproject.toml) pins complexipy 8.0.1, Ruff 0.16.9, Pyrefly 1.3.1, Interrogate 1.7.0, pytest 9.1.1, and ty 0.0.84. [uv.lock](uv.lock) locks their dependencies. ty is available for an additional development check with `uv run --no-sync ty check`; Pyrefly owns the workflow's `types` gate.

## What gets checked

The shared scanner finds every `.py` file under the repository root, including root-level files, new directories, tests, and hidden directories. It passes the same explicit file inventory to each code gate. Directory names in `[tool.pythonprs].exclude-dirs` are excluded at any depth; the defaults cover Git metadata, environments, caches, dependencies, and build outputs. Review this list when adopting the checks. A scan with no Python files fails.

Symbolic-link directories are not followed. A symbolic-link `.py` file fails the documentation check. Notebooks and `.pyi` stubs are outside this workflow's scope.

Every `def` and `async def` needs its own literal docstring. This includes private and nested functions, constructors, magic methods, properties and their setters/deleters, overload declarations, and test functions. An overriding method cannot rely on its parent's documentation. Modules and classes also require nonempty docstrings as an additional project convention. Lambdas and callables created at runtime are outside the literal-docstring check.

For functions, the length rule counts only the first paragraph, which can span several lines. Long `Args` or `Returns` sections after a blank line cannot compensate for a short summary. The 8-word/40-character minimum is a project baseline: length cannot prove that documentation is correct, useful, or free of filler. Review the explanation of purpose, inputs, outputs, and side effects yourself.

Interrogate's `style = "sphinx"` keeps constructor coverage separate from class coverage; it does not require Sphinx markup. The AST checker enforces missing documentation directly, avoiding percentage-rounding gaps.

The production runner disregards Ruff `noqa` comments and complexipy inline ignores. Complexity snapshots cannot excuse an over-limit function. Pyrefly ignore directives are disabled. The complexity gate also rejects alternate `complexipy.toml`/`.complexipy.toml` files, complexity diff policies, and nonempty complexipy exclusions; use the shared `exclude-dirs` list to change file scope. It validates and passes the configured ceiling explicitly. Change the reviewed policy in `pyproject.toml` when a different baseline is justified.

## Examples and regression tests

[examples/good.py](examples/good.py) shows small, annotated functions with documentation. Deliberately bad source lives in [tests/fixtures](tests/fixtures) as `*.py.txt` files, so it does not make the repository's normal checks permanently fail.

The tests copy those fixtures to real `.py` files in temporary project roots, use the production configuration, and require both exit status `1` and the expected diagnostic. They cover complexity boundaries, type errors, missing annotations, missing/blank/short docstrings, and documentation on less-visible kinds of functions. They also cover the percentage-rounding trap and repository-wide file discovery.

Run `uv run --no-sync python -m scripts.quality tests` to exercise the failing examples.

## Enable GitHub enforcement

[The workflow](.github/workflows/python-quality.yml) runs on every push, pull request, and manual dispatch. Its independent matrix jobs are `Quality (complexity)`, `Quality (ruff)`, `Quality (types)`, `Quality (docstrings)`, and `Quality (tests)`. A failing job does not cancel the others. Actions use commit SHA pins; the workflow has read-only repository permissions and does not retain checkout credentials.

The implementation is prepared locally. It becomes active on GitHub when these files are committed and pushed. To prevent merging a failing pull request, make all five job checks required in the target branch's protection rule or ruleset. The workflow does not change those repository settings. [GitHub required status checks](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches#require-status-checks-before-merging).

For another repository, copy `scripts/`, `examples/good.py`, `tests/` including its fixtures, the workflow, `.python-version`, and the tool configuration. Merge the development dependencies into its existing `pyproject.toml`, review the exclusion list, and generate its own `uv.lock` with `uv lock`; copy this lock only when adopting this project's dependency configuration unchanged. The fifth job requires a runnable pytest suite. Run all gates before activating required checks.

See [research and reviewed implementation plan](docs/research-and-plan.md) for the source-backed choices and tradeoffs.
