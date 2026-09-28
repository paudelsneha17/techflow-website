# GitHub Actions Workflow Analysis

This is my analysis of the `.github/workflows/deploy.yml` file, which is called "Deploy to GitHub Pages."

## 1. What triggers this workflow to run?

There are two triggers in the `on:` section. The first one is a push to the `main` branch, so any time I push or merge code into main, the workflow runs. The second one is a pull request into `main`. This means my changes get tested before I merge them, not just after.

## 2. What are the four main steps this workflow performs?

The `build-and-test` job has four steps:

1. **Checkout code** gets a copy of my repository onto the runner.
2. **Validate HTML** checks my HTML files for errors using `html5validator-action`.
3. **Check links** looks for broken links using `github-action-markdown-link-check`.
4. **Upload artifact** packages the site files so they are ready to deploy.

If all of these pass, a separate `deploy` job runs and publishes the site to GitHub Pages.

## 3. What does the "Checkout code" step do and why is it necessary?

This step uses `actions/checkout@v4` to copy my repository onto the runner. The runner is a brand new `ubuntu-latest` virtual machine, so it starts out empty. Without this step, it would not have my files, and the validation, link check, and upload steps would have nothing to work with. Every step after it would fail.

## 4. What is the purpose of the environment configuration?

The `environment` section connects the `deploy` job to the `github-pages` environment. The `name: github-pages` part tells GitHub where the site is being deployed, and it is why my deployments show up under "Deployments" on the repo page. The `url` part grabs the live site link from the deploy step and shows it in the Actions summary, so I can click right to my site after it deploys.

Environments can also have protection rules, like needing approval before a deployment. Along with the `permissions` section (`pages: write` and `id-token: write`), this gives the job only the access it needs to publish the site.

## 5. How does this automated deployment improve reliability compared to manual deployment?

The biggest thing is that broken code does not go live. The `deploy` job has `needs: build-and-test`, so it only runs if the checks pass. If I deployed by hand, I could upload a mistake without noticing.

It is also more consistent. The same steps run the same way every time, so I can't forget a step. I also get feedback early because tests run on pull requests before anything is merged. Every run is saved in the Actions tab, so if something breaks I can see what changed and when. And it is faster, since the site updates on its own when I merge.

## 6. What would happen if you pushed code to a different branch (not main)?

The push trigger only includes `branches: [ main ]`, so pushing to another branch like `feature/about-section` would not run the workflow and nothing would deploy.

If I opened a pull request from that branch into main, the `build-and-test` job would run to check my changes. The `deploy` job would still be skipped because of this condition:

`if: github.event_name == 'push' && github.ref == 'refs/heads/main'`

So the live site only updates after the changes are merged into main. I saw this happen in my own pull requests, where the deploy check said "Skipped."
