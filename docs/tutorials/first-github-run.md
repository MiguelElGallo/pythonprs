# Your first GitHub run

Make a small change in your own copy of pythonprs and watch the five quality checks run on a pull request.

You need a GitHub account. This exercise uses GitHub's web editor, so you do not need a local installation. A **pull request** proposes merging a branch's changes into another branch.

## 1. Make your own copy

Open [MiguelElGallo/pythonprs](https://github.com/MiguelElGallo/pythonprs) and choose **Fork**. Create the fork under your account and keep `main` as its default branch.

Open the **Actions** tab in your fork. If GitHub asks you to enable workflows, enable them. GitHub disables workflows in new forks until you do this. [GitHub's fork workflow behavior](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#workflows-in-forked-repositories).

## 2. Change one example

In your fork, open `examples/good.py` and choose the pencil icon to edit it. Find the `transitions` dictionary in `normalize_status` and add this line after `"running": "done",`. Keep the same indentation as the neighboring entries:

```python
        "done": "archived",
```

The surrounding dictionary becomes:

```python
    transitions = {
        "new": "queued",
        "queued": "running",
        "running": "done",
        "done": "archived",
        "failed": "retry",
    }
```

Choose **Commit changes**. Enter `Handle completed status` as the message, select **Create a new branch for this commit and start a pull request**, and name the branch `try-quality-checks`.

## 3. Open a pull request in your fork

On the pull-request form, set the **base repository to your own fork** and the base branch to `main`. The compare branch is `try-quality-checks` in your fork. GitHub can initially suggest the upstream repository; change it before submitting.

Choose **Create pull request**.

## 4. Read the results

Open the pull request's **Checks** tab. Look for:

```text
Quality (complexity)
Quality (ruff)
Quality (types)
Quality (docstrings)
Quality (tests)
```

Each check is a separate job: a group of steps that installs the tools and runs one part of the policy. Open a job to see its output. Wait for all five to show a successful result.

You may also see a **Documentation build** check. That belongs to the documentation site and does not publish changes from a pull request.

## 5. Finish with a passing change

Your change passes because it adds one dictionary entry without adding decisions, changing types, or removing documentation. You can close this practice pull request when finished.

The checks now run in your fork. To make GitHub prevent merging a failure, follow [require the checks](../how-to/require-checks.md). A green run and a required check are separate settings.
