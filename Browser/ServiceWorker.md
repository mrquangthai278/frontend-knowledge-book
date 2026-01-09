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

### Basic Service Worker Registration

```javascript
// In your main application (e.g., index.js)
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(registration => {
      console.log('Service Worker registered:', registration);
    })
    .catch(error => {
      console.error('Service Worker registration failed:', error);
    });
}
```

### Basic Service Worker File (sw.js)

```javascript
// Service Worker file
const CACHE_NAME = 'my-app-v1';
const urlsToCache = [
  '/',
  '/index.html',
  '/styles/main.css',
  '/scripts/main.js',
  '/images/logo.png'
];

// Install event - cache assets
self.addEventListener('install', event => {
  console.log('Service Worker installing...');

  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => {
        console.log('Caching app shell');
        return cache.addAll(urlsToCache);
      })
  );

  // Force the waiting Service Worker to become the active one
  self.skipWaiting();
});

// Activate event - cleanup old caches
self.addEventListener('activate', event => {
  console.log('Service Worker activating...');

  event.waitUntil(
    caches.keys().then(cacheNames => {
      return Promise.all(
        cacheNames.map(cacheName => {
          if (cacheName !== CACHE_NAME) {
            console.log('Deleting old cache:', cacheName);
            return caches.delete(cacheName);
          }
        })
      );
    })
  );

  // Take control of all pages immediately
  self.clients.claim();
});

// Fetch event - intercept network requests
self.addEventListener('fetch', event => {
  // Skip non-GET requests
  if (event.request.method !== 'GET') {
    return;
  }

  event.respondWith(
    caches.match(event.request)
      .then(response => {
        // Return cached response if available
        if (response) {
          return response;
        }

        return fetch(event.request).then(response => {
          // Don't cache non-successful responses
          if (!response || response.status !== 200 || response.type === 'error') {
            return response;
          }

          // Clone the response for caching
          const responseToCache = response.clone();
          caches.open(CACHE_NAME).then(cache => {
            cache.put(event.request, responseToCache);
          });

          return response;
        });
      })
      .catch(error => {
        console.error('Fetch failed:', error);
        // Return offline page or cached fallback
        return caches.match('/offline.html');
      })
  );
});
```

### Caching Strategies

```javascript
// 1. CACHE FIRST (Cache, falling back to network)
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request)
      .then(response => response || fetch(event.request))
      .catch(() => caches.match('/offline.html'))
  );
});

// 2. NETWORK FIRST (Network, falling back to cache)
self.addEventListener('fetch', event => {
  event.respondWith(
    fetch(event.request)
      .then(response => {
        const responseClone = response.clone();
        caches.open('dynamic-cache').then(cache => {
          cache.put(event.request, responseClone);
        });
        return response;
      })
      .catch(() => caches.match(event.request))
  );
});

// 3. STALE WHILE REVALIDATE
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request)
      .then(cachedResponse => {
        const fetchPromise = fetch(event.request)
          .then(response => {
            caches.open('dynamic-cache').then(cache => {
              cache.put(event.request, response.clone());
            });
            return response;
          });

        return cachedResponse || fetchPromise;
      })
  );
});

// 4. NETWORK ONLY
self.addEventListener('fetch', event => {
  event.respondWith(fetch(event.request));
});
```

### Communication with Main Thread

```javascript
// In main application (index.js)
const updateButton = document.getElementById('update-btn');

updateButton.addEventListener('click', () => {
  if ('serviceWorker' in navigator && navigator.serviceWorker.controller) {
    navigator.serviceWorker.controller.postMessage({
      type: 'UPDATE_CACHE',
      payload: { url: '/api/data' }
    });
  }
});

navigator.serviceWorker.addEventListener('message', event => {
  if (event.data.type === 'CACHE_UPDATED') {
    console.log('Cache updated:', event.data.payload);
  }
});

// In Service Worker (sw.js)
self.addEventListener('message', event => {
  if (event.data.type === 'UPDATE_CACHE') {
    const url = event.data.payload.url;

    fetch(url)
      .then(response => {
        caches.open('api-cache').then(cache => {
          cache.put(url, response.clone());
        });

        // Send message back to clients
        self.clients.matchAll().then(clients => {
          clients.forEach(client => {
            client.postMessage({
              type: 'CACHE_UPDATED',
              payload: { url }
            });
          });
        });
      });
  }
});
```

### Push Notifications

```javascript
// Request user permission (in main app)
Notification.requestPermission().then(permission => {
  if (permission === 'granted') {
    navigator.serviceWorker.ready.then(registration => {
      registration.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: 'YOUR_PUBLIC_KEY'
      });
    });
  }
});

// Handle push events (in Service Worker)
self.addEventListener('push', event => {
  const data = event.data.json();

  const options = {
    body: data.body,
    icon: '/icon-192x192.png',
    badge: '/badge-72x72.png',
    data: { url: data.url }
  };

  event.waitUntil(
    self.registration.showNotification(data.title, options)
  );
});

// Handle notification click
self.addEventListener('notificationclick', event => {
  event.notification.close();

  event.waitUntil(
    clients.matchAll({ type: 'window' })
      .then(clientList => {
        // Focus existing window if available
        for (let client of clientList) {
          if (client.url === event.notification.data.url && 'focus' in client) {
            return client.focus();
          }
        }
        // Open new window if not found
        if (clients.openWindow) {
          return clients.openWindow(event.notification.data.url);
        }
      })
  );
});
```

### Checking for Updates

```javascript
// In main application
navigator.serviceWorker.ready.then(registration => {
  // Check for updates periodically
  setInterval(() => {
    registration.update();
  }, 60000); // Check every minute
});

// Listen for controller change (new Service Worker activated)
navigator.serviceWorker.addEventListener('controllerchange', () => {
  console.log('New Service Worker activated');
  // Prompt user to reload or reload automatically
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
// Complete PWA example
const CACHE_NAME = 'pwa-v1';
const STATIC_ASSETS = [
  '/',
  '/index.html',
  '/styles/main.css',
  '/scripts/app.js',
  '/offline.html'
];

const API_CACHE = 'api-cache-v1';
const RUNTIME_CACHE = 'runtime-v1';

// Install
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => cache.addAll(STATIC_ASSETS))
      .then(() => self.skipWaiting())
  );
});

// Activate
self.addEventListener('activate', event => {
  event.waitUntil(
    caches.keys()
      .then(names => {
        return Promise.all(
          names
            .filter(name => ![CACHE_NAME, API_CACHE, RUNTIME_CACHE].includes(name))
            .map(name => caches.delete(name))
        );
      })
      .then(() => self.clients.claim())
  );
});

// Fetch with smart caching
self.addEventListener('fetch', event => {
  const { request } = event;
  const url = new URL(request.url);

  // API requests - network first, cache fallback
  if (url.pathname.startsWith('/api/')) {
    event.respondWith(
      fetch(request)
        .then(response => {
          caches.open(API_CACHE).then(cache => {
            cache.put(request, response.clone());
          });
          return response;
        })
        .catch(() => caches.match(request))
        .catch(() => new Response('Offline', { status: 503 }))
    );
  }

  // Static assets - cache first
  event.respondWith(
    caches.match(request)
      .then(response => {
        if (response) return response;

        return fetch(request)
          .then(response => {
            if (response.status === 200) {
              caches.open(RUNTIME_CACHE).then(cache => {
                cache.put(request, response.clone());
              });
            }
            return response;
          })
          .catch(() => caches.match('/offline.html'))
      })
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
