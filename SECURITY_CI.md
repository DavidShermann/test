# Security CI/CD Pipeline Documentation

## Overview

This repository is configured with an automated security scanning CI/CD pipeline that triggers on every code push. The pipeline performs comprehensive vulnerability scanning and publishes artifacts.

## Pipeline Features

### 🔒 Security Scanning Tools

1. **NPM Audit**
   - Scans package dependencies for known vulnerabilities
   - Checks the npm registry for security advisories
   - Generates JSON and text reports

2. **Snyk Security Scan**
   - Advanced vulnerability detection
   - License compliance checking
   - Fix recommendations
   - Requires `SNYK_TOKEN` secret (optional but recommended)

3. **OWASP Dependency-Check**
   - Identifies project dependencies
   - Checks for known, publicly disclosed vulnerabilities
   - Generates reports in multiple formats (HTML, JSON, SARIF)

4. **Package Outdated Check**
   - Identifies outdated dependencies
   - Helps maintain up-to-date packages

### 📦 Artifact Publishing

- Automatically creates and uploads build artifacts
- Artifacts include application bundle (tar.gz)
- 90-day retention for production builds
- Automatic release asset creation for tagged versions

## Workflow Triggers

The security scan runs on:
- Push to `main`, `master`, `develop`, or `claude/**` branches
- Pull requests to `main`, `master`, `develop` branches
- Manual trigger via GitHub Actions UI

## Setup Instructions

### 1. Add package.json

Rename `package.json.example` to `package.json` and configure your Node.js application:

```bash
mv package.json.example package.json
```

### 2. Configure Secrets (Optional but Recommended)

Add the following secrets to your GitHub repository:

- `SNYK_TOKEN`: Get your token from [Snyk.io](https://snyk.io/)
  - Go to Repository Settings → Secrets and variables → Actions
  - Click "New repository secret"
  - Add `SNYK_TOKEN` with your Snyk API token

### 3. Install Dependencies

```bash
npm install
```

### 4. Push Your Code

```bash
git add .
git commit -m "Add Node.js application"
git push
```

The CI pipeline will automatically start!

## Understanding the Pipeline

### Job 1: security-scan

Runs on multiple Node.js versions (18.x, 20.x) to ensure compatibility.

**Steps:**
1. Checkout code
2. Setup Node.js environment
3. Install dependencies
4. Run npm audit
5. Run Snyk scan
6. Run OWASP Dependency-Check
7. Check for outdated packages
8. Generate security summary report
9. Upload all reports as artifacts
10. Upload SARIF report to GitHub Security tab
11. Fail if critical vulnerabilities found

**Failure Conditions:**
- Any critical vulnerabilities found
- More than 5 high-severity vulnerabilities

### Job 2: build-and-publish-artifact

Runs only after security scan passes and only on main/master branch.

**Steps:**
1. Checkout code
2. Setup Node.js
3. Install dependencies
4. Run tests
5. Build application
6. Create artifact bundle
7. Upload artifacts
8. Create release assets (for tagged releases)

## Viewing Reports

### Security Reports

1. Go to Actions tab in GitHub
2. Click on the workflow run
3. Scroll down to "Artifacts" section
4. Download `security-reports-node-{version}` artifact

Reports included:
- `npm-audit-report.json` - NPM audit in JSON format
- `npm-audit-report.txt` - NPM audit human-readable format
- `snyk-report.json` - Snyk scan results
- `dependency-check-report/` - OWASP reports (HTML, JSON, SARIF)
- `outdated-packages.txt` - List of outdated packages
- `security-summary.md` - Consolidated summary

### Build Artifacts

1. Go to Actions tab
2. Click on successful workflow run
3. Download `nodejs-app-{sha}` artifact
4. Contains: `app-bundle-{sha}.tar.gz`

## Security Alerts Integration

The pipeline uploads SARIF reports to GitHub's Security tab:

1. Go to "Security" tab in your repository
2. Click "Code scanning alerts"
3. View detailed vulnerability information
4. Track fix status over time

## Local Security Testing

Run security scans locally before pushing:

```bash
# NPM audit
npm audit

# Fix automatically (where possible)
npm audit fix

# Check for outdated packages
npm outdated

# Install Snyk CLI
npm install -g snyk

# Run Snyk scan
snyk test
```

## Customization

### Adjust Vulnerability Thresholds

Edit `.github/workflows/security-scan.yml`:

```yaml
if [ "$CRITICAL" -gt 0 ]; then
  # Change this number to adjust threshold
  exit 1
fi
```

### Add More Security Tools

Consider adding:
- **ESLint security plugins** - Static code analysis
- **SonarCloud** - Code quality and security
- **Trivy** - Container scanning
- **GitGuardian** - Secret detection

### Modify Artifact Retention

Change retention days in workflow:

```yaml
retention-days: 90  # Change to your preferred duration
```

## Best Practices

1. **Keep dependencies updated** - Regular updates reduce vulnerabilities
2. **Review audit reports** - Don't ignore security warnings
3. **Use lock files** - Commit `package-lock.json` for reproducible builds
4. **Enable Dependabot** - Automated dependency updates
5. **Set up branch protection** - Require security checks to pass before merge

## Troubleshooting

### Pipeline Fails Immediately

- Ensure `package.json` exists in repository root
- Check Node.js version compatibility

### Snyk Scan Skipped

- Add `SNYK_TOKEN` secret to repository
- Snyk scan works without token but with limited features

### Too Many Vulnerability Alerts

- Review and update dependencies: `npm update`
- Use `npm audit fix` to automatically fix issues
- Consider adjusting threshold in workflow

### Artifacts Not Uploading

- Check workflow permissions in repository settings
- Ensure Actions have write permissions

## Support

For issues with the CI pipeline:
1. Check workflow logs in Actions tab
2. Review error messages in job output
3. Verify repository secrets are configured
4. Check Node.js version compatibility

## Pipeline Status

Add a badge to your README:

```markdown
![Security Scan](https://github.com/YOUR_USERNAME/YOUR_REPO/workflows/Security%20Scan%20CI/badge.svg)
```

Replace `YOUR_USERNAME` and `YOUR_REPO` with your GitHub details.
