# What docstrings and types tell you

Types describe the kinds of values a function accepts and returns. Documentation explains its purpose and behavior. Both help someone use the function correctly.

```python
def increment(value: int) -> int:
    """Add one to the supplied integer and return the result."""
    return value + 1
```

`value: int` is a **type annotation**: the input is expected to be an integer. `-> int` describes the return value. The triple-quoted string is the **docstring**: it explains the operation in words.

## Types can expose a mistake before execution

If code calls `increment("three")`, a type checker can report that a string does not match the expected integer. It does not need to run that call to notice the mismatch.

Pyrefly checks the annotated code and reports practical mistakes such as incompatible assignments, arguments, and returns. Ruff and Pyrefly also enforce the configured annotation requirements so functions have information the checker can use.

Annotations do not automatically validate values at runtime. Explicit `Any`, missing type information in dependencies, and external inputs can leave gaps. Keep application tests and validate inputs where your program needs to. [Python's typing documentation](https://docs.python.org/3/library/typing.html), [Pyrefly documentation](https://pyrefly.org/en/docs/).

## Why Interrogate plus an AST checker?

**Interrogate is an installed third-party package.** It finds documented and undocumented definitions and reports coverage.

**The AST checker is custom code in this project.** It uses Python's built-in `ast` module to read the structure of source files. An AST, or abstract syntax tree, represents definitions and statements without running the program. [Python AST documentation](https://docs.python.org/3/library/ast.html#ast.get_docstring).

Interrogate answers “how much is documented?” The custom checker answers “does every required definition have its own docstring, and is every function summary long enough?”

The direct presence check also closes a coverage-rounding gap: 199 documented objects out of 200 is 99.5%, which Interrogate can round to 100 before checking its threshold. A missing definition still fails the AST check. [Interrogate 1.7.0 coverage implementation](https://github.com/econchick/interrogate/blob/1.7.0/src/interrogate/coverage.py).

The two checks do not require a particular `Args`/`Returns` markup style. Interrogate's `sphinx` setting keeps constructor coverage separate from class coverage; it is not a requirement to write Sphinx markup.

## Why measure the first paragraph?

A summary should tell a reader what a function does before they study its parameter details. Counting the whole docstring would let a one-word summary pass simply because it had a long argument list below it.

The chosen minimum applies to the first paragraph only. That paragraph can span multiple lines, but a blank line ends it. Both the word and character counts must meet the [documented minimums](../reference/checks.md#documentation-requirements).

Private helpers, constructors, nested functions, and overrides need their own summaries. Modules and classes need nonempty documentation too, as an additional project convention.

## Enough text is only a starting point

The minimum length is a project choice. Eight words can explain a small operation well, or say very little with plenty of filler. The checker cannot decide which is true.

During review, ask whether the documentation explains the real purpose, inputs, result, and any important side effects or failures. Keep it accurate as the implementation changes. A passing length check helps prevent empty or very short documentation; it does not replace that judgment.

Related: [repair documentation failures](../how-to/fix-failed-checks.md#docstrings), [exact requirements](../reference/checks.md).
