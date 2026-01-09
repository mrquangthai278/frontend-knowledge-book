# Same-Origin Policy

## Definition / Concept
The **Same-Origin Policy** (SOP) is a critical browser security mechanism that restricts how documents or scripts from one origin can interact with resources from another origin. Two URLs have the same origin if they share the same **protocol**, **domain**, and **port**. This policy prevents malicious scripts on one page from accessing sensitive data on another page through the DOM or making unauthorized requests.

- Fundamental browser security model
- Origins must match: **protocol** + **domain** + **port**
- Prevents malicious sites from reading data from other sites
- Can be relaxed using **CORS**, **JSONP**, or **postMessage**

## Visual Representation
```
Same-Origin Check:
https://example.com:443/page1
  ↓ Compare with
https://example.com:443/page2

Protocol: https === https ✅
Domain:   example.com === example.com ✅
Port:     443 === 443 ✅
→ SAME ORIGIN ✅

Cross-Origin Examples:
https://example.com      vs http://example.com       ❌ (protocol)
https://example.com      vs https://api.example.com  ❌ (subdomain)
https://example.com      vs https://example.org      ❌ (domain)
https://example.com:443  vs https://example.com:8080 ❌ (port)

What's Restricted by SOP:
┌─────────────────────┐
│  Origin A           │
│  example.com        │
├─────────────────────┤
│ ❌ Read cookies     │←─── from Origin B
│ ❌ Access DOM       │
│ ❌ Read localStorage│
│ ✅ Link to pages    │←─── Allowed
│ ✅ Embed images     │
│ ✅ Load scripts     │
└─────────────────────┘
```

## Example
```javascript
// ========================================
// SAME-ORIGIN EXAMPLES
// ========================================

// Current page: https://example.com/page

// ✅ Same origin - ALL allowed
https://example.com/api/data
https://example.com:443/users  // Port 443 is default for HTTPS
https://example.com/path/to/resource

// ❌ Different origin - BLOCKED by default
http://example.com/api         // Different protocol
https://api.example.com/data   // Different subdomain
https://example.com:8080/api   // Different port
https://example.org/api        // Different domain

// ========================================
// WHAT'S RESTRICTED
// ========================================

// ❌ Cannot read content from different origin
const iframe = document.createElement('iframe');
iframe.src = 'https://different-origin.com';
document.body.appendChild(iframe);

// This will throw SecurityError
try {
  const content = iframe.contentDocument.body;
} catch (error) {
  console.error('Blocked by Same-Origin Policy');
}

// ❌ Cannot access cookies from different origin
document.cookie; // Only cookies from current origin

// ❌ Cannot access localStorage from different origin
localStorage.getItem('key'); // Only from current origin

// ❌ XMLHttpRequest/Fetch blocked without CORS
fetch('https://different-origin.com/api/data')
  .catch(error => {
    console.error('CORS error:', error);
    // No 'Access-Control-Allow-Origin' header
  });

// ========================================
// WHAT'S ALLOWED
// ========================================

// ✅ Can link to different origins
<a href="https://different-origin.com">Link</a>

// ✅ Can embed images
<img src="https://different-origin.com/image.jpg">

// ✅ Can load scripts (but can't read content)
<script src="https://cdn.different-origin.com/lib.js"></script>

// ✅ Can load stylesheets
<link rel="stylesheet" href="https://different-origin.com/styles.css">

// ✅ Can embed videos/audio
<video src="https://different-origin.com/video.mp4"></video>

// ✅ Can submit forms to different origins
<form action="https://different-origin.com/submit" method="POST">

// ✅ Can embed iframes (but can't access content)
<iframe src="https://different-origin.com"></iframe>

// ========================================
// BYPASSING SAME-ORIGIN POLICY (Safely)
// ========================================

// 1. CORS - Server grants permission
fetch('https://api.different-origin.com/data', {
  method: 'GET'
});
// Server responds with:
// Access-Control-Allow-Origin: https://your-origin.com

// 2. JSONP - Using script tags (legacy)
function handleData(data) {
  console.log(data);
}

const script = document.createElement('script');
script.src = 'https://api.com/data?callback=handleData';
document.body.appendChild(script);

// 3. postMessage - Cross-window communication
// Parent window
const iframe = document.getElementById('myIframe');
iframe.contentWindow.postMessage('Hello!', 'https://different-origin.com');

// Child window (different-origin.com)
window.addEventListener('message', (event) => {
  if (event.origin !== 'https://parent-origin.com') return;
  console.log('Message:', event.data);
  
  // Send response
  event.source.postMessage('Response!', event.origin);
});

// 4. document.domain (for subdomains only)
// On https://app.example.com
document.domain = 'example.com';

// On https://api.example.com
document.domain = 'example.com';

// Now they can access each other's DOM
// ⚠️ Deprecated, use postMessage instead

// 5. Proxy - Route through same origin
// Frontend (https://example.com)
fetch('/api/external-data') // Same origin

// Backend proxy (https://example.com/api/external-data)
app.get('/api/external-data', async (req, res) => {
  const response = await fetch('https://different-origin.com/data');
  const data = await response.json();
  res.json(data);
});

// ========================================
// SECURITY IMPLICATIONS
// ========================================

// ❌ Without SOP, malicious site could:
// 1. Read your emails
<iframe src="https://gmail.com"></iframe>
const emails = iframe.contentDocument.querySelector('.emails');
// → Blocked by SOP ✅

// 2. Steal your bank data
fetch('https://mybank.com/account')
  .then(r => r.json())
  .then(data => sendToAttacker(data));
// → Blocked by SOP (no CORS header) ✅

// 3. Make unauthorized requests
fetch('https://mybank.com/transfer', {
  method: 'POST',
  body: JSON.stringify({ to: 'attacker', amount: 1000 })
});
// → Blocked by SOP + CSRF protection ✅

// ========================================
// COMMON MISTAKES
// ========================================

// ❌ Thinking www and non-www are same origin
https://example.com !== https://www.example.com

// ❌ Forgetting about port
https://example.com !== https://example.com:8080

// ❌ Assuming HTTP and HTTPS are same
http://example.com !== https://example.com

// ❌ Using file:// protocol
file:///C:/page1.html !== file:///C:/page2.html
// Each file:// has unique origin

// ========================================
// TESTING SAME-ORIGIN
// ========================================

function isSameOrigin(url1, url2) {
  try {
    const a = new URL(url1);
    const b = new URL(url2);
    
    return a.protocol === b.protocol &&
           a.hostname === b.hostname &&
           a.port === b.port;
  } catch (error) {
    return false;
  }
}

console.log(isSameOrigin(
  'https://example.com/page1',
  'https://example.com/page2'
)); // true

console.log(isSameOrigin(
  'https://example.com',
  'https://api.example.com'
)); // false
```

## Usage
- **When it applies**: 
  - JavaScript accessing DOM across windows/iframes
  - AJAX/Fetch requests to different domains
  - Reading cookies, localStorage, sessionStorage
  - Canvas operations with cross-origin images
  - Web Workers loading scripts
  
- **Real-world example**:
  ```javascript
  // E-commerce checkout with payment iframe
  // Main page: https://shop.com
  // Payment iframe: https://payments.stripe.com
  
  // ❌ Cannot access iframe DOM directly
  const paymentFrame = document.getElementById('stripe-iframe');
  // paymentFrame.contentDocument // ← Blocked!
  
  // ✅ Use postMessage for communication
  paymentFrame.contentWindow.postMessage({
    type: 'PAYMENT_AMOUNT',
    amount: 99.99
  }, 'https://payments.stripe.com');
  
  window.addEventListener('message', (event) => {
    if (event.origin !== 'https://payments.stripe.com') {
      return; // Verify origin!
    }
    
    if (event.data.type === 'PAYMENT_SUCCESS') {
      completeOrder(event.data.transactionId);
    }
  });
  
  // Analytics dashboard loading data
  // Dashboard: https://dashboard.company.com
  // API: https://api.company.com
  
  // ❌ Blocked without CORS
  fetch('https://api.company.com/metrics')
  
  // ✅ Solution 1: Enable CORS on API
  // API server adds header
  res.header('Access-Control-Allow-Origin', 'https://dashboard.company.com');
  
  // ✅ Solution 2: Use proxy
  // Dashboard backend proxies requests
  fetch('/api/metrics') // Same origin, proxied to api.company.com
  
  // Social media embed with security
  // Main site: https://myblog.com
  // Embedded Twitter widget: https://platform.twitter.com
  
  // Twitter uses postMessage for resize events
  window.addEventListener('message', (event) => {
    if (event.origin !== 'https://platform.twitter.com') {
      return;
    }
    
    if (event.data.type === 'RESIZE') {
      twitterIframe.style.height = event.data.height + 'px';
    }
  });
  ```

- **Best practices**:
  - Always validate `event.origin` in postMessage handlers
  - Use CORS headers only for trusted origins, not `*`
  - Implement CSRF tokens for state-changing operations
  - Use SameSite cookie attribute for additional protection
  - Prefer postMessage over document.domain
  - Use proxy servers when CORS isn't available
  - Be aware of subdomain implications
  - Test cross-origin scenarios thoroughly

## FAQ / Interview Questions

**Q: What is the Same-Origin Policy and why is it important?**
A: **Same-Origin Policy** is a security mechanism that restricts how scripts from one origin can interact with resources from another origin. Two URLs have the same origin if they share:
- Same **protocol** (http/https)
- Same **domain** (example.com)
- Same **port** (80, 443, etc.)

It's important because without SOP:
- Malicious sites could read your Gmail, bank account
- Scripts could steal data from any site you're logged into
- XSS attacks would be far more dangerous

**Q: What's the difference between Same-Origin Policy and CORS?**
A:
- **Same-Origin Policy**: Browser's default security - blocks cross-origin requests
- **CORS**: Mechanism to safely relax SOP - allows specific cross-origin requests

```javascript
// SOP: Default behavior (restrictive)
fetch('https://different.com/api') // ❌ Blocked

// CORS: Server opts-in to allow access
// Server adds header:
Access-Control-Allow-Origin: https://mysite.com
// Now: fetch('https://different.com/api') // ✅ Allowed
```

SOP is the rule, CORS is the exception.

**Q: What are the different parts of an origin?**
A: An origin consists of three parts:
1. **Protocol** (scheme): http, https, file, etc.
2. **Domain** (hostname): example.com, www.example.com
3. **Port**: 80, 443, 8080, etc.

```javascript
// All different origins:
https://example.com:443      // Protocol: https, Domain: example.com, Port: 443
http://example.com:80        // Protocol: http (different)
https://www.example.com:443  // Domain: www.example.com (different)
https://example.com:8080     // Port: 8080 (different)
```

Even one difference = different origin!

**Q: How can you communicate between different origins safely?**
A: Several methods:
1. **postMessage API** (recommended for iframes/windows):
```javascript
// Sender
otherWindow.postMessage(data, 'https://target-origin.com');

// Receiver
window.addEventListener('message', (event) => {
  if (event.origin !== 'https://trusted-origin.com') return;
  console.log(event.data);
});
```

2. **CORS** (for AJAX requests):
```javascript
// Server allows cross-origin requests
Access-Control-Allow-Origin: https://frontend.com
```

3. **Server proxy**:
```javascript
// Frontend calls same-origin backend
fetch('/api/proxy-to-external')
// Backend forwards to different origin
```

**Q: What resources can be loaded cross-origin without CORS?**
A: These are allowed by default:
- `<img>` - Images
- `<script>` - JavaScript files
- `<link>` - Stylesheets
- `<video>` / `<audio>` - Media
- `<iframe>` - Embedded pages (but can't access content)
- `<form>` submissions

However, reading their content via JavaScript is still blocked:
```javascript
// ✅ Can load
<img src="https://other.com/image.jpg">

// ❌ Can't read pixel data (without CORS)
canvas.drawImage(img, 0, 0); // Taints canvas
canvas.toDataURL(); // SecurityError
```

## References
- [MDN Web Docs - Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
- [MDN Web Docs - Window.postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage)
- [OWASP - Same Origin Policy](https://owasp.org/www-community/Same_Origin_Policy)
- [Web Security - Same-Origin Policy](https://web.dev/same-origin-policy/)

---
*See also: [CORS](CORS.md), [CSP](CSP.md), [XSS](XSS.md), [CSRF](CSRF.md)*
