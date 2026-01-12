# Security Audit Report - vscode-antigravity-cockpit

**Date:** January 12, 2026  
**Repository:** seevali/vscode-antigravity-cockpit  
**Auditor:** GitHub Copilot Security Scanner  

## Executive Summary

This security audit identified **1 CRITICAL vulnerability** and several security observations in the vscode-antigravity-cockpit extension. The critical issue involves a hardcoded OAuth client secret that must be removed immediately.

---

## 🔴 CRITICAL VULNERABILITIES

### 1. Hardcoded OAuth Client Secret (CRITICAL)

**File:** `src/auto_trigger/oauth_service.ts:17`

**Issue:**
```typescript
const ANTIGRAVITY_CLIENT_SECRET = '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';
```

A Google OAuth client secret is hardcoded directly in the source code. This is a **critical security vulnerability** as:

1. **Public Exposure:** The secret is visible to anyone who can access the GitHub repository
2. **Token Theft Risk:** Attackers can use this secret to:
   - Impersonate the application
   - Exchange authorization codes for access tokens
   - Potentially access user data through the OAuth flow
3. **Cannot Be Easily Rotated:** Once committed to Git history, the secret remains accessible even if removed from current code

**Severity:** CRITICAL  
**CVE Risk:** High - This could lead to unauthorized access to user accounts

**Recommendation:**
1. **IMMEDIATE ACTION REQUIRED:**
   - Revoke this OAuth client secret in Google Cloud Console immediately
   - Generate a new client secret
   - Remove the hardcoded secret from the codebase
   
2. **Long-term Solution:**
   - For VS Code extensions that need OAuth, consider using VS Code's built-in authentication API (`vscode.authentication.getSession()`)
   - If custom OAuth is necessary, use environment variables or secure configuration files that are NOT committed to the repository
   - Add `.env*` files to `.gitignore`
   - Document that users need to configure their own OAuth credentials

**Impact:** Any malicious actor with access to this secret could potentially:
- Intercept OAuth authorization flows
- Access user's Google Cloud Platform resources
- Impersonate the Antigravity Cockpit application

---

## ⚠️ SECURITY OBSERVATIONS

### 2. Command Execution via child_process

**Files:** 
- `src/engine/hunter.ts`
- `src/engine/strategies.ts`
- `src/auto_trigger/local_auth_importer.ts`

**Issue:**
The extension uses Node.js `child_process` to execute system commands (PowerShell, shell commands, sqlite3).

**Current Implementation:**
```typescript
const { stdout, stderr } = await execAsync(cmd, {
    timeout: TIMING.PROCESS_CMD_TIMEOUT_MS,
});
```

**Analysis:**
✅ **GOOD PRACTICES OBSERVED:**
- Commands are constructed using predefined constants, not user input
- Process names and parameters are validated
- Timeouts are implemented to prevent hanging processes
- No direct use of `eval()` or `Function()` constructors
- Command injection vectors appear to be minimal

⚠️ **CONCERNS:**
- On Windows, PowerShell commands are constructed dynamically
- File paths from user environment (e.g., `getAntigravityStateDbPath()`) are used in sqlite3 commands
- While current implementation appears safe, any future modifications could introduce injection risks

**Severity:** MEDIUM (Currently mitigated, but requires ongoing vigilance)

**Recommendations:**
1. Continue to avoid user-provided input in command construction
2. Validate and sanitize all file paths before using in commands
3. Consider using parameterized approaches where available
4. Add code comments warning about injection risks for future developers
5. Use `execFile` instead of `exec` where possible for better security

### 3. Network Request Security

**Files:**
- `src/shared/cloudcode_client.ts`
- `src/shared/error_reporter.ts`
- `src/auto_trigger/oauth_service.ts`

**Analysis:**
✅ **GOOD PRACTICES OBSERVED:**
- All external API calls use HTTPS
- Request timeouts are implemented
- Proper error handling with retry logic
- Access tokens are not logged in plain text
- Authorization headers use Bearer token pattern correctly

⚠️ **OBSERVATIONS:**
- Multiple Cloud Code endpoint URLs (fallback mechanism)
- Error reporting to Sentry includes potentially sensitive diagnostic information
- OAuth callback uses HTTP on localhost (acceptable for OAuth redirect URIs)

**Severity:** LOW

**Recommendations:**
1. ✅ Continue using HTTPS for all external communications
2. Review what diagnostic information is sent to Sentry - ensure no sensitive data
3. Consider allowing users to opt-out of error reporting more prominently
4. Document all external network endpoints the extension communicates with

### 4. Credential Storage

**File:** `src/auto_trigger/credential_storage.ts`

**Analysis:**
✅ **GOOD PRACTICES OBSERVED:**
- Uses VS Code's `SecretStorage` API for sensitive credentials
- OAuth tokens are encrypted by VS Code's secure storage
- No credentials are written to disk in plain text
- Multi-account support with proper isolation
- Legacy credential migration is handled securely

**Security Features:**
```typescript
private secretStorage?: vscode.SecretStorage;
// Stores encrypted: accessToken, refreshToken, clientSecret
```

**Severity:** N/A (Properly Implemented)

**Recommendations:**
1. ✅ Current implementation follows best practices
2. Continue using VS Code's SecretStorage for all sensitive data
3. Consider adding credential expiration checks
4. Document the security model in README

### 5. SQL Injection Risk in SQLite Query

**File:** `src/auto_trigger/local_auth_importer.ts:52`

**Issue:**
```typescript
const { stdout } = await execFileAsync(
    'sqlite3',
    ['-readonly', dbPath, `SELECT value FROM ItemTable WHERE key = '${STATE_KEY}';`],
    { maxBuffer: 10 * 1024 * 1024 },
);
```

**Analysis:**
⚠️ The SQL query embeds a constant directly in the query string. While `STATE_KEY` is a hardcoded constant, this pattern is risky.

**Current Risk:** LOW (STATE_KEY is a constant)  
**Potential Risk if Modified:** HIGH

**Recommendations:**
1. Use parameterized queries if sqlite3 CLI supports it
2. Add validation that `STATE_KEY` contains only safe characters
3. Add a code comment warning about SQL injection if this pattern is changed

### 6. Telemetry and Error Reporting

**File:** `src/shared/error_reporter.ts`

**Analysis:**
✅ **GOOD PRACTICES OBSERVED:**
- Telemetry is disabled by default (`telemetryEnabled: false` in package.json line 207)
- Respects VS Code's global telemetry settings
- Users can opt-out via settings
- No personally identifiable information (PII) is collected
- Uses anonymous machine IDs provided by VS Code
- Error stack traces are sanitized

**Data Collected:**
- Error messages and stack traces
- Extension version
- Platform/OS information
- VS Code version
- Anonymous machine ID (provided by VS Code)
- Extension configuration (non-sensitive)

**Severity:** N/A (Properly Implemented)

**Recommendations:**
1. ✅ Current implementation follows best practices
2. Consider making telemetry opt-IN rather than opt-OUT (already disabled by default)
3. Add a clear privacy policy document
4. Review error messages to ensure no sensitive data is included

---

## 🟢 SECURITY STRENGTHS

### Positive Security Implementations:

1. **No eval() or Function() Constructors:** Code does not use dangerous dynamic code execution
2. **Input Validation:** User inputs are validated before processing
3. **Secure Storage:** Credentials use VS Code's encrypted SecretStorage
4. **HTTPS Only:** All external API calls use HTTPS
5. **Timeout Protection:** Network requests and subprocess calls have timeouts
6. **Error Handling:** Comprehensive error handling prevents information leakage
7. **No Hardcoded API Keys** (except the OAuth secret which must be removed)
8. **CSRF Token Sanitization:** Logs redact CSRF tokens to prevent leakage
9. **Least Privilege:** Extension requests minimal VS Code permissions
10. **Code Review:** TypeScript provides type safety and reduces runtime errors

---

## 📋 CHECKLIST FOR REMEDIATION

### Immediate Actions (Critical):
- [ ] **Revoke the exposed OAuth client secret in Google Cloud Console**
- [ ] **Remove hardcoded `ANTIGRAVITY_CLIENT_SECRET` from codebase**
- [ ] **Implement secure OAuth credential management**
- [ ] **Scan Git history and revoke any other exposed secrets**

### High Priority:
- [ ] Review SQL query construction in `local_auth_importer.ts`
- [ ] Add security warnings in code comments for command execution
- [ ] Document all external network endpoints
- [ ] Add privacy policy and security documentation

### Medium Priority:
- [ ] Review and minimize diagnostic data sent to Sentry
- [ ] Consider using `execFile` instead of `exec` where possible
- [ ] Add automated security scanning to CI/CD pipeline
- [ ] Implement dependency vulnerability scanning

### Low Priority:
- [ ] Add security best practices guide for contributors
- [ ] Regular security audits of dependencies
- [ ] Consider security.txt file for vulnerability disclosure

---

## 🔐 SECURITY BEST PRACTICES FOR THIS PROJECT

### For Developers:

1. **Never commit secrets:** Use environment variables or secure configuration
2. **Validate all inputs:** Even from trusted sources
3. **Use parameterized queries:** For any database operations
4. **Keep dependencies updated:** Regularly update npm packages
5. **Use `execFile` over `exec`:** When executing system commands
6. **Sanitize logs:** Never log sensitive data (tokens, passwords, etc.)
7. **Follow least privilege:** Request minimal permissions needed
8. **Test error paths:** Ensure errors don't leak sensitive information

### For Users:

1. **Review permissions:** Check what the extension can access
2. **Keep updated:** Install updates promptly for security fixes
3. **Review settings:** Understand what data is collected (telemetry)
4. **Report issues:** Use GitHub issues for security concerns (or private disclosure)

---

## 📊 VULNERABILITY SUMMARY

| Severity | Count | Status |
|----------|-------|--------|
| 🔴 Critical | 1 | **REQUIRES IMMEDIATE ACTION** |
| 🟠 High | 0 | N/A |
| 🟡 Medium | 1 | Mitigated, requires monitoring |
| 🔵 Low | 2 | Minor concerns |
| 🟢 Info | 2 | Best practices suggestions |

---

## 🎯 CONCLUSION

The vscode-antigravity-cockpit extension demonstrates **good security practices overall**, with proper use of VS Code's security APIs, encrypted credential storage, and secure network communications. However, the **hardcoded OAuth client secret is a critical vulnerability** that must be addressed immediately.

Once the OAuth secret is properly secured, the extension will have a strong security posture with only minor areas for improvement.

### Recommended Priority:
1. **URGENT:** Remove and revoke hardcoded OAuth secret
2. **HIGH:** Implement proper OAuth credential management  
3. **MEDIUM:** Review and enhance command execution safety
4. **LOW:** Documentation and ongoing security monitoring

---

## 📞 SECURITY CONTACT

For security vulnerabilities, please:
1. **Do NOT open a public GitHub issue for security vulnerabilities**
2. Contact the repository maintainers privately
3. Allow reasonable time for a fix before public disclosure
4. Consider using GitHub's private security advisory feature

---

## 📚 REFERENCES

- [VS Code Extension Security](https://code.visualstudio.com/api/references/extension-manifest#extension-security)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [Node.js Security Best Practices](https://nodejs.org/en/docs/guides/security/)
- [OAuth 2.0 Security Best Current Practice](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics)

---

**End of Security Audit Report**
