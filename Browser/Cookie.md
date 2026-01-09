# Cookie

## Definition / Concept

A **cookie** is a small piece of data stored on the client-side that is automatically sent with every HTTP request to the server. Cookies are used primarily for **session management**, **authentication**, and **personalization**. They consist of key-value pairs with optional attributes like **expiration**, **domain**, **path**, **secure**, and **httpOnly**. Unlike Web Storage, cookies are **automatically transmitted** with requests, making them ideal for authentication but also requiring careful security consideration.

- Small text data (typically **4KB** per cookie) sent automatically with requests
- Can be set by **server** (via Set-Cookie header) or **client** (via JavaScript)
- Optional attributes control scope, lifetime, and security
- **httpOnly** cookies are inaccessible to JavaScript (more secure)
- **Secure** flag ensures transmission only over HTTPS
- **SameSite** attribute protects against CSRF attacks

## Visual Representation

```
Cookie Lifecycle:

Server creates cookie:
┌─────────────────────────────────┐
│ Server Response                 │
│ Set-Cookie: sessionId=abc123... │
└──────────────────┬──────────────┘
                   │
                   ▼
┌─────────────────────────────────┐
│ Browser Storage (Automatic)     │
│ sessionId=abc123; path=/        │
└──────────────────┬──────────────┘
                   │
                   ▼
Client makes request:
┌─────────────────────────────────┐
│ Browser automatically includes  │
│ Cookie: sessionId=abc123        │
│ in every request to server      │
└─────────────────────────────────┘

Cookie Scope (Domain + Path):
┌──────────────────────────────────┐
│ example.com/auth/login           │
│  ├─ domain=example.com (all)     │
│  ├─ path=/auth (subtree)         │
│  └─ Matches: /auth/login, /auth/ │
└──────────────────────────────────┘

Cookie Attributes:
┌────────────────────────────────────┐
│ name=value                         │
│ Expires: Jan 1, 2027 (lifespan)   │
│ HttpOnly: ✓ (JS cannot access)    │
│ Secure: ✓ (HTTPS only)            │
│ SameSite: Strict (CSRF protection)│
└────────────────────────────────────┘
```

## Example

### Setting Cookies

```javascript
// Simple cookie
document.cookie = 'username=john_doe';
document.cookie = 'theme=dark';

// With expiration (7 days)
const date = new Date();
date.setTime(date.getTime() + (7 * 24 * 60 * 60 * 1000));
document.cookie = `username=john_doe; expires=${date.toUTCString()}`;

// With attributes
document.cookie = `sessionId=abc123; path=/; secure; samesite=Strict`;
```

### Reading Cookies

```javascript
// Get all cookies (string format)
console.log(document.cookie);

// Parse into object
function parseCookies() {
  const cookies = {};
  document.cookie.split(';').forEach(cookie => {
    const [name, value] = cookie.trim().split('=');
    if (name && value) cookies[name] = decodeURIComponent(value);
  });
  return cookies;
}

const allCookies = parseCookies();
```

### Deleting Cookies

```javascript
// Delete by setting to past date
function deleteCookie(name) {
  document.cookie = `${name}=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;`;
}

deleteCookie('theme');
```

### Cookie Manager Utility

```javascript
class CookieManager {
  static set(name, value, days = 30) {
    const date = new Date();
    date.setTime(date.getTime() + (days * 24 * 60 * 60 * 1000));
    document.cookie = `${name}=${value}; expires=${date.toUTCString()}; path=/`;
  }

  static get(name) {
    const value = `; ${document.cookie}`;
    const parts = value.split(`; ${name}=`);
    return parts.length === 2 ? parts.pop().split(';').shift() : null;
  }

  static delete(name) {
    this.set(name, '', -1);
  }
}

// Usage
CookieManager.set('userId', '12345', 30);
CookieManager.get('userId');
CookieManager.delete('userId');
```

## Usage

- **When to use**: Authentication tokens (server-side httpOnly), session management, user preferences, tracking, CSRF tokens
- **Real-world example**: Login sessions, remember-me functionality, CSRF protection, analytics tracking, A/B testing, language preferences
- **Best practices**:
  - Use **httpOnly** cookies for authentication (prevents XSS attacks)
  - Always use **Secure** flag for sensitive cookies (HTTPS only)
  - Set **SameSite=Strict** or **SameSite=Lax** to prevent CSRF attacks
  - Never store **sensitive data** like passwords in cookies
  - Use server-side cookie creation when possible (more secure than client-side)
  - Set appropriate **expiration times** (short for sessions, long for preferences)
  - Use **Path** to limit cookie scope to necessary routes
  - Use **Domain** carefully to avoid unintended sharing
  - Validate and sanitize cookie values on the server
  - Consider Web Storage or IndexedDB for large client-side data

## FAQ / Interview Questions

**Q: What's the difference between a cookie and Web Storage?**
A: Cookies are **automatically sent** with every HTTP request (adding overhead), have a small size limit (~4KB), and are primarily server-managed. Web Storage (localStorage/sessionStorage) is **not automatically sent**, has larger capacity (5-10MB), and is client-managed. Use cookies for authentication and server-side data, Web Storage for client-side preferences and caching.

**Q: Why should authentication tokens use httpOnly cookies?**
A: **httpOnly** cookies are inaccessible to JavaScript, protecting them from **XSS attacks**. If malicious JavaScript runs on the page, it cannot steal httpOnly cookies. Regular cookies set via JavaScript are vulnerable to XSS. For authentication, use httpOnly cookies set by the server. Never store sensitive tokens in regular cookies or Web Storage.

**Q: What is the SameSite attribute and why does it matter?**
A: **SameSite** prevents cookies from being sent in **cross-site requests**, protecting against **CSRF (Cross-Site Request Forgery)** attacks. Three levels exist:
- `SameSite=Strict`: Cookie sent only in same-site requests (safest)
- `SameSite=Lax`: Cookie sent in top-level navigation (default in most browsers)
- `SameSite=None; Secure`: Cookie sent in all requests (requires HTTPS)

**Q: Can cookies be deleted or modified by the client?**
A: Yes, JavaScript can **read and set** cookies (except httpOnly ones), and can **delete** them by setting expiration to the past. However, httpOnly cookies cannot be accessed or modified by JavaScript, only by the browser. Server-side validation is essential because clients can modify any non-httpOnly cookie.

**Q: What are first-party vs third-party cookies?**
A: **First-party cookies** are set by the website you're visiting and are generally trusted. **Third-party cookies** are set by external domains (e.g., ad networks, analytics) when you visit a website. Most browsers now restrict third-party cookies by default due to privacy concerns. Use first-party cookies for your own site, and be transparent about tracking.

## References
- [MDN - HTTP Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
- [MDN - Document.cookie](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie)
- [OWASP - Cookie Security](https://owasp.org/www-community/controls/Cookie_Security)
- [RFC 6265 - HTTP State Management Mechanism](https://tools.ietf.org/html/rfc6265)

---

*See also: [Web Storage](./WebStorage.md), [Session Management](./SessionManagement.md), [CSRF Protection](./CSRF.md), [XSS Prevention](./XSS.md), [Authentication](./Authentication.md)*
