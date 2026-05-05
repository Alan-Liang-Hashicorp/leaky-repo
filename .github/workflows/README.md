# Vault Radar GitHub Actions Setup

This directory contains GitHub Actions workflows for automated secret scanning using HashiCorp Vault Radar CLI.

## Overview

The `vault-radar.yml` workflow automatically scans your repository for exposed secrets, credentials, and sensitive data whenever code is pushed or a pull request is created. Results are uploaded to HashiCorp Cloud Platform (HCP) for centralized management.

## Prerequisites

1. **HCP Account**: Sign up at [HashiCorp Cloud Platform (HCP)](https://portal.cloud.hashicorp.com/)
2. **HCP Service Principal**: Create a service principal with Vault Radar permissions
3. **HCP Project**: Have an HCP project ID where Vault Radar results will be stored

## Setup Instructions

### Step 1: Create HCP Service Principal

1. Log in to [HCP Portal](https://portal.cloud.hashicorp.com/)
2. Navigate to **Access Control (IAM)** → **Service Principals**
3. Click **Create service principal**
4. Give it a name (e.g., "vault-radar-github-actions")
5. Assign appropriate roles (e.g., "Contributor" or "Vault Radar Admin")
6. Click **Create**
7. Copy the **Client ID** and **Client Secret** (you won't see the secret again!)

### Step 2: Get HCP Project ID

1. In HCP Portal, navigate to your project
2. Click on **Project Settings**
3. Copy the **Project ID** (format: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`)

### Step 3: Add Secrets to GitHub Repository

1. Go to your GitHub repository: `https://github.com/asliang123/leaky-repo`
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret** and add the following three secrets:

   - **Name**: `HCP_CLIENT_ID`
     - **Value**: Your HCP service principal client ID
   
   - **Name**: `HCP_CLIENT_SECRET`
     - **Value**: Your HCP service principal client secret
   
   - **Name**: `HCP_PROJECT_ID`
     - **Value**: Your HCP project ID

### Step 4: Verify Workflow Configuration

The workflow is already configured in `.github/workflows/vault-radar.yml` and will:
- Run on pushes to the `main` branch
- Run on pull requests targeting the `main` branch
- Automatically upload scan results to HCP

### Step 5: Test the Workflow

You can test the workflow by pushing a commit or creating a pull request:

```bash
cd /Users/alanliang/workspace/Hashicorp/leaky-repo
git add .github/
git commit -m "Add Vault Radar GitHub Actions workflow"
git push origin main
```

Or create a pull request to trigger the PR scanning workflow.

## Workflow Features

### Intelligent Scanning
- **Pull Request Scanning**: Scans only the changes in PRs for faster feedback
- **Push Scanning**: Scans commits pushed to the main branch
- **Adaptive Fetch Depth**: Configurable via `FETCH_DEPTH` environment variable (default: 1)
- **Latest Version**: Automatically downloads the latest Vault Radar CLI version

### Scan Modes
The workflow intelligently handles different scenarios:

1. **Shallow Clone (FETCH_DEPTH=1)**: Uses `--clone-pr-commits` to fetch PR commits
2. **Depth 2 Clone**: Scans merge commit with `--skip-not-latest`
3. **Full Clone**: Scans using head and base refs

### Results Handling
- **HCP Integration**: Results automatically uploaded to HCP with `--upload-risks-to-hcp`
- **GitHub Summary**: Scan summary displayed in GitHub Actions summary page
- **Severity Filtering**: Only reports high severity findings (`-s high`)
- **Pretty Output**: Formatted for GitHub Actions PR comments (`--pretty=gha_pr`)
- **Debug Logs**: Detailed logs available when workflow fails

### Viewing Results

#### In GitHub Actions
1. Go to the **Actions** tab in your repository
2. Click on a workflow run
3. View the summary in the workflow summary page
4. Check logs for detailed output

#### In HCP Portal
1. Log in to [HCP Portal](https://portal.cloud.hashicorp.com/)
2. Navigate to **Vault Radar**
3. View all detected risks across your repositories
4. Track risk remediation over time

## Customization Options

### Adjust Fetch Depth
Modify the `FETCH_DEPTH` environment variable at the top of the workflow:

```yaml
env:
  FETCH_DEPTH: 1  # Change to 0 for full history, 2 for merge commit + parent
```

### Change Severity Threshold
Edit the `-s` flag in the scan commands:

```yaml
-s high      # Only high severity (default)
-s medium    # Medium and above
-s low       # All severities
```

### Add More Branches
Modify the `on:` section to scan additional branches:

```yaml
on:
  push:
    branches:
      - "main"
      - "develop"  # Add more branches
  pull_request:
    branches:
      - main
      - develop
```

### Adjust Log Level
Change the `VAULT_RADAR_LOG_LEVEL` environment variable:

```yaml
VAULT_RADAR_LOG_LEVEL: debug  # Options: debug, info, warn, error
```

### Add Push Commit Scanning
The workflow currently has a TODO for push commit scanning. To implement:

```yaml
else
  # Scan pushed commits
  ./vault-radar scan ci push \
    --ref-name ${{ github.ref_name }} \
    --commit ${sha} \
    -s high \
    -o vault-radar.jsonl \
    -l vault-radar.log \
    --pretty=gha \
    --summary-pretty markdown \
    --summary-outfile $GITHUB_STEP_SUMMARY \
    --upload-risks-to-hcp
fi
```

## Troubleshooting

### Workflow Fails with Authentication Error
**Error**: `Failed to authenticate with HCP`

**Solutions**:
- Verify all three secrets are correctly set: `HCP_CLIENT_ID`, `HCP_CLIENT_SECRET`, `HCP_PROJECT_ID`
- Ensure the service principal has appropriate permissions
- Check that the service principal hasn't been deleted or disabled
- Verify the project ID is correct

### Workflow Fails to Download CLI
**Error**: `Failed to download Vault Radar CLI`

**Solutions**:
- Check your internet connectivity
- Verify the HashiCorp releases API is accessible
- Try pinning to a specific version instead of using `latest`

### No Secrets Detected (But You Know There Are Some)
**Possible Causes**:
- Severity threshold is set to `high` - lower it to `medium` or `low`
- Secrets might be in files not scanned (check scan scope)
- Vault Radar may need custom rules for your specific secret types

**Solutions**:
- Review `vault-radar.log` in the debugging step
- Check `vault-radar.jsonl` for raw scan results
- Lower the severity threshold
- Update to the latest Vault Radar version

### PR Comments Not Appearing
**Issue**: Scan runs but no PR comments

**Note**: The current workflow uses `--pretty=gha_pr` which formats output for GitHub Actions summary, not PR comments. To add PR comments, you would need to:
1. Parse the `vault-radar.jsonl` output
2. Use a GitHub Action to post comments (e.g., `actions/github-script`)

### Debugging Failed Scans
The workflow includes a debugging step that runs on failure:

```yaml
- name: debugging
  if: failure()
  run: |
    cat vault-radar.log
    cat vault-radar.jsonl
```

Check the workflow logs for these outputs to diagnose issues.

## Additional Resources

- [Vault Radar Documentation](https://developer.hashicorp.com/vault/docs/radar)
- [Vault Radar CLI Reference](https://developer.hashicorp.com/vault/docs/radar/cli)
- [HCP Portal](https://portal.cloud.hashicorp.com/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

## Security Best Practices

1. **Never commit secrets**: Use environment variables and secrets management
2. **Review findings promptly**: Address detected secrets immediately
3. **Rotate exposed credentials**: If secrets are found, rotate them ASAP
4. **Use .gitignore**: Prevent sensitive files from being committed
5. **Enable branch protection**: Require passing Vault Radar checks before merging

## Support

For issues or questions:
- GitHub Issues: https://github.com/asliang123/leaky-repo/issues
- HashiCorp Support: https://support.hashicorp.com/