# Web Storage

## Definition / Concept

**Web Storage** is a browser API that allows web applications to store key-value data **locally on the client-side** without sending it to the server. It includes two mechanisms: **localStorage** (persistent storage) and **sessionStorage** (temporary storage). Web Storage provides a simple alternative to cookies with larger capacity, better security, and a cleaner API. Data is stored in plain text and should never contain sensitive information like passwords or tokens.

- Stores data as **key-value pairs** on the client-side
- **localStorage**: Persists until explicitly deleted (survives browser restart)
- **sessionStorage**: Clears when the tab/window closes (session-specific)
- Capacity: Typically **5-10MB** per origin (much larger than cookies)
- Data is **synchronous** and **not sent** with every HTTP request like cookies

## Visual Representation

```
Web Storage Types & Lifecycle:

localStorage:
┌────────────────────────────────────┐
│ User Data (Persistent)             │
├────────────────────────────────────┤
│ Stored on: Page 1                  │
│ Accessible from: Page 1, Page 2    │
│ Lifetime: Until manually cleared   │
│ Survives: Browser restart, closure │
└────────────────────────────────────┘

sessionStorage:
┌────────────────────────────────────┐
│ User Data (Temporary)              │
├────────────────────────────────────┤
│ Stored on: Page 1                  │
│ Accessible from: Page 1 only       │
│ Lifetime: Until tab closes         │
│ Survives: Page refresh             │
└────────────────────────────────────┘

Comparison with Cookies:
┌──────────────────────────────────────┐
│ Cookies: Small (4KB), Auto-sent      │
│ Web Storage: Large (5-10MB), Manual  │
└──────────────────────────────────────┘
```

## Example

### localStorage - Persistent

```javascript
// Store data
localStorage.setItem('username', 'john_doe');
localStorage.setItem('user', JSON.stringify({ id: 1, name: 'John' }));

// Retrieve data
const username = localStorage.getItem('username');  // "john_doe"
const user = JSON.parse(localStorage.getItem('user'));

// Remove data
localStorage.removeItem('username');
localStorage.clear();  // Clear all
```

### sessionStorage - Temporary

```javascript
// Same API as localStorage, but cleared on tab close
sessionStorage.setItem('sessionId', 'abc123');
const id = sessionStorage.getItem('sessionId');
sessionStorage.removeItem('sessionId');
```

### Shopping Cart Example

```javascript
function saveCart(items) {
  localStorage.setItem('cart', JSON.stringify(items));
}

function getCart() {
  const cart = localStorage.getItem('cart');
  return cart ? JSON.parse(cart) : [];
}

saveCart([{ id: 1, name: 'Laptop', price: 999 }]);
const cart = getCart();  // Persists after reload!
```

### Theme Preference

```javascript
function setTheme(theme) {
  localStorage.setItem('theme', theme);
  document.documentElement.setAttribute('data-theme', theme);
}

const savedTheme = localStorage.getItem('theme') || 'light';
setTheme(savedTheme);
```

### Sync Across Tabs

```javascript
window.addEventListener('storage', (event) => {
  if (event.key === 'theme') {
    setTheme(event.newValue);  // Another tab changed theme
  }
});
```

## Usage

- **When to use**: Saving user preferences, caching data, persisting form inputs, storing authentication tokens (non-sensitive), maintaining app state
- **Real-world example**: Theme preferences, shopping carts, form drafts, recently visited items, user settings, offline app data
- **Best practices**:
  - Only store **non-sensitive** data (never passwords, API keys, or sensitive tokens)
  - Always **JSON serialize** complex objects before storing
  - Handle **storage exceptions** (quota exceeded, private mode)
  - Set an **expiration strategy** for outdated data
  - Use **sessionStorage** for temporary, session-specific data
  - Listen to `storage` events for syncing across tabs
  - Always wrap in try-catch due to quota limits and privacy modes
  - Validate and sanitize retrieved data before using

## FAQ / Interview Questions

**Q: What's the difference between localStorage and sessionStorage?**
A: Both store key-value data client-side with the same API. The main difference is **lifetime**: localStorage persists indefinitely (survives browser restart, page refresh), while sessionStorage clears when the tab/window closes. Use localStorage for permanent user preferences, sessionStorage for temporary session data like form progress or session IDs.

**Q: How much data can Web Storage hold?**
A: Most modern browsers allow **5-10MB** per origin (domain/protocol/port combination), compared to cookies' ~4KB. The exact limit varies by browser. You can exceed the limit if you store too much data—this throws a **QuotaExceededError**. Always wrap storage operations in try-catch blocks.

**Q: Is Web Storage secure? Can I store passwords or tokens?**
A: No! Web Storage is **not secure**. Data is stored in plain text and vulnerable to **XSS attacks**. Never store passwords, API keys, or sensitive tokens. For authentication tokens, consider **httpOnly cookies** (not accessible to JavaScript) or keep tokens in memory. If you must store tokens, use secure, refreshable tokens with short expiration times.

**Q: How do I sync data across multiple tabs/windows?**
A: Listen to the **storage event**, which fires when data changes in any other tab/window of the same origin:
```javascript
window.addEventListener('storage', (event) => {
  if (event.key === 'user') {
    console.log('User updated in another tab:', event.newValue);
  }
});
```
Note: The storage event **doesn't fire in the tab that made the change**, only in other tabs.

**Q: Can I use Web Storage if the browser is in private mode?**
A: Web Storage works in private mode, but with **important differences**: data is cleared when the private window closes, quota might be smaller, and some browsers may throw errors when storage is full. Always wrap operations in try-catch and handle the QuotaExceededError gracefully.

## References
- [MDN - Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)
- [MDN - localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [MDN - sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [Web Storage Specification](https://html.spec.whatwg.org/multipage/webstorage.html)

---

*See also: [Cookies](./Cookies.md), [IndexedDB](./IndexedDB.md), [LocalStorage Best Practices](./LocalStorageBestPractices.md), [Session Management](./SessionManagement.md)*
