# Python checks you can trust

Catch complicated functions, type mistakes, and missing documentation before you merge a Python change.
{ .lead }

**pythonprs** gives you a GitHub Actions workflow and the same checks on your own computer. It also tests deliberately bad examples, so a broken check cannot quietly look successful.

## Start with your goal

<div class="grid cards start-cards" markdown>

-   **Try the checks locally**

    Run a passing check, make a temporary example fail, and repair it.

    [Start the local tutorial →](tutorials/first-check.md)

-   **See a GitHub run**

    Make a small change in your own fork and watch its pull-request checks.

    [Start the GitHub tutorial →](tutorials/first-github-run.md)

-   **Set up your project**

    Add the workflow and rules to an existing Python repository.

    [Add GitHub Actions →](how-to/add-github-actions.md)

-   **Protect your branch**

    Make passing checks a requirement before anyone merges a change.

    [Require the checks →](how-to/require-checks.md)

-   **Fix a failure**

    Find the cause of a failed job and run its check again.

    [Repair a failed check →](how-to/fix-failed-checks.md)

-   **Publish your documentation**

    Preview a Zensical site and publish it to your own GitHub Pages URL.

    [Set up publishing →](how-to/publish-documentation.md)

</div>

## Learn, do, look up, understand

These pages follow [Diátaxis](https://diataxis.fr/): each kind of documentation serves a different need.

- **Tutorials** guide you through a complete practice exercise. Start [locally](tutorials/first-check.md), then try [GitHub](tutorials/first-github-run.md).
- **How-to guides** help you finish a task: [install the workflow](how-to/add-github-actions.md), [require checks](how-to/require-checks.md), [repair failures](how-to/fix-failed-checks.md), or [publish documentation](how-to/publish-documentation.md).
- **Reference** gives you exact [check requirements](reference/checks.md), [configuration settings](reference/configuration.md), and [commands](reference/commands.md).
- **Explanation** connects the ideas: [how the checks work](explanation/how-checks-work.md), [what types and docstrings tell you](explanation/docstrings-and-types.md), and [real findings and repairs from iParq](explanation/iparq-examples.md).

## What a passing result means

Each function stays within two complexity limits. Type checks find mistakes that can be detected from the code's type information. Every function has its own documentation, with a summary above a minimum length. The tests of the checks themselves also pass.

A passing result does not prove that a program is correct or that its documentation is useful. You still review the change and test its behavior. GitHub prevents merging a failed check only after you [configure required checks](how-to/require-checks.md).

For the original tool research and review evidence, see the [maintainer record](maintainers/research-and-plan.md).
