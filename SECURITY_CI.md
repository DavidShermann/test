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

### 📧 Email Notifications

- Sends formatted HTML email reports after each security scan
- Includes vulnerability summary with severity breakdown
- Attaches detailed reports (npm-audit, security-summary)
- Color-coded status indicators (Critical/Warning/Passed)
- Direct links to GitHub Actions workflow run

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

### 2. Configure Secrets and Variables

#### Email Notification Setup (Recommended)

To receive security scan reports via email:

**Step 1: Add Repository Variable**
1. Go to Repository Settings → Secrets and variables → Actions → Variables tab
2. Click "New repository variable"
3. Add variable:
   - Name: `SECURITY_EMAIL`
   - Value: `your-email@example.com` (or comma-separated list: `dev-team@example.com,security@example.com`)

**Step 2: Add Email Server Secrets**
Go to Repository Settings → Secrets and variables → Actions → Secrets tab and add:

- `MAIL_SERVER`: SMTP server address (e.g., `smtp.gmail.com`, `smtp.office365.com`, `smtp.sendgrid.net`)
- `MAIL_PORT`: SMTP port (default: `587` for TLS, `465` for SSL)
- `MAIL_USERNAME`: Your email account username
- `MAIL_PASSWORD`: Your email account password or app-specific password
- `MAIL_FROM`: (Optional) Email address to send from (defaults to `MAIL_USERNAME`)

**Common SMTP Configurations:**

**Gmail:**
```
MAIL_SERVER: smtp.gmail.com
MAIL_PORT: 587
MAIL_USERNAME: your-email@gmail.com
MAIL_PASSWORD: your-app-specific-password
```
Note: Enable 2-factor authentication and create an [App Password](https://myaccount.google.com/apppasswords)

**Office 365/Outlook:**
```
MAIL_SERVER: smtp.office365.com
MAIL_PORT: 587
MAIL_USERNAME: your-email@outlook.com
MAIL_PASSWORD: your-password
```

**SendGrid:**
```
MAIL_SERVER: smtp.sendgrid.net
MAIL_PORT: 587
MAIL_USERNAME: apikey
MAIL_PASSWORD: your-sendgrid-api-key
```

**AWS SES:**
```
MAIL_SERVER: email-smtp.us-east-1.amazonaws.com
MAIL_PORT: 587
MAIL_USERNAME: your-smtp-username
MAIL_PASSWORD: your-smtp-password
```

#### Other Secrets (Optional)

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
11. Prepare HTML email report with vulnerability summary
12. Send email notification (if configured)
13. Fail if critical vulnerabilities found

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

### Email Reports

If email notifications are configured, you'll receive:

**Email Features:**
- Professional HTML-formatted report
- Color-coded status header (🚨 CRITICAL / ⚠️ WARNING / ✅ PASSED)
- Vulnerability summary table with counts by severity
- Scan details (Node version, commit, triggered by, date)
- List of security tools used
- Direct link button to view full report in GitHub Actions
- Attached files: `npm-audit-report.txt`, `security-summary.md`

**Email Subject Format:**
```
[STATUS] - Security Scan: your-org/your-repo [branch-name]
```

**Example Subjects:**
- `✅ PASSED - Security Scan: acme/web-app [main]`
- `⚠️ WARNING - Security Scan: acme/web-app [develop]`
- `🚨 CRITICAL - Security Scan: acme/web-app [feature/auth]`

**When Emails Are Sent:**
- After every security scan (on push, PR, or manual trigger)
- Regardless of pass/fail status (always sent if configured)
- Separate email for each Node.js version tested

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

### Email Notifications Not Working

**Email not being sent:**
- Verify `SECURITY_EMAIL` variable is set (not a secret, but a variable)
- Check all required secrets are configured: `MAIL_SERVER`, `MAIL_USERNAME`, `MAIL_PASSWORD`
- Review workflow logs for email step errors
- Ensure email step shows "if: always() && vars.SECURITY_EMAIL != ''" condition is met

**Authentication errors:**
- For Gmail: Use App Password, not regular password (requires 2FA enabled)
- For Office 365: Ensure account allows SMTP access
- For SendGrid/AWS SES: Verify API credentials are correct
- Check SMTP server address and port are correct for your provider

**Email delivered to spam:**
- Add the sender email to your contacts
- Check SPF/DKIM records if using custom domain
- Use a reputable SMTP provider (Gmail, SendGrid, AWS SES)

**Testing email configuration:**
You can test email settings by triggering a manual workflow run:
1. Go to Actions tab → Security Scan CI
2. Click "Run workflow" button
3. Check logs for email step output

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
