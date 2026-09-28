# GitHub Actions Workflow Analysis

Analysis of `.github/workflows/deploy.yml` ("Deploy to GitHub Pages").

## 1. What triggers this workflow to run?

The `on:` section defines two triggers:

- **`push` to `main`**: any commit pushed or merged into the `main` branch starts the workflow.
- **`pull_request` targeting `main`**: opening or updating a pull request into `main` also starts the workflow, so changes are tested before they are merged.

## 2. What are the four main steps this workflow performs?

The `build-and-test` job runs four steps:

1. **Checkout code**: downloads the repository's code onto the runner.
2. **Validate HTML**: checks the HTML files for errors using `html5validator-action`.
3. **Check links**: scans for broken links using `github-action-markdown-link-check`.
4. **Upload artifact**: packages the site files so they can be deployed to GitHub Pages.

After these pass, a separate `deploy` job runs one more step, **Deploy to GitHub Pages**, which publishes the uploaded artifact to the live site.

## 3. What does the "Checkout code" step do and why is it necessary?

The "Checkout code" step uses `actions/checkout@v4` to clone the repository onto the GitHub Actions runner (a fresh `ubuntu-latest` virtual machine). The runner starts out empty, with no copy of the project. Without this step, the validation, link-check, and upload steps would have no files to work with, and every later step would fail.

## 4. What is the purpose of the environment configuration?

The `environment` block links the `deploy` job to the `github-pages` environment:

- **`name: github-pages`** tells GitHub which deployment target this job is publishing to. It is the environment GitHub Pages uses, and it shows up under "Deployments" in the repository.
- **`url: ${{ steps.deployment.outputs.page_url }}`** takes the live site URL produced by the deploy step and attaches it to the workflow run, so the deployed link appears directly in the Actions summary and on pull requests.

Environments also make it possible to add protection rules (such as required approvals) or environment-specific secrets before a deployment runs. Together with the `permissions` block (`pages: write` and `id-token: write`), this gives the job exactly the access it needs to publish the site and nothing more.

## 5. How does this automated deployment improve reliability compared to manual deployment?

- **Nothing broken goes live**: the `deploy` job has `needs: build-and-test`, so it only runs if the HTML validation and other checks succeed. With manual deployment, someone could upload broken code by mistake.
- **Consistency**: every deployment follows the exact same steps in a clean environment, removing "it worked on my machine" problems and forgotten steps.
- **Early feedback**: tests run on every pull request, so problems are caught before code is merged into `main`.
- **Traceability**: every run is logged in the Actions tab, showing who changed what, when, and whether it passed, which makes it easy to find and fix the cause of a problem.
- **Speed**: deployment happens automatically on merge instead of waiting on a person to do it by hand.

## 6. What would happen if you pushed code to a different branch (not main)?

The `push` trigger only lists `branches: [ main ]`, so pushing to any other branch (for example `feature/about-section`) **does not trigger the workflow** and nothing is deployed.

If a pull request is then opened from that branch into `main`, the `pull_request` trigger runs the **`build-and-test`** job to check the changes. However, the **`deploy`** job is **skipped**, because of the condition:

`if: github.event_name == 'push' && github.ref == 'refs/heads/main'`

This means the live site only updates after changes are actually merged (pushed) into `main`. This matched what happened in this project: the pull request checks showed "Deploy to GitHub Pages / deploy (pull_request): Skipped."

## Additional Notes

- The **Check links** step uses `continue-on-error: true`, which means a failed link check will show a warning but will **not** stop the workflow or block deployment. Only the HTML validation failing would prevent the site from deploying.
- The link checker (`github-action-markdown-link-check`) is designed for Markdown files, so it checks links in files like `README.md` rather than the links inside `index.html`.