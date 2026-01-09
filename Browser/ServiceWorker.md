# Service Worker

## Definition / Concept

A **Service Worker** is a JavaScript file that runs in the background, separate from the main browser thread, enabling offline functionality, caching strategies, and push notifications. It acts as a **proxy between the web app and the network**, intercepting requests and responses. Service Workers are essential for building **Progressive Web Apps (PWAs)** and can handle network requests even when the user is offline. They persist in the background and can be updated independently of the main application.

- **Background script** runs independently of web pages
- **Network proxy** intercepts and modifies requests/responses
- **Offline support** enables apps to work without internet
- **Cache management** allows strategic caching of assets
- **Push notifications** can deliver messages to users

## Visual Representation

```
SERVICE WORKER ARCHITECTURE
═══════════════════════════════════════════════════════════

┌──────────────────────────────────────────────────────────┐
│                   Browser Process                        │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │           Main Thread (Web Page)                 │  │
│  │  - DOM Manipulation                              │  │
│  │  - User Interactions                             │  │
│  └───────────────────┬────────────────────────────┘  │
│                      │ Register                       │
│                      ↓                                │
│  ┌──────────────────────────────────────────────────┐  │
│  │     Service Worker Thread (Background)          │  │
│  │  - Runs independently                            │  │
│  │  - Persists across page closes                   │  │
│  │  - Intercepts network requests                   │  │
│  │  - Manages cache                                 │  │
│  └──────────────────────────────────────────────────┘  │
│                      │                                 │
│                      ↓                                 │
│  ┌──────────────────────────────────────────────────┐  │
│  │          Network / Cache Storage                 │  │
│  │  - HTTP Requests                                 │  │
│  │  - Cached Responses                              │  │
│  └──────────────────────────────────────────────────┘  │
│                                                        │
└──────────────────────────────────────────────────────────┘

SERVICE WORKER LIFECYCLE
═══════════════════════════════════════════════════════════

1. REGISTRATION
   navigator.serviceWorker.register('sw.js')
                    ↓
2. INSTALLATION
   'install' event → Cache assets
                    ↓
3. ACTIVATION
   'activate' event → Clean up old caches
                    ↓
4. RUNNING
   Intercept requests via 'fetch' event
                    ↓
5. UPDATE
   New version detected → Repeat from Installation

FETCH EVENT FLOW
═══════════════════════════════════════════════════════════

Web Page Request
       ↓
Service Worker intercepts (fetch event)
       ├─ Cache Hit? → Return cached response ✓
       ├─ Network? → Fetch from network → Cache → Return ✓
       └─ No connection? → Return offline page ✓
```

## Example

### Service Worker Setup

```javascript
// Register Service Worker (main app)
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(reg => console.log('SW registered'))
    .catch(err => console.error('SW failed:', err));
}

// Service Worker file (sw.js)
const CACHE_NAME = 'my-app-v1';
const urlsToCache = ['/', '/index.html', '/styles.css', '/app.js'];

// Install: Cache assets
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CACHE_NAME).then(cache => cache.addAll(urlsToCache))
  );
  self.skipWaiting();
});

// Activate: Clean up old caches
self.addEventListener('activate', event => {
  event.waitUntil(
    caches.keys().then(names =>
      Promise.all(names.filter(name => name !== CACHE_NAME).map(name => caches.delete(name)))
    )
  );
  self.clients.claim();
});

// Fetch: Intercept requests (cache first)
self.addEventListener('fetch', event => {
  if (event.request.method !== 'GET') return;

  event.respondWith(
    caches.match(event.request)
      .then(response => response || fetch(event.request))
      .catch(() => caches.match('/offline.html'))
  );
});
```

### Caching Strategies

```javascript
// Cache First: Use cached, fallback to network
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request).then(response => response || fetch(event.request))
  );
});

// Network First: Try network, fallback to cache
self.addEventListener('fetch', event => {
  event.respondWith(
    fetch(event.request)
      .then(res => (caches.open('api-cache').then(cache => cache.put(event.request, res.clone())), res))
      .catch(() => caches.match(event.request))
  );
});

// Stale While Revalidate: Return cached, update in background
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request).then(cached => {
      const fetched = fetch(event.request).then(res =>
        caches.open('api-cache').then(cache => (cache.put(event.request, res.clone()), res))
      );
      return cached || fetched;
    })
  );
});
```

### Communication with Main Thread

```javascript
// Main app: Send message to Service Worker
navigator.serviceWorker.controller.postMessage({
  type: 'UPDATE_CACHE',
  payload: { url: '/api/data' }
});

// Main app: Listen for messages from Service Worker
navigator.serviceWorker.addEventListener('message', event => {
  console.log('Cache updated:', event.data.payload);
});

// Service Worker: Listen for messages from app
self.addEventListener('message', event => {
  if (event.data.type === 'UPDATE_CACHE') {
    fetch(event.data.payload.url).then(res =>
      caches.open('api-cache').then(cache => cache.put(event.data.payload.url, res))
    );
  }
});
```

### Push Notifications

```javascript
// Request permission and subscribe (main app)
Notification.requestPermission().then(perm => {
  if (perm === 'granted') {
    navigator.serviceWorker.ready.then(reg =>
      reg.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: 'YOUR_PUBLIC_KEY'
      })
    );
  }
});

// Handle push event (Service Worker)
self.addEventListener('push', event => {
  const { title, body, url } = event.data.json();
  event.waitUntil(
    self.registration.showNotification(title, { body, data: { url } })
  );
});

// Handle notification click (Service Worker)
self.addEventListener('notificationclick', event => {
  event.notification.close();
  event.waitUntil(
    clients.matchAll().then(list =>
      list.find(c => c.url === event.notification.data.url)?.focus()
      || clients.openWindow(event.notification.data.url)
    )
  );
});
```

### Checking for Updates

```javascript
// Check for updates periodically
navigator.serviceWorker.ready.then(reg => {
  setInterval(() => reg.update(), 60000);
});

// Reload when new Service Worker is activated
navigator.serviceWorker.addEventListener('controllerchange', () => {
  window.location.reload();
});
```

## Usage

### When to Use Service Workers

- **Offline functionality** — Apps should work without internet
- **Performance** — Cache static assets for faster loading
- **Progressive Web Apps (PWAs)** — Essential for app-like experience
- **Background sync** — Sync data when connection returns
- **Push notifications** — Send updates to users
- **Resource optimization** — Reduce bandwidth usage

### Real-world Example

```javascript
// Smart PWA caching: Different strategies for different content
self.addEventListener('fetch', event => {
  const url = new URL(event.request.url);

  // API: Network first, fallback to cache
  if (url.pathname.startsWith('/api/')) {
    return event.respondWith(
      fetch(event.request)
        .then(res => (caches.open('api-cache').then(c => c.put(event.request, res.clone())), res))
        .catch(() => caches.match(event.request))
    );
  }

  // Static assets: Cache first, fallback to network
  event.respondWith(
    caches.match(event.request)
      .then(res => res || fetch(event.request).then(res =>
        caches.open('static-cache').then(c => (c.put(event.request, res.clone()), res))
      ))
      .catch(() => caches.match('/offline.html'))
  );
});
```

### Best Practices

- **Version your caches** — Use `v1`, `v2` in cache names for easy cleanup
- **Handle errors gracefully** — Provide offline fallbacks and error pages
- **Update strategically** — Choose cache-first or network-first based on content type
- **Scope Service Workers carefully** — Register with appropriate paths
- **Test offline mode** — Use DevTools to simulate offline scenarios
- **Clean up old caches** — Remove unused caches in activate event
- **Monitor performance** — Track cache hit rates and update frequency

## FAQ / Interview Questions

**Q: What is a Service Worker and why is it important?**
A: A Service Worker is a background JavaScript process that runs independently of web pages, intercepting network requests. It's crucial for building offline-capable applications, implementing caching strategies, and enabling PWA features like push notifications. Service Workers improve performance by serving cached content and provide reliable user experiences even without internet.

**Q: What are the main events in a Service Worker lifecycle?**
A: The main events are: `install` (fired when registering, used to cache assets), `activate` (fired when Service Worker becomes active, used to clean up old caches), and `fetch` (fired for every network request from the page). Understanding these events is essential for implementing caching strategies and offline functionality.

**Q: What's the difference between Cache First and Network First strategies?**
A: **Cache First** returns cached content immediately if available, then falls back to the network (good for static assets). **Network First** tries the network first, caches successful responses, and falls back to cache if offline (good for dynamic content). Choose based on content freshness requirements.

**Q: How do Service Workers enable offline functionality?**
A: Service Workers intercept fetch requests via the `fetch` event listener. When offline, they serve previously cached responses instead of attempting network calls. Combined with comprehensive caching during installation and activation phases, this allows apps to remain functional without internet connectivity.

**Q: Can a Service Worker access the DOM?**
A: No, Service Workers cannot access the DOM because they run on a separate thread from the main page. They can communicate with the main thread using `postMessage()` to request DOM updates or receive information. This separation prevents blocking the UI with background tasks.

## References

- [MDN - Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
- [Google - Service Workers: An Introduction](https://developers.google.com/web/fundamentals/primers/service-workers)
- [Web Dev - Service Worker](https://web.dev/service-workers/)
- [Service Worker Cookbook](https://serviceworke.rs/)

---

*See also: [Web Storage](./WebStorage.md), [Cookie](./Cookie.md), [Progressive Web Apps](../Performance/ProgressiveWebApps.md)*
