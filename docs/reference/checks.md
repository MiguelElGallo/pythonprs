# Check requirements

The `Python quality` workflow runs five independent jobs. Every selected requirement must pass. Complexity limits apply to each function and include the stated maximum.

## Jobs and limits

| Local gate | GitHub check | Requirement |
| --- | --- | --- |
| `complexity` | `Quality (complexity)` | complexipy cognitive complexity **≤ 15** |
| `ruff` | `Quality (ruff)` | Ruff McCabe complexity **≤ 10**, selected lint rules, and formatting |
| `types` | `Quality (types)` | Pyrefly default checks and required annotations |
| `docstrings` | `Quality (docstrings)` | Interrogate coverage **100%** and direct AST checks below |
| `tests` | `Quality (tests)` | pytest suite passes |

Cognitive complexity 15 passes; 16 fails. McCabe complexity 10 passes; 11 fails. These limits use the tools' documented defaults. [complexipy threshold](https://complexipy.com/usage-guide/#setting-complexity-threshold), [Ruff threshold](https://docs.astral.sh/ruff/settings/#lint_mccabe_max-complexity).

For actual diagnostics and repaired code, see [the iParq examples](../explanation/iparq-examples.md).

## Ruff rules

| Selection | Checks |
| --- | --- |
| `E4`, `E7`, `E9`, `F` | Essential imports, syntax patterns, and name errors |
| `I` | Import ordering |
| `C901` | McCabe complexity |
| `ANN001`, `ANN002`, `ANN003` | Parameter annotations, including variadic parameters |
| `ANN201`, `ANN202`, `ANN204`, `ANN205`, `ANN206` | Return annotations for public/private functions and relevant special, static, and class methods |

Ruff's formatter runs separately in check mode. Formatting does not replace lint checks. See [Ruff's rule reference](https://docs.astral.sh/ruff/rules/).

## Type requirements

Pyrefly uses `preset = "default"`, checks unannotated function bodies, and sets `implicit-any-parameter` and `unannotated-return` to errors. Ruff also requires explicit parameter and return annotations through the rules above.

Tests cover incompatible assignments, arguments, and returns; missing attributes; unsupported operations; and missing annotations. The default preset contains additional diagnostics. This is not every available strict rule, and explicit `Any` can still reduce checking. [Pyrefly configuration](https://pyrefly.org/en/docs/configuration/), [error kinds](https://pyrefly.org/en/docs/error-kinds/).

## Documentation requirements

| Definition | Required documentation |
| --- | --- |
| Every `def` and `async def` | Own nonempty literal docstring; first paragraph **≥ 8 word tokens AND ≥ 40 non-whitespace characters** |
| Every module and class | Own nonempty literal docstring; function length minimum does not apply |

The function rule includes private and nested functions, constructors, magic methods, properties and their setters/deleters, overload declarations, tests, and overriding methods. Documentation inherited from a parent does not satisfy it.

The first paragraph can span multiple lines. A blank line ends it. The checker counts Unicode word tokens, allowing internal apostrophes and hyphens; it counts every non-whitespace character in that paragraph, including punctuation. Both minimums are inclusive.

| Code | Meaning |
| --- | --- |
| `DOC001` | Missing or blank documentation on a required definition |
| `DOC002` | Function summary below either minimum |
| `DOC003` | Source cannot be read or parsed, or a `.py` file is a symbolic link |

Interrogate is a third-party package. The AST checker is custom project code in [`scripts/check_docstrings.py`](https://github.com/MiguelElGallo/pythonprs/blob/main/scripts/check_docstrings.py), using Python's built-in [`ast`](https://docs.python.org/3/library/ast.html) module. Interrogate reports coverage; the custom checker enforces presence directly and measures summary length. `style = "sphinx"` controls constructor coverage, not the required markup style.

Length is a project baseline, not a measure of correctness. See [why both documentation checks are used](../explanation/docstrings-and-types.md).

## Versions

Tool packages are pinned in [`pyproject.toml`](https://github.com/MiguelElGallo/pythonprs/blob/main/pyproject.toml), with dependencies locked in `uv.lock`. Python is selected in `.python-version`, and the workflows pin uv.

| Component | Version | Role |
| --- | --- | --- |
| Python | 3.12 | Interpreter selected by `.python-version` |
| uv | 0.12.17 | CI environment setup |
| complexipy | 8.0.1 | Cognitive complexity |
| Ruff | 0.16.9 | Lint, annotations, McCabe complexity, formatting |
| Pyrefly | 1.3.1 | Type checking |
| Interrogate | 1.7.0 | Documentation coverage |
| pytest | 9.1.1 | Tests |
| ty | 0.0.84 | Additional local development type check |
| Zensical | 0.0.65 | Documentation site |

ty is available locally; Pyrefly owns the workflow's `types` gate. The documentation workflow is separate from the five quality jobs.

Related: [configuration](configuration.md), [commands](commands.md), [repairing failures](../how-to/fix-failed-checks.md).
