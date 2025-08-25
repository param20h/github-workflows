# Known Issues and Warnings in Example Workflows

This document explains the warnings you might see in the example workflows and how to resolve them.

## Expected Warnings

### Secret Context Warnings

You may see warnings like:
```
Context access might be invalid: SLACK_WEBHOOK_URL
Context access might be invalid: CODECOV_TOKEN
Context access might be invalid: NPM_TOKEN
```

**Why this happens**: These are example secret names that would need to be configured in your actual repository.

**How to fix**: 
1. Go to your repository Settings → Secrets and variables → Actions
2. Add the required secrets with the exact names shown in the workflows
3. The warnings will disappear once the secrets are properly configured

### Required Secrets by Workflow

#### `deploy.yml`
- `STAGING_DEPLOY_KEY` - SSH key for staging deployment
- `PRODUCTION_DEPLOY_KEY` - SSH key for production deployment  
- `PRODUCTION_SSH_KEY` - SSH key for production server access
- `SLACK_WEBHOOK_URL` - Slack webhook for notifications

#### `release.yml`
- `NPM_TOKEN` - Token for publishing to npm registry
- `SLACK_WEBHOOK_URL` - Slack webhook for notifications

#### `code-quality.yml`
- `CODECOV_TOKEN` - Token for Codecov coverage reporting
- `SEMGREP_APP_TOKEN` - Token for Semgrep security scanning

#### `scheduled.yml`
- `SLACK_WEBHOOK_URL` - Slack webhook for maintenance notifications

### Required Variables by Workflow

#### `deploy.yml`
- `STAGING_API_URL` - URL for staging environment API
- `PRODUCTION_API_URL` - URL for production environment API

## Setting Up Secrets and Variables

### Repository Secrets
1. Navigate to your repository on GitHub
2. Go to Settings → Secrets and variables → Actions
3. Click "New repository secret"
4. Add each secret with the exact name shown in the workflows

### Environment Variables
1. In the same Settings → Secrets and variables → Actions page
2. Click on the "Variables" tab
3. Click "New repository variable"
4. Add each variable with the exact name shown in the workflows

### Environment-Specific Secrets
For production deployments, consider using Environment secrets:
1. Go to Settings → Environments
2. Create environments (staging, production)
3. Add environment-specific secrets and variables
4. Set up protection rules for production environment

## Customizing for Your Project

### Remove Unused Features
If you don't need certain integrations, you can:
- Remove Slack notification steps
- Remove Codecov integration steps
- Remove npm publishing steps
- Simplify deployment workflows

### Example: Simple CI without External Services
```yaml
name: Simple CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm test
```

## Testing Your Workflows

1. **Start Small**: Begin with the basic-ci.yml workflow
2. **Use Feature Branches**: Test workflow changes in feature branches first
3. **Check Workflow Runs**: Monitor the Actions tab to see results
4. **Debug Issues**: Use the workflow logs to troubleshoot problems

Remember: These warnings are normal for example workflows and will resolve once you configure the secrets and variables for your specific use case!
