# godot-cicd

Reusable GitHub Actions workflow templates for **all GitHub Pages projects** — generic static sites and Godot 4 games.

See [CLAUDE.md](CLAUDE.md) for full usage conventions and onboarding instructions for new repos.

---

## Workflow Templates

### Generic (any project type)

| Workflow | Purpose |
|---|---|
| `pages-deploy.yml` | Deploy any pre-built static site to GitHub Pages + Playwright E2E tests |
| `pages-preview.yml` | Branch preview builds → `/preview/{slug}/` + PR comment |
| `godot-preview-cleanup.yml` | Remove preview directory when PR is closed |
| `playwright-iterate.yml` | Scheduled E2E monitoring, auto-open/close issues on failure |

### Godot 4 specific

| Workflow | Purpose |
|---|---|
| `godot-export-deploy.yml` | Full Godot 4 export → Web → GitHub Pages + E2E tests |
| `godot-preview.yml` | Godot preview build + deploy + PR comment with WASM/PCK sizes |

---

## Quick Start

### Generic static site (Next.js, Hugo, plain HTML, docs…)

1. Copy to `.github/workflows/` in your repo:
   - `pages-deploy.yml`
   - `pages-preview.yml`
   - `godot-preview-cleanup.yml`
   - `playwright-iterate.yml`

2. Create a trigger workflow that runs your build and calls `pages-deploy.yml`:

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
      - run: npm run build              # your build command
      - uses: actions/upload-artifact@v4
        with:
          name: site
          path: dist/                   # your output directory

  deploy:
    needs: build
    uses: ./.github/workflows/pages-deploy.yml
    with:
      artifact_name: site
      site_url: "https://your-org.github.io/your-repo"
```

### Godot 4 game

1. Copy to `.github/workflows/`:
   - `godot-export-deploy.yml`
   - `godot-preview.yml`
   - `godot-preview-cleanup.yml`
   - `playwright-iterate.yml`

2. Create a trigger workflow:

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

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  export:
    uses: ./.github/workflows/godot-export-deploy.yml
    with:
      godot_version: "4.6.1"
      godot_release: "stable"
      game_url: "https://your-org.github.io/your-repo"
      run_gut_tests: true
```

---

## E2E Tests

Copy `config/package.json` and `config/playwright.config.js` to your project root.
Copy `tests/e2e/game.test.js` to `tests/playwright/` and adapt to your site.

The test file reads `GAME_URL` from the environment — no hardcoded URLs.

---

## Directory Structure

```
godot-cicd/
├── CLAUDE.md                          # Usage conventions and onboarding guide
├── workflow-templates/
│   ├── pages-deploy.yml               # Generic: deploy any static site
│   ├── pages-preview.yml              # Generic: branch preview builds
│   ├── godot-preview-cleanup.yml      # Generic: remove preview on PR close
│   ├── playwright-iterate.yml         # Generic: scheduled E2E monitoring
│   ├── godot-export-deploy.yml        # Godot 4: export + deploy
│   └── godot-preview.yml             # Godot 4: preview builds
├── tests/
│   └── e2e/
│       └── game.test.js               # Template E2E test — copy and adapt
├── config/
│   ├── playwright.config.js           # Template Playwright config
│   └── package.json                   # Template package.json
└── README.md
```

---

## Conventions

| Item | Value |
|---|---|
| Preview branch pattern | `claude/**` |
| Preview URL | `https://{owner}.github.io/{repo}/preview/{branch-slug}/` |
| Playwright failure label | `playwright-failure` |
| E2E tests location | `tests/playwright/` |
| Node version | 20 |
