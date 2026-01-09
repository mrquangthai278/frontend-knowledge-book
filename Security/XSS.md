# XSS (Cross-Site Scripting)

## Definition / Concept

**XSS (Cross-Site Scripting)** is a security vulnerability that allows attackers to inject malicious scripts into web applications viewed by other users. The attacker's script runs in the victim's browser with the same privileges as the legitimate application, potentially stealing data, hijacking sessions, or modifying page content.

- **Three types**: Stored XSS (data persisted), Reflected XSS (immediate response), DOM-based XSS (client-side manipulation)
- **Root cause**: Improper handling of untrusted user input
- **Impact**: Session hijacking, credential theft, malware distribution, page defacement

## Visual Representation

```
Attacker Input → App (No Sanitization) → Database/Response
                                           ↓
                          Victim Browser executes script
                          - Steals cookies/tokens
                          - Redirects to phishing site
                          - Modifies DOM
```

## Example

```javascript
// ❌ VULNERABLE: Direct HTML injection
function displayComment(userComment) {
  document.getElementById('comments').innerHTML = userComment;
  // If userComment = '<img src=x onerror="fetch(/steal?c=" + document.cookie + ")">'
  // Script executes and steals cookies
}

// ✅ SAFE: Text content only
function displayComment(userComment) {
  const commentEl = document.createElement('p');
  commentEl.textContent = userComment; // Treats as text, not HTML
  document.getElementById('comments').appendChild(commentEl);
}

// ✅ SAFE: HTML sanitization
function displayComment(userComment) {
  const sanitized = DOMPurify.sanitize(userComment);
  document.getElementById('comments').innerHTML = sanitized;
}

// ✅ SAFE: Template escaping (frameworks like Vue, React)
<div>{{ userComment }}</div> // Vue auto-escapes by default
<div>{userComment}</div>     // React auto-escapes by default
```

## Usage

- **When to use protection**: Whenever displaying user-generated content, URL parameters, or untrusted data
- **Real-world example**: Comment sections, user profiles, search results, forum posts
- **Best practices**:
  - Use framework defaults (React/Vue escape by default)
  - Use `textContent` instead of `innerHTML`
  - Sanitize HTML with DOMPurify or similar libraries
  - Use **Content Security Policy (CSP)** headers
  - Validate and escape output based on context (HTML, URL, JavaScript, CSS)
  - Never use `eval()` or `Function()` constructor with user input

## FAQ / Interview Questions

**Q: What's the difference between Stored and Reflected XSS?**
A: **Stored XSS** saves malicious code in a database (comments, posts); it affects all users viewing that content. **Reflected XSS** is sent via URL/form and only affects the specific user clicking the malicious link. Stored XSS is more dangerous because it's persistent.

**Q: How do modern frameworks protect against XSS?**
A: React and Vue automatically escape text content by default, treating `{{ variable }}` as text unless explicitly marked as HTML. Angular uses sanitization by default. These frameworks prevent accidental XSS, but developers can still introduce vulnerabilities using `innerHTML`, `dangerouslySetInnerHTML`, or `v-html`.

**Q: What's the difference between XSS and CSRF?**
A: **XSS** (Cross-Site Scripting) injects malicious scripts to steal data or hijack sessions. **CSRF** (Cross-Site Request Forgery) tricks users into making unwanted requests on their behalf. XSS targets data theft; CSRF targets unauthorized actions.

**Q: Can Content Security Policy prevent XSS?**
A: **CSP** is a defense mechanism that restricts where scripts can be loaded from. It helps prevent XSS by disallowing inline scripts and external sources, but it's not a complete solution. You still need proper input validation and output escaping as the primary defense.

**Q: What's the safest way to handle user content in a single-page app?**
A: Use your framework's built-in escaping mechanisms, keep user content and code separate, use `textContent` for plain text, sanitize with DOMPurify for rich HTML, implement **CSP headers**, and validate input on both client and server sides.

## References

- [MDN Web Docs - Cross-site scripting (XSS)](https://developer.mozilla.org/en-US/docs/Glossary/Cross-site_scripting_(XSS))
- [OWASP - Cross Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/)
- [DOMPurify Documentation](https://github.com/cure53/DOMPurify)
- [Content Security Policy (CSP) Guide](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)

---

*See also: [CSRF](../WebStandards/CSRF.md), [CSP](../WebStandards/CSP.md), [Security](../WebStandards/Security.md)*
