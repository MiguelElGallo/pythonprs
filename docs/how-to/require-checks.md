# Require passing checks before merging

Configure a GitHub branch ruleset so a failing quality check prevents merging into your default branch. You need repository administration access or permission to edit rulesets. First complete a successful push or pull-request run in that repository.

## Create an active branch ruleset

1. Open your repository's **Settings → Rules → Rulesets**.
2. Choose **New ruleset → New branch ruleset** and give it a name such as `Python quality`.
3. Set **Enforcement status** to **Active**.
4. Under target branches, add **Include default branch**. Verify this selects the branch you actually merge into.
5. Enable **Require a pull request before merging**.
6. Enable **Require status checks to pass**, then add all five checks from the table below.
7. Where the check selector offers an expected source, select **GitHub Actions**. Review the bypass list: people or apps listed there may bypass the rules. Save the ruleset.

GitHub's [ruleset creation guide](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/creating-rulesets-for-a-repository) explains the available settings and permissions.

## Select the exact job names

| Required check | What must pass |
| --- | --- |
| `Quality (complexity)` | complexipy |
| `Quality (ruff)` | Ruff lint and formatting |
| `Quality (types)` | Pyrefly |
| `Quality (docstrings)` | Interrogate and the AST checker |
| `Quality (tests)` | pytest |

`Python quality` is the workflow name. Select the five job names above as required checks. Keep these names unique across your workflows so GitHub can identify the intended results.

If a check does not appear, run the workflow through a push or pull request and wait for it to finish. GitHub requires a successful check in the repository within the past seven days before it can be selected. A manual `workflow_dispatch` run can demonstrate execution, but does not qualify as a required pull-request check. [GitHub's required-check troubleshooting](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/troubleshooting-required-status-checks).

## Verify enforcement

Open a practice pull request against the protected branch. On the pull request's source branch, change a function's summary to `"""Short."""` and push the change. Confirm that `Quality (docstrings)` fails and merging is blocked for a user without bypass permission.

Restore the useful summary, push again, and confirm the latest commit passes. Close the practice pull request when done.

If you use a merge queue, first add the `merge_group` trigger described in the [setup guide](add-github-actions.md). Requiring a check without triggering it for the merge group leaves the queue waiting.
