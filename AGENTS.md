# PAM GitHub Actions - AI Agent Instructions

## Project Identity

**Type**: Reusable GitHub Actions (composite actions) repository for Pam AI projects

**Stack**: GitHub Actions YAML, Bash scripts, GitHub CLI

**Architecture**: Composite action pattern - each action is a self-contained, reusable workflow step

**Purpose**: Provide standardized, consistent CI/CD workflow components (testing, linting, deployment, notifications) that can be referenced across all Pam repositories

---

## Critical Context

### Node.js Version & Requirements

- **Node.js**: 22.14.0 (see `.nvmrc` - referenced in consuming workflows)
- **Module System**: N/A (this is a GitHub Actions repository, not a Node.js package)
- **Package Manager**: npm only (for Prettier formatting of YAML files)
- **No Runtime Code**: This repository contains YAML action definitions, not executable JavaScript/TypeScript

### Distribution Method

- **Published via**: GitHub (public repository)
- **Usage**: `uses: dream-lab-ai/pam-github-actions/{action-name}@{ref}`
- **Versioning**: Git tags (e.g., `v1.0.0`) and branches (`main`)
- **Reference Pattern**: `{owner}/{repo}/{path}@{ref}`

### Action Type

- **Composite Actions**: All actions use `runs: composite` (YAML-based workflow steps)
- **No Docker/JavaScript Actions**: For simplicity and portability
- **Shell: bash**: All shell commands use bash

---

## Development Environment

### Initial Setup

```bash
# Use correct Node version (for Prettier only)
nvm use  # reads from .nvmrc (v22.14.0)

# Install dependencies (Prettier only)
npm install

# Initialize Husky hooks
npm run prepare
```

### Development Commands

```bash
npm run format      # Format all YAML and Markdown files with Prettier
npm run npm-audit   # Run security audit (minimal dependencies)
```

### No Testing Infrastructure

- **No automated tests** for actions
- **Testing happens in consuming projects** (pam-tekion-integration, etc.)
- **Manual testing required** before releasing changes
- **Test in development workflows first** before updating production references

---

## Testing Strategy

### Manual Testing Approach

1. **Make changes** to action YAML
2. **Commit changes** to a feature branch
3. **Test in consuming project**:
   ```yaml
   # In consuming project's .github/workflows/test.yml
   - uses: dream-lab-ai/pam-github-actions/unit-test@feature-branch-name
   ```
4. **Verify** action works as expected in GitHub Actions UI
5. **Iterate** if needed
6. **Merge to main** when confident
7. **Create git tag** for release

### What to Test Manually

- ✅ Action inputs are parsed correctly
- ✅ Action outputs are set correctly
- ✅ Shell commands execute successfully
- ✅ Error handling works (test failure scenarios)
- ✅ Artifacts are uploaded correctly
- ✅ GitHub summaries display correctly
- ✅ Action works on ubuntu-latest runner

### Testing in Fork

For major changes, test in a fork:

1. Fork this repository
2. Update consuming project to use fork:
   ```yaml
   uses: your-username/pam-github-actions/unit-test@main
   ```
3. Test thoroughly
4. Submit PR to main repository

---

## Code Quality & Formatting

### Formatting

```bash
npm run format  # Prettier with auto-fix
```

- **Config**: Uses Prettier config from `@dream-lab-ai/pam-eslint-config`
- **Files**: YAML (`.yml`) and Markdown (`.md`)
- **Style**: Consistent indentation, line wrapping
- **Pre-commit**: Husky + lint-staged runs format automatically

### No Linting

- **No ESLint**: This repository doesn't contain JavaScript code
- **YAML Validation**: Implicit validation by GitHub Actions when workflow runs

### Pre-commit Hooks

- **Tool**: Husky v9
- **Config**: `lint-staged` in package.json
- **Triggers**: `npm run format` on staged files
- **Setup**: `npm run prepare` (runs after npm install)

---

## Architecture Patterns

### Repository Structure

```
pam-github-actions/
├── audit/
│   └── action.yml              # Security audit action
├── aws-configure/
│   └── action.yml              # AWS credentials configuration
├── create-issue/
│   └── action.yml              # GitHub issue creation
├── e2e-test/
│   └── action.yml              # E2E testing with coverage
├── format/
│   └── action.yml              # Code formatting checks
├── lint/
│   └── action.yml              # Linting checks
├── pr-comment/
│   └── action.yml              # PR comment creation
├── serverless-deploy/
│   └── action.yml              # Serverless Framework deployment
├── unit-test/
│   └── action.yml              # Unit testing with coverage
├── package.json                # npm for Prettier only
├── .nvmrc                      # Node version for development
├── .prettierrc.js              # Prettier configuration
└── README.md                   # Consumer documentation
```

**Pattern**: Each action is a directory with single `action.yml` file

### Composite Action Pattern

Every action follows this structure:

```yaml
name: 'Action Name'
description: 'What this action does'

inputs:
  input-name:
    description: 'Description of input'
    required: true/false
    default: 'default-value'

outputs:
  output-name:
    description: 'Description of output'
    value: ${{ steps.step-id.outputs.value }}

runs:
  using: 'composite'
  steps:
    - name: Step Name
      shell: bash
      run: |
        # Bash commands here
        echo "output-name=value" >> $GITHUB_OUTPUT
```

**Key Elements**:

- `inputs`: Define parameters (with descriptions, required flag, defaults)
- `outputs`: Define return values (linked to step outputs)
- `runs: composite`: Use composite action runner
- `steps`: List of shell commands or other actions
- `shell: bash`: All shell steps use bash

---

## Available Actions

### Unit Test Action (`unit-test/action.yml`)

**Purpose**: Run unit tests with coverage reporting and GitHub summaries

**Key Inputs**:

- `test-command`: Command to run tests (default: `npm run coverage:unit`)
- `node-version`: Node.js version (default: `'22'`)
- `package-manager`: npm/pnpm/yarn (default: `'npm'`)

**Key Outputs**:

- `test_passed`: Boolean (`'true'`/`'false'`)
- `coverage_text`: Text summary of coverage
- `coverage_summary_path`: Path to JSON coverage summary

**Key Features**:

- Runs tests and captures exit code
- Generates GitHub step summary with coverage table
- Uploads coverage artifacts (HTML, JSON)
- Sets outputs for downstream steps

### Lint Action (`lint/action.yml`)

**Purpose**: Run linting checks with GitHub summaries

**Key Inputs**:

- `lint-command`: Command to run linter (default: `npm run lint`)
- `node-version`: Node.js version (default: `'22'`)

**Key Outputs**:

- `lint_passed`: Boolean (`'true'`/`'false'`)
- `lint_results_path`: Path to lint results directory

**Key Features**:

- Runs linter and captures exit code
- Generates GitHub step summary
- Uploads lint results as artifacts

### Format Action (`format/action.yml`)

**Purpose**: Run code formatting checks with GitHub summaries

**Similar to lint action** but for formatting

### E2E Test Action (`e2e-test/action.yml`)

**Purpose**: Run end-to-end tests with coverage reporting

**Additional Inputs**:

- `test-env`: Environment to test (dev/prod)
- `api-url`: API URL to test against
- `wait-time`: Seconds to wait before tests (default: `'45'`)

**Key Features**:

- Waits for deployment to stabilize
- Runs E2E tests with environment variables
- Similar outputs to unit-test action

### AWS Configure Action (`aws-configure/action.yml`)

**Purpose**: Configure AWS credentials (access keys or OIDC)

**Key Inputs**:

- `auth-method`: `access-keys`, `oidc`, or `auto` (default: `'auto'`)
- `aws-access-key-id`: For access keys method
- `aws-secret-access-key`: For access keys method
- `role-to-assume`: For OIDC method
- `aws-region`: AWS region (default: `'us-east-1'`)

**Key Outputs**:

- `auth_method_used`: Method that was used
- `aws_region`: Configured region

**Key Features**:

- Auto-detects auth method based on inputs
- Supports both access keys and OIDC
- Uses official AWS actions internally

### Serverless Deploy Action (`serverless-deploy/action.yml`)

**Purpose**: Deploy Serverless Framework applications

**Key Inputs**:

- `stage`: Deployment stage (dev/prod)
- `service-name`: Service name for stack identification
- `wait-time`: Seconds to wait after deployment (default: `'30'`)
- `extract-url`: Extract API URL from deployment (default: `'true'`)

**Key Outputs**:

- `api_url`: Extracted API Gateway URL
- `deployment_time`: ISO timestamp
- `deployment_successful`: Boolean

**Key Features**:

- Deploys with `serverless deploy --stage {stage}`
- Extracts API URL from outputs
- Waits for deployment to stabilize
- Generates deployment summary

### Create Issue Action (`create-issue/action.yml`)

**Purpose**: Create GitHub issues with consistent formatting

**Key Inputs**:

- `issue-type`: Type of issue (test-failure, deployment-report)
- `title`: Issue title
- `body`: Issue body (Markdown)
- `labels`: Comma-separated labels
- `assignees`: Comma-separated assignees (optional)

**Key Outputs**:

- `issue_number`: Created issue number
- `issue_url`: URL to issue

**Key Features**:

- Uses GitHub CLI (`gh issue create`)
- Consistent formatting
- Supports templates

### PR Comment Action (`pr-comment/action.yml`)

**Purpose**: Create formatted PR comments with deployment/test status

**Key Inputs**:

- `pr-number`: Pull request number
- `deployment-environment`: dev/production
- `api-url`: Deployed API URL
- `deployment-status`: success/failed
- `unit-tests-status`: passed/failed/skipped
- `e2e-tests-status`: passed/failed/skipped
- `coverage-text`: Coverage summary (optional)

**Key Outputs**:

- `comment_id`: Created comment ID

**Key Features**:

- Formatted status table
- Emoji indicators (✅/❌/⏭️)
- Links to deployed API
- Includes coverage summary

### Security Audit Action (`audit/action.yml`)

**Purpose**: Run npm security audits

**Key Inputs**:

- `audit-command`: Command to run (default: `'npm audit'`)
- `node-version`: Node.js version (default: `'22'`)

**Key Features**:

- Simple audit runner
- Fails if vulnerabilities found
- Can be customized per project

---

## Action Development Patterns

### Input Validation Pattern

```yaml
- name: Validate Required Input
  shell: bash
  run: |
    if [[ -z "${{ inputs.required-input }}" ]]; then
      echo "::error::required-input is required"
      exit 1
    fi
```

### Output Setting Pattern

```yaml
- name: Set Output
  id: output-step
  shell: bash
  run: |
    result="computed-value"
    echo "output-name=$result" >> $GITHUB_OUTPUT

outputs:
  output-name:
    description: 'Description'
    value: ${{ steps.output-step.outputs.output-name }}
```

### Error Handling Pattern

```yaml
- name: Run Command
  id: command
  shell: bash
  continue-on-error: true # Don't fail action immediately
  run: |
    npm test

- name: Check Result
  shell: bash
  run: |
    if [[ "${{ steps.command.outcome }}" == "failure" ]]; then
      echo "::error::Tests failed"
      echo "test_passed=false" >> $GITHUB_OUTPUT
      exit 1
    else
      echo "test_passed=true" >> $GITHUB_OUTPUT
    fi
```

### Artifact Upload Pattern

```yaml
- name: Upload Artifacts
  if: always() # Upload even if tests failed
  uses: actions/upload-artifact@v4
  with:
    name: test-results-${{ github.run_id }}
    path: |
      coverage/
      test-results/
    retention-days: 30
```

### GitHub Summary Pattern

```yaml
- name: Generate Summary
  if: always()
  shell: bash
  run: |
    echo "## Test Results" >> $GITHUB_STEP_SUMMARY
    echo "" >> $GITHUB_STEP_SUMMARY
    echo "| Metric | Value |" >> $GITHUB_STEP_SUMMARY
    echo "|--------|-------|" >> $GITHUB_STEP_SUMMARY
    echo "| Status | ${{ steps.test.outputs.test_passed }} |" >> $GITHUB_STEP_SUMMARY
```

### Coverage Parsing Pattern

```yaml
- name: Parse Coverage
  shell: bash
  run: |
    if [[ -f coverage/coverage-summary.json ]]; then
      coverage=$(jq -r '.total.lines.pct' coverage/coverage-summary.json)
      echo "coverage_text=Lines: ${coverage}%" >> $GITHUB_OUTPUT
    fi
```

---

## Consumer Usage Patterns

### Basic Usage in Workflow

```yaml
name: Test

on:
  push:
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

      - name: Run Tests
        uses: dream-lab-ai/pam-github-actions/unit-test@main
```

### Using Action Outputs

```yaml
- name: Run Tests
  id: tests
  uses: dream-lab-ai/pam-github-actions/unit-test@main

- name: Comment on PR
  if: always()
  uses: dream-lab-ai/pam-github-actions/pr-comment@main
  with:
    pr-number: ${{ github.event.pull_request.number }}
    unit-tests-status: ${{ steps.tests.outputs.test_passed == 'true' && 'passed' || 'failed' }}
    coverage-text: ${{ steps.tests.outputs.coverage_text }}
```

### Chaining Actions

```yaml
- name: Configure AWS
  uses: dream-lab-ai/pam-github-actions/aws-configure@main
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

- name: Deploy
  id: deploy
  uses: dream-lab-ai/pam-github-actions/serverless-deploy@main
  with:
    stage: 'dev'

- name: Run E2E Tests
  uses: dream-lab-ai/pam-github-actions/e2e-test@main
  with:
    test-env: 'dev'
    api-url: ${{ steps.deploy.outputs.api_url }}
```

### Version Pinning Strategies

**Using main branch (development)**:

```yaml
uses: dream-lab-ai/pam-github-actions/unit-test@main
```

**Using git tag (production)**:

```yaml
uses: dream-lab-ai/pam-github-actions/unit-test@v1.0.0
```

**Using commit SHA (maximum stability)**:

```yaml
uses: dream-lab-ai/pam-github-actions/unit-test@abc123def
```

---

## Code Style & Conventions

### YAML Formatting

- **Indentation**: 2 spaces
- **String Quoting**: Single quotes for most strings, double quotes for templates
- **Line Length**: Prettier handles wrapping
- **Comments**: Use `#` for explanatory comments

### Action Naming

- **Directory Names**: kebab-case (e.g., `unit-test`, `aws-configure`)
- **Action Names**: Title Case (e.g., "Unit Test Action")
- **Step Names**: Descriptive, imperative (e.g., "Run Tests", "Upload Artifacts")

### Input/Output Naming

- **Inputs**: kebab-case (e.g., `node-version`, `test-command`)
- **Outputs**: snake_case (e.g., `test_passed`, `coverage_text`)
- **Reason**: Follows GitHub Actions conventions

### Shell Script Style

```bash
# Use strict mode
set -e  # Exit on error (if appropriate)

# Quote variables
echo "${{ inputs.my-input }}"

# Use [[ ]] for conditionals
if [[ -f coverage/coverage-summary.json ]]; then
  # ...
fi

# Meaningful variable names
test_result="passed"
coverage_percentage="85.5"
```

---

## Common Patterns & Recipes

### Adding a New Action

1. **Create directory**: `mkdir new-action`
2. **Create action.yml**:

   ```yaml
   name: 'New Action'
   description: 'What it does'

   inputs:
     input-name:
       description: 'Input description'
       required: false
       default: 'default-value'

   outputs:
     output-name:
       description: 'Output description'
       value: ${{ steps.step-id.outputs.value }}

   runs:
     using: 'composite'
     steps:
       - name: Do Something
         shell: bash
         run: echo "Hello"
   ```

3. **Test in consuming project**
4. **Update README.md** with documentation
5. **Commit and tag**

### Modifying Existing Action

1. **Make changes** to `action.yml`
2. **Test in consuming project** using feature branch:
   ```yaml
   uses: dream-lab-ai/pam-github-actions/unit-test@feature-branch
   ```
3. **Iterate** until working
4. **Merge to main**
5. **Create new tag** if breaking changes

### Adding Action Input

```yaml
inputs:
  new-input:
    description: 'Description of new input'
    required: false # Don't break existing consumers
    default: 'sensible-default'

steps:
  - name: Use Input
    shell: bash
    run: |
      echo "Value: ${{ inputs.new-input }}"
```

### Adding Action Output

```yaml
outputs:
  new-output:
    description: 'Description of new output'
    value: ${{ steps.compute.outputs.value }}

steps:
  - name: Compute Value
    id: compute
    shell: bash
    run: |
      result="computed-value"
      echo "value=$result" >> $GITHUB_OUTPUT
```

### Conditional Steps

```yaml
- name: Conditional Step
  if: ${{ inputs.skip-env-validation != 'true' }}
  shell: bash
  run: |
    echo "Running validation"
```

---

## Security & Secrets

### Secrets Handling

- **Never log secrets**: Avoid `echo ${{ secrets.SECRET }}`
- **Use GitHub masked values**: Secrets are automatically masked in logs
- **Pass to actions**: Use `with:` to pass secrets safely
- **Environment variables**: Set secrets as env vars for subprocesses

### Permission Management

- **Action permissions**: Actions inherit workflow permissions
- **Minimal permissions**: Consumers should use principle of least privilege
- **GITHUB_TOKEN**: Available by default for API operations

### Best Practices

```yaml
# ✅ Good: Secret passed via with
- uses: dream-lab-ai/pam-github-actions/aws-configure@main
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}

# ❌ Bad: Secret in shell command
- run: echo "Secret is ${{ secrets.AWS_ACCESS_KEY_ID }}"

# ✅ Good: Set as environment variable
- run: aws s3 ls
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
```

---

## Deployment & Versioning

### Release Process

1. **Make changes** and test in consuming project
2. **Update documentation** (README.md, this file)
3. **Commit changes**:
   ```bash
   git add .
   git commit -m "Add new feature to unit-test action"
   ```
4. **Push to main**:
   ```bash
   git push origin main
   ```
5. **Create git tag** (for major releases):
   ```bash
   git tag v1.1.0
   git push origin v1.1.0
   ```
6. **Notify consumers** of changes (if breaking)

### Versioning Strategy

- **Semantic Versioning**: `MAJOR.MINOR.PATCH`
- **Major**: Breaking changes (required input changes, removed outputs)
- **Minor**: New features (new inputs with defaults, new outputs)
- **Patch**: Bug fixes (no interface changes)

### Breaking Changes

**How to handle**:

1. Document in commit message
2. Update README.md with migration guide
3. Notify teams using the action
4. Bump major version (e.g., `v1.5.0` → `v2.0.0`)
5. Consider providing migration path (new action name, deprecation period)

### Backward Compatibility

**Always maintain** when possible:

- Add new inputs with defaults (don't break existing consumers)
- Don't remove outputs (mark as deprecated instead)
- Don't change input types or meanings
- Don't change shell: bash to another shell

---

## Documentation

### README.md

- **Audience**: Developers consuming these actions
- **Content**:
  - Available actions list
  - Input/output reference for each action
  - Usage examples (basic and advanced)
  - Versioning guidance
- **Keep Updated**: When adding actions or changing interfaces

### AGENTS.md (This File)

- **Audience**: AI agents and developers maintaining this repository
- **Content**:
  - Architecture patterns
  - Development workflow
  - Testing strategy
  - Common recipes
- **Keep Updated**: When changing patterns or adding best practices

### Inline Comments

- **Action files**: Comment complex shell logic
- **Example**:
  ```yaml
  - name: Extract API URL
    shell: bash
    run: |
      # Parse CloudFormation outputs for ServiceEndpoint key
      url=$(aws cloudformation describe-stacks ...)
      echo "api_url=$url" >> $GITHUB_OUTPUT
  ```

---

## Agent Permissions & Boundaries

### Allowed Actions (No Approval Required)

- ✅ Read any file in the repository
- ✅ List directory contents
- ✅ Search codebase
- ✅ Run formatter: `npm run format`
- ✅ View package dependencies: `cat package.json`
- ✅ Read documentation files

### Actions Requiring Approval

- ⚠️ Create/delete action directories
- ⚠️ Modify action.yml files
- ⚠️ Add/remove action inputs or outputs
- ⚠️ Change shell commands in actions
- ⚠️ Git operations: commit, push, tag
- ⚠️ Update README.md
- ⚠️ Install/remove npm packages

### Strictly Prohibited

- 🚫 Commit breaking changes without major version bump
- 🚫 Delete actions without deprecation notice
- 🚫 Force push to main
- 🚫 Modify git tags
- 🚫 Add secrets or credentials to repository
- 🚫 Change action type from composite to Docker/JavaScript

---

## Troubleshooting

### Common Issues

#### Action not found

- **Cause**: Wrong repository path or action doesn't exist
- **Fix**: Verify path: `uses: dream-lab-ai/pam-github-actions/{action-name}@main`

#### Input not recognized

- **Cause**: Typo in input name or action version mismatch
- **Fix**: Check `action.yml` for exact input name (kebab-case)

#### Output is empty

- **Cause**:
  - Step didn't set output
  - Wrong step ID referenced
  - Command failed before setting output
- **Fix**:
  - Check step ID matches: `value: ${{ steps.step-id.outputs.name }}`
  - Use `if: always()` to ensure step runs

#### Shell command fails

- **Cause**:
  - Missing dependency
  - Wrong working directory
  - Environment variable not set
- **Fix**:
  - Ensure consumer installs dependencies before action
  - Check if action expects specific working directory
  - Pass required environment variables

#### Coverage summary not found

- **Cause**:
  - Tests didn't generate coverage
  - Wrong coverage output path
- **Fix**:
  - Ensure test command includes `--coverage`
  - Verify `coverage/coverage-summary.json` exists after tests

### Debugging Actions

**In consuming workflow**:

```yaml
- name: Debug Action
  uses: dream-lab-ai/pam-github-actions/unit-test@main
  env:
    ACTIONS_STEP_DEBUG: true # Enable debug logging
```

**Check workflow logs**: GitHub Actions UI shows detailed step output

**Test locally**: Use `act` tool to run actions locally (limited support for composite actions)

### Where to Look for Help

1. **This file** (`AGENTS.md`) - Development context
2. **README.md** - Consumer documentation
3. **Action YAML files** - Exact implementation
4. **GitHub Actions docs** - [https://docs.github.com/en/actions](https://docs.github.com/en/actions)
5. **Consuming project workflows** - Real usage examples

---

## Project-Specific Terminology

- **Composite Action**: GitHub Actions action defined in YAML (not Docker/JavaScript)
- **Action Input**: Parameter passed to action via `with:`
- **Action Output**: Value returned from action via `outputs:`
- **Step Summary**: Markdown displayed in GitHub Actions UI (`$GITHUB_STEP_SUMMARY`)
- **Artifact**: File uploaded from workflow run (downloadable for 30-90 days)
- **Runner**: GitHub-hosted VM that executes workflows (e.g., ubuntu-latest)
- **Job**: Collection of steps that run on the same runner
- **Workflow**: YAML file defining CI/CD pipeline
- **GITHUB_OUTPUT**: Special file for setting action outputs
- **continue-on-error**: Allow step to fail without failing entire action

---

## Quick Reference

### Most Common Commands

```bash
npm run format      # Format YAML and Markdown
npm run npm-audit   # Security audit
```

### Most Important Files

- Each action's `action.yml` - Action definition
- `package.json` - Prettier dependency
- `.nvmrc` - Node version for development
- `README.md` - Consumer documentation

### Key Directories

- `unit-test/` - Unit testing action
- `lint/` - Linting action
- `format/` - Formatting action
- `e2e-test/` - E2E testing action
- `aws-configure/` - AWS setup action
- `serverless-deploy/` - Deployment action
- `create-issue/` - Issue creation action
- `pr-comment/` - PR comment action
- `audit/` - Security audit action

### Consuming Projects

- pam-tekion-integration
- pam-eslint-config
- pam-vitest-config
- Other Pam repositories with CI/CD pipelines

---

**Last Updated**: 2026-01-06  
**Node Version**: 22.14.0 (for development only)  
**Current Version**: 1.0.0  
**Action Type**: Composite Actions (YAML-based)
