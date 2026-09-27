# Publish documentation with Zensical and GitHub Pages

Build a documentation site from Markdown and publish it from your own GitHub repository. You need repository administration access to configure Pages. The project must allow GitHub Actions and GitHub Pages for its visibility and account plan.

This repository publishes at <https://miguelelgallo.github.io/pythonprs/>. Follow these steps to use your own destination.

## 1. Add the site files

If you are using the complete starter, these files are already present. Otherwise copy `docs/`, `zensical.toml`, and `.github/workflows/documentation.yml` from [pythonprs](https://github.com/MiguelElGallo/pythonprs) into your project. Merge an existing site or workflow rather than replacing it.

From your project's root, with [uv installed](https://docs.astral.sh/uv/getting-started/installation/), add the builder and install the locked environment:

```sh
uv add --dev zensical==0.0.65
uv sync --locked
```

Keep `site/` and `.cache/` in `.gitignore`. They are generated output and cache; Markdown under `docs/` is the source you commit.

## 2. Set the destination explicitly

In `zensical.toml`, change these values to your repository's owner and name:

```toml
[project]
site_url = "https://YOUR-OWNER.github.io/YOUR-REPO/"
repo_url = "https://github.com/YOUR-OWNER/YOUR-REPO"
repo_name = "YOUR-OWNER/YOUR-REPO"
```

Edit the existing `[project]` section; do not add a duplicate. Keep the trailing slash in `site_url`. For a user/organization site or custom domain, use its actual Pages URL instead. Update the README site link and any source links in your copied pages to point to the intended project.

The upload and deployment conditions in `documentation.yml` deliberately name `MiguelElGallo/pythonprs`. Replace that exact repository name in **both** conditions with yours. This prevents a fresh fork from attempting an unintended deployment.

If your publishing branch is not `main`, change `push.branches` and both `refs/heads/main` conditions to match. Review the `github-pages` environment's deployment branch policy too.

## 3. Preview and validate

```sh
uv run --no-sync zensical serve
```

Open the local address printed by Zensical. For this starter's current configuration it is <http://localhost:8000/pythonprs/>; after changing `site_url`, the preview path follows your site's path. Read a tutorial, follow internal links, and check narrow-screen navigation. Stop the server with `Ctrl+C`.

Then build what GitHub will publish:

```sh
uv run --no-sync zensical build --clean --strict
```

The generated site is in `site/`. Fix any reported warnings, including broken internal page or section links. `--clean` rebuilds without the previous site cache; `--strict` turns warnings into a failed build. [Zensical build options](https://zensical.org/docs/usage/build/).

## 4. Enable Pages and publish

In your repository, open **Settings → Pages**. Under **Build and deployment**, set **Source** to **GitHub Actions**. Ensure the `github-pages` environment permits deployment from your chosen branch.

Commit the source pages, config, workflow, `pyproject.toml`, and `uv.lock`. Push them to the publishing branch. In **Actions**, open the **Documentation** workflow and wait for **Documentation build** and **Publish documentation** to pass.

Open the URL shown by the deployment and verify the home page, a tutorial, navigation links, and styling. A successful upload alone is not a published site; the deployment job must finish. [GitHub Pages workflow setup](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages).

For the workflow's exact triggers, deployment conditions, and permissions, see the [site workflow reference](../reference/configuration.md#documentation-workflow).

## Repair a failed publication

For a build failure, run the strict build locally and repair the first warning or error. For a Pages configuration error, confirm the Source is **GitHub Actions**. For a skipped deployment, compare the repository and branch conditions with the actual run. For an environment approval requirement, follow your repository's configured approval process.

If a manual run is unavailable, ensure the workflow is on the default branch, includes `workflow_dispatch`, and you have write access. [GitHub manual runs](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/manually-run-a-workflow).

If HTML loads but links or assets fail, check `site_url` against the deployed Pages URL, including its repository path, then rebuild and push the correction.
