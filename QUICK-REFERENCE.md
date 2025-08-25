# GitHub Actions Quick Reference

## Common Workflow Syntax

### Basic Structure
```yaml
name: Workflow Name
on: [push, pull_request]
jobs:
  job-name:
    runs-on: ubuntu-latest
    steps:
      - name: Step Name
        uses: action@version
        with:
          parameter: value
      - name: Run Command
        run: echo "Hello World"
```

### Triggers
```yaml
# Single event
on: push

# Multiple events
on: [push, pull_request]

# Event with configuration
on:
  push:
    branches: [main, develop]
    paths: ['src/**']
  pull_request:
    types: [opened, synchronize]
  schedule:
    - cron: '0 0 * * 0'  # Sunday at midnight
  workflow_dispatch:  # Manual trigger
```

### Jobs and Steps
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    env:
      NODE_ENV: test
    steps:
      - uses: actions/checkout@v4
      - run: npm test
        env:
          API_KEY: ${{ secrets.API_KEY }}
```

### Matrix Strategy
```yaml
strategy:
  fail-fast: false
  matrix:
    os: [ubuntu-latest, windows-latest]
    node: [16, 18, 20]
    include:
      - os: ubuntu-latest
        node: 18
        experimental: true
    exclude:
      - os: windows-latest
        node: 16
```

### Conditionals
```yaml
# Job level
if: github.ref == 'refs/heads/main'

# Step level
- name: Deploy
  if: success() && github.event_name == 'push'
  run: ./deploy.sh

# Common conditions
if: always()  # Run regardless of previous step results
if: failure()  # Run only if previous step failed
if: success()  # Run only if previous steps succeeded
if: cancelled()  # Run only if workflow was cancelled
```

### Outputs
```yaml
jobs:
  build:
    outputs:
      version: ${{ steps.version.outputs.VERSION }}
    steps:
      - id: version
        run: echo "VERSION=1.0.0" >> $GITHUB_OUTPUT
        
  deploy:
    needs: build
    steps:
      - run: echo ${{ needs.build.outputs.version }}
```

## Essential Actions

### Repository and Environment Setup
```yaml
# Checkout code
- uses: actions/checkout@v4
  with:
    fetch-depth: 0  # Full history
    token: ${{ secrets.GITHUB_TOKEN }}

# Setup Node.js
- uses: actions/setup-node@v4
  with:
    node-version: '18'
    cache: 'npm'
    registry-url: 'https://registry.npmjs.org'

# Setup Python
- uses: actions/setup-python@v4
  with:
    python-version: '3.11'
    cache: 'pip'

# Cache dependencies
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('package-lock.json') }}
```

### Artifacts and Reports
```yaml
# Upload artifacts
- uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: test-results.xml
    retention-days: 30

# Download artifacts
- uses: actions/download-artifact@v4
  with:
    name: test-results
    path: ./artifacts
```

### Docker
```yaml
# Setup Docker Buildx
- uses: docker/setup-buildx-action@v3

# Login to registry
- uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}

# Build and push
- uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: ghcr.io/${{ github.repository }}:latest
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

## Context Variables

### GitHub Context
```yaml
${{ github.event_name }}      # push, pull_request, etc.
${{ github.ref }}             # refs/heads/main
${{ github.ref_name }}        # main
${{ github.sha }}             # commit SHA
${{ github.actor }}           # username who triggered
${{ github.repository }}      # owner/repo-name
${{ github.workspace }}       # workspace directory path
${{ github.run_id }}          # unique run identifier
${{ github.run_number }}      # run number
```

### Runner Context
```yaml
${{ runner.os }}              # Linux, Windows, macOS
${{ runner.arch }}            # X86, X64, ARM, ARM64
${{ runner.name }}            # runner name
${{ runner.temp }}            # temp directory path
${{ runner.tool_cache }}      # tool cache directory
```

### Job Context
```yaml
${{ job.status }}             # success, failure, cancelled
${{ job.container.id }}       # container ID
${{ job.services }}           # service containers
```

### Steps Context
```yaml
${{ steps.step-id.outputs.output-name }}
${{ steps.step-id.conclusion }}  # success, failure, skipped
${{ steps.step-id.outcome }}     # success, failure, skipped
```

## Environment Variables

### Setting Variables
```yaml
# Workflow level
env:
  NODE_ENV: production

# Job level
jobs:
  build:
    env:
      BUILD_TYPE: release

# Step level
- name: Build
  env:
    API_URL: https://api.example.com
  run: npm run build
```

### Default Environment Variables
```bash
GITHUB_TOKEN              # Default token
GITHUB_WORKSPACE           # Workspace directory
GITHUB_SHA                 # Commit SHA
GITHUB_REF                 # Git ref
GITHUB_REPOSITORY          # Repository name
GITHUB_ACTOR               # User who triggered
GITHUB_RUN_ID              # Unique run ID
GITHUB_RUN_NUMBER          # Run number
CI                         # Always set to true
```

## Secrets and Variables

### Using Secrets
```yaml
# In workflow
env:
  API_KEY: ${{ secrets.API_KEY }}

# In step
- run: echo $API_KEY
  env:
    API_KEY: ${{ secrets.API_KEY }}
```

### Repository Variables
```yaml
# Use vars context for non-sensitive data
env:
  API_URL: ${{ vars.API_URL }}
  ENVIRONMENT: ${{ vars.ENVIRONMENT }}
```

## Permissions

### GITHUB_TOKEN Permissions
```yaml
permissions:
  contents: read|write       # Repository contents
  issues: read|write         # Issues
  pull-requests: read|write  # Pull requests
  packages: read|write       # Packages
  pages: read|write          # GitHub Pages
  deployments: read|write    # Deployments
  security-events: write     # Security events (SARIF)
  actions: read|write        # GitHub Actions
  checks: read|write         # Checks API
  statuses: read|write       # Commit statuses
```

## Common Patterns

### Skip CI
```yaml
# In commit message
git commit -m "docs: update README [skip ci]"
# or
git commit -m "docs: update README [ci skip]"
```

### Path-based Triggers
```yaml
on:
  push:
    paths:
      - 'src/**'
      - 'tests/**'
      - '!docs/**'  # Exclude docs
```

### Manual Inputs
```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
      version:
        description: 'Version to deploy'
        required: false
        type: string

# Access inputs
${{ inputs.environment }}
${{ inputs.version }}
```

### Reusable Workflows
```yaml
# Define reusable workflow
on:
  workflow_call:
    inputs:
      config-path:
        required: true
        type: string
    secrets:
      api-key:
        required: true

# Use reusable workflow
jobs:
  call-reusable:
    uses: ./.github/workflows/reusable.yml
    with:
      config-path: './config.json'
    secrets:
      api-key: ${{ secrets.API_KEY }}
```

## Debugging

### Enable Debug Logging
```yaml
env:
  ACTIONS_STEP_DEBUG: true
  ACTIONS_RUNNER_DEBUG: true
```

### Debug Commands
```yaml
- run: |
    echo "::debug::Debug message"
    echo "::notice::Notice message"
    echo "::warning::Warning message"
    echo "::error::Error message"
```

### Common Debugging Steps
```yaml
- name: Dump GitHub context
  run: echo '${{ toJSON(github) }}'

- name: List environment variables
  run: env | sort

- name: Show working directory
  run: |
    pwd
    ls -la
```

## Tips and Best Practices

1. **Pin action versions**: Use `@v4` or specific commit SHA
2. **Use caching**: Cache dependencies for faster builds
3. **Fail fast**: Set `fail-fast: false` in matrix for better debugging
4. **Use timeouts**: Add `timeout-minutes` to prevent hanging jobs
5. **Secure secrets**: Never log secrets, use environment variables
6. **Optimize triggers**: Use path filters to avoid unnecessary runs
7. **Use environments**: Separate staging and production workflows
8. **Monitor usage**: Check Actions usage in repository insights
