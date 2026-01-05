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

### Unit Test Action

Runs unit tests with coverage reporting, generates comprehensive GitHub summaries with coverage metrics, and uploads test artifacts.

**Location:** `unit-test/action.yml`

#### Inputs

| Input                  | Description                                   | Required | Default                   |
| ---------------------- | --------------------------------------------- | -------- | ------------------------- |
| `node-version`         | Node.js version to use                        | No       | `'22'`                    |
| `package-manager`      | Package manager (npm, pnpm, yarn)             | No       | `'npm'`                   |
| `test-command`         | Test command to run (should include coverage) | No       | `'npm run coverage:unit'` |
| `skip-env-validation`  | Skip environment validation                   | No       | `'true'`                  |
| `results-directory`    | Directory to store test results               | No       | `'test-results'`          |
| `artifact-name-prefix` | Prefix for artifact name                      | No       | `'test-results'`          |

#### Outputs

| Output                  | Description                                                |
| ----------------------- | ---------------------------------------------------------- |
| `test_passed`           | Boolean indicating if tests passed (`'true'` or `'false'`) |
| `test_results_path`     | Path to the test results directory                         |
| `coverage_summary_path` | Path to coverage summary JSON file                         |
| `coverage_text`         | Text output of coverage summary for display                |

#### Basic Usage

```yaml
name: Test

on:
  push:
    branches: ['*']
  pull_request:
    branches: ['*']

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci

      - name: Run Unit Tests
        uses: dream-lab-ai/pam-github-actions/unit-test@main
```

#### Advanced Usage

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci

      - name: Run Unit Tests
        id: tests
        uses: dream-lab-ai/pam-github-actions/unit-test@main
        with:
          node-version: '22'
          package-manager: 'npm'
          test-command: 'npm run coverage:unit'
          results-directory: 'test-results'
          artifact-name-prefix: 'unit-test-results'

      - name: Check Test Results
        if: always()
        run: |
          echo "Tests passed: ${{ steps.tests.outputs.test_passed }}"
          echo "Coverage: ${{ steps.tests.outputs.coverage_text }}"
```

#### Coverage Visibility

The action provides multiple ways to view coverage results:

1. **GitHub Step Summary** - Coverage table displayed directly in the workflow run page:
   - Lines coverage percentage
   - Statements coverage percentage
   - Functions coverage percentage
   - Branches coverage percentage

2. **Downloadable Artifacts** - Full coverage reports uploaded as artifacts:
   - HTML coverage report (`coverage/index.html`)
   - JSON coverage data (`coverage/coverage-summary.json`)
   - Test output logs

3. **Action Outputs** - Programmatic access to coverage data:
   - Use `coverage_text` output for Slack notifications, PR comments, etc.
   - Use `coverage_summary_path` to parse detailed coverage data

#### Example with Custom Test Script

If your project has a different test script in `package.json`:

```json
{
  "scripts": {
    "test:coverage": "vitest run --coverage test/unit"
  }
}
```

You can use it in the workflow:

```yaml
- uses: dream-lab-ai/pam-github-actions/unit-test@main
  with:
    test-command: 'npm run test:coverage'
```

#### Requirements

- **Checkout:** You must checkout your code before calling this action
- **Node.js Setup:** You must setup Node.js before calling this action
- **Dependencies:** You must install dependencies before calling this action
- **Coverage Tool:** Your test command must generate a `coverage/coverage-summary.json` file (e.g., using Vitest with `@vitest/coverage-v8` or Jest with coverage enabled)

### Lint Action

Runs linting checks with configurable Node.js versions and package managers, generates GitHub summaries, and uploads lint results as artifacts.

**Location:** `lint/action.yml`

#### Inputs

| Input                  | Description                       | Required | Default          |
| ---------------------- | --------------------------------- | -------- | ---------------- |
| `node-version`         | Node.js version to use            | No       | `'22'`           |
| `package-manager`      | Package manager (npm, pnpm, yarn) | No       | `'npm'`          |
| `lint-command`         | Lint command to run               | No       | `'npm run lint'` |
| `skip-env-validation`  | Skip environment validation       | No       | `'true'`         |
| `results-directory`    | Directory to store lint results   | No       | `'lint-results'` |
| `artifact-name-prefix` | Prefix for artifact name          | No       | `'lint-results'` |

#### Outputs

| Output              | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `lint_passed`       | Boolean indicating if linting passed (`'true'` or `'false'`) |
| `lint_results_path` | Path to the lint results directory                           |

#### Basic Usage

```yaml
name: Lint

on:
  push:
    branches: ['*']
  pull_request:
    branches: ['*']

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci

      - name: Run Linting
        uses: dream-lab-ai/pam-github-actions/lint@main
```

#### Advanced Usage

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci

      - name: Run Linting
        id: lint
        uses: dream-lab-ai/pam-github-actions/lint@main
        with:
          node-version: '22'
          package-manager: 'npm'
          lint-command: 'npm run lint'
          results-directory: 'lint-results'
          artifact-name-prefix: 'lint-results'

      - name: Check Lint Results
        if: always()
        run: |
          echo "Linting passed: ${{ steps.lint.outputs.lint_passed }}"
```

#### Example with Custom Lint Script

If your project has a different lint script in `package.json`:

```json
{
  "scripts": {
    "lint:check": "eslint . --max-warnings 0"
  }
}
```

You can use it in the workflow:

```yaml
- uses: dream-lab-ai/pam-github-actions/lint@main
  with:
    lint-command: 'npm run lint:check'
```

#### Requirements

- **Checkout:** You must checkout your code before calling this action
- **Node.js Setup:** You must setup Node.js before calling this action
- **Dependencies:** You must install dependencies before calling this action
- **Lint Script:** Your project must have a lint command defined in `package.json`

### E2E Test Action

Runs E2E tests with coverage reporting, generates comprehensive GitHub summaries with coverage metrics, and uploads test artifacts.

**Location:** `e2e-test/action.yml`

#### Inputs

| Input                  | Description                                | Required | Default                  |
| ---------------------- | ------------------------------------------ | -------- | ------------------------ |
| `node-version`         | Node.js version to use                     | No       | `'22'`                   |
| `package-manager`      | Package manager (npm, pnpm, yarn)          | No       | `'npm'`                  |
| `test-command`         | E2E test command (should include coverage) | No       | `'npm run coverage:e2e'` |
| `test-env`             | Test environment (dev, prod)               | Yes      | -                        |
| `api-url`              | API URL to test against                    | Yes      | -                        |
| `skip-env-validation`  | Skip environment validation                | No       | `'true'`                 |
| `results-directory`    | Directory to store test results            | No       | `'e2e-results'`          |
| `artifact-name-prefix` | Prefix for artifact name                   | No       | `'e2e-test-results'`     |
| `wait-time`            | Seconds to wait before running tests       | No       | `'45'`                   |

#### Outputs

| Output                  | Description                                                |
| ----------------------- | ---------------------------------------------------------- |
| `test_passed`           | Boolean indicating if tests passed (`'true'` or `'false'`) |
| `test_results_path`     | Path to the test results directory                         |
| `coverage_summary_path` | Path to coverage summary JSON file                         |
| `coverage_text`         | Text output of coverage summary for display                |

#### Basic Usage

```yaml
- name: Run E2E Tests
  uses: dream-lab-ai/pam-github-actions/e2e-test@main
  with:
    test-env: 'dev'
    api-url: 'https://api.dev.example.com'
```

#### Advanced Usage

```yaml
- name: Run E2E Tests
  id: e2e
  uses: dream-lab-ai/pam-github-actions/e2e-test@main
  with:
    test-env: 'prod'
    api-url: ${{ steps.deploy.outputs.api_url }}
    test-command: 'npm run coverage:e2e'
    wait-time: '60'

- name: Check Results
  run: |
    echo "Tests passed: ${{ steps.e2e.outputs.test_passed }}"
    echo "Coverage: ${{ steps.e2e.outputs.coverage_text }}"
```

### AWS Configure Action

Configures AWS credentials using access keys or OIDC with auto-detection support.

**Location:** `aws-configure/action.yml`

#### Inputs

| Input                   | Description                                     | Required | Default         |
| ----------------------- | ----------------------------------------------- | -------- | --------------- |
| `aws-region`            | AWS region to configure                         | No       | `'us-east-1'`   |
| `auth-method`           | Authentication method (access-keys, oidc, auto) | No       | `'auto'`        |
| `aws-access-key-id`     | AWS Access Key ID (for access-keys method)      | No       | -               |
| `aws-secret-access-key` | AWS Secret Access Key (for access-keys method)  | No       | -               |
| `role-to-assume`        | IAM role ARN to assume (for OIDC method)        | No       | -               |
| `role-session-name`     | Role session name (for OIDC method)             | No       | `GitHubActions` |

#### Outputs

| Output             | Description                             |
| ------------------ | --------------------------------------- |
| `auth_method_used` | The authentication method that was used |
| `aws_region`       | The configured AWS region               |

#### Basic Usage (Access Keys)

```yaml
- name: Configure AWS
  uses: dream-lab-ai/pam-github-actions/aws-configure@main
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

#### Basic Usage (OIDC)

```yaml
- name: Configure AWS
  uses: dream-lab-ai/pam-github-actions/aws-configure@main
  with:
    role-to-assume: 'arn:aws:iam::123456789012:role/GitHubActionsRole'
```

### Serverless Deploy Action

Deploys serverless applications with automatic URL extraction and validation.

**Location:** `serverless-deploy/action.yml`

#### Inputs

| Input            | Description                                                 | Required | Default             |
| ---------------- | ----------------------------------------------------------- | -------- | ------------------- |
| `stage`          | Deployment stage (dev, prod, etc.)                          | Yes      | -                   |
| `service-name`   | Service name for stack identification                       | No       | -                   |
| `wait-time`      | Seconds to wait after deployment                            | No       | `'30'`              |
| `extract-url`    | Whether to extract API URL from deployment                  | No       | `'true'`            |
| `url-output-key` | Output key name to extract URL (ServiceEndpoint/HttpApiUrl) | No       | `'ServiceEndpoint'` |
| `verbose`        | Enable verbose output                                       | No       | `'false'`           |

#### Outputs

| Output                  | Description                                |
| ----------------------- | ------------------------------------------ |
| `api_url`               | Extracted API Gateway URL                  |
| `deployment_time`       | ISO timestamp of deployment                |
| `deployment_successful` | Boolean indicating if deployment succeeded |

#### Basic Usage

```yaml
- name: Deploy to Dev
  uses: dream-lab-ai/pam-github-actions/serverless-deploy@main
  with:
    stage: 'dev'
```

#### Advanced Usage

```yaml
- name: Deploy to Production
  id: deploy
  uses: dream-lab-ai/pam-github-actions/serverless-deploy@main
  with:
    stage: 'prod'
    service-name: 'my-api'
    wait-time: '60'
    verbose: 'true'

- name: Use Deployment Output
  run: echo "API deployed at ${{ steps.deploy.outputs.api_url }}"
```

### Create Issue Action

Creates GitHub issues with consistent formatting and templates.

**Location:** `create-issue/action.yml`

#### Inputs

| Input         | Description                                           | Required | Default |
| ------------- | ----------------------------------------------------- | -------- | ------- |
| `issue-type`  | Type of issue (test-failure, deployment-report, etc.) | Yes      | -       |
| `title`       | Issue title                                           | Yes      | -       |
| `body`        | Issue body (Markdown supported)                       | Yes      | -       |
| `labels`      | Comma-separated list of labels                        | Yes      | -       |
| `assignees`   | Comma-separated list of assignees                     | No       | -       |
| `environment` | Environment (dev, production)                         | No       | -       |

#### Outputs

| Output         | Description          |
| -------------- | -------------------- |
| `issue_number` | Created issue number |
| `issue_url`    | URL to created issue |

#### Basic Usage

```yaml
- name: Create Issue
  uses: dream-lab-ai/pam-github-actions/create-issue@main
  with:
    issue-type: 'test-failure'
    title: 'E2E Tests Failed in Production'
    body: 'E2E tests failed after deployment. Please investigate.'
    labels: 'production,test-failure,urgent'
```

#### Advanced Usage

```yaml
- name: Create Deployment Report
  id: issue
  uses: dream-lab-ai/pam-github-actions/create-issue@main
  with:
    issue-type: 'deployment-report'
    title: 'Production Deployment Report: ${{ steps.deploy.outputs.deployment_time }}'
    body: |
      # Deployment Report
      - Status: Success
      - API URL: ${{ steps.deploy.outputs.api_url }}
      - Time: ${{ steps.deploy.outputs.deployment_time }}
    labels: 'deployment,production,success'
    assignees: 'devops-team'
    environment: 'production'

- name: Show Issue
  run: echo "Created issue #${{ steps.issue.outputs.issue_number }}"
```

### PR Comment Action

Creates formatted PR comments with deployment status and test results.

**Location:** `pr-comment/action.yml`

#### Inputs

| Input                    | Description                                 | Required | Default     |
| ------------------------ | ------------------------------------------- | -------- | ----------- |
| `pr-number`              | Pull request number                         | Yes      | -           |
| `deployment-environment` | Deployment environment (dev, production)    | Yes      | -           |
| `api-url`                | Deployed API URL                            | Yes      | -           |
| `deployment-status`      | Deployment status (success, failed)         | Yes      | -           |
| `unit-tests-status`      | Unit tests status (passed, failed, skipped) | Yes      | -           |
| `e2e-tests-status`       | E2E tests status (passed, failed, skipped)  | Yes      | -           |
| `lint-status`            | Lint status (passed, failed, skipped)       | No       | `'skipped'` |
| `coverage-text`          | Coverage summary text                       | No       | -           |

#### Outputs

| Output       | Description        |
| ------------ | ------------------ |
| `comment_id` | Created comment ID |

#### Basic Usage

```yaml
- name: Comment on PR
  uses: dream-lab-ai/pam-github-actions/pr-comment@main
  with:
    pr-number: ${{ github.event.pull_request.number }}
    deployment-environment: 'dev'
    api-url: ${{ steps.deploy.outputs.api_url }}
    deployment-status: 'success'
    unit-tests-status: 'passed'
    e2e-tests-status: 'passed'
```

#### Advanced Usage

```yaml
- name: Comment on PR
  uses: dream-lab-ai/pam-github-actions/pr-comment@main
  with:
    pr-number: ${{ github.event.pull_request.number }}
    deployment-environment: 'dev'
    api-url: ${{ steps.deploy.outputs.api_url }}
    deployment-status: ${{ steps.deploy.outputs.deployment_successful == 'true' && 'success' || 'failed' }}
    unit-tests-status: ${{ steps.unit.outputs.test_passed == 'true' && 'passed' || 'failed' }}
    e2e-tests-status: ${{ steps.e2e.outputs.test_passed == 'true' && 'passed' || 'failed' }}
    lint-status: ${{ steps.lint.outputs.lint_passed == 'true' && 'passed' || 'failed' }}
    coverage-text: ${{ steps.unit.outputs.coverage_text }}
```

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
├── aws-configure/
│   └── action.yml          # AWS credentials configuration action
├── create-issue/
│   └── action.yml          # GitHub issue creation action
├── e2e-test/
│   └── action.yml          # E2E test with coverage composite action
├── lint/
│   └── action.yml          # Linting composite action
├── pr-comment/
│   └── action.yml          # PR comment action
├── serverless-deploy/
│   └── action.yml          # Serverless deployment action
├── unit-test/
│   └── action.yml          # Unit test with coverage composite action
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

- **Build** - Common build processes
- **Database** - Database migration and seeding

## Support

For questions or issues:

1. Check existing issues in this repository
2. Review the action's `action.yml` for configuration details
3. Create a new issue with details about your use case

## License

Internal use for Pam projects.
