# Pam GitHub Actions

A centralized repository for reusable GitHub Actions used across Pam (Personal Appointment Manager) projects.

## Purpose

This repository provides a single source of truth for common CI/CD workflows used across Pam repositories. By centralizing our actions, we can:

- **Maintain consistency** across all Pam projects
- **Simplify updates** - modify once, apply everywhere
- **Reduce duplication** - share common workflow steps
- **Improve reliability** - test actions in one place
- **Accelerate development** - reuse proven patterns

## Available Actions

### Security Audit Action

Runs npm security audits with configurable Node.js versions and package managers.

**Location:** `audit/action.yml`

#### Inputs

| Input                 | Description                       | Required | Default       |
| --------------------- | --------------------------------- | -------- | ------------- |
| `node-version`        | Node.js version to use            | No       | `'22'`        |
| `package-manager`     | Package manager (npm, pnpm, yarn) | No       | `'npm'`       |
| `skip-env-validation` | Skip environment validation       | No       | `'true'`      |
| `audit-command`       | Command to run for security audit | No       | `'npm audit'` |

#### Basic Usage

```yaml
name: Security Audit

on:
  push:
    branches: ['*']
  pull_request:
    branches: ['*']

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: dream-lab-ai/pam-github-actions/audit@main
```

#### Advanced Usage

```yaml
jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: dream-lab-ai/pam-github-actions/audit@main
        with:
          node-version: '22'
          package-manager: 'pnpm'
          audit-command: 'pnpm audit'
```

#### Example with Custom npm Script

If your project has a custom audit script in `package.json`:

```json
{
  "scripts": {
    "npm-audit": "npm audit"
  }
}
```

You can reference it in the workflow:

```yaml
jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: dream-lab-ai/pam-github-actions/audit@main
        with:
          node-version: '22'
          package-manager: 'npm'
          audit-command: 'npm run npm-audit'
```

## How to Use Actions from Another Repository

GitHub Actions supports referencing actions from other repositories using the following syntax:

```yaml
uses: {owner}/{repo}/{path}@{ref}
```

Where:

- `{owner}` - GitHub organization or user (e.g., `dream-lab-ai`)
- `{repo}` - Repository name (e.g., `pam-github-actions`)
- `{path}` - Path to the action directory (e.g., `audit`)
- `{ref}` - Git reference: branch name, tag, or commit SHA (e.g., `main`, `v1.0.0`, or a commit SHA)

### Best Practices for Versioning

#### Using Branches (Development)

```yaml
uses: dream-lab-ai/pam-github-actions/audit@main
```

✅ **Pros:** Always gets latest updates automatically  
⚠️ **Cons:** Breaking changes could affect your workflow

**Recommended for:** Development and testing

#### Using Tags (Production)

```yaml
uses: dream-lab-ai/pam-github-actions/audit@v1.0.0
```

✅ **Pros:** Stable, predictable, controlled updates  
✅ **Cons:** Requires manual updates to get new features

**Recommended for:** Production deployments

#### Using Commit SHAs (Maximum Stability)

```yaml
uses: dream-lab-ai/pam-github-actions/audit@a1b2c3d4
```

✅ **Pros:** Immutable, guaranteed consistency  
⚠️ **Cons:** Difficult to maintain, no automatic updates

**Recommended for:** Critical infrastructure requiring absolute stability

### Updating Actions

When we release updates to actions in this repository:

1. **For development workflows:** No action needed if using `@main`
2. **For production workflows:** Update the version tag when ready

   ```yaml
   # Before
   uses: dream-lab-ai/pam-github-actions/audit@v1.0.0

   # After
   uses: dream-lab-ai/pam-github-actions/audit@v1.1.0
   ```

## Contributing

### Development Setup

This repository uses Prettier for formatting YAML and Markdown files.

1. Install dependencies:

   ```bash
   npm install
   ```

2. Format all files:

   ```bash
   npm run format
   ```

3. Check formatting without changes:
   ```bash
   npm run format:check
   ```

The Prettier configuration is sourced from `@dream-lab-ai/pam-eslint-config` to maintain consistency across all Pam projects.

### Adding New Actions

1. Create a new directory for your action (e.g., `lint/`, `test/`, `deploy/`)
2. Add an `action.yml` file with:
   - Clear name and description
   - Well-documented inputs with defaults
   - Composite action steps or Docker/JavaScript runner
3. Update this README with:
   - Action documentation
   - Usage examples
   - Input/output reference
4. Test the action in a consuming repository
5. Submit a pull request

### Action Development Guidelines

- **Use composite actions** for simple workflow compositions
- **Provide sensible defaults** for all inputs
- **Document all inputs and outputs** clearly
- **Use semantic versioning** for releases
- **Test thoroughly** before publishing
- **Keep actions focused** - one action, one purpose
- **Support common configurations** through inputs

## Repository Structure

```
pam-github-actions/
├── audit/
│   └── action.yml          # Security audit composite action
├── package.json            # Node.js dependencies (Prettier)
├── .prettierrc.js          # Prettier configuration
├── .prettierignore         # Prettier ignore patterns
├── .nvmrc                  # Node version specification
├── .gitignore              # Git ignore patterns
├── README.md               # This file
└── .github/
    └── workflows/          # Future: workflows to test actions
```

## Future Actions

Potential actions to add:

- **Linting** - Standardized ESLint/Prettier checks
- **Testing** - Unit, integration, and E2E test runners
- **Build** - Common build processes
- **Deploy** - Deployment workflows for AWS, serverless, etc.
- **Notification** - Slack/email notifications
- **Database** - Database migration and seeding

## Support

For questions or issues:

1. Check existing issues in this repository
2. Review the action's `action.yml` for configuration details
3. Create a new issue with details about your use case

## License

Internal use for Pam projects.
