---
title: Micro Frontend
---

## Introduction

A **Microfrontend** architecture breaks a frontend application into smaller,
**independent, self-contained apps** (micro-apps), developed and deployed by
different teams, just like **microservices** on the backend.

Each team:

- Owns a specific business domain
- Builds, tests, and deploys their part of the UI independently
- Can even use different technologies (e.g., React + Angular in the same app)

## Pros and Cons of Microfrontends

| Benefit                 | Details                                                         |
| ----------------------- | --------------------------------------------------------------- |
| Team Autonomy           | Teams deploy independently (just like microservices)            |
| Tech Diversity          | One team uses Vue, another uses React, another plain JS         |
| Scalability             | Works well for large applications split across many teams       |
| Independent Deployments | Each microfrontend can be deployed separately (e.g., via CI/CD) |
| Maintainability         | Smaller codebases are easier to test, debug, and maintain       |

| Drawback                | Details                                                                |
| ----------------------- | ---------------------------------------------------------------------- |
| Initial Complexity      | Architecture, routing, shared state, and communication must be planned |
| Performance Overhead    | Loading multiple bundles can increase app size and startup time        |
| Shared Dependencies     | Duplicate libraries can bloat the app (fixable via Module Federation)  |
| Cross-Team Coordination | Teams must agree on contracts/interfaces                               |

## Core Ideas

### Be Technology Agnostic

> Teams should be free to choose their own stack (framework, build tools, etc.)

- Each microfrontend (MF) is self-contained.
- One team can use **Angular**, another **React**, another plain **JS**.
- Avoid forcing one tech across all teams.

### Isolate Team Code

> Never rely on shared runtime or global state.

- Microfrontends should **not leak CSS**, JS, or global variables.
- Use **Shadow DOM**, **CSS modules**, or **style prefixes**.
- JS should be **scoped** using module systems or `customElements`.

### Establish Clear Contracts

> Communicate via **well-defined APIs or interfaces**.

- Like microservices, microfrontends must have **clean boundaries**.
- Avoid direct DOM manipulation of sibling MFs.
- Use:

  - **Custom Events**:

    ```javascript
    // Micro App A (sender)
    window.dispatchEvent(
      new CustomEvent("user-logged-in", { detail: { userId: 123 } })
    );

    // Micro App B (receiver)
    window.addEventListener("user-logged-in", (e) => {
      console.log("User ID:", e.detail.userId);
    });
    ```

  - **Event Bus (Custom Pub/Sub Layer)**:

    ```javascript
    // pubsub.js
    const listeners = {};
    export const emit = (event, data) =>
      (listeners[event] || []).forEach((cb) => cb(data));
    export const on = (event, cb) =>
      (listeners[event] = [...(listeners[event] || []), cb]);

    // Usage
    on("theme-change", applyTheme);
    emit("theme-change", "dark");
    ```

  - **Props via Web Components**

    ```javascript
    <product-list user-id="123" theme="dark"></product-list>
    ```

  - **Shared Context or Services via Module Federation**:

    ```javascript
    // Host exposes service
    exposes: {
      './authService': './src/services/authService.js'
    }

    // Remote consumes
    import { getCurrentUser } from 'shell/authService';
    ```

  - **Shared Redux-like state**:

    ```javascript
    // Expose Redux store
    export const store = configureStore(...);
    // Consume in another MF
    import { store } from 'shared-state';
    ```

### Favor Native Browser Features

> Use **standards over custom tooling** when possible.

- Use **Web Components** (Custom Elements, Shadow DOM)
- Use **ES Modules** (`import()` at runtime)
- Avoid over-reliance on proprietary loaders or bundlers

### Build and Deploy Independently

> Each microfrontend should be **independently developed, tested, and
> deployed**.

- Teams manage their own pipelines
- One MF can be updated without rebuilding the whole app
- Use **CDN links**, **Module Federation**, or **import maps**

### Composition at Runtime or Build Time

- **Runtime composition**: Shell dynamically loads MFs (e.g., via `import()`,
  Module Federation)
- **Build-time composition**: All MFs bundled together by CI/CD

### Routing Isolation

Each MF should ideally:

- Handle its own **sub-route** (e.g., `/dashboard/*`)
- Use **mount/unmount** lifecycle hooks
- Or integrate with a shared **router in the host app**

### Shared Design System (Optional)

To ensure consistency:

- Use a shared **UI library** (e.g., design tokens, buttons, theming)
- But **don't force full reusability** — allow freedom

## Core Concepts

| Concept                  | Description                                                                 |
| ------------------------ | --------------------------------------------------------------------------- |
| **Micro App**            | A self-contained frontend unit (UI + logic)                                 |
| **Shell/Host/Container** | The main app that loads and orchestrates micro apps                         |
| **Integration Method**   | How micro apps are combined: iframe, Web Components, Module Federation, etc |

## How Microfrontends Work (Plain JS Example)

Let's use the **Web Components** approach, which is framework-agnostic and
browser-native.

We'll create:

1. A **host shell** (`index.html`)
2. Two **microfrontends** as web components (`mf-header.js`, `mf-products.js`)

### Folder Structure

```
/microfrontend-demo
│
├── index.html            <-- Shell app
├── mf-header.js          <-- Microfrontend 1: Header
└── mf-products.js        <-- Microfrontend 2: Product list
```

### `mf-header.js`

```javascript
class MFHeader extends HTMLElement {
  connectedCallback() {
    this.innerHTML = `
      <header style="background:#333;color:white;padding:10px;">
        <h1>My Shop</h1>
      </header>
    `;
  }
}
customElements.define("mf-header", MFHeader);
```

### `mf-products.js`

```javascript
class MFProducts extends HTMLElement {
  connectedCallback() {
    this.innerHTML = `
      <section>
        <h2>Products</h2>
        <ul>
          <li>Apples</li>
          <li>Bananas</li>
          <li>Oranges</li>
        </ul>
      </section>
    `;
  }
}
customElements.define("mf-products", MFProducts);
```

### `index.html` (Shell App)

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Microfrontend Demo</title>
  </head>
  <body>
    <!-- Microfrontends loaded as components -->
    <mf-header></mf-header>
    <main style="padding:10px;">
      <mf-products></mf-products>
    </main>

    <!-- Load microfrontends -->
    <script type="module" src="./mf-header.js"></script>
    <script type="module" src="./mf-products.js"></script>
  </body>
</html>
```

## Composition in Microfrontends

### Build-Time Composition

> All microfrontends are compiled and bundled together during the build.

- Usually done with **monorepos**
- Shared build pipeline (e.g., Nx, Turborepo)
- Strong integration, tighter coupling

#### Pros

- Easier local development
- Simplified performance optimization

#### Cons

- No independent deployment
- Teams must coordinate builds

#### Use When

- Teams are small or release together
- Strong cross-MF contracts

### Runtime Composition (Client-Side)

> The shell app loads each microfrontend dynamically in the browser.

#### Common tools

- `import()` (dynamic ES module loading)
- **Web Components**
- **Webpack Module Federation**
- **Single-SPA**

#### Pros

- Microfrontends are truly decoupled
- Teams deploy independently
- Better for scalability

#### Cons

- More complex setup
- Performance & caching concerns
- JS-heavy (more hydration time)

#### Use When

- Teams own full CI/CD
- Features update independently

#### Example: Module Federation (Client-Side)

```javascript
// Host app webpack config
remotes: {
  dashboard: 'dashboardApp@http://localhost:3001/remoteEntry.js',
}
```

- Host app loads `dashboard` at runtime
- Dashboard is deployed independently

### Runtime Composition (Server-Side)

> The server assembles microfrontend fragments into a full HTML response.

#### Techniques

- **Edge-side includes (ESI)**
  ```html
  <html>
    <body>
      <esi:include src="/header.html" />
      <h1>Welcome to our site!</h1>
      <esi:include src="/promo.html" />
      <esi:include src="/footer.html" />
    </body>
  </html>
  ```
- **Server-side includes (SSI)**
- **Fragment rendering via microservices**
- SSR with **Next.js / Angular Universal**

#### Pros

- Fast initial page load (SSR)
- Can cache individual fragments
- Better SEO

#### Cons

- Tighter server dependencies
- Deployment coordination still needed
- Server-side error handling is trickier

#### Use When

- You need **high performance + SEO**
- Prefer **monolithic SSR** but want MFE benefits

### Iframe Composition (Legacy / Strict Isolation)

Each microfrontend runs in an `iframe`.

#### Pros

- Strong sandboxing
- Easy integration

#### Cons

- Poor UX
- Slower performance
- Hard to share data/state

#### Use When

- Strict isolation is required (e.g., plugins, admin panels)

## Interview Questions

### What is a Microfrontend architecture?

Microfrontend is an architectural style where a frontend app is divided into
**independent, self-contained micro-apps**, each owned by a different team and
potentially built with different technologies. It mirrors microservices on the
frontend.

### What are the main integration methods for microfrontends?

- **Iframes** – Easy isolation, poor UX and performance.
- **Web Components** – Native browser support, framework-agnostic.
- **Module Federation (Webpack 5)** – Dynamic runtime loading and shared
  dependencies.
- **Single SPA** – Framework for orchestrating micro-apps. It is a JavaScript
  framework for building microfrontends. It allows you to combine multiple
  frameworks (React, Angular, Vue, etc.) into one single frontend application,
  all living together on the same page.

### What are the main benefits of using microfrontends?

- Independent deployments
- Team autonomy
- Scalability
- Technology diversity
- Better code ownership and separation of concerns

### What are the challenges of microfrontends?

- Shared state management across apps
- Performance overhead from multiple bundles
- Routing coordination
- Consistent styling
- Dependency duplication and version mismatches

### How do you share data between microfrontends?

- Using **custom events** (in Web Components)
- Shared **global state** (e.g., Redux or Zustand in a shared module)
- **Props** passed down via the host app
- **Browser storage** (e.g., `localStorage`, `sessionStorage`)
- **PostMessage** or **Pub/Sub systems**

### How can you ensure consistent UI across different microfrontends?

- Share a **design system or component library**
- Enforce common **CSS reset/base styles**
- Use **CSS modules** or **Shadow DOM** to avoid style leakage
- Use **theming tokens** and share them across apps

### What is Webpack Module Federation and how is it used in microfrontends?

Webpack 5's Module Federation allows apps to **expose and consume modules at
runtime** without bundling them together. It enables true **runtime
composition** of microfrontends and shared libraries.

Example use:

```javascript
// shell app
exposes: {
  './Header': './src/Header',
}
```

### How would you implement microfrontends using Web Components?

- Create each micro app as a **custom element** using the `CustomElements` API
- Register with `customElements.define()`
- Host app loads them via `<script>` or `import()` and place `<mf-header>` in
  the DOM

### How can microfrontends handle routing?

Options:

- Use **centralized routing** in the host app and load micro-apps based on route
- Let each micro-app handle **its own internal routing**
- Use tools like **Single SPA**, which support both global and app-level routing

### When would you NOT recommend microfrontends?

- For **small teams** or **simple applications** — too much complexity
- When you don't need **independent deployments**
- If you need **tight performance** and **minimal bundle size**
- If all teams already share the same release cycle

---

## SPA (Single Page Application)

A **Single Page Application** is a web app that loads a **single HTML page** and
dynamically updates the content using **JavaScript** without refreshing the
page.

Instead of fetching new HTML from the server for each page, a SPA fetches data
via **AJAX/Fetch** or uses **APIs** (often REST or GraphQL) and updates the DOM
using a framework (like Angular, React, or Vue).

### How it works

- Initial load -> downloads `index.html`, JS, CSS
- Then -> uses `fetch()` or `XMLHttpRequest` to get data
- Updates the view using JS (client-side rendering)

### Pros and Cons of SPA

| Benefit                | Description                               |
| ---------------------- | ----------------------------------------- |
| Fast UX                | No full-page reloads; faster interactions |
| Mobile-like feel       | Smooth, app-like behavior                 |
| Decoupled backend      | Works well with REST/GraphQL APIs         |
| Reusable UI components | Good fit for component-based frameworks   |

| Drawback                 | Description                                           |
| ------------------------ | ----------------------------------------------------- |
| SEO Challenges           | Harder for search engines to index unless SSR is used |
| Initial Load Time        | Larger JS bundles can slow first load                 |
| JS Dependency            | Breaks if JS fails or is blocked                      |
| Browser History Handling | Requires client-side routing (e.g., `pushState`)      |
| Security                 | More surface area for XSS if not coded carefully      |

## Fat vs Thin Client and Server

| Concept                 | Thin Client               | Fat Client                            |
| ----------------------- | ------------------------- | ------------------------------------- |
| **Who does rendering?** | Server                    | Client                                |
| **Who handles logic?**  | Server                    | Mostly client                         |
| **Network usage**       | Sends HTML                | Sends JSON/API data                   |
| **Examples**            | Classic PHP/.NET web apps | SPA (Angular, React)                  |
| **Offline use**         | Rare                      | Possible with caching/service workers |

## Interview Questions with Answers

### How does routing work in a SPA?

SPAs use the **History API** (`pushState`, `popstate`) to update the URL without
reloading the page. The router then dynamically loads or displays views based on
the URL.

### How do you improve SEO in a SPA?

- Use **Server-Side Rendering (SSR)** (e.g., Angular Universal, Next.js)
- Use **pre-rendering** or **static site generation**
- Ensure proper **meta tags** and **structured data**
- Add **sitemap.xml** and use **robots.txt** properly

### Can you use SPAs offline? How?

Yes, using **Service Workers** and **Caching** (via PWA techniques), SPAs can
work offline or partially offline. This includes caching assets and fallback
responses.

---

## Isomorphic Application

An **isomorphic (or universal) application** is a JavaScript app that can **run
both on the client and on the server** — using the **same codebase**.

### Key Idea

- The **server** renders the first HTML view -> fast load, SEO friendly.
- Then the **client** takes over (hydration) -> adds interactivity with
  JavaScript.

This is common in frameworks like **Next.js**, **Nuxt.js**, or **Angular
Universal**.

### Pros and Cons of Isomorphic Apps

| Benefit         | Why it matters                         |
| --------------- | -------------------------------------- |
| Fast first load | Server renders HTML instantly          |
| SEO friendly    | Crawlers see real content, not just JS |
| Code reuse      | Same code on server and client         |
| Better UX       | Smooth transitions after hydration     |

| Drawback               | Why it’s tricky                             |
| ---------------------- | ------------------------------------------- |
| More complex           | Must handle both server and client contexts |
| Server setup needed    | Need Node.js to run server-side JS          |
| Hydration bugs         | Mismatched DOMs cause warnings or errors    |
| Bundle size management | Must separate client-only code carefully    |

---

## JAMstack

**JAMstack** stands for:

- **J**avaScript — handles dynamic functionality (frontend logic)
- **A**PIs — reusable, stateless services (e.g., REST, GraphQL)
- **M**arkup — prebuilt HTML, often generated at build time using tools like
  Hugo, Jekyll, or Next.js

> JAMstack isn't a framework — it's a **web architecture pattern** focused on
> **decoupling** the frontend from the backend.

JAMstack Apps Are:

- **Pre-rendered**: HTML is generated at build time
- **Static-first**: Served via CDN for speed and scalability
- **API-driven**: Dynamic content comes from APIs (e.g., Stripe, Auth0,
  Contentful)
- **Frontend-focused**: Everything starts from the client

### Pros and Cons of JAMstack

| Benefit                 | Why it matters                                    |
| ----------------------- | ------------------------------------------------- |
| Blazing fast            | Content is pre-rendered and served from CDN       |
| Scalable                | CDNs handle traffic, not your server              |
| Secure                  | No backend = fewer attack surfaces                |
| Developer friendly      | Decoupled — use best tools for each piece         |
| Works with Headless CMS | Contentful, Sanity, etc. plug right in            |
| Cheap hosting           | Static hosting (Netlify, Vercel) is free or cheap |

| Drawback           | Why it's tricky                                              |
| ------------------ | ------------------------------------------------------------ |
| Build complexity   | Build-time generation gets complex for large/real-time sites |
| Not truly dynamic  | Real-time content needs client-side JS or revalidation       |
| Over-fetching APIs | Many client-side calls can bloat performance                 |
| SEO challenges     | When content is loaded client-side only                      |

---

## Module Federation

**Module Federation** is a **Webpack 5 feature** that allows multiple apps to
**share and load modules at runtime**, even across different deployments.

In Angular, it enables **true microfrontends** by allowing:

- Independent builds/deployments
- Lazy loading remote components
- Shared libraries (like Angular, RxJS, Material)
- Communication between micro apps

## Architecture

We'll create:

- **Host App (Shell)**: Loads microfrontends
- **Remote App (MFE1)**: Exposes a module/component
- Communication: via shared services or custom event bus

## Setup Summary

We’ll use Angular CLI + Webpack 5 + Module Federation Plugin.

Assume:

- `shell` on port 4200
- `mfe1` on port 4300

## 1. Install Dependencies

In both projects:

```bash
ng add @angular-architects/module-federation
```

This sets up `webpack.config.js` and `webpack.prod.config.js` automatically.

## 2. Configure `mfe1`

`projects/mfe1/webpack.config.js`:

```javascript
const {
  withModuleFederationPlugin,
} = require("@angular-architects/module-federation/webpack");

module.exports = withModuleFederationPlugin({
  name: "mfe1",
  exposes: {
    "./Module": "./src/app/feature/feature.module.ts",
  },
  shared: {
    "@angular/core": { singleton: true, strictVersion: true },
    "@angular/common": { singleton: true, strictVersion: true },
    "@angular/router": { singleton: true, strictVersion: true },
  },
});
```

## 3. Configure `shell`

`shell/webpack.config.js`:

```javascript
const {
  withModuleFederationPlugin,
} = require("@angular-architects/module-federation/webpack");

module.exports = withModuleFederationPlugin({
  remotes: {
    mfe1: "mfe1@http://localhost:4300/remoteEntry.js",
  },
  shared: {
    "@angular/core": { singleton: true, strictVersion: true },
    "@angular/common": { singleton: true, strictVersion: true },
    "@angular/router": { singleton: true, strictVersion: true },
  },
});
```

## 4. Lazy Load Remote Module in Shell

In `app.routes.ts` of Shell:

```typescript
{
  path: 'feature',
  loadChildren: () =>
    import('mfe1/Module').then(m => m.FeatureModule)
}
```

This lazy-loads `FeatureModule` from `mfe1`.

## 5. Communication Between Apps

### Option A: Shared Service (via `shared` in Webpack)

Create a service in a shared library and mark it as `singleton`.

```typescript
@Injectable({ providedIn: "root" })
export class MessageBusService {
  private subject = new Subject<string>();
  messages$ = this.subject.asObservable();

  send(msg: string) {
    this.subject.next(msg);
  }
}
```

Use the same service in both apps (via `shared` config), and you can send
messages like a pub-sub.

### Option B: `window.postMessage`

For looser coupling, use:

```typescript
// Send
window.postMessage({ type: "event", payload: "hello" }, "*");

// Listen
window.addEventListener("message", (event) => {
  if (event.data.type === "event") {
    console.log(event.data.payload);
  }
});
```

## Test

1. Run `mfe1`:

```bash
ng serve mfe1 --port 4300
```

2. Run `shell`:

```bash
ng serve shell --port 4200
```

3. Visit `http://localhost:4200/feature`

You’ll see the remote Angular module/component from `mfe1` loaded **lazily at
runtime** via Webpack Module Federation.
