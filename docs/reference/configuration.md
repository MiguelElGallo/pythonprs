# Configuration

The quality runner reads the selected repository root's `pyproject.toml`. `.python-version` selects Python 3.12, and `uv.lock` fixes the dependency environment. The complete starter values are in [the source configuration](https://github.com/MiguelElGallo/pythonprs/blob/main/pyproject.toml).

## Shared file scope and documentation policy

| Setting in `[tool.pythonprs]` | Type | Starter value | Meaning |
| --- | --- | --- | --- |
| `exclude-dirs` | List of directory-name strings | List below | Skip matching directory names at any depth |
| `docstring-min-words` | Positive integer | `8` | Minimum word tokens in each function's first documentation paragraph |
| `docstring-min-characters` | Positive integer | `40` | Minimum non-whitespace characters in that paragraph |

The starter excludes these directory names:

```toml
exclude-dirs = [
    ".git", ".venv", "venv", "__pycache__", ".pytest_cache",
    ".ruff_cache", ".complexipy_cache", ".mypy_cache", ".tox",
    "node_modules", "build", "dist",
]
```

The scanner finds all other files ending in `.py`, including root-level files, tests, examples, hidden directories, new directories, and gitignored files. The same explicit inventory goes to all four code gates. Tool-specific exclusions are not the shared scope control.

Directory symbolic links are skipped. A symbolic-link `.py` file fails the documentation gate. An absent root, unreadable directory, or scan containing no Python files fails setup. Notebooks, `.pyi` stubs, lambdas, and callables generated at runtime are outside the literal-docstring policy.

## Complexity

| Setting in `[tool.complexipy]` | Starter value | Runner behavior |
| --- | --- | --- |
| `max-complexity-allowed` | `15` | Must be a non-negative integer; supplied in the runner's tool command |
| `snapshot-ignore` | `true` | Runner command option prevents snapshots from excusing a violation |
| `snapshot-create` | `false` | Runner command option prevents creating a snapshot |
| `ignore-complexity` | `false` | Runner command option makes violations fail |
| `no-ignore` | `true` | Runner command option prevents inline ignores from excusing a violation |

The runner rejects root-level `complexipy.toml` or `.complexipy.toml`, any `[tool.complexipy.diff]` policy, and nonempty complexipy `exclude` settings. It checks every included Python file against the configured maximum on each run. There is no option to check only files changed since another commit or selected for a commit.

## Ruff

| Setting | Starter value | Meaning |
| --- | --- | --- |
| `[tool.ruff].target-version` | `"py312"` | Python syntax target |
| `[tool.ruff].line-length` | `88` | Formatter line-length target |
| `[tool.ruff.lint].select` | [Selected rules](checks.md#ruff-rules) | Enabled lint families and annotation checks |
| `[tool.ruff.lint.mccabe].max-complexity` | `10` | Inclusive per-function McCabe limit |

The runner loads this configuration explicitly and supplies `--ignore-noqa`. It checks the scanner's file list with both `ruff check` and `ruff format --check`.

## Pyrefly

| Setting in `[tool.pyrefly]` | Starter value | Meaning |
| --- | --- | --- |
| `preset` | `"default"` | Default diagnostic set |
| `python-version` | `"3.12"` | Python target |
| `check-unannotated-defs` | `true` | Inspect bodies without annotations too |
| `infer-return-types` | `"checked"` | Infer missing return types from function bodies Pyrefly checks |
| `project-includes` | `["**/*.py"]` | Python scope pattern |
| `project-excludes` | `[]` | No additional project exclusions |
| `disable-project-excludes-heuristics` | `true` | Do not automatically skip additional directories |
| `use-ignore-files` | `false` | Do not filter through ignore files |
| `ignore-errors-in-generated-code` | `false` | Generated-code markers do not excuse errors |
| `enabled-ignores` | `[]` | Disable ignore directives |

`[tool.pyrefly.errors]` sets both `implicit-any-parameter` and `unannotated-return` to `"error"`. The runner supplies the shared file inventory and selected root's configuration. [Pyrefly setting definitions](https://pyrefly.org/en/docs/configuration/).

## Interrogate

`[tool.interrogate]` sets `fail-under = 100`, `style = "sphinx"`, and `verbose = 1`. All supplied `ignore-*` options are `false`: init methods/modules, magic methods, modules, nested functions/classes, overloaded functions, private/semiprivate definitions, properties, and setters stay included.

The AST checker runs after Interrogate even when coverage fails. Its limits come from `[tool.pythonprs]`, and its diagnostics are defined in [check requirements](checks.md#documentation-requirements).

## Tests and the documentation site

`[tool.pytest.ini_options]` sets `testpaths = ["tests"]` and `addopts = "-ra"`. The runner calls `pytest -q` in the selected root. Keep application tests in the suite when adopting this project.

`zensical.toml` configures the site separately. Its source directory is `docs`, output directory is `site`, and `site_url` is `https://miguelelgallo.github.io/pythonprs/`. Change `site_url`, `repo_url`, and `repo_name` before publishing from another repository. See the [publishing guide](../how-to/publish-documentation.md).

### Site appearance

The site uses Zensical's `modern` theme with a teal accent, light and dark palettes, a check-mark logo, navigation tabs and expanded sections, a back-to-top button, and copy buttons for code blocks. The initial palette follows the system preference; the header toggle lets readers choose another.

`[project.theme]` controls the variant, features, and logo. The two `[[project.theme.palette]]` entries control light and dark mode. `[project].extra_css` loads `docs/stylesheets/extra.css` for typography, rounded cards, code blocks, and keyboard focus outlines. Keep that file when copying the site. The home page uses Zensical's responsive card grid. [Zensical theme colors](https://zensical.org/docs/setup/colors/), [card grids](https://zensical.org/docs/authoring/grids/).

## Documentation workflow

`.github/workflows/documentation.yml` runs **Documentation build** for pull requests, pushes to `main`, and manual requests. It installs the locked tools and runs `zensical build --clean --strict` with read-only repository permissions.

Artifact upload and **Publish documentation** run only when all three conditions match:

- Repository is `MiguelElGallo/pythonprs`.
- Ref is `refs/heads/main`.
- Event is not `pull_request`.

The deployment job waits for a successful build. It receives `pages: write` and `id-token: write` and uses the `github-pages` environment. Pull requests never upload a Pages artifact or deploy. Actions are pinned by commit SHA. uv's dependency cache is allowed; Zensical's site cache is not persisted. [Zensical publishing guidance](https://zensical.org/docs/publish-your-site/), [GitHub Pages workflow requirements](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages).

The workflow's repository and branch guards must be changed deliberately when publishing from another repository. See [set the destination](../how-to/publish-documentation.md#2-set-the-destination-explicitly).

Related: [commands](commands.md), [adopting the configuration](../how-to/add-github-actions.md).
