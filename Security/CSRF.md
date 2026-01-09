# CSRF (Cross-Site Request Forgery)

## Definition / Concept

**CSRF (Cross-Site Request Forgery)** is a security vulnerability where an attacker tricks an authenticated user into performing unwanted actions on a website without their knowledge. The attacker crafts a malicious request that leverages the user's existing authentication session to perform unauthorized operations like transferring funds, changing passwords, or modifying account settings.

- **Core issue**: The browser automatically includes credentials (cookies) in cross-site requests
- **Attack vector**: Malicious links, forms, or images on attacker-controlled sites
- **Impact**: Unauthorized actions performed as the logged-in user, data modification, account compromise

## Visual Representation

```
User logged into Bank (session cookie stored)
           ↓
User visits Attacker's site (without logging out)
           ↓
Attacker's page sends hidden request to Bank
(GET/POST to bank.com/transfer)
           ↓
Browser automatically includes Bank's session cookie
           ↓
Bank processes request (thinks it's legitimate)
           ↓
Transfer completes without user's consent
```

## Example

```javascript
// ❌ VULNERABLE: No CSRF protection
app.post('/api/transfer-money', (req, res) => {
  if (req.user) {  // Only checks authentication, not origin
    transferMoney(req.user.id, req.body.recipient, req.body.amount);
    res.json({ success: true });
  }
});

// ✅ SAFE: Verify CSRF token
app.post('/api/transfer-money', (req, res) => {
  // Check token matches session
  if (req.session.csrfToken !== req.body.csrfToken) {
    return res.status(403).json({ error: 'CSRF token invalid' });
  }
  transferMoney(req.user.id, req.body.recipient, req.body.amount);
  res.json({ success: true });
});

// Frontend: Include token in request header
fetch('/api/transfer-money', {
  method: 'POST',
  headers: { 'X-CSRF-Token': getCsrfToken() },
  body: JSON.stringify({ amount: 10000, recipient: 'alice' })
});
```

## Usage

- **When to use protection**: All state-changing requests (POST, PUT, DELETE, PATCH) on authenticated routes
- **Real-world example**: Banking transfers, password changes, profile updates, permission modifications
- **Best practices**:
  - Use **CSRF tokens** for all state-changing operations
  - Implement **SameSite cookie attribute** (`SameSite=Strict` or `SameSite=Lax`)
  - Use **POST/PUT/DELETE** instead of GET for sensitive operations
  - Verify **Origin** and **Referer** headers on the server
  - Require user confirmation for critical actions
  - Use frameworks with built-in CSRF protection (Express, Django, Rails)

## FAQ / Interview Questions

**Q: How does a CSRF attack actually work?**
A: A CSRF attack exploits the browser's automatic inclusion of credentials (cookies) in cross-site requests. An attacker tricks a logged-in user into visiting their malicious site, which silently sends a request to the target site. The browser automatically includes the user's session cookie, and the server processes the request as if it came from the user.

**Q: What's the difference between CSRF and XSS?**
A: **CSRF** tricks the user into making unwanted requests on their behalf (requires user to be logged in). **XSS** injects malicious scripts to steal data or hijack the session. CSRF targets actions; XSS targets data. A user can be protected from CSRF by logging out, but XSS works even on public sites.

**Q: How do CSRF tokens prevent attacks?**
A: CSRF tokens are unique values tied to a user's session. The server generates a token, sends it to the client, and requires it in state-changing requests. Since attackers can't access tokens from other domains (due to Same-Origin Policy), they can't craft valid CSRF requests. The token verifies that the request came from the legitimate application.

**Q: What's the SameSite cookie attribute and how does it help?**
A: **SameSite** controls whether cookies are sent with cross-site requests. `SameSite=Strict` never sends cookies cross-site (most secure). `SameSite=Lax` allows cookies for safe cross-site navigations (clicks, but not forms). `SameSite=None` requires `Secure` flag and sends cookies everywhere. SameSite significantly reduces CSRF risk for modern browsers.

**Q: Can a CSRF attack happen with POST requests?**
A: Yes, absolutely. Attackers can submit hidden forms that auto-POST to target sites. This is why CSRF protection applies to all state-changing methods (POST, PUT, DELETE, PATCH), not just GET requests. The HTTP method doesn't matter—what matters is whether it changes server state.

## References

- [MDN Web Docs - Cross-Site Request Forgery (CSRF)](https://developer.mozilla.org/en-US/docs/Glossary/CSRF)
- [OWASP - Cross-Site Request Forgery (CSRF)](https://owasp.org/www-community/attacks/csrf)
- [SameSite Cookie Explained](https://web.dev/samesite-cookie-explained/)
- [Express CSRF Protection with csurf](https://github.com/expressjs/csurf)

---

*See also: [XSS](../WebStandards/XSS.md), [Security](../WebStandards/Security.md), [Authentication](../WebStandards/Authentication.md)*
