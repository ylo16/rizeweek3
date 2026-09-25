# Workflow Analysis

## 1. What triggers this workflow to run?
The workflow triggers on two events: a `push` to the `main` branch, and a `pull_request` targeting the `main` branch. This means it runs both when code is merged/pushed directly to main, and whenever someone opens or updates a PR into main - giving early feedback before the merge even happens.

## 2. What are the four main steps this workflow performs?
These are the four steps in the `build-and-test` job:
1. **Checkout code** — downloads the repository's files so later steps can access them.
2. **Validate HTML** — runs an HTML5 validator against the files to catch markup errors.
3. **Check links** — scans for broken links using a markdown link checker (set to continue even if it finds issues, via `continue-on-error: true`).
4. **Upload artifact** — packages the site files and uploads them as a deployable artifact for GitHub Pages.

(There's also a separate `deploy` job with its own step, "Deploy to GitHub Pages," but it only runs after `build-and-test` succeeds and only on a push to main.)

## 3. What does the "Checkout code" step do and why is it necessary?
It uses `actions/checkout@v4` to pull a copy of the repository's code into the GitHub Actions runner (a fresh, temporary virtual machine). It's necessary because the runner starts with an empty environment — without this step, there would be no files for the validator, link checker, or deploy step to act on.

## 4. What is the purpose of the environment configuration?
The `environment: name: github-pages` block in the `deploy` job tells GitHub this deployment targets the `github-pages` environment, which is what enables GitHub Pages deployment permissions and lets GitHub track deployment history/status for that environment. The `url: ${{ steps.deployment.outputs.page_url }}` line captures the live URL that the deploy step outputs, so it shows up as a clickable link in the GitHub Actions run summary and in the repo's deployments tab.

## 5. How does this automated deployment improve reliability compared to manual deployment?
It removes human error and inconsistency from the process. Every push or PR runs the exact same validation and link-checking steps in a clean, identical environment — nobody can forget a step, skip testing, or deploy from a machine with a different local setup ("works on my machine" problems). The `deploy` job is also gated with `needs: build-and-test`, so broken or invalid code never reaches the live site; deployment only happens if validation passes first.

## 6. What would happen if you pushed code to a different branch (not main)?
Nothing would run automatically — the `on:` trigger only fires for pushes to `main` (or PRs targeting `main`). A push to another branch, like a feature branch, wouldn't kick off the workflow at all unless that branch's changes are part of a pull request into `main`, in which case the `build-and-test` job would run (to validate the PR) but the `deploy` job still wouldn't run, since it's explicitly restricted to `github.event_name == 'push' && github.ref == 'refs/heads/main'`.