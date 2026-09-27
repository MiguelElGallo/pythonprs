# How the checks work

A pull request changes code that other people must read and maintain. These checks set a shared baseline: functions stay manageable, type information agrees with the code, and documentation is present. Everyone runs the same policy before deciding whether to merge.

## One policy on your computer and GitHub

A **workflow** is GitHub's recipe for running commands automatically. This project's `Python quality` workflow starts on a push, pull request, or manual request. It creates five **jobs**, each responsible for one check.

Each job installs the versions recorded in the lock and calls the same Python runner you use locally. That runner discovers the project's Python files and gives the four code checks the same list. pytest uses its configured test suite.

```text
Push or pull request
        ↓
Install the locked tools in five independent jobs
        ↓
complexity · Ruff · types · docstrings · tests
        ↓
GitHub records five pass/fail results
        ↓
Your branch rules decide whether those results permit merging
```

One failed job does not cancel the other four. You can see all the problems from a single run. The code-quality workflow needs only read access to the repository and does not deploy your application.

## Why two complexity measures?

Consider a function with many separate `if` statements. Each decision adds a possible path through it. **McCabe complexity**, checked by Ruff C901, measures these decision paths.

Now put several decisions inside one another. The reader must keep track of the surrounding conditions to understand the innermost action. **Cognitive complexity**, checked by complexipy, gives extra weight to patterns such as nesting. [Ruff C901](https://docs.astral.sh/ruff/rules/complex-structure/), [complexipy](https://github.com/rohaquinlop/complexipy).

The scores are not interchangeable. A function can pass one and fail the other. The tests include these cases and verify the exact passing and failing boundaries in the [reference](../reference/checks.md).

A low score does not make a function correct. Refactoring to pass should also make its purpose clearer, and application tests should confirm that its behavior still works.

## Why inspect the whole repository?

Checking only changed files can leave an existing violation invisible. Checking a fixed `src/` directory can miss a new root-level script or a newly added folder.

This runner finds every `.py` file outside the explicitly excluded directory names. Tests and private functions receive the same baseline as application code. The [configuration reference](../reference/configuration.md#shared-file-scope-and-documentation-policy) describes exclusions and scope limits.

This is an absolute policy, so adopting an older project may expose existing failures immediately. The setup guide asks you to review the scope and repair failures before making the checks required.

## Why test the checks themselves?

A tool can fail because it found the intended problem, because it could not start, or because its configuration was invalid. A useful negative test distinguishes these outcomes.

The deliberately bad examples are stored as `.py.txt` files. Tests copy them into temporary projects as real Python files, apply the production configuration, and require the expected diagnostic plus exit status 1. They also verify examples at the passing boundaries.

Storing bad examples as text keeps ordinary repository scans from reporting those intentional violations. The tests exercise real tool commands. Tests run Python code; the AST documentation inspection itself reads source without executing the inspected functions.

## A failed job becomes a merge rule through settings

The workflow records results. A branch ruleset decides which results must pass before merging. Without that setting, a failed workflow can still be visible while GitHub allows a merge.

[Require all five checks](../how-to/require-checks.md) to connect the policy to your merge process. The documentation site has its own build and deployment workflow; it is separate from this code policy.
