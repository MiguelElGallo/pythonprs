# Research and reviewed implementation plan

Research date: 27 September 2026. The goal is a blocking GitHub Actions workflow for Python complexity, essential type safety, and documentation on every function, supported by deliberately failing examples.

## Complexity limits

[complexipy](https://github.com/rohaquinlop/complexipy) measures cognitive complexity: nesting and control-flow patterns contribute to how difficult a function is to follow. Its [documented default threshold](https://complexipy.com/usage-guide/#setting-complexity-threshold) is 15. This project allows scores up to 15 and fails above 15. Snapshots, inline ignores, and report-only success behavior are disabled so existing violations cannot silently pass.

[Ruff C901](https://docs.astral.sh/ruff/rules/complex-structure/) measures McCabe complexity, based on decision paths. Its [default maximum](https://docs.astral.sh/ruff/settings/#lint_mccabe_max-complexity) is 10. This project allows scores up to 10 and fails above 10. Both metrics are useful: a function can have many separate decisions or fewer deeply nested decisions. Neither score proves correctness. These are per-function ceilings; module-level script complexity is not enabled.

The Ruff gate also enables essential syntax/import/name checks (`E4`, `E7`, `E9`, `F`), import ordering (`I`), and parameter/return annotation rules (`ANN001`, `ANN002`, `ANN003`, `ANN201`, `ANN202`, `ANN204`, `ANN205`, `ANN206`). Formatting must match Ruff's formatter. The runner supplies `--ignore-noqa` so inline suppression cannot hide a selected violation.

## Essential type checking

The [Pyrefly default preset](https://pyrefly.org/en/docs/configuration/#preset) supplies the base checks. The configuration adds `implicit-any-parameter` and `unannotated-return` as errors, checks unannotated bodies, and uses checked return inference. This covers practical errors such as incompatible assignments, argument and return types, unknown names, and unsafe optional-value use without enabling every stricter diagnostic. [Pyrefly error kinds](https://pyrefly.org/en/docs/error-kinds/).

The configuration explicitly chooses `preset = "default"`; automatic unconfigured runs can select a reduced basic preset. It disables ignore directives, ignore-file filtering, heuristic exclusions, and exemptions for generated code. The runner supplies explicit file paths and the production configuration. [Pyrefly configuration](https://pyrefly.org/en/docs/configuration/).

This is static checking against annotations, not a runtime test or a guarantee that every use of `Any` is safe. Application dependencies and their available type information still affect results. Add an adopted application's dependencies to its environment rather than hiding missing imports.

## Choosing the documentation check

| Candidate | Useful capability | Why it is insufficient alone |
| --- | --- | --- |
| [Interrogate 1.7.0](https://pypi.org/project/interrogate/), selected | Finds documented and undocumented definitions, including private, nested, and async functions. | Measures presence, not minimum summary length; percentage rounding can mask a missing docstring. |
| [docstr-coverage 2.3.2](https://pypi.org/project/docstr-coverage/) | Reports coverage and supports a blocking threshold. | Excludes setters/deleters by default; comments can excuse or inherit missing docs; no minimum-length policy. |
| [Ruff D103](https://docs.astral.sh/ruff/rules/undocumented-public-function/) / [pydocstyle](https://pydocstyle.readthedocs.io/en/latest/error_codes.html#publicity) | Checks public-documentation presence and docstring conventions. | Private definitions and descendants of private constructs can be exempt from missing-documentation rules. |
| [pydoclint 0.9.1](https://pypi.org/project/pydoclint/) | Compares documented arguments, returns, yields, and raises with the signature and implementation. | [Skips functions without docstrings](https://github.com/jsh9/pydoclint#271-how-to-not-document-certain-functions); description-only docs are skipped by default; no minimum-length setting. |

Interrogate is configured with every function-exclusion option disabled and a 100% coverage threshold. `style = "sphinx"` is intentional: the Google coverage mode treats a class docstring as covering its missing `__init__` docstring. This setting controls that coverage treatment, not markup validation. [Interrogate configuration](https://interrogate.readthedocs.io/en/latest/), [constructor coverage implementation](https://github.com/econchick/interrogate/blob/1.7.0/src/interrogate/coverage.py).

Interrogate rounds coverage before comparing it with `fail-under`. With an integer threshold of 100, 199 documented objects out of 200 can pass because 99.5 rounds to 100. The second check therefore inspects definitions directly and fails whenever a required literal docstring is missing. [Threshold implementation](https://github.com/econchick/interrogate/blob/1.7.0/src/interrogate/coverage.py).

The small standard-library checker uses [Python's AST](https://docs.python.org/3/library/ast.html#ast.get_docstring) to find every function, async function, class, and module without executing inspected code. Each function's first documentation paragraph must contain at least 8 word tokens and 40 non-whitespace characters. The two limits are an explicit project baseline, not a published universal standard. A long or padded docstring can still be inaccurate; human review remains necessary. Modules and classes need nonempty documentation but do not receive the function-summary length rule.

Pydoclint could be added later to enforce a selected Args/Returns/Raises convention, but it is unnecessary for the requested presence-and-length gate. Also, [Pylint's `docstring-min-length`](https://pylint.readthedocs.io/en/stable/user_guide/configuration/all-options.html#docstring-min-length) refers to the length of a function/class that requires documentation, not the length of its docstring.

## File scope and strictness

One scanner discovers `.py` files across the repository, including hidden and newly added directories, root files, tests, and examples. Directory exclusions come from `[tool.pythonprs].exclude-dirs`; an empty scan fails. All four code gates receive that explicit inventory. Python source encodings are read with `tokenize.open()`. Unreadable or invalid source fails the documentation check.

The documentation policy includes private functions, nested functions, async definitions, constructors, magic methods, property accessors, overload declarations, and tests. Each definition needs its own literal docstring, including overrides. Lambdas, dynamically generated callables, notebooks, and `.pyi` files are outside the check. Symbolic-link directories are skipped; symbolic-link Python files fail documentation inspection.

This strict scope is deliberate, but adopting projects must review exclusions and the extra module/class convention. Repositories with many generated files or stub-only APIs may need an explicitly reviewed policy change. The implementation does not use a legacy baseline or silently exempt existing violations.

## Plan and peer review

Before implementation, three independent agent reviews examined the complexity tools, type-check configuration, and documentation policy. Their findings were incorporated into this plan:

1. Pin Python 3.12 and exact tool versions, then lock dependencies so local and CI runs share an environment.
2. Use one file inventory and one gate runner for every code check. Disable inline suppressions and complexity snapshots.
3. Pair Interrogate with direct AST enforcement to close rounding and constructor-coverage gaps.
4. Store bad examples as text fixtures; materialize Python files in temporary project roots with the production configuration.
5. Test passing and failing complexity boundaries, essential type errors, missing annotations, and missing/blank/short documentation. Include private, nested, async, constructor, property, magic, overload, and inherited-override cases, plus first-paragraph and near-100% rounding regressions.
6. Run four independent code gates and the test suite in a GitHub Actions matrix on pushes, pull requests, and manual dispatch. Keep read-only permissions, SHA-pinned actions, and visible diagnostics.
7. Explain local usage, adoption, scope, and required branch checks. Preparing the local implementation does not activate it on GitHub or configure repository protection.

The important review corrections were to enforce every definition directly, make scan scope identical across tools, preserve strictness despite ignore comments, and describe documentation length honestly. The complexity gate rejects root-level `complexipy.toml`/`.complexipy.toml`, any `[tool.complexipy.diff]` policy, and nonempty complexipy exclusions. It reads and validates the `pyproject.toml` maximum and passes that value explicitly. These checks prevent alternate configuration, diff-only enforcement, and tool-specific exclusions from weakening the shared policy.

Negative tests require a named diagnostic as well as exit status 1, so an unrelated failure cannot count as proof that the intended rule works. The runner preserves normal tool failures and converts signal termination into a blocking status of 2.

## Locked environment and workflow

| Component | Pinned version | Primary reference |
| --- | --- | --- |
| Python | 3.12 | [`.python-version`](https://github.com/MiguelElGallo/pythonprs/blob/main/.python-version) |
| uv | 0.12.17 | [Workflow setup](https://github.com/MiguelElGallo/pythonprs/blob/main/.github/workflows/python-quality.yml) |
| complexipy | 8.0.1 | [Published release](https://pypi.org/project/complexipy/8.0.1/) |
| Ruff | 0.16.9 | [Published release](https://pypi.org/project/ruff/0.16.9/) |
| Pyrefly | 1.3.1 | [Published release](https://pypi.org/project/pyrefly/1.3.1/) |
| Interrogate | 1.7.0 | [Published release](https://pypi.org/project/interrogate/1.7.0/) |
| pytest | 9.1.1 | [Published release](https://pypi.org/project/pytest/9.1.1/) |
| ty, available for development validation | 0.0.84 | [Published release](https://pypi.org/project/ty/0.0.84/) |

The [`Python quality` workflow](https://github.com/MiguelElGallo/pythonprs/blob/main/.github/workflows/python-quality.yml) runs `uv sync --locked` before its selected gate. `fail-fast: false` preserves results from the other matrix jobs when one fails. The required check names are `Quality (complexity)`, `Quality (ruff)`, `Quality (types)`, `Quality (docstrings)`, and `Quality (tests)`.

GitHub checks block merging only when repository protection or a ruleset requires them. [GitHub status-check requirements](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches#require-status-checks-before-merging). The implementation was subsequently committed and pushed at the user's request; branch enforcement remains a repository setting described in the [required-check guide](../how-to/require-checks.md).

## Validation completed

All five gates passed locally using the locked Python 3.12 environment, including a run with `GITHUB_ACTIONS=true` to exercise GitHub annotation output. The suite passed **78 tests** using **13 text fixtures**. Actual CLI tests verified cognitive scores 15/16 and McCabe scores 10/11, named type errors, missing/short documentation, suppression resistance, and root/new/hidden/gitignored file discovery. Configuration overrides and terminated tools also produced blocking results.

Interrogate reported 100% coverage for all 55 objects; direct AST inspection found 47 functions across 7 Python files with no documentation violations. Ruff lint and formatting, Pyrefly, `ty check`, `actionlint`, locked dependency synchronization, and whitespace checks passed.

Commit `4c99161c448e94e123148c2a53fb3b3f2fea70e3` published the implementation to `MiguelElGallo/pythonprs`. All five hosted quality jobs passed in [GitHub Actions run 36311260681](https://github.com/MiguelElGallo/pythonprs/actions/runs/36311260681). Later documentation work adds the Zensical dependency and a separate Pages workflow; its review evidence is in the [documentation review record](documentation-review.md).
