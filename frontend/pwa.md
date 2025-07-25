---
title: PWA
---

## Introduction

A Progressive Web App is a web application that takes advantage of modern
browser capabilities like _Service Workers_, _Web App Manifests_, and _Push
APIs_ to deliver an app-like experience: offline support, fast load times, and
home‑screen installation, all without requiring a traditional app store.

- Platform Independence: Runs on any device with a modern browser.
- Low Friction: Users can _install_ from the browser without app‑store approval
  delays.
- Performance: Caching strategies and background sync deliver near-native speed
  and resilience.

## Features

### Service Worker

- A background script between the network and your app.
- Enables offline capabilities, caching strategies, background sync, and
  intercepting network requests.

### Web App Manifest

- A JSON file (`manifest.json`) that defines app metadata: name, icons, theme
  colors, and display mode (`standalone` to remove browser UI).

### Caching

- Fine‑grained control over resource caching, from static assets to dynamic API
  responses.
- Strategies include _Cache First_, _Network First_, _Stale‑While‑Revalidate_,
  etc.

### Push Notifications

- Use the Push API + Notification API to send timely, contextual messages even
  when the PWA isn't open.
- Requires user permission and a push service (e.g., Firebase Cloud Messaging).

### Installability & Home-Screen

- Prompt users to install the PWA; once installed, it runs in a standalone
  window, like a native app.

## Caching Strategies in Detail

| Strategy                   | Use Case                                  | Behavior                                                           |
| -------------------------- | ----------------------------------------- | ------------------------------------------------------------------ |
| **Cache First**            | Static assets (CSS, JS, images)           | Serve from cache; fallback to network if missing.                  |
| **Network First**          | API calls where freshness is critical     | Attempt network; fallback to cache if offline.                     |
| **Stale‑While‑Revalidate** | Balance freshness and speed               | Serve cached copy immediately; fetch & update cache in background. |
| **Cache Only**             | Immutable resources                       | Only from cache; ideal for hashed asset filenames.                 |
| **Network Only**           | Real‑time data, analytics, authentication | Always fetch from network; never cache.                            |

## PWAs vs. Native Apps

| Aspect              | PWA                                                       | Native App                                         |
| ------------------- | --------------------------------------------------------- | -------------------------------------------------- |
| **Installation**    | One‑click via browser; no store review                    | App store submission; installation via store       |
| **Platform Reach**  | Single codebase for all platforms                         | Platform‑specific code (iOS, Android)              |
| **Performance**     | Very fast for web workloads; limited GPU/OS access        | Full access to hardware, sensors, background tasks |
| **Capabilities**    | Limited vs. native APIs (e.g., Bluetooth LE, file system) | Rich ecosystem of OS APIs                          |
| **Discoverability** | Indexed by search engines; shareable via URL              | Store search; SEO limited                          |
| **Maintenance**     | Deploy once; updates are instant                          | Store updates; users may delay updating            |

---

## Common Interview Questions & Answers

### What are the core pillars or "must-have" components of a PWA?

- HTTPS (secure origin)
- A registered Service Worker (for offline caching and network interception)
- A Web App Manifest with at least `name`/`short_name`, `start_url`, `display`,
  and icons (192px and 512px)

### What is a Service Worker and how does it enable offline functionality?

A Service Worker is a background JavaScript file that intercepts network
requests. In its `install` phase, you pre‑cache assets; in `fetch` events, you
serve cached resources when offline, ensuring the app remains functional without
connectivity.

### Explain the Cache‑First vs. Network‑First strategies

- Cache‑First: Looks in cache first, then network. Ideal for static assets for
  fast loads.
- Network‑First: Attempts network first, with cache fallback. Suited for dynamic
  data where freshness matters.

### Describe stale-while-revalidate strategy in caching APIs

First return cached content immediately, then in background fetch fresh content
from network. Once fetched, update cache for future use. This balances fast UX
with data freshness.

### What is the `beforeinstallprompt` event and how do you use it?

When a PWA meets install criteria, browsers fire `beforeinstallprompt`. You can
intercept it, prevent default UI, and show your custom **Install** button. Then
call `event.prompt()` to show the dialog and check `event.userChoice` to track
acceptance.

### What are the limitations of PWAs compared to native apps?

PWAs have limited access to device hardware and OS-level features (e.g.,
low‑level Bluetooth, background geolocation on iOS), and rely on browser support
variability across platforms.

### How do you implement push notifications in a PWA?

- Request Notification permission via `Notification.requestPermission()`.
- In Service Worker, use `pushManager.subscribe()` with a VAPID public key.
- Send subscription to server; send pushes via a push service.
- Handle `push` event in SW to call `showNotification()`.

### Can you explain the Web App Manifest?

It's a JSON file declaring metadata (name, icons, theme colors, display mode)
that instructs the browser how to present the PWA when installed (e.g.,
standalone, fullscreen).

### How does Angular integrate PWA features?

Angular CLI's `@angular/pwa` schematic scaffolds a service worker
(`ngsw-config.json`), adds a manifest, icons, and registers `ngsw-worker.js`
automatically in production builds.

### Explain App Shell Architecture. Why is it helpful?

App Shell separates static UI (shell) from dynamic content. The shell is
pre-cached and served instantly, giving immediate UI access while content loads.
This improves perceived performance and enables offline-first behavior.

### Which browsers support PWAs and what limitations exist?

Chromium-based browsers (Chrome, Edge, Brave, Opera) fully support PWA features
like installability and push notifications. Firefox has partial support; Safari
on iOS has more limited support (e.g., restricted push, background sync).

### How do service workers handle version updates?

When a new SW script is detected, the browser downloads it into the install
state; it waits to activate until all PWA pages are closed. Once activated, the
old cache can be deleted in the activate event. This ensures consistency and
avoids version conflicts.

### What are common security concerns with service workers?

Because SWs run as trusted scripts, vulnerabilities include abuse for cross‑site
scripting/malvertising via push notifications, unauthorized background syncing,
or persistent cached malware. Mitigations include strict Content Security
Policy, limiting external scripts, and regular audits.

### How do you debug service workers and caching issues?

Use Chrome DevTools -> Application panel -> Service Workers and Cache Storage.
From there, you can unregister SWs, clear caches, simulate offline mode, step
through lifecycle events, and inspect intercepted fetch events.

### How do you implement background sync with service workers?

Background Sync (via the Sync API) allows PWAs to defer important tasks (like
sending data to a server) when offline. The task is retried later automatically
once the device regains connectivity, even if the PWA/Tab is closed.

- Register a sync event:
  ```javascript
  navigator.serviceWorker.ready.then((sw) => sw.sync.register("syncTag"));
  ```
- In Service Worker:
  ```javascript
  self.addEventListener("sync", (event) => {
    if (event.tag === "syncTag") {
      event.waitUntil(doDeferredWork());
    }
  });
  ```

---

## Pure JavaScript: Step-by-Step PWA Setup

### Project Structure

```html
<!--
Directory structure:
/
|-- index.html
|-- offline.html
|-- manifest.json
|-- styles.css
|-- app.js
|-- sw.js
\-- icons/
    |-- icon-192x192.png
    \-- icon-512x512.png
-->
```

### `manifest.json`

```json
{
  "name": "Pure JS PWA Demo",
  "short_name": "JS PWA",
  "start_url": ".",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#317EFB",
  "icons": [
    {
      "src": "icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    { "src": "icons/icon-512x512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

### `index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Pure JS PWA Demo</title>

    <!-- Web App Manifest -->
    <link rel="manifest" href="manifest.json" />
    <meta name="theme-color" content="#317EFB" />

    <!-- Fallback offline page -->
    <link rel="stylesheet" href="styles.css" />
  </head>
  <body>
    <h1>Welcome to Pure JS PWA</h1>
    <p>
      This PWA demonstrates installation, caching strategies, and push
      notifications.
    </p>
    <button id="notify-btn">Enable Push Notifications</button>
    <script src="app.js"></script>
  </body>
</html>
```

### `offline.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Offline</title>
  </head>
  <body>
    <h1>Offline</h1>
    <p>Sorry, you are offline and this resource is not cached.</p>
  </body>
</html>
```

### `sw.js`

```javascript
const CACHE_NAME = "pwa-cache-v1";
const STATIC_ASSETS = [
  "/",
  "/index.html",
  "/styles.css",
  "/app.js",
  "/offline.html",
  "/icons/icon-192x192.png",
  "/icons/icon-512x512.png",
];

// Utility to limit cache size
async function trimCache(cacheName, maxItems) {
  const cache = await caches.open(cacheName);
  const keys = await cache.keys();
  if (keys.length > maxItems) {
    cache.delete(keys[0]).then(() => trimCache(cacheName, maxItems));
  }
}

self.addEventListener("install", (event) => {
  // Pre-cache static assets
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => cache.addAll(STATIC_ASSETS))
  );
});

self.addEventListener("activate", (event) => {
  // Delete old caches
  event.waitUntil(
    caches
      .keys()
      .then((keys) =>
        Promise.all(
          keys
            .filter((key) => key !== CACHE_NAME)
            .map((key) => caches.delete(key))
        )
      )
  );
  return self.clients.claim();
});

self.addEventListener("fetch", (event) => {
  const url = new URL(event.request.url);

  // Network-first for API calls
  if (url.pathname.startsWith("/api/")) {
    event.respondWith(
      fetch(event.request)
        .then((res) => {
          const copy = res.clone();
          caches.open(CACHE_NAME).then((cache) => {
            cache.put(event.request, copy);
            trimCache(CACHE_NAME, 50);
          });
          return res;
        })
        .catch(() => caches.match(event.request))
    );
    return;
  }

  // Cache-first for static assets
  if (
    STATIC_ASSETS.includes(url.pathname) ||
    url.pathname.startsWith("/icons/")
  ) {
    event.respondWith(
      caches
        .match(event.request)
        .then((cached) => cached || fetch(event.request))
    );
    return;
  }

  // Stale-while-revalidate for others
  event.respondWith(
    caches
      .match(event.request)
      .then((cached) => {
        const networkFetch = fetch(event.request).then((res) => {
          caches
            .open(CACHE_NAME)
            .then((cache) => cache.put(event.request, res.clone()));
          return res;
        });
        return cached || networkFetch;
      })
      .catch(() => caches.match("/offline.html"))
  );
});

// Push notifications
self.addEventListener("push", (event) => {
  const data = event.data.json();
  const options = {
    body: data.body,
    icon: "/icons/icon-192x192.png",
    data: data.url,
  };
  event.waitUntil(self.registration.showNotification(data.title, options));
});

self.addEventListener("notificationclick", (event) => {
  event.notification.close();
  event.waitUntil(clients.openWindow(event.notification.data));
});
```

### `app.js`

```javascript
// Register service worker
if ("serviceWorker" in navigator) {
  navigator.serviceWorker
    .register("/sw.js")
    .then((reg) => console.log("SW registered", reg))
    .catch((err) => console.error("SW registration failed", err));
}

// Installation prompt handling
let deferredPrompt;
const installBtn = document.createElement("button");
installBtn.textContent = "Install App";
installBtn.style.display = "none";
document.body.appendChild(installBtn);

window.addEventListener("beforeinstallprompt", (e) => {
  e.preventDefault();
  deferredPrompt = e;
  installBtn.style.display = "block";
});

installBtn.addEventListener("click", () => {
  installBtn.style.display = "none";
  deferredPrompt.prompt();
  deferredPrompt.userChoice.then((choice) => {
    console.log(choice.outcome);
    deferredPrompt = null;
  });
});

// Push Notifications setup
const notifyBtn = document.getElementById("notify-btn");
notifyBtn.addEventListener("click", () => {
  if (!("Notification" in window) || !("PushManager" in window)) {
    return alert("Push notifications not supported");
  }
  Notification.requestPermission().then((permission) => {
    if (permission === "granted") subscribeUser();
  });
});

async function subscribeUser() {
  const registration = await navigator.serviceWorker.ready;
  // Replace with your VAPID public key
  const vapidPublicKey = "YOUR_PUBLIC_VAPID_KEY";
  const convertedKey = urlBase64ToUint8Array(vapidPublicKey);

  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: convertedKey,
  });

  // Send subscription to server
  await fetch("/api/subscribe", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(subscription),
  });
  alert("Subscribed successfully!");
}

function urlBase64ToUint8Array(base64String) {
  const padding = "=".repeat((4 - (base64String.length % 4)) % 4);
  const base64 = (base64String + padding).replace(/-/g, "+").replace(/_/g, "/");
  const rawData = atob(base64);
  return Uint8Array.from([...rawData].map((char) => char.charCodeAt(0)));
}
```

---

## Angular: Step-by-Step PWA Setup

### Add PWA Support

```bash
ng add @angular/pwa
```

### Configure `ngsw-config.json`

```json
{
  "index": "/index.html",
  "assetGroups": [
    {
      "name": "app",
      "installMode": "prefetch",
      "resources": {
        "files": ["/favicon.ico", "/index.html", "/*.css", "/*.js"]
      }
    },
    {
      "name": "assets",
      "installMode": "lazy",
      "updateMode": "prefetch",
      "resources": {
        "files": ["/assets/**", "/*.(png|jpg)"]
      }
    }
  ],
  "dataGroups": [
    {
      "name": "api-calls",
      "urls": ["https://api.example.com/**"],
      "cacheConfig": {
        "maxSize": 100,
        "maxAge": "1h",
        "strategy": "freshness"
      }
    }
  ]
}
```

### Service Worker Registration

Automatically handled in `app.module.ts`:

```typescript
ServiceWorkerModule.register("ngsw-worker.js", {
  enabled: environment.production,
});
```

### Push Notifications

- Install _RxJS Web Push_ (or use `@angular/service-worker` PET).
- Subscribe:
  ```typescript
  this.swPush.requestSubscription({
    serverPublicKey: this.vapidPublicKey
  })
  .then(sub => /* send to server */)
  .catch(err => console.error('Could not subscribe', err));
  ```
- Handle in Service Worker: customize `ngsw-worker.js` or extend via a separate
  SW file.
