# Fix a failed check

Open the failed job in the pull request's **Checks** tab. Read the first diagnostic that names your source file and line. Then reproduce that job from your repository root with the command below.

If you have just pulled a change to dependencies, run `uv sync --locked` first.

## Complexity

```sh
uv run --no-sync python -m scripts.quality complexity
```

Find the function reported above the [cognitive complexity limit](../reference/checks.md). Reduce nesting with an early return, split distinct tasks into named functions, or simplify repeated decisions. Preserve the behavior with application tests, then rerun the check.

## Ruff lint or formatting

```sh
uv run --no-sync python -m scripts.quality ruff
```

For `C901`, simplify the reported function's decision paths. For other rule codes, follow the diagnostic: for example, remove an unused import or add a missing type annotation.

If the output says a file **would be reformatted**, apply formatting to that file and review the result:

```sh
uv run --no-sync ruff format path/to/file.py
uv run --no-sync python -m scripts.quality ruff
```

Replace `path/to/file.py` with the reported path. Formatting is separate from linting; both must pass. The CI command checks formatting without editing files.

## Types

```sh
uv run --no-sync python -m scripts.quality types
```

Compare the expected type with the supplied value. Add parameter and return annotations where missing. For a value that could be `None`, check for `None` before using it. For a missing third-party import, install the application's dependency and update its lock.

An annotation should describe the value your program really accepts or returns. Changing an annotation solely to silence an error can hide a bug. See [what types tell you](../explanation/docstrings-and-types.md).

## Docstrings

```sh
uv run --no-sync python -m scripts.quality docstrings
```

| Diagnostic | Repair |
| --- | --- |
| `DOC001` | Add a nonempty docstring as the first statement in the reported module, class, or function. |
| `DOC002` | Explain the function's purpose in the first paragraph. Meet both summary minimums. |
| `DOC003` | Fix invalid syntax, encoding, file access, or a symbolic-link Python file so it can be inspected. |

For example:

```python
def increment(value: int) -> int:
    """Add one to the supplied integer and return the result."""
    return value + 1
```

Long `Args` or `Returns` sections after a blank line cannot make up for a short first paragraph. Each function needs its own docstring, including private functions, constructors, and overrides.

Interrogate may separately report missing documentation coverage. Fix that reported definition too; the job requires both tools to pass.

## Tests

```sh
uv run --no-sync python -m scripts.quality tests
```

Read the failed assertion. The gate tests expect certain bad examples to fail with a specific diagnostic. A failing assertion means the check's behavior has changed or the test setup failed.

Run the affected test while investigating:

```sh
uv run --no-sync pytest path/to/test_file.py::test_name -q
```

Replace `path/to/test_file.py` with the failed test file and `test_name` with the function name reported by pytest. If you intentionally changed the policy, update its boundary examples and assertions together. Also repair any failed application tests in an adopted project.

## Setup or configuration errors

For `Quality configuration or scan error`, check the reported setting and ensure the selected root contains `pyproject.toml` and at least one Python file. For a missing executable, rerun `uv sync --locked`. See the [command exit behavior](../reference/commands.md#exit-status).

After repairing the cause, run `uv run --no-sync python -m scripts.quality all` and push the fix. The production runner ignores inline suppression comments; repair the code or make an explicitly reviewed policy change in `pyproject.toml`.
