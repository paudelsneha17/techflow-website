# TechFlow Solutions Website

![Deploy to GitHub Pages](https://github.com/paudelsneha17/techflow-website/actions/workflows/deploy.yml/badge.svg)

The company website for TechFlow Solutions, a startup that builds custom websites for small businesses.

**Live site:** https://paudelsneha17.github.io/techflow-website/

## Features

- **About section** with the company story, mission, and key stats
- **Team section** with profiles of the lead developer, designer, and full-stack developer
- **Contact form** for potential clients
- Responsive layout for desktop and mobile

## Development Workflow

- Each feature is built on its own branch (for example `feature/about-section` and `feature/profiles`)
- Changes are merged into `main` through pull requests
- `main` is protected and requires a pull request before merging
- Every push to `main` runs a GitHub Actions workflow that validates the HTML, checks for broken links, and deploys the site to GitHub Pages

## Built With

- HTML5
- CSS3
- JavaScript
- GitHub Actions and GitHub Pages