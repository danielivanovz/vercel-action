# Deploy to Vercel Action

A comprehensive GitHub Action for deploying Next.js, React, Python, and Go applications to Vercel with full monorepo support (Turbo, Nx, pnpm workspaces).

## Features

- **Next.js & React** optimized with intelligent auto-detection
- **Monorepo support** for Turbo, Nx, and pnpm workspaces
- **Python backends** with requirements.txt/pyproject.toml support
- **Go backends** with go.mod support
- **All package managers** (npm, pnpm, yarn, bun)
- **Private npm packages** with GitHub Packages authentication
- **Smart project detection** with manual override options
- **Automatic PR comments** with deployment URLs
- **Flexible configuration** for custom build workflows
- **Performance optimized** with intelligent caching

## Quick Start

### Basic Next.js App

```yaml
name: Deploy to Vercel
on: [push, pull_request]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: danielivanovz/deploy-to-vercel-action@v1
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
```

### Turbo Monorepo

```yaml
- uses: danielivanovz/deploy-to-vercel-action@v1
  with:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
    VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
    VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
    WORKING_DIRECTORY: './apps/web'
    PACKAGE_MANAGER: 'pnpm'
    TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
    TURBO_TEAM: 'my-team'
```

## Complete Input Reference

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `VERCEL_TOKEN` | Vercel deployment token | ✅ | - |
| `VERCEL_ORG_ID` | Vercel organization ID | ✅ | - |
| `VERCEL_PROJECT_ID` | Vercel project ID | ✅ | - |
| `GITHUB_TOKEN` | GitHub token for PR comments and private packages | ✅ | - |
| `ENVIRONMENT` | Deployment environment (`production`, `development`, `preview`) | ❌ | Auto-detect |
| `NODE_VERSION` | Node.js version | ❌ | `18` |
| `PYTHON_VERSION` | Python version (for Python projects) | ❌ | `3.11` |
| `GO_VERSION` | Go version (for Go projects) | ❌ | `1.21` |
| `PACKAGE_MANAGER` | Package manager (`npm`, `pnpm`, `yarn`, `bun`) | ❌ | `npm` |
| `WORKING_DIRECTORY` | Working directory for deployment | ❌ | `.` |
| `BUILD_COMMAND` | Custom build command | ❌ | Auto-detect |
| `INSTALL_COMMAND` | Custom install command | ❌ | Auto-detect |
| `PROJECT_TYPE` | Project type (`auto`, `nextjs`, `react`, `python`, `go`, `turbo`, `nx`) | ❌ | `auto` |
| `TURBO_TEAM` | Turbo team for remote caching | ❌ | - |
| `TURBO_TOKEN` | Turbo token for remote caching | ❌ | - |
| `NPM_SCOPE` | NPM scope for private packages (e.g., `@mycompany`) | ❌ | `@*` |

## Outputs

| Output | Description |
|--------|-------------|
| `url` | The deployment URL |
| `environment` | The deployment environment used |
| `project_type` | The detected project type |

## Project Type Detection

The action automatically detects your project type:

| Detection | Project Type | Build Strategy |
|-----------|-------------|----------------|
| `turbo.json` found | `turbo` | Turbo monorepo with remote caching |
| `nx.json` found | `nx` | Nx monorepo with workspace support |
| `requirements.txt` or `pyproject.toml` | `python` | Python with pip/poetry |
| `go.mod` found | `go` | Go with module support |
| `next.config.*` found | `nextjs` | Next.js optimized build |
| `package.json` with React | `react` | React SPA build |
| Default | `nextjs` | Safe fallback |

## Environment Detection

When `ENVIRONMENT` is not provided, auto-detects based on:

| Condition | Environment |
|-----------|-------------|
| Push to `main`/`master` | `production` |
| Push to `develop`/`development` | `development` |
| Pull requests | `preview` |
| Other branches | `preview` |

## Use Cases & Examples

### 1. Turbo Monorepo with Multiple Apps

```yaml
jobs:
  deploy-web:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy Web App
        uses: danielivanovz/deploy-to-vercel-action@v1
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_WEB_PROJECT_ID }}
          WORKING_DIRECTORY: './apps/web'
          PACKAGE_MANAGER: 'pnpm'
          TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
          BUILD_COMMAND: 'pnpm turbo build --filter=web'

  deploy-docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy Documentation
        uses: danielivanovz/deploy-to-vercel-action@v1
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_DOCS_PROJECT_ID }}
          WORKING_DIRECTORY: './apps/docs'
          PACKAGE_MANAGER: 'pnpm'
          BUILD_COMMAND: 'pnpm turbo build --filter=docs'
```

### 2. Nx Monorepo

```yaml
- name: Deploy Nx App
  uses: danielivanovz/deploy-to-vercel-action@v1
  with:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
    VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
    VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
    WORKING_DIRECTORY: './apps/frontend'
    PROJECT_TYPE: 'nx'
    BUILD_COMMAND: 'npx nx build frontend --configuration=production'
```

### 3. Python Backend + Next.js Frontend

```yaml
jobs:
  deploy-api:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy Python API
        uses: danielivanovz/deploy-to-vercel-action@v1
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_API_PROJECT_ID }}
          WORKING_DIRECTORY: './api'
          PROJECT_TYPE: 'python'
          PYTHON_VERSION: '3.11'

  deploy-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy Next.js Frontend
        uses: danielivanovz/deploy-to-vercel-action@v1
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_WEB_PROJECT_ID }}
          WORKING_DIRECTORY: './web'
          NODE_VERSION: '20'
```

### 4. Go Backend

```yaml
- name: Deploy Go API
  uses: danielivanovz/deploy-to-vercel-action@v1
  with:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
    VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
    VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
    WORKING_DIRECTORY: './api'
    PROJECT_TYPE: 'go'
    GO_VERSION: '1.21'
```

### 5. Custom Build with Bun

```yaml
- name: Deploy with Bun
  uses: danielivanovz/deploy-to-vercel-action@v1
  with:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
    VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
    VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
    PACKAGE_MANAGER: 'bun'
    NODE_VERSION: '20'
    BUILD_COMMAND: 'bun run build:production'
    INSTALL_COMMAND: 'bun install --production=false'
```

### 6. Private Packages with Scoped Registry

```yaml
- name: Deploy with Private Packages
  uses: danielivanovz/deploy-to-vercel-action@v1
  with:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
    VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
    VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
    NPM_SCOPE: '@mycompany'  # Only @mycompany packages from GitHub
```

## Monorepo Best Practices

### Turbo Configuration

1. **Repository structure:**
```
my-turbo-repo/
├── apps/
│   ├── web/          # Next.js app
│   └── docs/         # Documentation site
├── packages/
│   └── ui/           # Shared components
├── turbo.json
└── package.json
```

2. **Multiple deployments:**
```yaml
strategy:
  matrix:
    app: [web, docs, admin]
    include:
      - app: web
        project_id: VERCEL_WEB_PROJECT_ID
        directory: ./apps/web
      - app: docs
        project_id: VERCEL_DOCS_PROJECT_ID
        directory: ./apps/docs

steps:
  - uses: danielivanovz/deploy-to-vercel-action@v1
    with:
      VERCEL_PROJECT_ID: ${{ secrets[matrix.project_id] }}
      WORKING_DIRECTORY: ${{ matrix.directory }}
      TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
```

### Nx Configuration

1. **Selective deployment:**
```yaml
- name: Deploy affected apps
  uses: danielivanovz/deploy-to-vercel-action@v1
  with:
    BUILD_COMMAND: 'npx nx affected:build --base=origin/main'
```

## Private NPM Packages

The action supports private packages from GitHub Packages:

### Global Scope (all scoped packages)
```yaml
NPM_SCOPE: # Not specified - all @* packages go to GitHub
```

### Specific Organization
```yaml
NPM_SCOPE: '@mycompany'  # Only @mycompany packages from GitHub
```

This creates appropriate `.npmrc` configuration:
```
//npm.pkg.github.com/:_authToken=YOUR_TOKEN
@mycompany:registry=https://npm.pkg.github.com
registry=https://registry.npmjs.org/
```

## Setup Guide

### 1. Get Vercel Credentials

```bash
# Install Vercel CLI
npm i -g vercel

# Login and link project
vercel login
vercel link

# Get project info
cat .vercel/project.json
```

### 2. Create GitHub Secrets

Required secrets in your repository:

- `VERCEL_TOKEN` - Get from [Vercel Dashboard](https://vercel.com/account/tokens)
- `VERCEL_ORG_ID` - From `.vercel/project.json`
- `VERCEL_PROJECT_ID` - From `.vercel/project.json`
- `GITHUB_TOKEN` - Automatically available or use PAT

Optional for Turbo:
- `TURBO_TOKEN` - From `npx turbo login`
- `TURBO_TEAM` - Your Turbo team name

### 3. Repository Structure Examples

**Turbo Monorepo:**
```
my-app/
├── apps/
│   ├── web/
│   │   ├── package.json
│   │   └── next.config.js
│   └── api/
│       ├── requirements.txt
│       └── main.py
├── packages/
│   └── ui/
├── turbo.json
└── package.json
```

**Multi-language:**
```
fullstack-app/
├── frontend/
│   ├── package.json
│   └── next.config.js
├── backend/
│   ├── go.mod
│   └── main.go
└── .github/workflows/
```

## Performance Optimizations

The action includes several performance optimizations:

1. **Smart caching** - Language-specific cache strategies
2. **Conditional steps** - Only runs steps needed for your project type
3. **Parallel builds** - Multiple apps can deploy in parallel
4. **Turbo remote caching** - Leverages Turbo's build cache
5. **Package manager optimization** - Uses fastest commands for each manager

## Troubleshooting

### Common Issues

**1. Cache issues with bun:**
```yaml
# Fixed automatically in v1+
cache: ${{ inputs.PACKAGE_MANAGER != 'bun' && inputs.PACKAGE_MANAGER || '' }}
```

**2. Monorepo detection:**
```yaml
# Explicitly set project type
PROJECT_TYPE: 'turbo'
WORKING_DIRECTORY: './apps/web'
```

**3. Private package authentication:**
```yaml
# Use specific scope
NPM_SCOPE: '@mycompany'
```

**4. Custom builds not working:**
```yaml
# Use both custom commands
BUILD_COMMAND: 'bun run build:prod'
INSTALL_COMMAND: 'bun install --no-save'
```

### Debug Mode

Enable detailed logging:

```yaml
- name: Deploy with Debug
  uses: danielivanovz/deploy-to-vercel-action@v1
  env:
    ACTIONS_STEP_DEBUG: true
  with:
    # ... your config
```

## License

MIT License - see [LICENSE](LICENSE) file for details.
