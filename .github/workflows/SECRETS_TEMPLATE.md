# GitHub Secrets Template

Add these secrets to your GitHub repository at:
`https://github.com/asliang123/leaky-repo/settings/secrets/actions`

## Required Secrets

### HCP_CLIENT_ID
```
Description: HCP Service Principal Client ID
Format: UUID format (e.g., xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx)
Where to get: HCP Portal → Access Control (IAM) → Service Principals
```

### HCP_CLIENT_SECRET
```
Description: HCP Service Principal Client Secret
Format: Long alphanumeric string
Where to get: HCP Portal → Access Control (IAM) → Service Principals
Note: Only shown once during service principal creation!
```

### HCP_PROJECT_ID
```
Description: HCP Project ID where Vault Radar results will be stored
Format: UUID format (e.g., xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx)
Where to get: HCP Portal → Project Settings
```

## How to Create HCP Service Principal

1. Log in to [HCP Portal](https://portal.cloud.hashicorp.com/)
2. Navigate to **Access Control (IAM)** → **Service Principals**
3. Click **Create service principal**
4. Name: `vault-radar-github-actions` (or your preferred name)
5. Assign role: **Contributor** or **Vault Radar Admin**
6. Click **Create**
7. **IMPORTANT**: Copy both Client ID and Client Secret immediately!
   - Client Secret is only shown once
   - Store it securely

## How to Get HCP Project ID

1. In HCP Portal, select your project from the dropdown
2. Click **Project Settings** (gear icon)
3. Copy the **Project ID** from the settings page

## Verification Checklist

Before pushing the workflow, verify:

- [ ] All three secrets are added to GitHub
- [ ] Service principal has appropriate permissions
- [ ] Project ID is from the correct HCP project
- [ ] No typos in secret names (they are case-sensitive)

## Testing Secrets

After adding secrets, you can test by:

1. Creating a test pull request
2. Pushing to the main branch
3. Checking the workflow run in GitHub Actions

If authentication fails, double-check:
- Secret names match exactly: `HCP_CLIENT_ID`, `HCP_CLIENT_SECRET`, `HCP_PROJECT_ID`
- Values are copied correctly without extra spaces
- Service principal is active and has permissions