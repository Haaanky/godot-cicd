# CI/CD for GitHub Pages — Claude Instructions

This repository (`haaanky/godot-cicd`) is the **central CI/CD template library** for all GitHub Pages projects owned by haaanky. Every GitHub Pages repo should use these workflows for a consistent deploy, preview, and monitoring experience.

---

## Available Workflow Templates

### Generic (any project type)

| File | Purpose |
|---|---|
| `pages-deploy.yml` | Deploy any pre-built static site to GitHub Pages + run Playwright E2E tests |
| `pages-preview.yml` | Deploy branch preview to `/preview/{slug}/` + post URL to PR |
| `godot-preview-cleanup.yml` | Remove preview directory when PR is closed |
| `playwright-iterate.yml` | Scheduled Playwright monitoring — auto-open/close GitHub issues |

### Godot 4 specific

| File | Purpose |
|---|---|
| `godot-export-deploy.yml` | Full Godot 4 export → Web → GitHub Pages + E2E tests (self-contained) |
| `godot-preview.yml` | Godot branch preview build + deploy + PR comment with WASM/PCK sizes |

---

## Setting Up a New GitHub Pages Repository

### 1. Choose the right templates

- **Godot 4 game** → use `godot-export-deploy.yml` + `godot-preview.yml` + `godot-preview-cleanup.yml` + `playwright-iterate.yml`
- **Any other static site** (Next.js, Hugo, plain HTML, docs…) → use `pages-deploy.yml` + `pages-preview.yml` + `godot-preview-cleanup.yml` + `playwright-iterate.yml`

### 2. Copy workflow templates

```bash
mkdir -p .github/workflows
# Copy the templates you need from workflow-templates/ into .github/workflows/
```

### 3. Add trigger workflows

Each template uses `workflow_call`. You need a separate trigger workflow in the consuming repo.

**Example `deploy.yml` for a generic static site:**

```yaml
name: Deploy

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: npm run build           # replace with your build command
      - uses: actions/upload-artifact@v4
        with:
          name: site
          path: dist/                # replace with your output directory

  deploy:
    needs: build
    uses: ./.github/workflows/pages-deploy.yml
    with:
      artifact_name: site
      site_url: "https://haaanky.github.io/your-repo"
```

**Example `preview.yml` for branch previews:**

```yaml
name: Branch Preview

on:
  push:
    branches: ["claude/**"]
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: site
          path: dist/

  preview:
    needs: build
    uses: ./.github/workflows/pages-preview.yml
    with:
      artifact_name: site
```

**Example `preview-cleanup.yml`:**

```yaml
name: Preview Cleanup

on:
  pull_request:
    types: [closed]

permissions:
  contents: write

jobs:
  cleanup:
    uses: ./.github/workflows/godot-preview-cleanup.yml
    with:
      branch_prefix: "claude/"
```

**Example `playwright-monitor.yml` for scheduled monitoring:**

```yaml
name: Playwright Monitor

on:
  schedule:
    - cron: "*/30 * * * *"
  workflow_dispatch:

permissions:
  contents: read
  issues: write

jobs:
  test:
    uses: ./.github/workflows/playwright-iterate.yml
    with:
      game_url: "https://haaanky.github.io/your-repo"
```

### 4. Add E2E test files

Copy `config/package.json` and `config/playwright.config.js` to the repo root.
Copy `tests/e2e/game.test.js` to `tests/playwright/` and adapt it for the site.

The test file reads `GAME_URL` from the environment — no hardcoded URLs needed.

---

## Conventions (apply to ALL repos)

| Convention | Value |
|---|---|
| Preview branch pattern | `claude/**` |
| Preview URL pattern | `https://haaanky.github.io/{repo}/preview/{branch-slug}/` |
| Preview PR comment marker | `<!-- preview-bot -->` (one comment, updated in-place) |
| Playwright failure label | `playwright-failure` |
| E2E tests location | `tests/playwright/` |
| Playwright config | `playwright.config.js` at project root |
| Node version | 20 |
| Playwright version | as defined in `config/package.json` |

---

## When Working in a Consuming Repo

When asked to set up or modify CI/CD in a GitHub Pages repository:

1. **Check what project type it is** (Godot, Next.js, Hugo, etc.)
2. **Use templates from `haaanky/godot-cicd`** — don't invent new workflows from scratch
3. **Copy, don't reference** — workflow files are copied into the consuming repo's `.github/workflows/`, not referenced via `uses: haaanky/godot-cicd/...@main`
4. **Set the correct `site_url` / `game_url`** — always `https://haaanky.github.io/{repo}`
5. **Keep workflow versions in sync** — if a template changes in this repo, update it in consuming repos too

---

## Updating Templates Across Repos

When a workflow template is changed in this repo:
1. Update the template file here and push
2. Identify which consuming repos use that template
3. Copy the updated file to each consuming repo
4. Test in one repo before rolling out to all

---

## Repository Notes

Add consuming repo-specific notes here as they are onboarded:

<!-- Example:
### haaanky/my-game
- Uses: godot-export-deploy.yml, godot-preview.yml, godot-preview-cleanup.yml, playwright-iterate.yml
- Godot version: 4.6.1
- Site URL: https://haaanky.github.io/my-game
-->
