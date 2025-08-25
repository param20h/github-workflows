# GitHub Workflows Step-by-Step Guide

This guide walks you through each workflow in this repository, explaining the concepts and helping you understand how to implement them.

## Table of Contents

1. [Getting Started](#getting-started)
2. [Workflow Deep Dives](#workflow-deep-dives)
3. [Setting Up Your Repository](#setting-up-your-repository)
4. [Common Patterns](#common-patterns)
5. [Troubleshooting](#troubleshooting)

## Getting Started

### Prerequisites

Before implementing these workflows, make sure you have:

- A GitHub repository
- Basic understanding of YAML syntax
- Node.js project (for the examples, but concepts apply to any language)
- Understanding of CI/CD concepts

### Your First Workflow

Start with the **Basic CI Workflow** (`basic-ci.yml`). This introduces fundamental concepts:

1. **Triggers**: When workflows run
2. **Jobs**: Units of work
3. **Steps**: Individual tasks
4. **Actions**: Reusable components

## Workflow Deep Dives

### 1. Basic CI Workflow (`basic-ci.yml`)

**Purpose**: Automated testing and quality checks for every code change.

**Key Concepts Learned**:
- Workflow triggers (`on: push`, `on: pull_request`)
- Using pre-built actions (`actions/checkout`, `actions/setup-node`)
- Running shell commands (`npm ci`, `npm test`)
- Conditional execution

**Implementation Steps**:
1. Create `.github/workflows/basic-ci.yml`
2. Add the workflow content
3. Ensure your `package.json` has the required scripts
4. Push to trigger the workflow

**Customization Options**:
```yaml
# Change Node.js version
node-version: '20'

# Add different triggers
on:
  push:
    branches: [ main, develop ]
    paths: ['src/**', 'tests/**']

# Add environment variables
env:
  NODE_ENV: test
  API_URL: https://api.example.com
```

### 2. Multi-Platform Testing (`multi-platform.yml`)

**Purpose**: Ensure your code works across different environments.

**Key Concepts Learned**:
- Matrix strategies for parallel execution
- Testing multiple OS and versions
- Job dependencies (`needs`)
- Conditional steps (`if`)

**Matrix Strategy Explained**:
```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node-version: [16, 18, 20]
```

This creates 9 jobs (3 OS × 3 Node versions), all running in parallel.

**When to Use**:
- Cross-platform applications
- Libraries used by others
- When you need to support multiple versions

### 3. Docker Build and Push (`docker-build.yml`)

**Purpose**: Automate container image creation and distribution.

**Key Concepts Learned**:
- Docker Buildx for multi-platform builds
- Registry authentication
- Metadata extraction for tags
- Layer caching for performance
- Security scanning

**Registry Options**:
- GitHub Container Registry (ghcr.io)
- Docker Hub
- AWS ECR
- Azure Container Registry

**Security Best Practices**:
- Use multi-stage builds
- Run as non-root user
- Scan for vulnerabilities
- Use specific base image tags

### 4. Deployment Workflow (`deploy.yml`)

**Purpose**: Automated deployment to different environments.

**Key Concepts Learned**:
- Environment protection rules
- Manual approval workflows
- Secrets management
- Rollback strategies
- Environment-specific configuration

**Environment Setup**:
1. Go to repository Settings → Environments
2. Create environments (staging, production)
3. Add protection rules for production
4. Configure environment secrets and variables

**Deployment Strategies**:
- **Blue-Green**: Zero downtime deployments
- **Rolling**: Gradual rollout
- **Canary**: Test with small user percentage

### 5. Release Automation (`release.yml`)

**Purpose**: Automate the release process from tag creation to publishing.

**Key Concepts Learned**:
- Tag-based triggers
- Asset compilation and upload
- Changelog generation
- Multi-registry publishing
- Semantic versioning

**Release Process**:
1. Create a version tag: `git tag v1.0.0`
2. Push the tag: `git push origin v1.0.0`
3. Workflow automatically creates release
4. Compiles and uploads assets
5. Publishes to package registries

### 6. Code Quality Checks (`code-quality.yml`)

**Purpose**: Maintain code quality standards automatically.

**Key Concepts Learned**:
- Multiple parallel jobs
- SARIF format for security findings
- Coverage thresholds
- Quality gates
- Bundle analysis

**Quality Metrics**:
- **Linting**: Code style and errors
- **Testing**: Code coverage percentage
- **Security**: Vulnerability scanning
- **Performance**: Bundle size tracking

### 7. Scheduled Maintenance (`scheduled.yml`)

**Purpose**: Automate routine maintenance tasks.

**Key Concepts Learned**:
- Cron scheduling
- Dependency updates
- Health monitoring
- Artifact cleanup
- Automated pull requests

**Cron Examples**:
```yaml
# Every day at 2 AM
- cron: '0 2 * * *'

# Every Monday at 9 AM
- cron: '0 9 * * 1'

# First day of every month
- cron: '0 0 1 * *'
```

### 8. Issue and PR Automation (`issue-automation.yml`)

**Purpose**: Automate project management tasks.

**Key Concepts Learned**:
- GitHub API integration
- Event-driven automation
- Stale issue management
- Auto-labeling and assignment
- Community management

## Setting Up Your Repository

### Required Repository Structure

```
your-repo/
├── .github/
│   └── workflows/
│       ├── basic-ci.yml
│       ├── multi-platform.yml
│       └── ...
├── src/
├── tests/
├── package.json
├── Dockerfile (optional)
└── README.md
```

### Required Secrets

Set up these secrets in your repository settings:

| Secret | Purpose | Required For |
|--------|---------|--------------|
| `CODECOV_TOKEN` | Code coverage reporting | Code Quality |
| `NPM_TOKEN` | Publishing to npm | Release |
| `SLACK_WEBHOOK_URL` | Notifications | Multiple workflows |
| `PRODUCTION_DEPLOY_KEY` | Production deployment | Deployment |

### Required Environment Variables

Configure these in environment settings:

| Variable | Purpose | Example |
|----------|---------|---------|
| `STAGING_API_URL` | Staging API endpoint | `https://staging.api.com` |
| `PRODUCTION_API_URL` | Production API endpoint | `https://api.com` |

## Common Patterns

### 1. Conditional Execution

```yaml
# Only run on main branch
if: github.ref == 'refs/heads/main'

# Only run for specific file changes
if: contains(github.event.head_commit.modified, 'src/')

# Only run for external contributors
if: github.event.pull_request.head.repo.full_name != github.repository
```

### 2. Sharing Data Between Jobs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.VERSION }}
    steps:
      - id: version
        run: echo "VERSION=1.0.0" >> $GITHUB_OUTPUT
        
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying version ${{ needs.build.outputs.version }}"
```

### 3. Reusable Workflows

Create `.github/workflows/reusable-test.yml`:
```yaml
name: Reusable Test Workflow

on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm test
```

Use it in other workflows:
```yaml
jobs:
  test:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '18'
```

### 4. Dynamic Matrix Generation

```yaml
jobs:
  generate-matrix:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.matrix.outputs.matrix }}
    steps:
      - id: matrix
        run: |
          # Generate matrix based on changed files
          echo "matrix={\"node\":[\"16\",\"18\",\"20\"]}" >> $GITHUB_OUTPUT
          
  test:
    needs: generate-matrix
    strategy:
      matrix: ${{ fromJson(needs.generate-matrix.outputs.matrix) }}
    runs-on: ubuntu-latest
    # ... rest of job
```

## Troubleshooting

### Common Issues and Solutions

#### 1. Workflow Not Triggering

**Problem**: Workflow doesn't run when expected.

**Solutions**:
- Check trigger syntax in `on:` section
- Verify branch names match exactly
- Ensure workflow file is in `main` branch
- Check if workflow is disabled

#### 2. Permission Denied Errors

**Problem**: `Error: Resource not accessible by integration`

**Solutions**:
- Add required permissions to job:
  ```yaml
  permissions:
    contents: write
    pull-requests: write
  ```
- Use `GITHUB_TOKEN` or personal access token
- Check repository settings for action permissions

#### 3. Secrets Not Available

**Problem**: Secrets are undefined or empty.

**Solutions**:
- Verify secret names match exactly (case-sensitive)
- Check if secrets are set at repository level
- For organization secrets, verify repository access
- Use `env:` to expose secrets to steps

#### 4. Job Dependencies Not Working

**Problem**: Jobs run in wrong order or skip.

**Solutions**:
- Use `needs:` to define dependencies
- Check conditional logic in `if:` statements
- Verify job names match exactly

#### 5. Caching Issues

**Problem**: Cache not working or causing errors.

**Solutions**:
- Use correct cache key pattern
- Include file hash in cache key:
  ```yaml
  cache: npm
  cache-dependency-path: package-lock.json
  ```
- Clear cache if corrupted (GitHub settings)

### Debugging Techniques

#### 1. Enable Debug Logging

```yaml
env:
  ACTIONS_STEP_DEBUG: true
  ACTIONS_RUNNER_DEBUG: true
```

#### 2. Add Debug Steps

```yaml
- name: Debug Context
  run: |
    echo "Event: ${{ github.event_name }}"
    echo "Ref: ${{ github.ref }}"
    echo "SHA: ${{ github.sha }}"
    echo "Actor: ${{ github.actor }}"
```

#### 3. Upload Debug Artifacts

```yaml
- name: Upload debug info
  uses: actions/upload-artifact@v4
  if: failure()
  with:
    name: debug-logs
    path: |
      /tmp/
      ~/.npm/_logs/
```

### Performance Tips

1. **Use Caching**: Cache dependencies and build artifacts
2. **Optimize Matrix**: Don't test unnecessary combinations
3. **Parallel Jobs**: Run independent jobs in parallel
4. **Skip Unnecessary Steps**: Use conditionals to skip steps
5. **Use Smaller Runners**: Use appropriate runner sizes

## Next Steps

1. **Start Simple**: Begin with basic-ci.yml
2. **Iterate**: Add complexity gradually
3. **Monitor**: Watch workflow runs and optimize
4. **Customize**: Adapt examples to your specific needs
5. **Learn More**: Explore GitHub Actions marketplace

Remember: Workflows are code too! Version control them, review changes, and test in feature branches before merging to main.
