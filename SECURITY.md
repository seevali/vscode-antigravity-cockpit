# Security Policy

## Supported Versions

We take security seriously and actively maintain security updates for the following versions:

| Version | Supported          |
| ------- | ------------------ |
| 1.8.x   | :white_check_mark: |
| < 1.8.0 | :x:                |

## Reporting a Vulnerability

**Please do NOT report security vulnerabilities through public GitHub issues.**

### How to Report

If you discover a security vulnerability in Antigravity Cockpit, please report it by:

1. **GitHub Security Advisory** (Preferred)
   - Navigate to the [Security tab](https://github.com/seevali/vscode-antigravity-cockpit/security)
   - Click "Report a vulnerability"
   - Provide detailed information about the vulnerability

2. **Email** (Alternative)
   - Contact the maintainers directly via email
   - Include "SECURITY" in the subject line
   - Provide detailed steps to reproduce the vulnerability

### What to Include in Your Report

Please include the following information:

- Type of vulnerability (e.g., code injection, credential exposure, etc.)
- Full paths of source file(s) related to the vulnerability
- Location of the affected source code (tag/branch/commit or direct URL)
- Step-by-step instructions to reproduce the issue
- Proof-of-concept or exploit code (if possible)
- Impact of the vulnerability
- Any potential mitigations you've identified

### What to Expect

- **Response Time:** We aim to respond to security reports within 48 hours
- **Updates:** We will keep you informed about our progress
- **Disclosure Timeline:** We request 90 days to address the vulnerability before public disclosure
- **Credit:** We will credit researchers who responsibly disclose vulnerabilities (if desired)

## Security Measures

### Current Security Features

1. **Credential Storage**
   - All OAuth tokens and credentials are encrypted using VS Code's SecretStorage API
   - No credentials are stored in plain text on disk
   - Credentials are only accessible to the extension

2. **Network Security**
   - All external API communications use HTTPS
   - Request timeouts prevent hanging connections
   - Proper error handling prevents information leakage

3. **Code Execution**
   - No use of `eval()` or dynamic code execution
   - System commands use predefined constants
   - Input validation on all user-provided data

4. **Privacy**
   - Telemetry is **disabled by default**
   - Users must explicitly opt-in to error reporting
   - No personally identifiable information (PII) is collected
   - Respects VS Code's global telemetry settings

5. **Dependency Management**
   - Regular dependency updates
   - Minimal external dependencies
   - Regular security audits

### Known Limitations

- The extension requires local system access to detect Antigravity processes
- OAuth flow requires temporary local HTTP server (localhost only)
- Some features require network access to Google Cloud APIs

## Security Best Practices for Users

### Installation

1. **Official Sources Only**
   - Install from official VS Code Marketplace or Open VSX Registry
   - Verify the publisher is `jlcodes`
   - Check the download count and reviews

2. **Review Permissions**
   - Review the extension's required permissions before installation
   - Understand what the extension can access

3. **Keep Updated**
   - Enable automatic extension updates in VS Code
   - Review changelogs for security fixes
   - Update promptly when security patches are released

### Configuration

1. **Telemetry Settings**
   - Review telemetry settings in extension preferences
   - Opt-out of error reporting if concerned about privacy
   - Understand what data is collected (see Privacy section)

2. **Authorization**
   - Only authorize the extension when you need to use authorized quota features
   - Review OAuth permissions before granting access
   - Revoke authorization when no longer needed

3. **Sensitive Data**
   - Enable data masking in settings if sharing screenshots
   - Be cautious when sharing logs or error messages
   - Use the "Hide Profile" option if needed

## Privacy Policy

### Data Collection

**Telemetry (Opt-In, Disabled by Default):**
- Error messages and stack traces (sanitized)
- Extension version and configuration
- Platform and VS Code version information
- Anonymous machine ID (provided by VS Code)

**NOT Collected:**
- Personal information (name, email, etc.)
- Source code or project files
- OAuth tokens or credentials
- File paths or directory structures
- IP addresses (beyond what's necessary for error reporting)

### Data Usage

Collected data is used solely for:
- Debugging and fixing issues
- Improving extension stability
- Understanding platform-specific problems

### Data Storage

- Telemetry data is sent to Sentry (if enabled)
- OAuth credentials are stored locally in VS Code SecretStorage
- No data is shared with third parties beyond error reporting service

### User Rights

You have the right to:
- Opt-out of telemetry at any time
- Request deletion of your error reports
- Review what data is collected
- Control what information is shared

## Security Development Practices

### For Contributors

If you're contributing to this project, please follow these security guidelines:

1. **Never Commit Secrets**
   - No API keys, tokens, or passwords in code
   - Use environment variables for sensitive configuration
   - Review changes before committing

2. **Input Validation**
   - Validate all user inputs
   - Sanitize data before using in commands or queries
   - Use parameterized queries for databases

3. **Code Review**
   - All changes require code review
   - Security-sensitive changes require additional review
   - Use static analysis tools

4. **Dependencies**
   - Minimize external dependencies
   - Keep dependencies up to date
   - Review dependency security advisories
   - Use `npm audit` before releases

5. **Testing**
   - Write tests for security-critical code
   - Test error handling paths
   - Verify sensitive data is not leaked in errors

## Security Changelog

### Version 1.8.33
- Improved OAuth credential storage
- Enhanced multi-account support
- Better error handling and sanitization

### Previous Versions
- Initial security implementation
- VS Code SecretStorage integration
- HTTPS-only external communications

## Acknowledgments

We thank the security researchers and contributors who help keep Antigravity Cockpit secure:

- (Your name could be here - report responsibly!)

## Contact

For security concerns:
- **Security Issues:** Use GitHub Security Advisory (preferred)
- **General Security Questions:** Open a GitHub discussion
- **Project Issues:** https://github.com/seevali/vscode-antigravity-cockpit/issues

---

**Note:** This security policy applies to the Antigravity Cockpit VS Code extension. For security issues with Antigravity itself, please contact the Antigravity team directly.

Last Updated: January 2026
