# GitHub Environments Setup for Manual Approvals

To enable manual approval steps in your GitHub Actions workflows, you need to set up **Environments** with protection rules.

## Step 1: Create Environments

1. Go to your GitHub repository
2. Click on **Settings** tab
3. In the left sidebar, click **Environments**
4. Click **New environment**

Create these environments:

### Environment 1: `production-approval`
- **Name:** `production-approval`
- **Description:** Basic production deployment approval

### Environment 2: `traffic-switch-approval`  
- **Name:** `traffic-switch-approval`
- **Description:** Approval required before switching traffic between blue/green environments

### Environment 3: `cleanup-approval`
- **Name:** `cleanup-approval`  
- **Description:** Approval required before cleaning up old cluster resources

## Step 2: Configure Protection Rules

For each environment, add these protection rules:

### Required Reviewers
1. Click on the environment name
2. Check **Required reviewers**
3. Add team members or teams who can approve deployments
4. Set **Number of required reviewers** (recommended: 1-2)

### Wait Timer (Optional)
- Set a wait timer if you want a minimum delay before approval
- Example: 5 minutes for basic checks

### Allowed Branches (Recommended)
- Restrict which branches can deploy to this environment
- Example: Only `main` branch for production environments

## Step 3: Environment-Specific Secrets (Optional)

If you need different AWS credentials for different environments:

1. In each environment settings
2. Add environment-specific secrets:
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`

## Step 4: Test the Approval Process

1. Run the workflow with manual approval
2. The workflow will pause at approval steps
3. Approvers will receive notifications
4. Go to **Actions** → **Running workflow** → **Review deployments**
5. Click **Approve and deploy** or **Reject**

## Approval Workflow Behavior

### Traffic Switch Approval Flow:
```
Deploy → Smoke Tests → ⏸️ WAIT FOR APPROVAL → Switch Traffic → Validation
```

### Cleanup Approval Flow:
```
Traffic Switch → Post-Switch Validation → ⏸️ WAIT FOR APPROVAL → Cleanup
```

## Notifications

Approvers will be notified via:
- GitHub notifications
- Email (if enabled)
- Slack/Teams (if configured)

## Best Practices

### 1. Multiple Approval Levels
```yaml
environment: 
  name: production-approval
  url: https://your-app.com  # Link to deployed app for testing
```

### 2. Conditional Approvals
```yaml
# Only require approval for production deployments
environment: ${{ github.ref == 'refs/heads/main' && 'production-approval' || '' }}
```

### 3. Timeout Settings
```yaml
timeout-minutes: 60  # Auto-fail if no approval within 1 hour
```

### 4. Custom Approval Messages
Use step summaries to provide context:
```yaml
- name: Approval Context
  run: |
    echo "## 🚀 Ready for Production" >> $GITHUB_STEP_SUMMARY
    echo "Version: ${{ inputs.version }}" >> $GITHUB_STEP_SUMMARY
    echo "Tests: ✅ All passed" >> $GITHUB_STEP_SUMMARY
```

## Troubleshooting

### Common Issues:

1. **No approval notification received**
   - Check if reviewer has repository access
   - Verify notification settings

2. **Environment not found error**
   - Ensure environment name matches exactly in workflow
   - Case-sensitive matching

3. **Approval bypassed**
   - Check branch protection rules
   - Verify environment is properly configured

### Emergency Bypass

Repository admins can bypass approval requirements if needed:
1. Go to the running workflow
2. Click **Review deployments** 
3. Select **Override protection rules** (admin only)

## Example Approval Email

When approval is required, reviewers receive:
```
🚀 Deployment approval required

Repository: your-org/your-repo
Environment: traffic-switch-approval  
Branch: main
Commit: abc1234 - "Deploy version v1.2.3"

Review deployment: [View workflow]
```

This setup provides a robust approval process for your blue-green deployments while maintaining flexibility and security.