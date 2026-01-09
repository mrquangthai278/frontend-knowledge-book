# Content Security Policy (CSP)

## Definition / Concept
**Content Security Policy (CSP)** is a security layer that helps detect and mitigate certain types of attacks, including **Cross-Site Scripting (XSS)** and data injection attacks. It works by allowing developers to specify which sources of content are trusted, preventing the browser from loading malicious resources. CSP is implemented through HTTP headers or meta tags that define allowed sources for scripts, styles, images, and other resources.

- Browser security mechanism to prevent XSS and injection attacks
- Defines **whitelist** of trusted content sources
- Configured via HTTP header: `Content-Security-Policy`
- Blocks inline scripts and `eval()` by default (can be enabled with unsafe-inline)

## Visual Representation
```
Without CSP:
Browser loads ANY script from ANYWHERE
  ↓
Malicious script executes
  ↓
Data stolen / XSS attack

With CSP:
HTTP Response Header:
Content-Security-Policy: script-src 'self' https://trusted.com
  ↓
Browser checks script source
  ↓
✅ https://mysite.com/script.js → Allowed ('self')
✅ https://trusted.com/lib.js → Allowed (whitelisted)
❌ https://evil.com/malware.js → Blocked (not in whitelist)
❌ <script>alert('XSS')</script> → Blocked (inline script)
```

## Example
```javascript
// Setting CSP via HTTP header (server-side)
// Node.js/Express example
app.use((req, res, next) => {
  res.setHeader(
    'Content-Security-Policy',
    "default-src 'self'; script-src 'self' https://cdn.example.com; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' https://fonts.googleapis.com"
  );
  next();
});

// Setting CSP via meta tag (less secure, not recommended for critical policies)
// HTML
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; script-src 'self' https://cdn.example.com">

// Common CSP directives
const cspDirectives = {
  'default-src': "'self'",                    // Default for all resources
  'script-src': "'self' https://cdn.com",     // JavaScript sources
  'style-src': "'self' 'unsafe-inline'",      // CSS sources
  'img-src': "'self' data: https:",           // Image sources
  'font-src': "'self' https://fonts.com",     // Font sources
  'connect-src': "'self' https://api.com",    // AJAX, WebSocket sources
  'media-src': "'self'",                      // Audio/video sources
  'object-src': "'none'",                     // Plugins (Flash, etc.)
  'frame-src': "'self'",                      // iframes
  'base-uri': "'self'",                       // <base> element
  'form-action': "'self'",                    // Form submissions
  'frame-ancestors': "'none'",                // Embedding in iframes
  'upgrade-insecure-requests': ''             // Upgrade HTTP to HTTPS
};

// Strict CSP example (most secure)
const strictCSP = `
  default-src 'none';
  script-src 'self' 'nonce-{random}';
  style-src 'self' 'nonce-{random}';
  img-src 'self';
  font-src 'self';
  connect-src 'self';
  frame-ancestors 'none';
  base-uri 'self';
  form-action 'self';
`;

// Using nonce for inline scripts (recommended)
// Server generates random nonce per request
const nonce = generateRandomNonce(); // e.g., "r4nd0m123"

// HTTP Header
res.setHeader(
  'Content-Security-Policy',
  `script-src 'self' 'nonce-${nonce}'`
);

// HTML with nonce
// ✅ Allowed (has matching nonce)
<script nonce="r4nd0m123">
  console.log('This is allowed');
</script>

// ❌ Blocked (no nonce)
<script>
  console.log('This is blocked');
</script>

// Using hash for inline scripts (alternative to nonce)
const scriptContent = "console.log('hello');";
const hash = sha256(scriptContent); // Generate SHA-256 hash

// Header
res.setHeader(
  'Content-Security-Policy',
  `script-src 'self' 'sha256-${hash}'`
);

// HTML
<script>console.log('hello');</script> // ✅ Allowed (matches hash)

// Report-only mode (testing CSP without blocking)
res.setHeader(
  'Content-Security-Policy-Report-Only',
  "default-src 'self'; report-uri /csp-report"
);

// CSP violation report endpoint
app.post('/csp-report', (req, res) => {
  console.log('CSP Violation:', req.body);
  // {
  //   "csp-report": {
  //     "document-uri": "https://example.com/page",
  //     "violated-directive": "script-src 'self'",
  //     "blocked-uri": "https://evil.com/malware.js",
  //     "original-policy": "default-src 'self'; script-src 'self'"
  //   }
  // }
  res.status(204).end();
});

// Modern CSP with report-to
res.setHeader(
  'Content-Security-Policy',
  "default-src 'self'; report-to csp-endpoint"
);
res.setHeader(
  'Report-To',
  JSON.stringify({
    group: 'csp-endpoint',
    max_age: 10886400,
    endpoints: [{ url: 'https://example.com/csp-reports' }]
  })
);
```

## Usage
- **When to use**: 
  - All web applications handling sensitive data
  - Preventing XSS attacks
  - Protecting against code injection
  - Preventing clickjacking (frame-ancestors)
  - Defense-in-depth security strategy
  
- **Real-world example**:
  ```javascript
  // E-commerce site CSP
  const ecommerceCSP = [
    "default-src 'self'",
    "script-src 'self' https://cdn.stripe.com https://www.google-analytics.com",
    "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
    "img-src 'self' data: https:",
    "font-src 'self' https://fonts.gstatic.com",
    "connect-src 'self' https://api.stripe.com https://analytics.google.com",
    "frame-src https://js.stripe.com",
    "object-src 'none'",
    "base-uri 'self'",
    "form-action 'self'",
    "frame-ancestors 'none'",
    "upgrade-insecure-requests"
  ].join('; ');
  
  // SPA with API CSP
  const spaCSP = [
    "default-src 'self'",
    "script-src 'self'",
    "style-src 'self'",
    "img-src 'self' data: https:",
    "connect-src 'self' https://api.example.com",
    "font-src 'self'",
    "object-src 'none'",
    "frame-ancestors 'none'"
  ].join('; ');
  
  // Express middleware for CSP
  const helmet = require('helmet');
  
  app.use(helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'nonce-{random}'"],
      styleSrc: ["'self'", "'nonce-{random}'"],
      imgSrc: ["'self'", "data:", "https:"],
      connectSrc: ["'self'", "https://api.example.com"],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      frameAncestors: ["'none'"],
      baseUri: ["'self'"],
      formAction: ["'self'"]
    }
  }));
  ```

- **Best practices**:
  - Start with `Content-Security-Policy-Report-Only` to test
  - Use `default-src 'self'` as baseline
  - Avoid `'unsafe-inline'` and `'unsafe-eval'` when possible
  - Use nonces or hashes for inline scripts
  - Set `object-src 'none'` to block plugins
  - Use `frame-ancestors 'none'` to prevent clickjacking
  - Monitor CSP violation reports
  - Gradually tighten policy as you fix violations

## FAQ / Interview Questions

**Q: What is Content Security Policy and why is it important?**
A: **CSP** is a security header that tells browsers which sources are trusted for loading resources. It's important because:
- **Prevents XSS attacks**: Blocks malicious scripts from executing
- **Mitigates data injection**: Prevents unauthorized content loading
- **Defense-in-depth**: Additional security layer even if XSS vulnerability exists
- **Clickjacking protection**: Controls iframe embedding with `frame-ancestors`
- **HTTPS enforcement**: Can upgrade all HTTP requests to HTTPS

**Q: What's the difference between 'self', 'unsafe-inline', and 'unsafe-eval'?**
A:
- **'self'**: Allows resources from same origin (protocol + domain + port)
  ```javascript
  script-src 'self' // Only scripts from your domain
  ```
- **'unsafe-inline'**: Allows inline scripts/styles (NOT recommended, enables XSS)
  ```javascript
  script-src 'unsafe-inline' // <script>...</script> allowed
  ```
- **'unsafe-eval'**: Allows eval() and similar methods (NOT recommended)
  ```javascript
  script-src 'unsafe-eval' // eval(), new Function() allowed
  ```
Use nonces or hashes instead of `'unsafe-inline'`.

**Q: How do nonces work in CSP?**
A: A **nonce** (number used once) is a random value generated per request:
1. Server generates random nonce: `"abc123"`
2. Adds to CSP header: `script-src 'nonce-abc123'`
3. Adds same nonce to inline script: `<script nonce="abc123">...</script>`
4. Browser only executes scripts with matching nonce
5. New nonce for each page load prevents replay attacks

```javascript
// Server (per request)
const nonce = crypto.randomBytes(16).toString('base64');
res.setHeader('Content-Security-Policy', `script-src 'nonce-${nonce}'`);

// HTML
<script nonce="${nonce}">console.log('allowed');</script>
```

**Q: What's the difference between Content-Security-Policy and Content-Security-Policy-Report-Only?**
A:
- **Content-Security-Policy**: Enforces the policy, blocks violations
  - Use in production after testing
  - Violations are blocked and reported
  
- **Content-Security-Policy-Report-Only**: Reports violations but doesn't block
  - Use for testing before deployment
  - Violations are only reported, not blocked
  - Helps identify what will break

Both can be used simultaneously to test new policies while enforcing current ones.

**Q: How do you handle third-party scripts (Google Analytics, CDNs) with CSP?**
A: Several approaches:
1. **Whitelist domains**:
```javascript
script-src 'self' https://www.google-analytics.com https://cdn.jsdelivr.net
```

2. **Use Subresource Integrity (SRI)** for CDN scripts:
```html
<script src="https://cdn.example.com/lib.js"
        integrity="sha384-hash..."
        crossorigin="anonymous"></script>
```

3. **Self-host third-party libraries** when possible

4. **Use nonce** for inline tracking code:
```html
<script nonce="random123">
  ga('send', 'pageview');
</script>
```

## References
- [MDN Web Docs - Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [Google Web Fundamentals - CSP](https://developers.google.com/web/fundamentals/security/csp)
- [CSP Evaluator](https://csp-evaluator.withgoogle.com/)
- [Content Security Policy Reference](https://content-security-policy.com/)

---
*See also: [XSS](XSS.md), [CORS](CORS.md), [Same-Origin Policy](SameOriginPolicy.md)*
