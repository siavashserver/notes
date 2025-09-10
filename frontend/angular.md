---
title: Angular
---

## Introduction

Angular is a **TypeScript-based**, open-source, component‑based framework
developed by Google.

Key features include:

- **Component-based architecture** for isolated, reusable UI units
- Rich **data binding**: interpolation, property/event binding, and ngModel
- Built-in **dependency injection** system promoting modularity and testability
- **Directives** (structural vs. attribute), **pipes**, and robust **routing**
- Compilation strategies like **AOT**, **Ivy**, and **tree shaking** for
  optimised performance

---

## Service Injection Methods

### Constructor Injection (Traditional)

The most common approach—inject services by declaring them in the constructor.

- Angular uses TypeScript metadata to know what to pass based on the token type.
- Works for components, services, directives, and more.

```typescript
constructor(private authService: AuthService) {}
```

#### Pros

- Dependencies are explicit and easy to see.
- Well-supported in tooling and documentation.

#### Cons

- If you extend a base class that requires injection, subclasses must also
  declare and pass in parameters (via `super(...)`)

### `inject()` (Angular v14+)

Angular introduced the `inject()` function, usable in class fields or standalone
functions.

- Supports injecting services without the constructor.
- Ideal for base classes or mixins to avoid boilerplate in subclasses.

```typescript
private userService = inject(UserService);
```

#### Pros

- Cleaner code when extending classes.
- Can be used in functions—not only constructors.
- In TypeScript 6+, avoids issues with `useDefineForClassFields` behavior.

#### Cons

- Makes dependencies less explicit—can obfuscate what's injected, complicating
  tests or refactoring.
- Should be used carefully to avoid the Service Locator anti-pattern.

### Injector-based (Runtime Retrieval)

You can inject the Angular `Injector` and call `.get()` to fetch dependencies
dynamically.

- Used when injection needs to happen at runtime or lazily (e.g. in
  `APP_INITIALIZER`).

```typescript
constructor(private injector: Injector) {
  this.configService = this.injector.get(ConfigService);
}
```

#### Pros

- Allows deferred injection based on runtime conditions.

#### Cons

- Dependencies are hidden and scattered.
- Breaks static analysis of dependencies.
- Not idiomatic and can reduce readability.

### Setter Injection

Use a setter method to inject services after instantiation.

- Useful for optional dependencies or dependencies that may change at runtime.

```typescript
setService(dep: SomeService) {
  this.dep = dep;
}
```

### Interface Injection

Define a method in an interface and implement it to receive dependencies.

- Rarely used in Angular but aligns with SOLID OOP principles.

```typescript
interface DepInjector {
  inject(dep: MyService): void;
}

class Service implements DepInjector {
  inject(dep: MyService) {
    this.dep = dep;
  }
}
```

---

## Provider Patterns

Providers are DI instructions that tell Angular **how to create or return a
dependency**, identifying a token and giving a recipe for instantiation. Tokens
might be classes, strings, or `InjectionToken`s.

You register providers via:

- `providedIn: 'root'` in `@Injectable()`
- Adding to `providers:` at module or component level Component-level providers
  create **child injectors**, allowing scoping or overriding per instance
  (\[turn0search0]turn0search7).

### Provider Types

#### `useClass` (Class Provider)

The default when you register a class for a token. Angular will instantiate
`ConsoleLogger` whenever something requests `Logger`. Great for swapping
implementations, no change required in consuming code.

```typescript
providers: [{ provide: Logger, useClass: ConsoleLogger }];
// or shorthand
providers: [Logger];
```

#### `useValue` (Value Provider)

Provides a static value, configuration object, or primitive. Inject with
`@Inject(APP_CONFIG)` to read config data.

```typescript
export const APP_CONFIG = new InjectionToken<AppConfig>("token_description");

providers: [
  { provide: APP_CONFIG, useValue: { apiEndpoint: "https://api.example.com" } },
];
```

#### `useFactory` (Factory Provider)

Uses a factory function to create the dependency—especially useful when logic or
runtime context is needed:

```typescript
export function heroFactory(auth: AuthService) {
  return new HeroService(auth.isAdmin());
}

providers: [
  {
    provide: HeroService,
    useFactory: heroFactory,
    deps: [AuthService],
  },
];
```

#### `useExisting` (Alias Provider)

Creates an alias so two tokens refer to the same instance—no new instance is
created. Requests for `OldLogger` or `NewLogger` yield the same singleton
instance.

```typescript
providers: [NewLogger, { provide: OldLogger, useExisting: NewLogger }];
```

#### Multi-Providers (`multi: true`)

Allows multiple providers for the **same token**; Angular aggregates them into
an array. Then inject `@Inject(LOGGER_PROVIDERS) private loggers: Logger[]` to
get both implementations.

```typescript
export const LOGGER_PROVIDERS = new InjectionToken<Logger[]>(
  "token_description"
);

providers: [
  ConsoleLoggerService,
  { provide: LOGGER_PROVIDERS, useExisting: ConsoleLoggerService, multi: true },
  FileLoggerService,
  { provide: LOGGER_PROVIDERS, useExisting: FileLoggerService, multi: true },
];
```

### Scoping & Lifecycles

| Scope Location           | Effect                                  |
| ------------------------ | --------------------------------------- |
| `providedIn: 'root'`     | Singleton app-wide                      |
| Module-level provider    | Shared within that module               |
| Component-level provider | New instance per component view subtree |

### Best Practices

- Prefer `providedIn: 'root'` unless you need scoped instances.
- Use **injection tokens** instead of strings to avoid collisions.
- Choose the appropriate provider type based on need:

  - Config/constants -> `useValue`
  - Dynamic runtime initialization -> `useFactory`
  - Swap implementations without altering consumers -> `useClass` or
    `useExisting`
  - Multiple contributors -> `multi: true`

- Place providers thoughtfully—module-level for shared singletons,
  component-level for isolated or reusable patterns.

---

## Angular Binding Methods

| Binding Method   | Syntax              | Direction        | Use Case                             |
| ---------------- | ------------------- | ---------------- | ------------------------------------ |
| Interpolation    | `{{ expr }}`        | Component → View | Display text or simple string output |
| Property Binding | `[prop]="expr"`     | Component → View | Non-string data or DOM properties    |
| Event Binding    | `(event)="handler"` | View → Component | React to user or DOM events          |
| Two-Way Binding  | `[(ngModel)]`       | Bidirectional    | Form control syncing, child props    |

### Interpolation (`{{ }}`) — Component → View (one-way)

Angular converts interpolation to a property binding and casts output to a
string.

```html
<p>{{ message }}</p>
```

```typescript
export class MyComponent {
  message = "Hello, Angular!";
}
```

### Property Binding (`[property]`) — Component → View (one-way)

```html
<button [disabled]="isDisabled">Click me</button>
```

```typescript
export class MyComponent {
  isDisabled = true;
}
```

### Event Binding (`(event)`) — View → Component (one-way)

```html
<button (click)="sayHello()">Say Hello</button>
<p>{{ greeting }}</p>
```

```typescript
export class MyComponent {
  greeting = "";
  sayHello() {
    this.greeting = "Hello, Angular!";
  }
}
```

### Two-Way Binding (`[(...)]`) — View ⇄ Component

Allow component and view to sync data automatically. Sugar syntax combining
`[property]` and `(propertyChange)` (Angular-specific convention), often via
`ngModel` or a custom input/output pair named `something` and `somethingChange`.

```html
<input [(ngModel)]="username" />
<p>Your username is: {{ username }}</p>
```

```typescript
export class MyComponent {
  username = "JohnDoe";
}
```

#### Custom component 2‑way binding:

Sets up `[count]` and `(countChange)` bindings behind the scenes.

```html
<app-counter [(count)]="initialCount"></app-counter>
```

```typescript
// Parent
initialCount = 10;

// Child component
@Input() count!: number;
@Output() countChange = new EventEmitter<number>();

increment() {
  this.countChange.emit(this.count + 1);
}
```

---

## Directives

Angular directives let you manipulate the DOM, behavior, and structure of
templates. They come in three types:

1. **Component directives** (components with templates)
2. **Structural directives** (modify DOM structure, e.g. `*ngIf`)
3. **Attribute directives** (modify behavior or styling, e.g. `ngClass`)

### Built-in Structural Directives

Angular includes these common structural directives:

- **`*ngIf`**: conditionally adds or removes elements.
- **`*ngFor`**: loops over a collection and instantiates DOM per item.
- **`*ngSwitch`, `*ngSwitchCase`, `*ngSwitchDefault`**: choose between multiple
  views based on a value.

```html
<div *ngIf="isLoggedIn">Welcome back!</div>

<ul>
  <li *ngFor="let item of items; index as i">{{ i + 1 }}: {{ item }}</li>
</ul>

<div [ngSwitch]="status">
  <p *ngSwitchCase="'active'">Active</p>
  <p *ngSwitchCase="'inactive'">Inactive</p>
  <p *ngSwitchDefault>Unknown</p>
</div>
```

### Built-in Attribute Directives

These directives adjust styling or behavior without changing structure:

- **`[ngClass]`**: dynamically adds/removes CSS classes
- **`[ngStyle]`**: adjusts inline styles
- **`[ngModel]`**: two-way form binding

```html
<p [ngClass]="{ active: isActive }">Status</p>
<p [ngStyle]="{ color: textColor, 'font-size.px': fontSize }">Styled text</p>
<input [(ngModel)]="user.name" placeholder="Name" />
```

### Creating Custom Directives

#### Attribute Directive

Use when you want to modify an element's behavior or appearance.

```typescript
@Directive({ selector: "[appHighlight]" })
export class HighlightDirective implements OnInit {
  @Input() appHighlight = "";
  constructor(private el: ElementRef, private renderer: Renderer2) {}
  ngOnInit() {
    this.renderer.setStyle(
      this.el.nativeElement,
      "backgroundColor",
      this.appHighlight || "yellow"
    );
  }
}
```

```html
<p appHighlight="lightgreen">Hover over me!</p>
```

#### Structural Directive

```typescript
@Directive({ selector: "[appShowIf]" })
export class ShowIfDirective {
  private hasView = false;

  @Input() set appShowIf(condition: boolean) {
    if (condition && !this.hasView) {
      this.vcRef.createEmbeddedView(this.templateRef);
      this.hasView = true;
    } else if (!condition && this.hasView) {
      this.vcRef.clear();
      this.hasView = false;
    }
  }

  constructor(
    private templateRef: TemplateRef<any>,
    private vcRef: ViewContainerRef
  ) {}
}
```

```html
<div *appShowIf="isVisible">Only when visible is true</div>
```

---

## Pipes

### Common Built‑in Pipes

Angular provides several out-of-the-box pipes (all are **pure**, except
`AsyncPipe`, `JsonPipe`, `SlicePipe`) for formatting and data transformation

- **DatePipe** – Format dates: `<p>{{ now | date:'fullDate' }}</p>` _
  **CurrencyPipe** – Format numbers as currencies: `{{ price |
currency:'USD':'symbol':'1.2-2' }}` _ **DecimalPipe / PercentPipe** – Format
  numbers: `{{ value | number:'1.0-2' }}` or `{{ 0.25 | percent:'1.0-0' }}` _
  **UpperCasePipe / LowerCasePipe / TitleCasePipe** – String casing: `{{ text |
uppercase }}` _ **SlicePipe** – Extract portion of string/array: `{{ items |
slice:0:3 }}` \* **JsonPipe** – Debug by serializing objects: `{{ obj | json }}`
- **AsyncPipe** – Automatically subscribes to Observables or Promises and
  unwraps values.

### Pure vs Impure Pipes

| Pipe Type       | Runs When…                              | Best Use Cases                            |
| --------------- | --------------------------------------- | ----------------------------------------- |
| **Pure pipe**   | Only when input value/reference changes | Data formatting: strings, dates, decimals |
| **Impure pipe** | On _every_ change detection cycle       | Live filtering, mutable array updates     |

#### Pure Pipes (default)

- Executed **only** when:

  - Primitive input value changes, or
  - Object/array reference changes.

- Very efficient and performant.

#### Impure Pipes

- Run **on every change detection cycle**, even if input reference stays the
  same. Useful for arrays or mutable objects (like inline filter/sort).
- Built-in impure pipes: `AsyncPipe`, `JsonPipe`, `SlicePipe`.

### Writing Custom Pipes

#### Pure Pipe `capitalize`

```typescript
import { Pipe, PipeTransform } from "@angular/core";

@Pipe({ name: "capitalize" /* pure by default */ })
export class CapitalizePipe implements PipeTransform {
  transform(value: string): string {
    if (!value) return value;
    return value.charAt(0).toUpperCase() + value.slice(1);
  }
}
```

```typescript
<p>{{ user.name | capitalize }}</p>
```

#### Impure Pipe `impureFilter`

```typescript
import { Pipe, PipeTransform } from "@angular/core";

@Pipe({ name: "impureFilter", pure: false })
export class ImpureFilterPipe implements PipeTransform {
  transform(items: any[], searchText: string): any[] {
    if (!items || !searchText) return items;
    return items.filter((item) =>
      item.name.toLowerCase().includes(searchText.toLowerCase())
    );
  }
}
```

```html
<ul>
  <li *ngFor="let item of items | impureFilter: searchText">{{ item.name }}</li>
</ul>
```

---

## Angular Lifecycle Hooks

Angular provides several lifecycle hooks that allow you to tap into key events
in a component's lifecycle. These hooks are called in a specific order:

1. `ngOnChanges`: Called when any data-bound input property changes.
2. `ngOnInit`: Called once, after the first `ngOnChanges`.
3. `ngDoCheck`: Called during every change detection run.
4. `ngAfterContentInit`: Called after content (ng-content) has been projected
   into the component.
5. `ngAfterContentChecked`: Called after every check of projected content.
6. `ngAfterViewInit`: Called after the component's view and child views have
   been initialized.
7. `ngAfterViewChecked`: Called after every check of the component's view and
   child views.
8. `ngOnDestroy`: Called just before Angular destroys the component.

```typescript
import { Component, OnInit, OnChanges, SimpleChanges } from "@angular/core";

@Component({
  selector: "app-lifecycle-hooks",
  template: `<p>{{ message }}</p>`,
})
export class LifecycleHooksComponent implements OnInit, OnChanges {
  message = "Hello, Angular!";

  ngOnChanges(changes: SimpleChanges) {
    console.log("ngOnChanges:", changes);
  }

  ngOnInit() {
    console.log("ngOnInit: Component initialized");
  }
}
```

---

## `ngOnChanges`

`ngOnChanges` is a lifecycle hook that is called when any data-bound property of
a directive changes. It receives a `SimpleChanges` object that contains the
current and previous values of the properties.

```typescript
import { Component, Input, OnChanges, SimpleChanges } from "@angular/core";

@Component({
  selector: "app-input-change",
  template: `<p>{{ data }}</p>`,
})
export class InputChangeComponent implements OnChanges {
  @Input() data: string;

  ngOnChanges(changes: SimpleChanges) {
    console.log("ngOnChanges:", changes);
  }
}
```

```html
<app-input-change [data]="parentData"></app-input-change>
```

In this example, whenever `parentData` changes, `ngOnChanges` will be triggered
in the child component.

---

## `ChangeDetectorRef`

`ChangeDetectorRef` is a service that allows you to control change detection
manually.

```typescript
import {
  Component,
  ChangeDetectorRef,
  ChangeDetectionStrategy,
} from "@angular/core";

@Component({
  selector: "app-manual-change-detection",
  template: `<p>{{ counter }}</p>`,
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ManualChangeDetectionComponent {
  counter = 0;

  constructor(private cdRef: ChangeDetectorRef) {}

  increment() {
    this.counter++;
    this.cdRef.detectChanges(); // Manually trigger change detection
  }
}
```

---

## `@ViewChild` and `@ViewChildren`

### `@ViewChild`

`@ViewChild` allows a parent component to access a single child component,
directive, or DOM element.

```typescript
import { Component, ViewChild, AfterViewInit } from "@angular/core";
import { ChildComponent } from "./child.component";

@Component({
  selector: "app-parent",
  template: `<app-child></app-child>`,
})
export class ParentComponent implements AfterViewInit {
  @ViewChild(ChildComponent) child: ChildComponent;

  ngAfterViewInit() {
    console.log(this.child); // Access child component instance
  }
}
```

### `@ViewChildren`

`@ViewChildren` is used to access multiple child components or elements.

```typescript
import {
  Component,
  ViewChildren,
  QueryList,
  AfterViewInit,
} from "@angular/core";
import { ChildComponent } from "./child.component";

@Component({
  selector: "app-parent",
  template: `<app-child></app-child><app-child></app-child>`,
})
export class ParentComponent implements AfterViewInit {
  @ViewChildren(ChildComponent) children: QueryList<ChildComponent>;

  ngAfterViewInit() {
    this.children.forEach((child) => console.log(child)); // Access all child components
  }
}
```

---

## `ng-template` vs `ng-container`

- Use `ng-template` when you want **deferred, reusable, context-aware
  fragments**—especially with `ngTemplateOutlet` or custom directives.
- Use `ng-container` when you need to **apply structural directives** to
  multiple elements without impacting the DOM.
- Know that `*ngIf`, `*ngFor`, and `*ngSwitchCase` are syntactic sugar over
  `ng-template` blocks.
- You can mention the internal lifecycle details: `ng-template` never renders
  unless consumed; `ng-container` renders children immediately but without its
  own tag.

| Feature                        | `ng-template`                                 | `ng-container`                                   |
| ------------------------------ | --------------------------------------------- | ------------------------------------------------ |
| Renders itself?                | No—not in DOM until explicitly used           | No—even children render directly                 |
| Requires structural directive? | Yes—via `*ngIf` etc. or explicit outlet       | Optional, but most useful with `*ngIf`, `*ngFor` |
| Reusable / dynamic?            | ✅ via `ngTemplateOutlet` and context passing | ❌ Static grouping only                          |
| Internal Angular use           | Used under the hood for `*` sugar-rendering   | Primarily grouping and directive host            |
| Supports context/injector      | Yes                                           | No                                               |

**Use `ng-container` to group without extra HTML:**

```html
<ng-container *ngIf="isLoggedIn; else loginTpl">
  <p>Welcome, {{ user.name }}!</p>
  <p>Last login: {{ lastLogin }}</p>
</ng-container>

<ng-template #loginTpl>
  <p>Please <a href="/login">log in</a>.</p>
</ng-template>
```

**Define reusable template and render via outlet:**

```html
<ng-template #itemTpl let-item>
  <div class="item-card">
    <h4>{{ item.title }}</h4>
    <p>{{ item.desc }}</p>
  </div>
</ng-template>

<ng-container *ngFor="let product of products">
  <ng-container
    *ngTemplateOutlet="itemTpl; context: { $implicit: product }"
  ></ng-container>
</ng-container>
```

- Here, `ng-template` defines the reusable UI; `ng-container` hosts both the
  loop and outlet injection.

---

## `<ng-content>`

Angular allows projecting multiple pieces of content into specific slots inside
a component template by using the `select` attribute on `<ng-content>` elements.
This is especially useful for building reusable container components (e.g.,
cards, modals) that accept content for header, body, footer, and more.

### Example: Card Component with Header, Body & Footer Slots

#### Component Template (`card.component.html`)

```html
<div class="card">
  <header class="card-header">
    <ng-content select="[header]"></ng-content>
  </header>
  <section class="card-body">
    <ng-content select="[body]"></ng-content>
  </section>
  <footer class="card-footer">
    <ng-content select="[footer]"></ng-content>
  </footer>
</div>
```

- `<ng-content select="[header]="">` matches any projected element tagged with
  the `header` attribute.
- You can also include an unselective `<ng-content>` for default slot content if
  needed.

#### Usage in Parent Template (`app.component.html`)

```html
<app-card>
  <div header>
    <h3>Card Title</h3>
  </div>
  <div body>
    <p>This is the main card content.</p>
  </div>
  <div footer>
    <button>Save</button>
  </div>
</app-card>
```

### Multi-Slot Projection with CSS Selectors or Elements

Instead of attributes, you can use CSS selectors or component selectors:

```html
<ng-content select="h1, h2, h3"></ng-content>
<ng-content select=".special-content"></ng-content>
```

### Default / Fallback Slot Behavior

If you include an unqualified `<ng-content>` (without `select`), any content not
matching named slots will be projected there:

```html
<ng-content select="[header]"></ng-content> <ng-content></ng-content>
<!-- fallback/default slot -->
```

### Slot Ordering & Defaults

- Angular matches content to the **first matching slot only**. Once content is
  projected, it’s removed from the host and placed. It won’t appear in other
  slots.
- To reuse content across slots or conditionally reinsert it, use `ng-template`
  with `ngTemplateOutlet` referencing that content fragment separately.
- Angular initializes projected content even if it’s hidden via conditional
  logic (`*ngIf`). Angular won’t delay content initialization in `<ng-content>`;
  you must use `ng-template` to avoid instantiation unless required.

### Conditional Rendering or Default Content

To support optional content (e.g., show custom header if provided, otherwise
fallback), you can:

- Provide default fallback content within `<ng-content>` tags:

```html
<ng-content select="[header]">Default Header</ng-content>
```

- Or use **`@ContentChildren`** or a custom directive (`slot`) to detect if
  content was provided, then conditionally render or default to fallback using
  `ngTemplateOutlet`.

---

## `APP_INITIALIZER` – Run Startup Logic Before App Load

Use `APP_INITIALIZER` to delay application bootstrap until async tasks complete,
like loading configuration, fetching user settings, or initializing services.

- Angular waits for the `Promise` to resolve before rendering the app.
- Use `multi: true` to support multiple initializers.
- Use cases: Feature toggles, i18n settings, auth token initialization.

```typescript
@Injectable({ providedIn: "root" })
class AppConfigService {
  config: any;
  loadConfig(): Promise<void> {
    return this.http
      .get("/assets/app-config.json")
      .toPromise()
      .then((data) => (this.config = data));
  }
}

@NgModule({
  providers: [
    AppConfigService,
    {
      provide: APP_INITIALIZER,
      useFactory: (cfg: AppConfigService) => () => cfg.loadConfig(),
      deps: [AppConfigService],
      multi: true,
    },
  ],
})
export class AppModule {}
```

---

## HTTP Interceptors – Centralized HTTP Request/Response Handling

Angular's `HttpInterceptor` interface allows you to intercept outgoing requests
and incoming responses to add headers, show loaders, or handle errors.

Common scenarios:

- Adding auth headers
- Global loading spinner (using `finalize`)
- Mapping error format responses
- Retrying/transformation or caching logic

```typescript
@Injectable()
export class HttpAuthErrorInterceptor implements HttpInterceptor {
  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    const token = this.authService.getToken();
    const authReq = req.clone({
      setHeaders: { Authorization: `Bearer ${token}` },
    });

    return next.handle(authReq).pipe(
      tap((evt) => {
        /* logging logic */
      }),
      catchError((error: HttpErrorResponse) => {
        if (error.status === 401) {
          this.router.navigate(["/login"]);
        }
        return throwError(() => error);
      })
    );
  }
}
```

```typescript
providers: [
  {
    provide: HTTP_INTERCEPTORS,
    useClass: HttpAuthErrorInterceptor,
    multi: true,
  },
];
```

---

## Global Error Handler – Catch Uncaught Runtime Errors

Angular's `ErrorHandler` captures uncaught exceptions anywhere in the app
(lifecycle hooks, event handlers, template expressions). Combined with
interceptors, you can unify error handling.

```typescript
@Injectable({ providedIn: "root" })
export class GlobalErrorHandler implements ErrorHandler {
  constructor(private logger: ErrorLoggerService, private injector: Injector) {}

  handleError(err: any): void {
    const ngZone = this.injector.get(NgZone);
    ngZone.run(() => {
      console.error("Handled globally:", err);
      this.logger.log(err);
      // user-friendly UI or navigation
    });
  }
}
```

```typescript
{ provide: ErrorHandler, useClass: GlobalErrorHandler }
```

### Important Notes

- Use `ErrorHandler` for client-side exceptions.
- When interceptors catch and **re-throw** HTTP errors, `ErrorHandler` can also
  catch those.
- If subscriber-level `.catchError(…)` handles the error and doesn't re-throw,
  `ErrorHandler` won't be invoked for that error.

### Interaction between Interceptors and ErrorHandler

- An HTTP error first goes through **HttpInterceptor**.
- If interceptor re-throws the original `HttpErrorResponse`, it can also be
  caught by the **Global ErrorHandler**.
- But if the interceptor **transforms** the error into a generic `Error`,
  `ErrorHandler` won't detect it as HTTP error. Better to re-throw original
  object.

---

## Standalone Components

Standalone components are Angular components, directives, or pipes that don't
require declaration within an `NgModule`. Introduced in Angular 14 and refined
in later versions, they simplify architecture by colocating dependencies and
metadata inside the component itself — leading to modular, lightweight, and more
maintainable codebases.

Starting in Angular 19, `standalone: true` will become the **default**, marking
a paradigm shift away from module-heavy architectures.

### Key Advantages

- Reduced Boilerplate: No need for feature modules just to declare components.
- Improved Tree-Shaking: Dependencies are explicit, which enables better
  dead-code elimination, smaller bundle sizes, and improved startup performance.
- Simplified Lazy Loading: Components can be lazy-loaded directly using
  `loadComponent`, no module scaffolding required.
- Streamlined Testing: Standalone components can be tested without complex
  module setup, improving clarity and reducing test boilerplate.
- Better Developer Experience: Dependencies are visible within the component
  decorator and auto-import handled by IDEs. Great for new developers and rapid
  prototyping.

### Basic Usage & Code Examples

#### Declaring a Standalone Component

```typescript
import { Component } from "@angular/core";
import { CommonModule } from "@angular/common";

@Component({
  selector: "app-hello-world",
  standalone: true,
  imports: [CommonModule],
  template: `<h1>Hello, Standalone!</h1>`,
})
export class HelloWorldComponent {}
```

No `NgModule` required — this component can be imported directly where needed.

#### Using a Standalone Component

```typescript
@Component({
  selector: "app-parent",
  standalone: true,
  imports: [HelloWorldComponent],
  template: `<hello-world></hello-world>`,
})
export class ParentComponent {}
```

Just import the component in the `imports:` array of another standalone
component — no module needed.

### When to Use vs. When to Keep NgModules

#### Use Standalone When:

- Building new features or small apps.
- Creating reusable component libraries.
- You want component-level lazy loading or deferred strategy.
- Simplifying tests and reducing boilerplate.

#### Stick with Modules When:

- Existing large legacy app structured around modules.
- You need group-level exports or feature boundaries.
- Certain libraries or tooling still depend on `NgModule` configuration.

---

## Angular Routing

Angular's `RouterModule` enables navigation within a Single Page Application
without full page reloads. You define routes that match URL paths to components:

```typescript
const routes: Routes = [
  { path: "", component: HomeComponent, pathMatch: "full" },
  { path: "dashboard", component: DashboardComponent },
  { path: "**", component: NotFoundComponent },
];
@NgModule({ imports: [RouterModule.forRoot(routes)], exports: [RouterModule] })
export class AppRoutingModule {}
```

Use `<router-outlet></router-outlet>` in templates as a placeholder where
matched components render.

## Route Guards: Controlling Access & Navigation

Angular provides five main guard types:

| Guard Type         | When It Runs                          | Return Type                 | Use Case                               |
| ------------------ | ------------------------------------- | --------------------------- | -------------------------------------- |
| `CanActivate`      | Before activating a route             | boolean or UrlTree or Async | Auth checks                            |
| `CanActivateChild` | Before entering child routes          | boolean or UrlTree          | Protect feature sections               |
| `CanDeactivate`    | Before leaving the current route      | boolean / Async boolean     | Unsaved change confirmation            |
| `CanMatch`         | While matching a route (lazy loading) | boolean or UrlTree          | Feature flag routing, prevent download |
| `Resolve`          | Before route activation               | Observable data             | Pre-fetch route data                   |

### `CanActivate`

Prevents unauthorized navigation into a route. Should return a boolean,
`UrlTree`, `Promise`, or `Observable` of these.

#### Class-based

```typescript
@Injectable({ providedIn: "root" })
export class AuthGuard implements CanActivate {
  constructor(private auth: AuthService, private router: Router) {}
  canActivate() {
    return this.auth.isLoggedIn() || this.router.createUrlTree(["/login"]);
  }
}
```

Apply it:

```typescript
{ path: 'dashboard', component: DashboardComponent, canActivate: [AuthGuard] }
```

#### Functional guard (Angular 16+)

```typescript
export const authGuard: CanActivateFn = () => {
  const auth = inject(AuthService);
  const router = inject(Router);
  return auth.isLoggedIn()
    ? true
    : router.createUrlTree(['/login']);
};
// usage
{ path: 'dashboard', canActivate: [authGuard], component: DashboardComponent }
```

**Avoid manual `router.navigate()` inside guards—return a `UrlTree` instead.**

### `CanActivateChild`

Runs only for child routes. Good for protecting a set of nested routes:

```typescript
{ path: 'admin', canActivateChild: [adminGuard], children: [
   { path: 'users', component: UserListComponent },
   { path: 'settings', component: SettingsComponent }
]}
```

**If navigating within child routes, only `CanActivateChild` runs—not the parent
`CanActivate`.**

### `CanDeactivate`

Prevents a user from leaving a route—useful for unsaved changes or cleanup
prompts.

```typescript
export interface CanComponentDeactivate {
  canDeactivate(): boolean | Observable<boolean>;
}

export const unsavedGuard: CanDeactivateFn<CanComponentDeactivate> = (
  component
) => component.canDeactivate();
```

On routes:

```typescript
{ path: 'editor', component: EditorComponent, canDeactivate: [unsavedGuard] }
```

Editors can implement:

```typescript
canDeactivate(): boolean {
  return this.form.dirty
    ? confirm('Unsaved changes, leave?')
    : true;
}
```

Reusable approaches delegate logic via a common interface.

### `CanLoad` & `CanMatch`

- `CanLoad` (now deprecated class-based) prevents downloading a lazy module for
  unauthorized users.
- `CanMatch` replaces it in Angular 14+: it prevents route matching and
  therefore lazy chunk loading entirely.

```typescript
export const featureFlagGuard: CanMatchFn = (route, segments) => {
  const fs = inject(FeatureService);
  return fs.isEnabled('betaFeature');
};

{ path: 'beta', loadChildren: () => import('./beta.module').then(m => m.BetaModule), canMatch: [featureFlagGuard] }
```

This prevents large chunks from downloading unless allowed.

### `Resolve`

Fetches data before navigation completes; ensures components have pre-fetched
data available. Often used to avoid blank intermediate states.

```typescript
@Injectable({providedIn: 'root'})
export class ItemResolver implements Resolve<Item> {
  constructor(private svc: ItemService) {}
  resolve(route: ActivatedRouteSnapshot) {
    return this.svc.getItem(route.params['id']);
  }
}

{ path: 'item/:id', component: ItemComponent, resolve: { item: ItemResolver } }
```

---

## Lazy Loading

Lazy loading in Angular is a performance optimization technique that defers the
loading of feature modules until they are needed, rather than loading all
modules at the application's startup. This approach reduces the initial bundle
size, leading to faster load times and improved user experience, especially in
large applications.

- Reduced Initial Load Time: Only essential code is loaded initially, speeding
  up the application's startup.
- Improved Performance: By loading modules on demand, the application becomes
  more responsive.
- Efficient Resource Usage: Unused modules are not loaded, saving bandwidth and
  memory.

### How to Implement Lazy Loading

To implement lazy loading in Angular, follow these steps:

#### 1. Create a Feature Module with Routing

Generate a new module with routing enabled using the Angular CLI:

```bash
ng generate module features/products --route products --module app.module
```

This command creates a `ProductsModule` with its own routing configuration,
automatically adding it to the main routing module.

#### 2. Define Routes in the Feature Module

In the generated `products-routing.module.ts`, define the routes for the feature
module:

```typescript
const routes: Routes = [
  {
    path: "",
    component: ProductsComponent,
  },
];
```

#### 3. Configure Lazy Loading in the Main Routing Module

In `app-routing.module.ts`, set up the lazy loading for the feature module:

```typescript
const routes: Routes = [
  {
    path: "products",
    loadChildren: () =>
      import("./features/products/products.module").then(
        (m) => m.ProductsModule
      ),
  },
];
```

This configuration tells Angular to load the `ProductsModule` only when the user
navigates to the `/products` route.

---

## Built-in Preloading Strategies in Angular

When you use **lazy loading**, Angular delays loading certain modules until the
user navigates to a route that uses them. While this improves the initial load
time, it can introduce a **delay on first navigation** to the lazy route.

**Preloading strategies solve this by loading lazy-loaded modules _after_ the
initial app load**, so they're ready when the user navigates to them — improving
perceived performance.

Angular provides the following strategies via the `RouterModule.forRoot()` call.

### `NoPreloading` (default)

- **Nothing is preloaded.**
- Lazy-loaded modules are loaded only when needed.

```typescript
RouterModule.forRoot(routes, { preloadingStrategy: NoPreloading });
```

### `PreloadAllModules`

- **All lazy-loaded modules are preloaded in the background** after the app is
  stable.
- Useful for apps with good bandwidth and not many lazy-loaded modules.

```typescript
// app-routing.module.ts
import { NgModule } from "@angular/core";
import { Routes, RouterModule, PreloadAllModules } from "@angular/router";

const routes: Routes = [
  {
    path: "admin",
    loadChildren: () =>
      import("./admin/admin.module").then((m) => m.AdminModule),
  },
  {
    path: "dashboard",
    loadChildren: () =>
      import("./dashboard/dashboard.module").then((m) => m.DashboardModule),
  },
  { path: "", redirectTo: "/dashboard", pathMatch: "full" },
];

@NgModule({
  imports: [
    RouterModule.forRoot(routes, {
      preloadingStrategy: PreloadAllModules, // 👈 this line is key
    }),
  ],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

### Custom Preloading Strategy

You can define a custom class that implements `PreloadingStrategy` to
selectively preload modules.

```typescript
import { PreloadingStrategy, Route } from "@angular/router";
import { Observable, of } from "rxjs";

export class CustomPreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    return route.data?.["preload"] ? load() : of(null);
  }
}
```

Define routes with metadata:

```typescript
const routes: Routes = [
  {
    path: "admin",
    loadChildren: () =>
      import("./admin/admin.module").then((m) => m.AdminModule),
    data: { preload: true },
  },
  {
    path: "reports",
    loadChildren: () =>
      import("./reports/reports.module").then((m) => m.ReportsModule),
    data: { preload: false },
  },
];
```

Register in your `AppModule`:

```typescript
@NgModule({
  imports: [
    RouterModule.forRoot(routes, {
      preloadingStrategy: CustomPreloadingStrategy,
    }),
  ],
  providers: [CustomPreloadingStrategy],
})
export class AppModule {}
```

Only modules with `data: { preload: true }` will be preloaded.

---

## Angular Change Detection & NgZone

`NgZone` is Angular's wrapper over **Zone.js**, an execution context that
patches async APIs (like `setTimeout`, Promises, DOM events, HTTP requests) to
monitor asynchronous tasks. Angular uses these patches to automatically trigger
UI updates when asynchronous code finishes.

- When async code runs **within** the Angular zone, `NgZone` notifies Angular to
  run change detection.
- Code executed **outside** the zone (via `ngZone.runOutsideAngular`) is ignored
  by change detection, unless explicitly brought back with `ngZone.run()`.

### How Angular Change Detection Works

- App wrapped in Angular Zone: All your app’s code runs inside `NgZone`.
  Asynchronous operations are tracked and lead to change detection cycles.

- Dirty marking on events: Angular wraps event listeners and input changes to
  mark components as "dirty" (needing refresh), cascading up to ancestor
  components.

- `onMicrotaskEmpty` triggers view updates: When the Zone reports no pending
  microtasks, Angular calls `ApplicationRef.tick()` which runs change detection
  across the component tree.

- Traversing the component tree: Angular checks bindings for each component. If
  values differ, the UI updates. In **Default** strategy, this runs for all
  components every cycle; with **OnPush**, only for marked and relevant
  components.

### Change Detection Strategies

#### Default Strategy

- Every change detection cycle checks **all components** in the tree.
- Ensures full UI sync but can become performance-intensive in large apps.

#### OnPush Strategy

- `ChangeDetectionStrategy.OnPush` avoids re-evaluating components unless:

  - `@Input()` reference changes (`===` comparison)
  - An event originates from within the component
  - `markForCheck()` is called manually

- Fewer checks yield better performance in immutable-data or event-driven
  scenarios.

### Running Code Inside vs Outside Angular Zone

#### Inside (`ngZone.run(...)`)

- Ensures change detection triggers when async tasks finish.
- Use this when updating UI-bound state.

#### Outside (`ngZone.runOutsideAngular(...)`)

- Avoids unnecessary change detection triggers.
- Best for CPU-heavy or non-UI tasks (e.g. polling, animation loops).

```typescript
constructor(private ngZone: NgZone) { }

runHeavyLoop() {
  this.ngZone.runOutsideAngular(() => {
    for (let i = 0; i < 1e9; i++) { /* cpu work */ }
  });
}
```

If you later need to update the UI:

```typescript
ngZone.run(() => {
  this.data = newData;
});
```

### Zoneless Angular & Signals (Advanced)

Angular now supports **zoneless mode**, removing dependency on `Zone.js`. In
this mode:

- Change detection must be triggered manually via:

  - `ChangeDetectorRef.markForCheck()`
  - Signals, `AsyncPipe`, or API updates like `ComponentRef.setInput`

- Components often use `OnPush` to be eligible for zoneless behavior.

---

## Angular Signals

Angular **Signals** provide a declarative, fine-grained way to manage state and
trigger updates—acting like lightweight, built-in reactive variables.

- A **signal** wraps a value: to read it you call the signal as a function
  (`count()`), and to update it you use `.set(...)` or `.update(...)`
  (`count.set(3)` or `count.update(v => v + 1)`).
- A **computed** signal derives its value from other signals; it's lazily
  evaluated and memoized, recalculating only when dependencies change.

```typescript
import { signal, computed } from "@angular/core";

const count = signal(0);
const double = computed(() => count() * 2);

count.set(5);
// double() now returns 10 and updates only when count changes
```

### Signals & Change Detection

- Angular tracks where signals are read—typically in component templates. When
  these signals change, Angular marks only **that component** (and its
  ancestors) for re-rendering—not the whole component tree.
- This contrasts with the traditional model where Angular re-evaluates all
  components from top to bottom on each async event (via Zone.js).

### Benefits of Signals in Angular

- Targeted reactivity: Only components relying on changed signals get updated.
- Improved runtime performance: Avoids redundant change detection and DOM
  updates.
- Simplicity & readability: Easier than callback-based reactive streams; great
  for UI state and local logic.
- Zone-less future support: Signals pave the way for Angular to eventually
  operate without Zone.js.
- Interop with RxJS: Convert signals to Observables for more complex
  async/reactive workflows when needed.

---

## Webpack and its role in Angular

Webpack is a powerful **module bundler** that takes JavaScript (TS compiled),
HTML, CSS, and assets, builds a dependency graph, and outputs optimized bundles
for the browser.

- Webpack enables **tree shaking**, removing unused Angular and application code
  automatically.
- It handles **lazy-loading modules** for route-based code splitting.
- It powers **source maps, HMR**, and platform optimizations without manual
  setup.

### How Angular CLI Uses Webpack Under the Hood

#### Entry Points

Webpack's entry point is typically `src/main.ts`, which bootstraps the Angular
application (`bootstrapModule(AppModule)`).

It also handles polyfills and vendor code internally via the integrated Angular
Webpack Plugin (`@ngtools/webpack`), configured within `angular.json` rather
than `webpack.config.js`.

#### Loaders

- TypeScript: Files are transpiled to JavaScript using `ts-loader` or
  `babel-loader` combined with Angular-specific loaders (like
  `angular2-template-loader`) for inlining templates/styles.
- HTML & CSS: Template and style assets are processed via loaders (`raw-loader`,
  `html-loader`, `css-loader`, `style-loader`) so they can be bundled and
  inlined into components.

#### Plugins & Optimization

- **HtmlWebpackPlugin** injects generated bundles into `index.html`.
- Webpack's optimization features—like tree shaking, minification, code
  splitting, and HMR (Hot Module Replacement)—are enabled automatically by
  Angular CLI for development and production builds.

#### Outputs

Webpack emits optimized bundles (often hashed filenames), along with source
maps, static assets, and lazy-loaded chunks.

### Bundle Contents

| Bundle Part         | Content & Role                                        |
| ------------------- | ----------------------------------------------------- |
| **runtime.js**      | Webpack runtime bootloader for chunk management       |
| **polyfills.js**    | Browser compatibility logic                           |
| **main.js**         | Core app logic + essential Angular/vendor modules     |
| **common chunk(s)** | Shared dependencies across lazy modules               |
| **lazy bundles**    | Feature modules loaded on demand (via dynamic import) |
| **styles**          | Global and extracted component styles                 |

#### runtime.js

- Contains the minimal Webpack runtime logic used to bootstrap the application
  and load other chunks.
- Manages module loading, chunk mapping, and application startup coordination.
- Typically very small (\~2–3 kB) but crucial for coalescing the rest of the
  build.

#### polyfills.js

- Includes browser polyfills needed for older browsers (Promises, Intl, etc.).
- Enabled or trimmed based on browser support and differential loading config.

#### main.js

- The core application logic and such vendor code (Angular framework, RxJS,
  etc.) needed for bootstrapping.
- May include vendor modules in the same file to optimize initial load,
  depending on config.

#### common chunk(s)

- Automatically created by Webpack when multiple lazy-loaded feature modules
  share code (e.g. shared components/services).
- Moves shared code out of duplicated lazy chunks into a `common.*.js` to avoid
  duplication.

#### lazy-loaded chunks

- Generated for route-based or dynamic imports (e.g. `loadChildren` modules or
  `import()`).
- Loaded on-demand when navigating to that feature, minimizing the initial
  bundle size.

#### styles.css (or styles.\*.js)

- Aggregates global CSS and component-specific styles (if configured).
- Depending on build configuration, CSS may be emitted separately or bundled
  into JS.

### Build Bundle Priority & Loading Sequence

On app load, bundles are fetched in this order:

1. **runtime.js** – Bootstraps loaders
2. **polyfills.js** – Ensures browser compatibility
3. **main.js** – Contains the entry code and shared logic
4. **common chunk(s)** – Shared dependencies between lazy modules
5. **Lazy bundles** – Downloaded when feature routes/components load dynamically

### How to Analyze & Optimize Bundles

- Use **`ng build --stats-json`** and tools like **Webpack Bundle Analyzer** to
  inspect bundle composition and spot large modules.
- Pay attention to how many modules come from `node_modules` (often 80–90% of
  bundle weight).
- Identify duplicates and large third-party imports—for instance, entire lodash
  or Angular Material when only partial usage is needed. Purge or lazily load
  heavy assets.

---

## Angular Compilation Strategies

### Ahead-of-Time (AOT) Compilation

Templates and components are compiled at build time, producing optimized
JavaScript. Default mode since Angular 9.

#### Workflow

Use `ng build --aot` or `ng build --prod`.

#### Pros

- Faster startup and smaller bundles (no compiler in runtime).
- Templates inline HTML and CSS into code, removing extra HTTP load.
- Errors found at build time for higher safety and predictability.

#### Cons

- Slightly longer build times than JIT.
- Source maps may differ from TypeScript, making debugging less intuitive.

### Ivy & Local/Incremental Compilation

Ivy is Angular's runtime and compilation engine optimized for small code and
fast builds:

- Compiles **per component locally** instead of entire app at once, enabling
  faster incremental rebuilds.
- Emits instruction-based render code (such as `ɵɵelementStart`), which triggers
  more effective tree-shaking and runtime performance benefits.
- Applies to both development (ng serve) and production builds: Ivy-powered AOT
  is now default across all Angular CLI workflows (including tests).
- Some enterprise apps noted **\~68% bundle size** and **\~72% build time**
  reductions post-Ivy migration.

### Library Compilation Modes: Partial vs Full

When building Angular libraries, two Ivy compilation modes exist:

- Partial compilation: Packages metadata and leaves final Ivy transformation for
  the consumer application's build phase. Promotes version compatibility across
  Angular versions.

- Full compilation: Produces Ivy-specific compiled output. Faster in-app builds,
  but tightly coupled to a specific Angular version; incompatible across minor
  updates.

#### Common practice

Use **partial mode** for libraries published to npm; full compilation is
generally only used for internal packages targeting a fixed Angular version.

### Just-in-Time (JIT) Compilation (LEGACY)

Templates are compiled **in the browser at runtime** using the Angular compiler
embedded in the bundle.

#### Workflow

Use `ng serve` or `ng build --no-aot`. Templates are compiled on the fly at load
time.

#### Pros

- Faster builds during development and easy iteration.
- No compilation step needed before deployment.

#### Cons

- Larger bundle size (includes the Angular compiler).
- Slower application startup; templates compiled in browser.
- Template errors only caught at runtime, which can be less robust.

### Summary

| Strategy / Mode     | When Used          | Output Location      | Bundle Size                    | Build Speed                      | Runtime Speed                   | Compatibility                       |
| ------------------- | ------------------ | -------------------- | ------------------------------ | -------------------------------- | ------------------------------- | ----------------------------------- |
| **JIT**             | Dev, legacy use    | Browser at runtime   | Larger (compiler included)     | Fast builds                      | Slower startup                  | Works anywhere                      |
| **AOT (Ivy)**       | Default dev & prod | Build-time output    | Smaller bundles                | Moderate (with Ivy improvements) | Fast startup                    | App-version matched                 |
| **Ivy Local Build** | Everywhere (Ivy)   | Component-level code | Lean, tree-shaky               | Fast incremental rebuilds        | Efficient runtime               | Standard Angular CI                 |
| **Library Partial** | Library packaging  | Intermediate code    | Neutral (\~app compiles later) | Slight overhead                  | App handles runtime compilation | Broad Angular version compatibility |
| **Library Full**    | Internal libs      | Ivy-compiled code    | Lean                           | Faster app builds                | Runtime ready Ivy               | Only specific Angular version       |

---

## Ivy

**Ivy** is the modern **compilation and rendering engine** that Angular
introduced starting in **version 9** and made mandatory in **version 13**,
replacing the legacy View Engine.

- It compiles each component **locally**, generating efficient JavaScript
  instructions for the browser.
- Ivy integrates both **compiler and renderer** into a streamlined pipeline for
  production and development builds.
- It delivers **smaller bundle sizes**, **faster incremental builds**, **more
  precise change detection**, and **better developer ergonomics**.

### Smaller Bundle Sizes

Ivy emits only the code actually used by your app. Combined with improved tree
shaking, this can reduce bundle sizes by **25–40%**, and in some cases even
more; reports show up to **80% smaller** for large projects.

### Faster Compilation & Incremental Builds

Ivy supports **incremental compilation**, recompiling only changed components
instead of the entire app. This drastically speeds up development builds and CI
pipelines.

### Enhanced Runtime Performance & Change Detection

Using an **incremental DOM** model, Ivy only updates parts of the DOM that
change, rather than redrawing entire sections—leading to smoother UI updates and
better performance.

### Better Debugging & Template Type Checking

Ivy produces **clearer error messages**, stack traces, and template-level type
checking to catch mistakes early during build time.

### Simplified Dynamic and Lazy Component Loading

With Ivy, APIs like `ViewContainerRef.createComponent()` no longer require
`ComponentFactoryResolver`, making dynamic component loading easier and less
boilerplate-heavy.

### Backward-Compatible & Future-Proof

Ivy is **fully backward-compatible** with Angular 8+ projects and future Angular
versions (including Angular 15+, where Ivy is the only supported engine).

---

## Build Optimization Techniques

### AOT Compilation

Ensures faster startup, smaller bundle sizes, early template error detection,
and better security via pre-compilation.

### Tree Shaking & Dead Code Elimination

Removes unused modules imported via ES modules. Works seamlessly with Ivy and
esbuild to trim bundle size.

### Differential Loading

Bundles separate outputs for modern (ES2015+) and legacy browsers. Even though
newer Angular versions simplify this, it's still supported via build
configuration.

### Size Budgets

You can enforce maximum sizes for initial bundles, scripts, styles, or overall
app size using `budgets` in `angular.json`; exceeding thresholds triggers
warnings or errors.

### CommonJS Dependency Warnings

CommonJS modules hinder static analysis and tree shaking. Angular warns when
these are detected. You can whitelist dependencies or prefer ES modules when
possible.

- **CommonJS (CJS)** is a module system introduced around 2009 and became the
  standard for **server-side JavaScript** and **Node.js** applications.

- It uses **synchronous loading** and the `require()` function to import
  modules, and `module.exports` or `exports` to expose content from a module.

  ```javascript
  // math.js
  exports.add = (a, b) => a + b;

  // app.js
  const math = require("./math");
  console.log(math.add(2, 3)); // 5
  ```

- Modules wrap code in function closures, giving each file its own private scope
  and avoiding global namespace pollution.

#### CJS vs ES Modules (ESM)

- **ES Modules (ESM)** use static `import/export` syntax and are supported
  natively in modern browsers and in Node.js with `.mjs` or `"type":"module"`
  configuration.
- ESM supports **static analysis**, enabling **tree-shaking** — dead code
  elimination — which improves bundle size and performance.
- In contrast, **CommonJS modules load and resolve at runtime**, making static
  analysis difficult and tree-shaking far less effective.

### Source Maps & Named Chunks

Disabling source maps (`sourceMap: false`) and named chunks (`namedChunks:
false`) speeds up production builds and shrinks output size.

---

## Performance Optimization Strategies

| Strategy                        | Benefits                              |
| ------------------------------- | ------------------------------------- |
| **AOT Compilation**             | Faster rendering, smaller bundle size |
| **Lazy Loading**                | Reduced initial load time             |
| **OnPush Change Detection**     | Fewer change detection cycles         |
| **Pure Pipes**                  | Less computation in templates         |
| **trackBy in ngFor**            | Optimized list rendering              |
| **Web Workers**                 | Offload heavy computations            |
| **Bundle Optimization**         | Faster load times                     |
| **Server-Side Rendering (SSR)** | Improved SEO and initial load time    |

### Use Ahead-of-Time (AOT) Compilation

AOT compiles your Angular components and templates during the build process,
rather than in the browser.

- Reduces the size of the JavaScript bundles.
- Speeds up the rendering process.
- Detects template errors early.

### Implement Lazy Loading

Lazy loading defers the loading of feature modules until they are needed,
reducing the initial load time.

```typescript
const routes: Routes = [
  {
    path: "feature",
    loadChildren: () =>
      import("./feature/feature.module").then((m) => m.FeatureModule),
  },
];
```

### Optimize Change Detection with `OnPush` Strategy

The `OnPush` change detection strategy tells Angular to check for changes only
when new references are passed to the component.

- Reduces the number of checks Angular performs.
- Improves performance, especially in large applications.

```typescript
@Component({
  selector: "app-my-component",
  changeDetection: ChangeDetectionStrategy.OnPush,
  templateUrl: "./my-component.component.html",
})
export class MyComponent {
  // Component logic
}
```

### Use Pure Pipes

Pure pipes are only executed when the input data changes, preventing unnecessary
recalculations.

```typescript
@Pipe({
  name: "currencyFormat",
  pure: true,
})
export class CurrencyFormatPipe implements PipeTransform {
  transform(value: number): string {
    return `$${value.toFixed(2)}`;
  }
}
```

### Avoid Complex Expressions in Templates

Using complex expressions in templates can cause Angular to re-evaluate them
frequently, leading to performance issues.

- Move complex logic to component properties or methods.
- Use getters to compute values lazily.
- Avoid using `ngIf` with complex conditions directly in the template.

### Implement `trackBy` in `ngFor` Loops

The `trackBy` function helps Angular identify which items have changed, been
added, or are removed, optimizing rendering.

```typescript
trackByFn(index: number, item: any): number {
  return item.id;
}
```

```html
<ul>
  <li *ngFor="let item of items; trackBy: trackByFn">{{ item.name }}</li>
</ul>
```

### Use Web Workers for Heavy Computations

Web workers allow you to run scripts in background threads, offloading heavy
computations from the main thread. Create a new worker file and use Angular's
`WorkerPlugin` to integrate it into your build process.

- Keeps the UI responsive.
- Improves performance for CPU-intensive tasks.

### Optimize Bundle Size

Reducing the size of your JavaScript bundles decreases load times and improves
performance.

- Use the `webpack-bundle-analyzer` to visualize bundle sizes.
- Remove unused dependencies.
- Use tree-shaking to eliminate dead code.

### Implement Server-Side Rendering (SSR) with Angular Universal

SSR renders your application on the server, sending a fully rendered page to the
client.

- Improves initial load time.
- Enhances SEO.

---

## Bundling Strategies

Modern build pipelines use bundlers like Webpack to produce optimized and
cache-friendly asset bundles, supporting code splitting and dynamic loading.

- **Code Splitting:** Divides the codebase into async-loaded chunks to optimize
  startup time.
- **Shared Dependencies:** Via `SplitChunksPlugin` or `dependOn`, deduplicates
  repeated modules.
- **Dynamic Imports:** Loads modules on demand.

---

## Cache Busting

Cache busting ensures clients receive new code versions via unique file hashes
in bundle names, reducing bugs from stale assets. Proper CDN, prefetch/preload
hints, and analysis tools (webpack-bundle-analyzer) complete the modern
optimization suite.

---

## RxJS

### Common Angular + RxJS Scenarios

#### HTTP Requests with Debounce / Switching

Handling user input (search fields) and canceling inflight HTTP requests:

```typescript
this.searchControl.valueChanges
  .pipe(
    debounceTime(300),
    distinctUntilChanged(),
    switchMap((query) => this.http.get<Item[]>(`/api/search?q=${query}`)),
    catchError(() => of([]))
  )
  .subscribe((items) => (this.items = items));
```

- `debounceTime`: waits for typing to pause
- `distinctUntilChanged`: avoids duplicate queries
- `switchMap`: cancels previous HTTP call on new query
- `catchError`: handles network errors gracefully

#### Cancelling HTTP Requests

Angular `HttpClient` supports request cancellation via **RxJS `unsubscribe()`**
or **`takeUntil()`**.

```typescript
import { Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

private destroy$ = new Subject<void>();

this.http.get('/api/data')
  .pipe(takeUntil(this.destroy$))
  .subscribe(data => {
    console.log(data);
  });

// Cancel the request (e.g., on component destroy)
this.destroy$.next();
this.destroy$.complete();
```

> 🔍 Angular 16+ `HttpClient` now **supports `AbortSignal`** via `signal` option
> if you're using fetch under the hood.

```typescript
const controller = new AbortController();

this.http
  .get("/api/data", {
    signal: controller.signal as any, // TS compatibility needed
  })
  .subscribe({
    next: (data) => console.log(data),
    error: (err) => {
      if (err.name === "AbortError") {
        console.log("Request cancelled");
      }
    },
  });

// To cancel:
controller.abort();
```

#### One-Time Subscription / Preventing Memory Leaks

Read a single value from an observable once (e.g. route parameters):

```typescript
this.route.params.pipe(take(1)).subscribe((params) => {
  /* use params */
});
```

Prefer `take(1)` over `first()` when the stream might complete without emitting,
to avoid `EmptyError`.

#### Parallel Requests with Aggregation

Run multiple HTTP calls in parallel and combine results:

```typescript
forkJoin([this.http.get("/api/a"), this.http.get("/api/b")])
  .pipe(catchError((err) => of([null, null])))
  .subscribe(([a, b]) => {
    this.a = a;
    this.b = b;
  });
```

- `forkJoin`: emits once when all complete
- Handles each API result as an array of outputs

#### Streaming & State Accumulation

Accumulate values over time using `scan()`—useful for counters or lists:

```typescript
fromEvent(button, "click")
  .pipe(
    map(() => 1),
    scan((total, curr) => total + curr, 0)
  )
  .subscribe((count) => console.log(`Clicked ${count} times`));
```

- `scan`: like array reduce, but emits intermediate results

#### Logging / Side Effects Without Transformation

Use `tap()` when you need to debug or assign data without altering the stream:

```typescript
httpCall$
  .pipe(
    tap((response) => console.log("Response:", response)),
    map((res) => res.data),
    catchError((err) => {
      console.error(err);
      return of([]);
    })
  )
  .subscribe((data) => (this.data = data));
```

- `tap`: perform actions without modifying the stream

### Essential RxJS Operators & Use Cases

| Operator                     | Purpose                                                | Usage                               |
| ---------------------------- | ------------------------------------------------------ | ----------------------------------- |
| `of()` / `from()`            | Create streams from values, arrays or promises         | `of(1,2,3).subscribe(...)`          |
| `map()`                      | Transform each emitted value                           | `map(x => x * 2)`                   |
| `filter()`                   | Only emit values matching a predicate                  | `filter(x => x>5)`                  |
| `debounceTime()`             | Wait for pauses in emission                            | `debounceTime(300)`                 |
| `distinctUntilChanged()`     | Skip duplicate consecutive values                      |                                     |
| `switchMap()`                | Cancel previous inner observable on new emission       | `switchMap(query => http.get(...))` |
| `mergeMap()` / `concatMap()` | Handle multiple inner streams (parallel vs sequential) | useful in task pipelines            |
| `forkJoin()`                 | Combine final values from multiple Observables         | parallel API calls                  |
| `retry()`                    | Retry a failed stream N times                          | `retry(3)`                          |
| `catchError()`               | Graceful error recovery                                | `catchError(() => of([]))`          |
| `scan()`                     | Accumulate over time                                   | running sums or histories           |
| `share()` / `shareReplay()`  | Multicast to multiple subscribers                      | avoid duplicate HTTP calls          |

### What Is `catchError` and How Does It Work?

- `catchError()` is an RxJS operator that intercepts error notifications within
  a stream and **replaces** the error by returning a new observable. It behaves
  like `try…catch` for observables.
- Signature: `catchError((err, caught) => ObservableInput)`. You must return a
  new observable: e.g., `of(value)`, `EMPTY`, `caught`, or `throwError(...)`.
- Once an observable emits an error, it completes—**so retrying or returning
  another observable is essential** to continue a stream.

| Scenario                                | `catchError` Return                        | Result                                      |
| --------------------------------------- | ------------------------------------------ | ------------------------------------------- |
| Provide fallback value                  | `of(defaultValue)`                         | Stream continues with safe default          |
| Silent complete                         | `EMPTY`                                    | Stream ends silently; no error thrown       |
| Retry same stream                       | `caught$`                                  | Automatically resubscribes upon error       |
| Propagate error to subscriber           | `throwError(err)`                          | Subscription error handler is triggered     |
| Keep outer stream alive, inner fallback | `innerObs.pipe(catchError(() => of(...)))` | Outer observable continues on input changes |

#### Return Fallback Value

Best when you want the stream to succeed with a default:

```typescript
this.data$ = this.http.get<Data>("/api/data").pipe(
  catchError((err) => {
    console.error(err);
    return of({ items: [] });
  })
);
```

#### Silence Errors and Complete

Use `EMPTY` to resolve the stream silently:

```typescript
.pipe(catchError(err => {
  console.error(err);
  return EMPTY;
}))
```

#### Retry or Re-subscribe Automatically

Return the caught observable to effectively restart the stream:

```typescript
.pipe(
  catchError((err, caught$) => {
    console.error(err);
    return caught$;
  })
);
```

Use with caution—may loop endlessly if error source isn't resolved.

---

## Interview Questions

### What is the role of `NgZone` in Angular, and when would you opt out of Angular's change detection?

`NgZone` is a service that helps Angular know when to update the view. It allows
you to run code outside of Angular's zone to prevent unnecessary change
detection cycles, improving performance. You might opt out of Angular's change
detection when integrating with third-party libraries that manage their own
rendering, such as D3.js or certain charting libraries.

### What are resolution modifiers in Angular, and how do you use them?

Resolution modifiers are used in Angular's dependency injection system to
control how dependencies are provided. They include:

- `providedIn`: Specifies the injector to which the service is provided.

- `useClass`: Defines the class to instantiate.

- `useValue`: Provides a specific value.

- `useFactory`: Uses a factory function to create the dependency.

- `deps`: Lists the dependencies required by the factory.

These modifiers help in customizing the dependency injection behavior.

### Why would you use a `trackBy` function in an `ngFor` loop, and how does it work?

The `trackBy` function is used in `ngFor` loops to improve performance by
helping Angular identify which items have changed, been added, or been removed.
It reduces the number of DOM manipulations by tracking items based on a unique
identifier. For example:

```typescript
trackById(index: number, item: any): number {
  return item.id;
}
```

This ensures that Angular only updates the DOM elements that have changed.

### What is the difference between `providers` and `viewProviders` in Angular?

Both `providers` and `viewProviders` are used to define services in Angular, but
they differ in scope:

- `providers`: Services are available to the component and its child components.

- `viewProviders`: Services are available only to the component's view and its
  child views.

In Angular, `providers` and `viewProviders` are both used to configure
**dependency injection (DI)** at the component level, but they differ in **where
their services are available**.

### What is Ahead-of-Time (AOT) compilation in Angular? Is it enabled by default?

AOT compilation is a process where Angular compiles the application during the
build phase, before the browser downloads and runs the code. This results in
faster rendering, fewer asynchronous requests, and smaller Angular framework
download sizes. Yes, AOT is enabled by default in Angular CLI for production
builds.

### What are Angular Guards, and how do you use them?

Angular Guards are interfaces that determine whether a route can be activated,
deactivated, or loaded. They are used to protect routes and control navigation.
There are four types:

- `CanActivate`: Determines if a route can be activated.

- `CanActivateChild`: Determines if a child route can be activated.

- `CanDeactivate`: Determines if a route can be deactivated.

- `CanLoad`: Determines if a route can be loaded.

Guards are implemented as services and can be added to routes in the routing
module.

### How do you handle HTTP errors globally in Angular?

HTTP errors can be handled globally using an HTTP interceptor. The interceptor
can catch errors from all HTTP requests and responses, allowing for centralized
error handling.

```typescript
@Injectable()
export class ErrorInterceptor implements HttpInterceptor {
  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      catchError((error: HttpErrorResponse) => {
        // Handle error
        return throwError(error);
      })
    );
  }
}
```

### What is the difference between Observables and Promises in Angular?

Both Observables and Promises handle asynchronous operations, but they have key
differences:

- Observables: Can emit multiple values over time, are cancellable, and support
  operators like `map`, `filter`, and `mergeMap`.

- Promises: Resolve a single value once and cannot be cancelled.

### What is Time To First Byte (TTFB)?

TTFB measures the time from a user's request to when the browser receives the
first byte of a response. It encompasses network latency, backend processing,
and dynamic resource fetching, and influences all paint and interactivity
metrics that follow. Optimal TTFB is ≤ 0.8 seconds; higher times are symptomatic
of hosting, network, or backend issues. TTFB can be improved via CDNs,
compression, cache control, efficient database access, and server-side rendering
optimizations.

### What is Optimistic UI?

Implements state changes on the client as soon as an action is triggered (e.g.,
form submit), rolling back only if the operation fails on the server side.
