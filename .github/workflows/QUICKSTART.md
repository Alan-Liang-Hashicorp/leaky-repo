# Vault Radar Quick Start Guide

## 🚀 Quick Setup (5 minutes)

### 1. Get HCP Credentials

Visit [HCP Portal](https://portal.cloud.hashicorp.com/) and collect:

- ✅ **HCP_CLIENT_ID**: Service principal client ID
- ✅ **HCP_CLIENT_SECRET**: Service principal client secret  
- ✅ **HCP_PROJECT_ID**: Your HCP project ID

### 2. Add GitHub Secrets

Go to: `https://github.com/asliang123/leaky-repo/settings/secrets/actions`

Add these three secrets:
```
HCP_CLIENT_ID       → Your service principal client ID
HCP_CLIENT_SECRET   → Your service principal client secret
HCP_PROJECT_ID      → Your HCP project ID
```

### 3. Push the Workflow

```bash
cd /Users/alanliang/workspace/Hashicorp/leaky-repo
git add .github/
git commit -m "Add Vault Radar GitHub Actions workflow"
git push origin main
```

### 4. Verify

- Go to: `https://github.com/asliang123/leaky-repo/actions`
- Watch the "Vault Radar Scan" workflow run
- Check results in HCP Portal

## 📋 What Gets Scanned?

- ✅ Pull requests (only changed files)
- ✅ Pushes to main branch
- ✅ High severity secrets only (configurable)

## 🔍 Where to View Results?

### GitHub Actions
- Workflow summary page shows scan results
- Failed workflows show detailed logs

### HCP Portal
- Centralized dashboard for all repositories
- Historical tracking of risks
- Risk remediation workflow

## ⚙️ Common Configurations

### Scan All Severities
Edit `.github/workflows/vault-radar.yml`, change:
```yaml
-s high    →    -s low
```

### Scan More Branches
Edit the `on:` section:
```yaml
on:
  push:
    branches:
      - "main"
      - "develop"  # Add this
```

### Full History Scan
Change at the top of the workflow:
```yaml
env:
  FETCH_DEPTH: 1    →    FETCH_DEPTH: 0
```

## 🆘 Quick Troubleshooting

| Issue | Solution |
|-------|----------|
| Authentication failed | Verify all 3 secrets are set correctly |
| No secrets found | Lower severity threshold from `high` to `medium` |
| Workflow fails | Check debugging step output in workflow logs |
| Results not in HCP | Verify `HCP_PROJECT_ID` is correct |

## 📚 Full Documentation

See [README.md](./README.md) for complete documentation.

## 🔗 Useful Links

- [HCP Portal](https://portal.cloud.hashicorp.com/)
- [Vault Radar Docs](https://developer.hashicorp.com/vault/docs/radar)
- [GitHub Actions](https://github.com/asliang123/leaky-repo/actions)