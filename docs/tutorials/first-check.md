# Your first local check

Run the checks, make one function fail, and repair its documentation. You will finish with a passing result and see exactly what the documentation rule measures.

You need Git, [uv](https://docs.astral.sh/uv/getting-started/installation/), and a terminal using a POSIX shell, such as bash or zsh on Linux or macOS. You do not need to install the quality tools separately. uv installs the locked tools and obtains Python 3.12 if needed.

## 1. Get the example project

Run these commands in a directory where you keep projects:

```sh
git clone https://github.com/MiguelElGallo/pythonprs.git
cd pythonprs
uv sync --locked
```

Keep this terminal in the `pythonprs` directory for the rest of the exercise. It contains `pyproject.toml`, which defines the rules, and `uv.lock`, which fixes the installed versions.

## 2. Run a passing check

```sh
uv run --no-sync python -m scripts.quality docstrings
```

A **docstring** is the string at the beginning of a function that explains what it does. The check first reports documentation coverage, then prints a line ending in:

```text
0 violations.
```

The command succeeds. You have checked every Python file in the project, including the tests.

## 3. Make a temporary example fail

Create a practice directory outside the project and give it the same rules:

```sh
example_dir=$(mktemp -d)
cp pyproject.toml "$example_dir/pyproject.toml"
cp tests/fixtures/short_docstring.py.txt "$example_dir/example.py"
uv run --no-sync python -m scripts.quality docstrings --root "$example_dir"
```

This failure is expected. The output contains `DOC002` and says the summary requires at least 8 words and 40 non-whitespace characters.

To find the file in your editor, print its location:

```sh
echo "$example_dir/example.py"
```

The file contains this function:

```python
def increment(value: int) -> int:
    """Increment.

    Args:
        value: The supplied integer input used for this increment operation.

    Returns:
        The supplied integer input plus one additional unit of increment.
    """
    return value + 1
```

It already has documentation. Its first paragraph, `Increment.`, is too short. The longer paragraphs below it do not count toward the summary minimum.

## 4. Repair the summary

In that temporary file, replace only `Increment.` with:

```text
Add one to the supplied integer and return the result.
```

Save the file and run the same command again:

```sh
uv run --no-sync python -m scripts.quality docstrings --root "$example_dir"
```

You should now see:

```text
Documentation: 1 files, 1 functions, 0 violations.
```

The practice function passes. You have made a documentation failure and fixed its cause without changing the project's own source files.

## 5. Check the whole project

```sh
uv run --no-sync python -m scripts.quality all
```

All five checks pass, including the tests that create bad examples in temporary directories. The temporary practice directory is yours to remove when you no longer need it.

Continue with [your first GitHub run](first-github-run.md). For the exact summary rule, see [check requirements](../reference/checks.md).
