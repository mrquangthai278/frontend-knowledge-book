# CORS (Cross-Origin Resource Sharing)

## Definition / Concept
**CORS** (Cross-Origin Resource Sharing) is a browser security mechanism that allows or restricts web applications running at one origin to access resources from a different origin. It uses HTTP headers to tell browsers whether to allow a web application from one origin to access selected resources from a server at a different origin. CORS relaxes the **Same-Origin Policy** in a controlled manner.

- Browser mechanism for cross-origin HTTP requests
- Uses HTTP headers to grant permission (Access-Control-Allow-Origin)
- Required when frontend and backend are on different domains
- Only applies to **browser requests** (not server-to-server)

## Visual Representation
```
Same-Origin (No CORS needed):
https://example.com/page.html
   ↓ XHR/Fetch
https://example.com/api/data
✅ Same origin → Allowed

Cross-Origin (CORS required):
https://frontend.com/page.html
   ↓ XHR/Fetch
https://api.backend.com/data
   ↓
Browser sends: Origin: https://frontend.com
   ↓
Server responds: Access-Control-Allow-Origin: https://frontend.com
   ↓
✅ CORS header present → Allowed
❌ No CORS header → Blocked

Preflight Request (for complex requests):
Browser → [OPTIONS] → Server
         "Can I send POST with custom headers?"
         ↓
Server → Response → Browser
        "Yes, allowed methods: POST, GET"
        "Allowed headers: Content-Type, X-Custom"
         ↓
Browser → [POST] → Server (actual request)
```

## Example
```javascript
// Simple CORS request (GET, HEAD, POST with simple content-type)
fetch('https://api.example.com/data')
  .then(response => response.json())
  .then(data => console.log(data));

// Browser automatically adds:
// Origin: https://mysite.com

// Server must respond with:
// Access-Control-Allow-Origin: https://mysite.com
// or
// Access-Control-Allow-Origin: *

// ========================================
// SERVER-SIDE CORS CONFIGURATION
// ========================================

// Node.js/Express - Basic CORS
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*'); // Allow all origins
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  next();
});

// Using cors middleware (recommended)
const cors = require('cors');

// Allow all origins
app.use(cors());

// Allow specific origin
app.use(cors({
  origin: 'https://frontend.com'
}));

// Allow multiple origins
const allowedOrigins = [
  'https://frontend.com',
  'https://app.frontend.com',
  'http://localhost:3000'
];

app.use(cors({
  origin: function(origin, callback) {
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true, // Allow cookies
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  exposedHeaders: ['X-Total-Count'], // Client can access this header
  maxAge: 86400 // Preflight cache duration (24 hours)
}));

// Handle preflight requests
app.options('*', cors());

// ========================================
// PREFLIGHT REQUEST
// ========================================

// Complex request triggers preflight
fetch('https://api.example.com/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Custom-Header': 'value'
  },
  body: JSON.stringify({ data: 'value' })
});

// Browser sends OPTIONS request first:
// OPTIONS /data HTTP/1.1
// Origin: https://frontend.com
// Access-Control-Request-Method: POST
// Access-Control-Request-Headers: Content-Type, X-Custom-Header

// Server must respond:
// Access-Control-Allow-Origin: https://frontend.com
// Access-Control-Allow-Methods: POST, GET, OPTIONS
// Access-Control-Allow-Headers: Content-Type, X-Custom-Header
// Access-Control-Max-Age: 86400

// Then browser sends actual POST request

// ========================================
// CORS WITH CREDENTIALS (Cookies)
// ========================================

// Client-side
fetch('https://api.example.com/data', {
  credentials: 'include' // Send cookies cross-origin
})
  .then(response => response.json());

// Server-side
app.use(cors({
  origin: 'https://frontend.com', // Must be specific, not *
  credentials: true
}));

// Response headers:
// Access-Control-Allow-Origin: https://frontend.com (NOT *)
// Access-Control-Allow-Credentials: true

// ========================================
// COMMON CORS ERRORS AND SOLUTIONS
// ========================================

// ❌ Error: "No 'Access-Control-Allow-Origin' header"
// Solution: Add CORS headers on server
res.header('Access-Control-Allow-Origin', 'https://frontend.com');

// ❌ Error: "The value of 'Access-Control-Allow-Origin' must not be '*' when credentials is true"
// Solution: Specify exact origin
app.use(cors({
  origin: 'https://frontend.com',
  credentials: true
}));

// ❌ Error: "Method POST not allowed by Access-Control-Allow-Methods"
// Solution: Add method to allowed methods
res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');

// ❌ Error: "Request header X-Custom not allowed"
// Solution: Add header to allowed headers
res.header('Access-Control-Allow-Headers', 'Content-Type, X-Custom');

// ========================================
// TESTING CORS
// ========================================

// Test with curl
// Preflight request
curl -X OPTIONS https://api.example.com/data \
  -H "Origin: https://frontend.com" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: Content-Type" \
  -v

// Actual request
curl -X POST https://api.example.com/data \
  -H "Origin: https://frontend.com" \
  -H "Content-Type: application/json" \
  -d '{"test": "data"}' \
  -v
```

## Usage
- **When to use**: 
  - Frontend and backend on different domains/ports
  - Making API calls from browser to external services
  - Microservices architecture with separate services
  - Development (frontend on localhost:3000, backend on localhost:5000)
  - Public APIs accessed from browsers
  
- **Real-world example**:
  ```javascript
  // Development setup
  // Frontend: http://localhost:3000
  // Backend: http://localhost:5000
  
  // Backend (Express)
  const cors = require('cors');
  
  if (process.env.NODE_ENV === 'development') {
    app.use(cors({
      origin: 'http://localhost:3000',
      credentials: true
    }));
  } else {
    app.use(cors({
      origin: process.env.FRONTEND_URL,
      credentials: true
    }));
  }
  
  // API with authentication
  app.post('/api/login', async (req, res) => {
    const { email, password } = req.body;
    const user = await authenticate(email, password);
    
    res.cookie('token', user.token, {
      httpOnly: true,
      secure: true,
      sameSite: 'none' // Required for cross-origin cookies
    });
    
    res.json({ user });
  });
  
  // Frontend
  fetch('https://api.example.com/login', {
    method: 'POST',
    credentials: 'include', // Send cookies
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ email, password })
  });
  
  // Proxy approach (alternative to CORS)
  // In development, use proxy in package.json or webpack
  // package.json
  {
    "proxy": "http://localhost:5000"
  }
  
  // Now fetch without full URL
  fetch('/api/data') // Proxied to http://localhost:5000/api/data
  
  // Nginx reverse proxy (production)
  // Frontend: https://example.com
  // Backend: https://example.com/api (proxied to http://backend:5000)
  location /api {
    proxy_pass http://backend:5000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
  }
  ```

- **Best practices**:
  - Never use `Access-Control-Allow-Origin: *` with credentials
  - Whitelist specific origins, don't allow all in production
  - Use `Access-Control-Max-Age` to cache preflight responses
  - Handle OPTIONS requests properly
  - Use HTTPS for cross-origin requests with credentials
  - Consider using a reverse proxy instead of CORS when possible
  - Set appropriate `SameSite` cookie attribute
  - Validate the Origin header on server

## FAQ / Interview Questions

**Q: What is CORS and why do we need it?**
A: **CORS** (Cross-Origin Resource Sharing) is a mechanism that allows browsers to make requests to domains different from the one serving the web page. We need it because:
- **Same-Origin Policy** blocks cross-origin requests by default (security)
- Modern apps often have separate frontend/backend domains
- CORS provides a secure way to allow specific cross-origin requests
- Without CORS, you couldn't call APIs from different domains in the browser

**Q: What's the difference between simple and preflight CORS requests?**
A: 
**Simple requests** (no preflight):
- Methods: GET, HEAD, POST
- Headers: Accept, Accept-Language, Content-Language, Content-Type
- Content-Type: application/x-www-form-urlencoded, multipart/form-data, text/plain
- Browser sends request directly

**Preflight requests** (requires OPTIONS first):
- Other methods: PUT, DELETE, PATCH
- Custom headers: Authorization, X-Custom-Header
- Content-Type: application/json
- Browser sends OPTIONS first to check permissions

```javascript
// Simple - no preflight
fetch('https://api.com/data') // GET request

// Preflight required
fetch('https://api.com/data', {
  method: 'DELETE', // Custom method
  headers: { 'Authorization': 'Bearer token' } // Custom header
});
```

**Q: How do you enable CORS with credentials (cookies)?**
A: Three requirements:
1. **Client**: Set `credentials: 'include'`
2. **Server**: Set specific origin (NOT `*`)
3. **Server**: Set `Access-Control-Allow-Credentials: true`

```javascript
// Client
fetch('https://api.com/data', {
  credentials: 'include'
});

// Server
res.header('Access-Control-Allow-Origin', 'https://frontend.com');
res.header('Access-Control-Allow-Credentials', 'true');

// ❌ Won't work:
res.header('Access-Control-Allow-Origin', '*'); // Can't use * with credentials
```

**Q: What are common CORS errors and how do you fix them?**
A: Common errors:
1. **"No Access-Control-Allow-Origin header"**
   - Fix: Add CORS headers on server

2. **"Origin not allowed"**
   - Fix: Add origin to whitelist

3. **"Method not allowed"**
   - Fix: Add method to Access-Control-Allow-Methods

4. **"Header not allowed"**
   - Fix: Add header to Access-Control-Allow-Headers

5. **"Wildcard '*' with credentials"**
   - Fix: Use specific origin instead of *

**Q: What's the difference between CORS and JSONP?**
A:
- **JSONP** (old approach):
  - Uses `<script>` tags (not subject to CORS)
  - Only supports GET requests
  - Less secure, vulnerable to XSS
  - No error handling
  - Legacy technique

- **CORS** (modern approach):
  - Uses XHR/Fetch with proper headers
  - Supports all HTTP methods
  - More secure with proper configuration
  - Better error handling
  - Standard approach

```javascript
// JSONP (old)
<script src="https://api.com/data?callback=handleData"></script>
function handleData(data) { console.log(data); }

// CORS (modern)
fetch('https://api.com/data')
  .then(r => r.json())
  .then(data => console.log(data));
```

## References
- [MDN Web Docs - CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [MDN Web Docs - Access-Control-Allow-Origin](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin)
- [W3C CORS Specification](https://www.w3.org/TR/cors/)
- [Enable CORS](https://enable-cors.org/)

---
*See also: [Same-Origin Policy](SameOriginPolicy.md), [CSP](CSP.md), [Cookie](../Browser/Cookie.md)*
