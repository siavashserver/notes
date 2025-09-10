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

## Benefits of Adopting Microfrontends

**Microfrontends** address many critical challenges of modern web development.
The most tangible benefits, consistently documented in both industry and
academic literature, are:

- **Scalability:** Teams and features grow independently without increasing
  overall system complexity.
- **Team Autonomy:** Developers take full ownership of features, fostering
  accountability and enabling parallel workstreams.
- **Continuous and Independent Deployment:** Modules can be released, upgraded,
  or rolled back without impacting the broader system—enabling rapid innovation
  cycles.
- **Technology Diversity:** Freedom to choose the most appropriate tool or
  framework per module; experimentation without risking the stability of the
  integrated experience.
- **Incremental Upgrades and Migration:** Legacy applications can be modernized
  feature by feature, minimizing risk and “big-bang” rewrites.
- **Fault Isolation:** Issues or failures in one module rarely impact the entire
  application, increasing system resilience and reliability.
- **Faster Time-to-Market:** Modular development and testing pipelines reduce
  bottlenecks common in monolithic codebases, allowing new features and bug
  fixes to be delivered more quickly.

## Monolithic Frontend vs Microfrontend Architectures

### Monolithic Frontend Architecture

Traditional **monolithic frontends** are developed, tested, and deployed as a
single, unified codebase. All features, dependencies, and UI components reside
in one project or repository, managed by a centralized team or set of teams.

**Key characteristics of monoliths:**

- All UI logic, state, and styling are tightly coupled.
- Technology stack remains uniform across the application.
- Scaling requires scaling the entire app, regardless of which parts experience
  increased demand.
- Introduction of new features, upgrades, or bug fixes often risks affecting
  unrelated parts of the application.
- Coordination overhead increases as more developers and teams are added.

**Advantages:**

- Simple to develop and manage for small projects.
- Lower upfront complexity.

**Disadvantages:**

- Difficult to scale development when multiple teams are involved.
- Risk of bottlenecks and merge conflicts.
- Slower innovation cycles and deployment times when updates are queued behind
  unrelated changes.
- Harder to incrementally modernize; upgrades often require large rewrites.

### Microfrontend Architecture

Conversely, **microfrontend architectures** divide the UI into independently
deployable modules, each responsible for one feature or business domain. Each
module is owned end-to-end—code, infrastructure, deployment—by a single team.

**Key characteristics:**

- Modules are developed, deployed, and tested independently.
- Teams choose their tech stacks, tooling, and processes.
- Applications grow organically; legacy monoliths can be “strangled” piece by
  piece into microfrontends.
- Clear ownership reduces cross-team conflicts.
- Teams and features can scale horizontally, in parallel architectures.

**Advantages:**

- Rapid iteration and experimentation by small, focused teams.
- Technology flexibility—the application can evolve technology stacks
  incrementally.
- Incremental upgrades and modernization.
- Failure isolation; issues in one module are less likely to bring down the
  whole UI.
- Easier legacy migration; new features or pages are delivered as new
  microfrontends without touching existing monolithic parts.

**Disadvantages:**

- Higher operational and governance overhead.
- Potential for inconsistent user experience without strong central design
  systems.
- More complex inter-module communication requirements.

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

#### HTML Templates

The `<template>` element allows hidden, inert chunks of HTML markup to be
defined and instantiated by JavaScript at runtime. This defers rendering and
enables dynamic, repetitive UI creation. Templates are pivotal for defining
reusable content skeletons in custom element development.

```html
<template id="card-template">
  <div class="card">
    <h2 class="title"></h2>
    <p class="content"></p>
  </div>
</template>

<script>
  const template = document.getElementById("card-template");
  const clone = template.content.cloneNode(true);
  clone.querySelector(".title").textContent = "Hello, Template!";
  clone.querySelector(".content").textContent =
    "This content was cloned from a template.";
  document.body.appendChild(clone);
</script>
```

#### Custom Elements API

Developers can define new HTML element types (autonomous or as extensions of
built-ins), encapsulating logic, state, and style. Elements include lifecycle
callbacks for attachment, removal, and attribute mutation, supporting
sophisticated UI patterns with native support in modern browsers.

```html
<script>
  class MyGreeting extends HTMLElement {
    connectedCallback() {
      this.textContent = `Hello, ${this.getAttribute("name") || "World"}!`;
    }
  }

  customElements.define("my-greeting", MyGreeting);
</script>

<my-greeting name="David"></my-greeting>
```

#### Shadow DOM

Enables encapsulation of a subtree and styles within a host element,
guaranteeing that styles and structure are isolated from the rest of the page
and preventing global CSS/JS conflicts. Shadow roots can be open (accessible
from the host) or closed (hidden), facilitating robust composability and
abstraction for design systems and micro-frontends.

```html
<script>
  class FancyButton extends HTMLElement {
    constructor() {
      super();
      const shadow = this.attachShadow({ mode: "open" });
      shadow.innerHTML = `
        <style>
          button {
            background: royalblue;
            color: white;
            padding: 10px;
            border: none;
            border-radius: 5px;
          }
        </style>
        <button>Click Me</button>
      `;
    }
  }

  customElements.define("fancy-button", FancyButton);
</script>

<fancy-button></fancy-button>
```

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

Design systems encapsulate reusable UI patterns, component libraries, and
standardized style guides, enforcing consistency and scalability in multi-team,
component-driven development. Advantages include:

- Centralized decision-making (styles, tokens)
- Framework-agnostic adoption (web components)
- Improved designer–developer collaboration and handoff
- Automatic enforcement of corporate branding and accessibility

## Routing and State Management Design Patterns

**Best Practice:** Prefer state and routing isolation, only exposing
well-defined interfaces for cross-module communication. Use design systems and
shared libraries only when necessary, and version shared dependencies to limit
upgrade risk.

### Routing

- Typically orchestrated at the container level, with child microfrontends
  controlling routing within their scope.
- Strategies include passing the navigation object (e.g., React Router's
  history), using location-based parameters, or event buses to notify modules to
  render based on URL changes.
- Technologies like single-spa or Luigi provide advanced, configuration-driven
  routing for multi-module apps.

### State Management

- Each microfrontend maintains its own state (e.g., Redux store).
- Shared state is generally avoided; when necessary, state is passed via global
  events, shared services, localStorage, or APIs.
- Event-based communication (pub-sub, browser events) helps reduce tight
  coupling.
- Common interface patterns include custom DOM events, message-passing
  (especially for iframes), or integrations via backend APIs.

## Best Practices for Implementing Microfrontends

- **Define Clear Boundaries:** Each microfrontend should have a well-defined,
  functionally cohesive domain to minimize integration complexity and maximize
  maintainability.
- **Consistent Design Systems:** Establish a shared design system (e.g., with
  Storybook, Design Tokens) and style guide to ensure visual and UX consistency
  across microfrontends, despite the potential use of multiple frameworks.
- **API-First Contracts:** Standardize communication and data exchange through
  explicit, versioned API contracts.
- **Version Control and Dependency Management:** Use tools like import maps or
  semantic versioning; maintain backward compatibility across shared modules and
  expose APIs for communication.
- **Automated Testing & CI/CD:** Each microfrontend requires its own automated
  tests (unit, integration, e2e) and independent CI/CD pipelines. This supports
  fast, autonomous releases and minimizes risk.
- **Lazy Loading:** Load microfrontends only when required (on navigation,
  feature requirement), reducing initial bundle sizes and improving performance.
- **Minimize Shared State:** Avoiding global state across microfrontends reduces
  coupling; state is either isolated or shared through backend APIs or
  well-documented event systems.
- **Performance Optimization:** Deduplicate dependencies, use caching, and
  monitor overall payload size.
- **Governance:** Guard against architectural and UX anarchy by codifying
  contribution and design standards, and appoint platform or design system
  maintainers.
- **Security:** Isolate microfrontends via sandboxing or strict CSP, manage
  cross-origin asset loading, audit dependencies, and enforce secure coding
  guidelines.

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

> The server assembles microfrontend fragments into a full HTML response. A
> fragment is an isolated embeddable mini-app in page.

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

### Web Components and Custom Elements

These are natively supported HTML elements, leveraging the Shadow DOM for style
and DOM isolation. Teams wrap their feature modules as custom elements
(`<my-checkout-widget>`) which the container injects into the DOM.

### Single SPA (single-spa)

This meta-framework lets each microfrontend be built with a different stack.
Single-spa provides orchestration by registering and dynamically mounting the
microfrontends against routes or other activation events.

### Module Federation (Webpack 5)

Webpack's Module Federation feature allows multiple projects to share code and
dynamically import modules from one another at runtime, facilitating independent
deployment with shared dependencies.

## Build-Time vs. Runtime Integration Strategies

### Build-Time Integration

- Microfrontend modules are aggregated at build time, resulting in a single,
  optimized bundle.
- Ensures consistent dependencies and styling.
- However, any update requires the whole app to be rebuilt and redeployed;
  reduced autonomy and flexibility.

### Runtime Integration

- Microfrontends are loaded on-demand by the container from independently
  deployed sources.
- Supports independent deployment, flexible technology choices, and incremental
  rollout.
- Common methods: Module Federation, Single-SPA, Web Components, SystemJS,
  dynamic script tags, or import maps.
- Requires careful management of runtime dependencies (e.g., avoiding duplicate
  React bundles) and consistent inter-module communication contracts.

| Factor                   | Build-Time Integration       | Runtime Integration            |
| ------------------------ | ---------------------------- | ------------------------------ |
| Deployment flexibility   | Lower                        | Higher                         |
| Performance              | Higher (single bundle)       | Potentially lower (many files) |
| Independent updates      | Poor                         | Excellent                      |
| Risk of version conflict | Low                          | Medium-High                    |
| Use case                 | Small projects, stable teams | Large, dynamic applications    |

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

### What is Shared Library Optimization in microfrontends?

Module Federation (Webpack 5+) solves the duplication of shared dependencies in
microfrontend (MFE) architectures. Only one copy of a singleton dependency (such
as React or Redux) is loaded, managed via federation-specific shared API
registries and workspace-level policies (e.g., Nx SVP recommends enforcing a
single version for team consistency and API compatibility). This eliminates
runtime conflicts, reduces bundle size, and maintains shared library integrity
despite decentralized feature team deployments.

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

---

## Custom Elements API

The **CustomElements API** lets developers:

- Define entirely new HTML tags (autonomous custom elements)
- Extend existing elements (customized built-in elements, with browser
  limitations)
- Associate ES6 classes with HTML tags, giving each DOM instance powerful,
  class-based logic, state, and reactivity
- Utilize lifecycle hooks for reacting to changes in the element's connection,
  attributes, or parent document
- Integrate with other DOM APIs, events, and even frameworks thanks to the
  standards-based approach

### Types of Custom Elements

| Type                        | Base Class              | Example Tag Syntax        | Extends Native? | Browser Support\*   |
| --------------------------- | ----------------------- | ------------------------- | --------------- | ------------------- |
| Autonomous Custom Element   | HTMLElement             | `<my-card></my-card>`     | No              | Universal (modern)  |
| Customized Built-in Element | HTMLButtonElement, etc. | `<button is="fancy-btn">` | Yes             | Limited (see below) |

> \*Browser support: Custom built-in elements are not supported in Safari, and
> their practical use is niche. The mainstream practice is to use **autonomous
> custom elements** wherever possible for maximum compatibility and flexibility.

#### Autonomous Custom Elements

- **Definition**: Inherit directly from `HTMLElement`, introducing entirely new
  tags and syntax.
- **Usage**: `<my-dialog>`, `<todo-list>`, or any unique tag containing a
  hyphen.
- **Full power of the API**: All lifecycle hooks and encapsulation strategies
  are available.

```javascript
// Autonomous Custom Element
class MyCard extends HTMLElement {
  // ...custom logic...
}
customElements.define("my-card", MyCard);
```

#### Customized Built-in Elements

- **Definition**: Extend from an existing native element interface (e.g.,
  `HTMLButtonElement`) to specialize its behavior.
- **Usage**: `<button is="rounded-button">`
- **Syntax**: Custom element name via the `is` attribute, not the tag name.
- **Browser Limitation**: Not supported in Safari and largely discouraged except
  for specialist legacy cases.

```javascript
// Customized Built-in Element (limited support)
class FancyButton extends HTMLButtonElement {
  // ...custom logic...
}
customElements.define("fancy-btn", FancyButton, { extends: "button" });
// Usage: <button is="fancy-btn"></button>
```

### Defining and Registering Custom Elements

#### Defining a Custom Element

```javascript
class MyGreeting extends HTMLElement {
  constructor() {
    super();
    // Initial state setup, event binding, etc.
  }
  connectedCallback() {
    this.innerHTML = "<p>Hello, Custom Elements!</p>";
  }
}

customElements.define("my-greeting", MyGreeting); // Registration
```

```html
<my-greeting></my-greeting>
```

#### The Registration Process: customElements.define

```javascript
customElements.define(name, constructor, options);
```

- **name (string):** Must include a hyphen (e.g., `my-element`).
- **constructor (class):** The JavaScript class implementing the element
- **options (object, optional):** Used for customized built-in elements only;
  e.g., `{ extends: 'button' }`.

##### Registration Rules and Errors

1. **The name must include a hyphen.** This avoids future conflicts with native
   tags.
2. **Multiple registrations with the same name are not allowed.** Re-registering
   throws an error.
3. **Registration must occur before elements of that type are present in DOM.**
   Otherwise, those pre-existing nodes are not upgraded (i.e., they act as
   unknown elements until registration).
4. **Registration order & timing matter:** Careful initialization sequencing
   prevents race conditions in complex apps.

### Core Class Methods for Custom Elements

- A **constructor** (ES6 class constructor)
- Lifecycle hooks: `connectedCallback`, `disconnectedCallback`,
  `attributeChangedCallback`, and `adoptedCallback`
- Static getter for `observedAttributes`
- Additional application-specific methods and property accessors

```javascript
class MyElement extends HTMLElement {
  constructor() {
    /* ... */
  }
  connectedCallback() {
    /* ... */
  }
  disconnectedCallback() {
    /* ... */
  }
  static get observedAttributes() {
    return ["foo", "bar"];
  }
  attributeChangedCallback(name, oldValue, newValue) {
    /* ... */
  }
  adoptedCallback() {
    /* ... */
  }
  // Other methods, event handlers, accessors, etc.
}
```

> **Note:** All lifecycle hooks are optional except the constructor, but at
> least `connectedCallback` is almost always used for element setup.

### Lifecycle Callbacks

| Method                    | Triggered When...                                         | Common Use Cases                                             |
| ------------------------- | --------------------------------------------------------- | ------------------------------------------------------------ |
| constructor               | Element is created or upgraded                            | Initialize internal state, event bindings, create shadow DOM |
| connectedCallback         | Element inserted into or moved within the document        | Setup DOM/UI, data fetching, event listeners                 |
| disconnectedCallback      | Element removed from the document                         | Tear-down, cleanup, remove event listeners                   |
| static observedAttributes | Specifies which attributes to monitor                     | (Not called directly; enables attribute callbacks)           |
| attributeChangedCallback  | Monitored attribute is changed, added, removed, replaced  | Respond to attribute changes (e.g., update UI/state)         |
| adoptedCallback           | Element moved to a new document (e.g., via `adoptNode()`) | Adjust logic for cross-document context                      |

#### constructor

Called when the element is instantiated (via parser or
`document.createElement`). Useful for setting up initial state, default values,
and event handler bindings.

- Must always call `super()` as the first statement.
- **Do not interact with attributes or child elements** in the constructor—they
  may not yet exist or be set by the parser.
- Adding attributes and making DOM modifications should be delayed to
  `connectedCallback`.

```javascript
class HelloPopup extends HTMLElement {
  constructor() {
    super();
    this._isOpen = false;
    // Declared state property, but defer DOM to connectedCallback
  }
}
```

#### connectedCallback

**Invoked every time** the custom element is inserted into the document's DOM or
moved within it. This is the preferred place to set up DOM, register event
listeners, fetch data, and update rendering.

- Can be called multiple times during an element's lifecycle.
- DOM availability is guaranteed, but child custom elements may not be fully
  upgraded yet (take care accessing children).
- Use for any interactivity tied to presence in the document, not for one-time
  initializations (those belong in the constructor).

```javascript
connectedCallback() {
  this.innerHTML = `
    <button>Click me</button>
  `;
  this.querySelector('button').addEventListener('click', this._handleClick);
}
```

> **Note:** Cleanup should be paired with `disconnectedCallback`.

#### disconnectedCallback

Called each time the element is **removed** from the document, either by DOM
manipulation or page navigation.

- Used to remove event listeners, stop timers, or clean up resources to prevent
  memory leaks.
- May be called multiple times if an element is repeatedly inserted and removed
  from the DOM.

```javascript
disconnectedCallback() {
  this.querySelector('button').removeEventListener('click', this._handleClick);
}
```

#### static get observedAttributes()

A static getter that returns an array of **attribute names** to be observed for
changes. Only attributes listed here will trigger `attributeChangedCallback`
when modified.

```javascript
static get observedAttributes() {
  return ['color', 'size'];
}
```

This selective monitoring improves performance and avoids unnecessary callback
invocations.

#### attributeChangedCallback

Called whenever one of the `observedAttributes` is added, removed, or changed
(including via JavaScript `setAttribute()`/`removeAttribute()` or direct
attribute manipulation).

- Highly useful for re-rendering the element when attribute-based configuration
  or state changes.
- Attributes are always passed as strings (JSON encoding/decoding is needed for
  object values, if needed).

```javascript
attributeChangedCallback(name, oldValue, newValue) {
  if (oldValue !== newValue) {
    if (name === 'color') {
      this.style.color = newValue;
    }
  }
}
```

For more advanced usage, map attributes to properties and keep them in sync.

#### adoptedCallback

This rarely-used callback triggers when the custom element is moved into a **new
document** via methods like `document.adoptNode()`. Practical use cases include
moving elements between iframes or handling dynamic imports with unique document
contexts.

```javascript
adoptedCallback() {
  // E.g., reinitialize templates, listeners, or resources tied to a specific document
}
```

Most applications will never need adoptedCallback, but multi-document scenarios
benefit from it.

### Handling Attributes and Element Properties

A subtle but crucial distinction exists between **attributes** (defined in
markup or via `setAttribute`) and **properties** (JS object fields on the
element).

| Aspect     | Attributes                                                               | Properties                                 |
| ---------- | ------------------------------------------------------------------------ | ------------------------------------------ |
| Definition | Written in HTML as `<tag attr="v">`                                      | JS-accessible fields, e.g., `el.value`     |
| Value type | Always a string                                                          | Any type (string, object, function, array) |
| Sync?      | Initial parse: attribute values reflected as properties where applicable | Not always, must be manually reflected     |

#### Best Practices

- Use **attributes** for declarative, serializable configuration (HTML).
- Use **properties** for dynamic, programmatic interaction.
- Manual reflection between the two helps maintain state consistency.

#### Syncing Attributes and Properties

```javascript
class CountingButton extends HTMLElement {
  static get observedAttributes() {
    return ["count"];
  }
  get count() {
    return Number(this.getAttribute("count"));
  }
  set count(val) {
    this.setAttribute("count", val);
  }
  attributeChangedCallback(name, oldValue, newValue) {
    if (name === "count") {
      this._updateButton();
    }
  }
  _updateButton() {
    this.textContent = `Clicked ${this.count} times`;
  }
}
customElements.define("counting-button", CountingButton);
```

This example illustrates how **custom elements can expose ergonomic
getters/setters** for working with properties, while also syncing those with
HTML attributes for interoperability and reactivity.

### Integrating Custom Elements into the DOM

#### Programmatic Creation

```javascript
const myEl = document.createElement("my-greeting");
document.body.appendChild(myEl);
```

#### Moving and Removing Elements

- Insertion via `appendChild`, `insertBefore`, etc., triggers
  `connectedCallback`.
- Removal via `removeChild`, `remove()`, etc., triggers `disconnectedCallback`.

Custom elements are natively part of the DOM event flow and can be targeted,
queried, or iterated as with any element.

#### Slots and Content Projection

Shadow DOM (next section) combined with `<slot>` allows custom elements to
accept and render child content, maintaining flexibility and composability for
complex UI structures.

### Using Shadow DOM with Custom Elements

The **Shadow DOM** is an essential feature for encapsulating the internal DOM
and CSS of custom elements, preventing style and markup leaks both in and out of
the component.

#### Attaching a Shadow Root

Call `attachShadow({ mode: 'open' })` (or `'closed'`) in the constructor of your
element class:

```javascript
class ShadowBox extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: "open" });
  }
}
```

- **`mode: 'open'`** exposes the `shadowRoot` property for direct access (e.g.,
  `el.shadowRoot`).
- **`mode: 'closed'`** keeps the shadow root inaccessible outside the class.

#### Using Shadow DOM for Markup and Style Encapsulation

```javascript
class StyledTextarea extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: "open" });
    this.shadowRoot.innerHTML = `
      <style>
        textarea { color: #336699; border-radius: 8px; padding: 8px; }
      </style>
      <textarea></textarea>
    `;
  }
}
customElements.define("styled-textarea", StyledTextarea);
```

Shadow DOM ensures that the textarea’s CSS and structure won’t “leak” into the
global document, nor will outside styles affect it—enabling true component
isolation. Use `<slot>` tags within your shadow root to facilitate content
projection from outside the element for maximum flexibility.
