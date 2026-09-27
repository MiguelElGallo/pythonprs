# pythonprs

Python quality checks for GitHub pull requests, with tested examples that show what passes and what fails.

**[Read the documentation](https://miguelelgallo.github.io/pythonprs/)**

The workflow checks function complexity with complexipy (maximum 15) and Ruff C901 (maximum 10), checks types with Pyrefly, and requires a docstring on every function. Function summaries need at least 8 words and 40 non-whitespace characters. A fifth job tests the checks themselves. All five jobs must pass.

From the repository root, with [uv installed](https://docs.astral.sh/uv/getting-started/installation/):

```sh
uv sync --locked
uv run --no-sync python -m scripts.quality all
```

- [Try the checks locally](docs/tutorials/first-check.md)
- [Run them on a GitHub pull request](docs/tutorials/first-github-run.md)
- [Add them to an existing project](docs/how-to/add-github-actions.md)
- [Require passing checks before merging](docs/how-to/require-checks.md)
- [Look up limits and tool versions](docs/reference/checks.md)

The documentation uses [Zensical](https://zensical.org/) and publishes through GitHub Pages. To preview it:

```sh
uv run --no-sync zensical serve
```

Open <http://localhost:8000/pythonprs/>. See the [publishing guide](docs/how-to/publish-documentation.md) to set up your own site.
