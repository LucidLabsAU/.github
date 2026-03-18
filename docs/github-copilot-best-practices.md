# GitHub & Copilot Best Practices

> Patterns and configurations extracted from LucidLabsAU repos for consistent developer experience across all projects.

## Table of Contents

1. [VS Code Configuration](#vs-code-configuration)
2. [Copilot Instructions](#copilot-instructions)
3. [Custom Prompts](#custom-prompts)
4. [Copilot Coding Agent](#copilot-coding-agent)
5. [CI/CD Quality Gates](#cicd-quality-gates)
6. [Dependency Management](#dependency-management)
7. [Security](#security)
8. [Agentic Workflows](#agentic-workflows)
9. [Claude Code](#claude-code)

---

## VS Code Configuration

### GitHub MCP Server (`.vscode/mcp.json`)

Enables Copilot Chat to access GitHub context (issues, PRs, code search):

```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    }
  }
}
```

**Required for all repos.** This is tech-stack agnostic and enables richer Copilot Chat interactions.

### Editor Settings (`.vscode/settings.json`)

Minimum recommended settings:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll": "explicit"
  },
  "github.copilot.chat.codeGeneration.instructions": [
    { "file": ".github/copilot-instructions.md" }
  ]
}
```

**Notes:**

- Only include the `github.copilot.chat.codeGeneration.instructions` reference if the repo has a `.github/copilot-instructions.md` file
- Adapt `editor.defaultFormatter` to match the project's formatter (Prettier, Black, dotnet-format, etc.)
- Add language-specific settings as needed (e.g., `[python]`, `[typescript]`, `[powershell]`)

### Extensions (`.vscode/extensions.json`)

Universal recommendations for all repos:

```json
{
  "recommendations": [
    "github.copilot",
    "github.copilot-chat"
  ]
}
```

**Merge with existing recommendations** — never overwrite. Add project-specific extensions alongside:

| Stack | Additional Extensions |
|-------|----------------------|
| TypeScript/React | `esbenp.prettier-vscode`, `dbaeumer.vscode-eslint` |
| Python | `ms-python.python`, `charliermarsh.ruff` |
| PowerShell | `ms-vscode.powershell` |
| .NET | `ms-dotnettools.csdevkit` |
| Flutter/Dart | `dart-code.dart-code`, `dart-code.flutter` |

---

## Copilot Instructions

### Central Instructions (`.github/copilot-instructions.md`)

The main instruction file that Copilot reads for every interaction. Include:

- **Project overview** — what the project does, tech stack
- **Essential commands** — build, test, lint, deploy
- **Architecture** — folder structure, key patterns
- **Code style** — naming conventions, formatting rules
- **Security rules** — secrets handling, input validation

**Template:**

````markdown
# Project Name

## Overview
Brief description of what this project does.

## Tech Stack
- Language/Framework
- Key libraries

## Essential Commands
- `npm run build` — Build for production
- `npm test` — Run test suite
- `npm run lint` — Lint validation

## Architecture
```text
src/
├── components/
├── pages/
└── lib/
```

## Code Style
- Use TypeScript strict mode
- Prefer functional components
- Follow existing naming conventions

## Security
- Never hardcode secrets
- Use environment variables for sensitive config
- Validate all user inputs
````

### Path-Specific Instructions (`.github/instructions/*.instructions.md`)

For targeted guidance that only applies to specific file patterns. Uses `applyTo:` frontmatter:

```markdown
---
applyTo: "client/src/components/**/*.tsx"
---
# Component Guidelines
- Use TypeScript with explicit prop types
- Prefer composition over inheritance
- Keep components focused and single-purpose
```

**Categories to consider:**

- Security rules for auth/API files
- Styling conventions for CSS/UI files
- Build configuration for config files
- Test patterns for test files
- Image handling for asset files

---

## Custom Prompts

### Prompt Files (`.github/prompts/*.prompt.md`)

Reusable prompts that team members can invoke from Copilot Chat. Uses `agent:` frontmatter:

```markdown
---
agent: "copilot"
---
# Validation Suite

Run the full validation suite and report results:
1. `npm run build` — check for build errors
2. `npm run check` — TypeScript type checking
3. `npm run lint` — ESLint validation
4. `npm run format` — formatting check

Report pass/fail for each step.
```

**Use cases:**

- Validation/CI checks
- Code review checklists
- Component scaffolding
- Security audits

---

## Copilot Coding Agent

### Setup Steps (`.github/workflows/copilot-setup-steps.yml`)

Pre-seeds the agent's environment with the correct runtime and dependencies:

#### Node.js Example

```yaml
name: "Copilot Setup Steps"
on: workflow_dispatch

jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version-file: '.nvmrc'
          cache: 'npm'
      - run: npm ci
```

#### Python Example

```yaml
name: "Copilot Setup Steps"
on: workflow_dispatch

jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version-file: '.python-version'
          cache: 'pip'
      - run: pip install -r requirements.txt
```

#### .NET Example

```yaml
name: "Copilot Setup Steps"
on: workflow_dispatch

jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '9.0.x'
      - run: dotnet restore
```

#### PowerShell Example

```yaml
name: "Copilot Setup Steps"
on: workflow_dispatch

jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install PowerShell modules
        shell: pwsh
        run: |
          Install-Module -Name Az -Force -AllowClobber
          Install-Module -Name Microsoft.Graph -Force
```

---

## CI/CD Quality Gates

### Workflow Permissions

**Always** include explicit `permissions` per job:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      issues: write
```

### Zero-Tolerance Policies

Enforce strict quality in CI:

- **Zero TypeScript errors** — `tsc --noEmit` must exit 0
- **Zero lint warnings** — treat warnings as errors (`--max-warnings 0`)
- **Zero audit vulnerabilities** — `npm audit --audit-level=moderate`
- **All tests pass** — no skipped tests in CI

### Visual Regression Testing

Use Playwright for screenshot-based regression detection:

```yaml
- name: Visual regression tests
  run: npx playwright test
- name: Upload screenshots
  if: failure()
  uses: actions/upload-artifact@v4
  with:
    name: visual-regression-results
    path: test-results/
```

---

## Dependency Management

### Dependabot Configuration (`.github/dependabot.yml`)

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    groups:
      production-dependencies:
        dependency-type: "production"
      dev-dependencies:
        dependency-type: "development"
    reviewers:
      - "LucidLabsAU/developers"
```

**Adapt `package-ecosystem`** for your stack: `npm`, `pip`, `nuget`, `github-actions`, `docker`.

### Auto-Merge for Minor/Patch

Create a workflow to auto-merge low-risk dependency updates:

```yaml
name: Auto-merge Dependabot
on:
  pull_request:
    types: [opened, synchronize]

permissions:
  contents: write
  pull-requests: write

jobs:
  auto-merge:
    if: github.actor == 'dependabot[bot]'
    runs-on: ubuntu-latest
    steps:
      - uses: dependabot/fetch-metadata@v2
        id: metadata
      - if: steps.metadata.outputs.update-type != 'version-update:semver-major'
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## Security

### Automated Scanning

- **CodeQL** — static analysis for security vulnerabilities
- **Microsoft Security DevOps (MSDO)** — comprehensive security scanning
- **Dependency review** — block PRs introducing vulnerable dependencies

### Content Security Policy

For web projects, maintain strict CSP in deployment config:

- Use `'self'` for trusted sources
- Avoid `'unsafe-inline'` except for styles (required by some UI frameworks)
- Whitelist specific domains, never wildcards
- Enforce HSTS with long max-age (2 years recommended)

### Secrets Management

- **Never** hardcode API keys, tokens, or passwords
- Use GitHub Secrets for CI/CD
- Use environment variables for runtime config
- Use Azure Key Vault for production secrets
- Add `.env` to `.gitignore`

---

## Agentic Workflows

> **Status: Private preview** — prepare files now, activate when preview access is granted.

GitHub Agentic Workflows are Markdown-based automation files that run coding agents in GitHub Actions. They enable "Continuous AI" — automated, intent-driven workflows triggered by repository events.

### Directory Structure

```
.github/
├── agents/                           # Specialist agent definitions
│   └── <role>.agent.md
├── prompts/                          # Reusable task prompts
│   └── <task>.prompt.md
└── agentic-workflows/                # AI-automated workflows
    └── <workflow>.md
```

### Agent Definitions (`.github/agents/*.agent.md`)

Define specialist personas with YAML frontmatter:

```markdown
---
name: API-Developer
description: Expert agent for type-safe API endpoints
---

# API Developer Agent

<Role description, responsibilities, patterns, quality standards>
```

Invoke in Copilot Chat: `@workspace /agents/<name>`

### Prompt Templates (`.github/prompts/*.prompt.md`)

Reusable task prompts for common operations:

```markdown
# Add Feature

## Steps
1. Define types and schemas
2. Implement data layer
3. Create API endpoints
4. Build UI components
5. Write tests

## Validation
- [ ] Type check passes
- [ ] Tests pass
- [ ] Build succeeds
```

Invoke in Copilot Chat: `@workspace /prompts/<name>`

### Agentic Workflow Format

```markdown
---
name: Workflow Name
description: What this workflow does
triggers:
  - type: issues
    activity: opened
    label: bug
permissions:
  contents: read
  issues: write
safe-outputs:
  - comment-on-issue
  - create-pull-request
---

# Natural Language Instructions

<What the agent should do when triggered>
```

**Key concepts:**
- **Triggers** — repository events (`issues`, `pull_request`, `check_run`, `schedule`)
- **Safe outputs** — explicitly declared allowed actions (`create-pull-request`, `comment-on-issue`, `add-label`)
- **Permissions** — read-only by default, minimum required scope
- **Compilation** — `gh aw compile` generates `.lock.yml` files for execution

### Common Automation Patterns

| Pattern | Trigger | Safe Outputs |
|---------|---------|-------------|
| Issue triage | `issues.opened` | `add-label`, `comment-on-issue` |
| Docs sync | `pull_request.closed` (merged) | `create-pull-request` |
| CI failure analysis | `check_run.completed` (failure) | `comment-on-pull-request` |
| Weekly audit | `schedule` (cron) | `create-pull-request`, `comment-on-issue` |

### Security

- **Read-only default** — workflows can only read repository contents
- **Safe outputs** — explicitly whitelist allowed write actions
- **No secrets** — agents don't have access to repository secrets
- **Sandboxed** — execution is isolated in GitHub Actions
- **Human review** — PRs created by agents require human approval

### Claude Code Skill

Use the `github-agentic-workflows` Claude Code skill to scaffold agent definitions, prompt templates, and agentic workflow files for any repository.

---

## Claude Code

### CLAUDE.md

Place a `CLAUDE.md` at the repo root with:

- **Project overview** — what the project does
- **Essential commands** — build, test, lint with typical run times
- **Architecture** — folder structure and key patterns
- **Validation workflow** — what to run after changes
- **Security guidelines** — project-specific security rules

**This overlaps with copilot-instructions.md** — keep both in sync or reference one from the other.

---

## Rollout Checklist

When setting up a new repo or bringing an existing one to standard:

- [ ] `.vscode/mcp.json` — GitHub MCP Server
- [ ] `.vscode/extensions.json` — Copilot + stack-specific extensions
- [ ] `.vscode/settings.json` — editor defaults + Copilot instruction reference
- [ ] `.github/copilot-instructions.md` — project instructions
- [ ] `.github/instructions/` — path-specific instructions (optional)
- [ ] `.github/prompts/` — reusable prompts (optional)
- [ ] `.github/agents/` — specialist agent definitions (optional)
- [ ] `.github/agentic-workflows/` — AI automation files (optional, preview)
- [ ] `.github/workflows/copilot-setup-steps.yml` — Copilot coding agent environment
- [ ] `.github/dependabot.yml` — dependency management
- [ ] `CLAUDE.md` — Claude Code instructions
- [ ] Security scanning workflows (CodeQL, MSDO)
