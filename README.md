# GitHub Workflows Learning Repository

A comprehensive guide to understanding and implementing GitHub Actions workflows. This repository contains practical examples and explanations to help you master CI/CD automation with GitHub Actions.

## Table of Contents

1. [What are GitHub Workflows?](#what-are-github-workflows)
2. [Basic Concepts](#basic-concepts)
3. [Workflow Examples](#workflow-examples)
4. [Best Practices](#best-practices)
5. [Common Use Cases](#common-use-cases)
6. [Resources](#resources) 

## What are GitHub Workflows?

GitHub Workflows are automated processes that you can set up in your repository to build, test, package, release, or deploy your project. They are defined using YAML syntax and stored in the `.github/workflows` directory of your repository.

## Basic Concepts

### Key Components

- **Workflow**: An automated procedure made up of one or more jobs
- **Job**: A set of steps that execute on the same runner
- **Step**: An individual task that can run commands or actions
- **Action**: A reusable unit of code that can be combined into steps
- **Runner**: A server that runs your workflows when triggered
- **Event**: A specific activity that triggers a workflow run

### Workflow Syntax

```yaml
name: Workflow Name
on: [trigger_events]
jobs:
  job_name:
    runs-on: ubuntu-latest
    steps:
      - name: Step Name
        uses: action@version
        with:
          parameter: value
```

## Workflow Examples

This repository includes the following workflow examples:

### 1. Basic CI Workflow (`basic-ci.yml`)
- **Purpose**: Simple continuous integration for Node.js projects
- **Triggers**: Push and pull requests to main branch
- **Features**: 
  - Checkout code
  - Setup Node.js
  - Install dependencies
  - Run tests
  - Run linting

### 2. Multi-Platform Testing (`multi-platform.yml`)
- **Purpose**: Test across multiple operating systems and Node.js versions
- **Triggers**: Push to main, pull requests
- **Features**:
  - Matrix strategy for OS and Node versions
  - Parallel job execution
  - Cross-platform compatibility testing

### 3. Docker Build and Push (`docker-build.yml`)
- **Purpose**: Build Docker images and push to registry
- **Triggers**: Push to main, tags
- **Features**:
  - Docker build and tag
  - Multi-stage builds
  - Push to Docker Hub or GitHub Container Registry

### 4. Deployment Workflow (`deploy.yml`)
- **Purpose**: Deploy application to production
- **Triggers**: Push to main branch
- **Features**:
  - Environment-specific deployments
  - Secrets management
  - Rollback capabilities

### 5. Release Automation (`release.yml`)
- **Purpose**: Automate release creation and asset publishing
- **Triggers**: Tag creation
- **Features**:
  - Semantic versioning
  - Changelog generation
  - Asset compilation and upload

### 6. Code Quality Checks (`code-quality.yml`)
- **Purpose**: Maintain code quality standards
- **Triggers**: Pull requests
- **Features**:
  - Code linting
  - Security scanning
  - Code coverage reporting
  - Static analysis

### 7. Scheduled Maintenance (`scheduled.yml`)
- **Purpose**: Run periodic maintenance tasks
- **Triggers**: Cron schedule
- **Features**:
  - Dependency updates
  - Health checks
  - Cleanup tasks

### 8. Issue and PR Automation (`issue-automation.yml`)
- **Purpose**: Automate issue and pull request management
- **Triggers**: Issue/PR events
- **Features**:
  - Auto-labeling
  - Stale issue management
  - Welcome messages

## Best Practices

### Security
- ✅ Use secrets for sensitive data
- ✅ Limit permissions with `GITHUB_TOKEN`
- ✅ Pin action versions to specific commits
- ✅ Use environment protection rules
- ❌ Never hardcode credentials in workflows

### Performance
- ✅ Use caching for dependencies
- ✅ Optimize Docker layer caching
- ✅ Use matrix strategies for parallel execution
- ✅ Skip unnecessary jobs with conditions
- ❌ Don't run heavy jobs on every push

### Maintainability
- ✅ Use descriptive names for workflows and jobs
- ✅ Add comments for complex logic
- ✅ Use reusable workflows
- ✅ Keep workflows focused and simple
- ❌ Don't create monolithic workflows

### Reliability
- ✅ Use timeout settings
- ✅ Handle failures gracefully
- ✅ Use appropriate retry strategies
- ✅ Test workflows in feature branches
- ❌ Don't ignore error conditions

## Common Use Cases

### Development Workflows
- **Continuous Integration**: Automated testing on code changes
- **Code Quality**: Linting, formatting, and security checks
- **Documentation**: Auto-generate and deploy documentation

### Deployment Workflows
- **Staging Deployment**: Deploy to staging environment
- **Production Deployment**: Deploy to production with approvals
- **Blue-Green Deployment**: Zero-downtime deployments

### Maintenance Workflows
- **Dependency Updates**: Automated dependency updates
- **Security Scanning**: Regular security vulnerability checks
- **Backup and Cleanup**: Automated maintenance tasks

### Release Workflows
- **Semantic Versioning**: Automated version bumping
- **Release Notes**: Auto-generate changelogs
- **Package Publishing**: Publish to npm, PyPI, etc.

## Common Triggers

### Event Triggers
- `push`: When code is pushed to repository
- `pull_request`: When PR is opened, updated, or closed
- `release`: When a release is published
- `issues`: When issues are opened, closed, etc.
- `workflow_dispatch`: Manual trigger from GitHub UI

### Scheduled Triggers
```yaml
on:
  schedule:
    - cron: '0 0 * * 0'  # Every Sunday at midnight
```

### Conditional Triggers
```yaml
on:
  push:
    branches: [main, develop]
    paths: ['src/**', 'tests/**']
```

## Environment Variables and Secrets

### Default Environment Variables
- `GITHUB_TOKEN`: Authentication token
- `GITHUB_REPOSITORY`: Repository name
- `GITHUB_SHA`: Commit SHA
- `GITHUB_REF`: Branch or tag ref

### Using Secrets
```yaml
steps:
  - name: Deploy
    env:
      API_KEY: ${{ secrets.API_KEY }}
    run: ./deploy.sh
```

## Debugging Workflows

### Common Issues
1. **Syntax Errors**: Use YAML validators
2. **Action Not Found**: Check action names and versions
3. **Permission Denied**: Review GITHUB_TOKEN permissions
4. **Timeout**: Add timeout settings to jobs/steps

### Debugging Tips
- Use `actions/upload-artifact` to save debug files
- Add debug outputs with `echo "::debug::message"`
- Enable step debugging with `ACTIONS_STEP_DEBUG`
- Use `jobs.<job_id>.if` for conditional execution

## Resources

### Official Documentation
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Action Marketplace](https://github.com/marketplace?type=actions)

### Useful Actions
- [`actions/checkout`](https://github.com/actions/checkout): Check out repository
- [`actions/setup-node`](https://github.com/actions/setup-node): Setup Node.js
- [`actions/cache`](https://github.com/actions/cache): Cache dependencies
- [`actions/upload-artifact`](https://github.com/actions/upload-artifact): Upload build artifacts
- [`docker/build-push-action`](https://github.com/docker/build-push-action): Build and push Docker images

### Learning Resources
- [GitHub Actions Learning Lab](https://lab.github.com/github/github-actions:-hello-world)
- [Awesome Actions](https://github.com/sdras/awesome-actions)
- [GitHub Actions Community](https://github.community/c/code-to-cloud/github-actions/41)

## Getting Started

1. Clone this repository
2. Examine the workflow files in `.github/workflows/`
3. Read the comments and documentation for each workflow
4. Try modifying the examples for your own projects
5. Test workflows in a fork or test repository first

## Contributing

Feel free to contribute additional workflow examples or improvements to existing ones. Please follow the established naming conventions and include thorough documentation.

---

*Happy automating! 🚀*
