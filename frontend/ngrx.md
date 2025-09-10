---
title: NgRx
---

## Introduction

NgRx is a powerful, Redux-inspired state management library specifically
designed for Angular applications. Its fundamental purpose is to centralize
application state into a single, immutable store and orchestrate state changes
through a predictable, unidirectional data flow. NgRx helps developers manage
complex stateful logic, facilitate robust UI updates, and enhance application
scalability and maintainability—qualities increasingly necessary as applications
grow in size and complexity.

---

## Core Concepts of NgRx

Understanding NgRx requires fluency with the following interlinked core
concepts:

### Store

The **Store** is the central repository for all application state—a single
source of truth. This immutable object tree holds domain entities, UI state, and
metadata, and is structured hierarchically, allowing for modularization around
features.

### Actions

**Actions** are plain objects describing "what happened" in the system. Each
action has a `type` (a readable string that usually combines the feature and the
event, e.g., `[Counter] Increment`) and an optional payload containing data
required for the state change:

```typescript
export const increment = createAction("[Counter Component] Increment");
export const addUser = createAction("[User] Add", props<{ user: User }>());
```

Actions are dispatched (sent) from components or effects to the store to drive
state transitions or trigger side-effects.

### Reducers

**Reducers** are pure, deterministic functions that take the current state and
an action, and return a new, updated state. Reducers enforce immutability by
returning new objects on every state transition:

```typescript
const counterReducer = createReducer(
  initialState,
  on(increment, (state) => ({ ...state, count: state.count + 1 })),
  on(decrement, (state) => ({ ...state, count: state.count - 1 }))
);
```

Reducers should never mutate the state directly and must cover all recognized
action types.

### Selectors

**Selectors** are optimized, memoized functions used to extract, calculate, and
transform slices of state from the store tree:

```typescript
export const selectCounter = createSelector(
  (state: AppState) => state.counter,
  (counter) => counter.value
);
```

Selectors help in keeping components decoupled from the actual store structure,
boosting modularity and performance.

### Effects

**Effects** allow developers to handle **side-effects** (such as HTTP calls,
navigation, or browser storage) outside of reducers. Effects listen for
dispatched actions, perform asynchronous or impure work, and dispatch new
actions:

```typescript
@Injectable()
export class UserEffects {
  loadUsers$ = createEffect(() =>
    this.actions$.pipe(
      ofType(loadUsers),
      switchMap(() =>
        this.userService.fetchAll().pipe(
          map((users) => loadUsersSuccess({ users })),
          catchError((error) => of(loadUsersFailure({ error })))
        )
      )
    )
  );
}
```

Reducers remain pure, while side-effecting logic remains in effects, ensuring
separation of concerns.

### Entity Management

For managing collections (e.g., lists of users, products), `@ngrx/entity`
provides utilities to simplify CRUD logic, enforce normalized state, and
generate selectors automatically.

### Signals and SignalStore

Starting with Angular 17 and NgRx 18/19, `@ngrx/signals` and **SignalStore**
provide an alternative to the classic observable-based store. SignalStore
leverages Angular signals for fine-grained change detection, and replaces
explicit actions/reducers with state-updating methods:

```typescript
const CounterStore = signalStore(
  { providedIn: "root" },
  withState({ count: 0 }),
  withMethods((store) => ({
    increment() {
      patchState(store, { count: store.count() + 1 });
    },
    decrement() {
      patchState(store, { count: store.count() - 1 });
    },
    reset() {
      patchState(store, { count: 0 });
    },
  }))
);
```

This new paradigm streamlines state management and makes the API more
approachable while retaining the advantages of Redux-style predictability.

---

## Interview Questions

### What is NgRx and why use it in Angular applications?

1. NgRx is a reactive state management library for Angular built on Redux
   principles and RxJS.
2. It centralizes application state in a single store, making state changes
   predictable and traceable.
3. It improves scalability by enforcing a clear separation between state,
   actions, and side effects.
4. Ideal for complex applications where multiple components share and update
   state consistently.

### Explain the key building blocks of NgRx.

1. Store: Holds the immutable application state.
2. Actions: Plain objects that describe state changes.
3. Reducers: Pure functions that take the current state and an action, and
   return a new state.
4. Effects: Handle side effects (e.g., HTTP calls) outside of reducers using
   RxJS operators.
5. Selectors: Pure functions to derive and memoize data slices from the store.

### How do you define and dispatch an action?

```typescript
// Define
export const loadItems = createAction('[Items Page] Load Items');

// Dispatch
constructor(private store: Store) {}
ngOnInit() {
  this.store.dispatch(loadItems());
}
```

1. createAction returns a unique action type.
2. dispatch triggers the action pipeline through reducers and effects.

### How do reducers work? Provide an example.

```typescript
export interface AppState {
  count: number;
}

export const initialState: AppState = { count: 0 };

export const counterReducer = createReducer(
  initialState,
  on(increment, (state) => ({ ...state, count: state.count + 1 })),
  on(decrement, (state) => ({ ...state, count: state.count - 1 }))
);
```

1. Reducers are pure functions: no side effects or async logic.
2. They use `on` to map actions to state transformations.

### What are NgRx Effects and when would you use them?

1. Effects listen for specific actions and perform side effects (HTTP requests,
   logging).
2. They return new actions to the store when side effects complete.
3. Use them to keep reducers pure and handle async tasks outside of state
   transitions.

```typescript
loadItems$ = createEffect(() =>
  this.actions$.pipe(
    ofType(loadItems),
    mergeMap(() =>
      this.api.getItems().pipe(
        map((items) => loadItemsSuccess({ items })),
        catchError((error) => of(loadItemsFailure({ error })))
      )
    )
  )
);
```

### How do you create and use selectors?

```typescript
// Feature selector
const selectFeature = createFeatureSelector<AppState>("featureKey");

// Property selector
export const selectCount = createSelector(
  selectFeature,
  (state) => state.count
);

// In component
this.count$ = this.store.select(selectCount);
```

1. Selectors are memoized, boosting performance.
2. They let components subscribe to specific state slices.

### How can you organize NgRx modules in a large application?

1. Split state by feature modules, each with its own actions, reducers, effects,
   and selectors.
2. Register each feature via `StoreModule.forFeature` and
   `EffectsModule.forFeature`.
3. Use a core module for app-wide state and shared utilities.
4. Keep feature folders consistent: actions.ts, reducer.ts, effects.ts,
   selectors.ts.

### How do you debug NgRx state changes?

1. Install Redux DevTools and integrate via `StoreDevtoolsModule.instrument()`.
2. Use `provideMockStore` in unit tests to inspect dispatched actions and
   selectors.
3. Log actions and state snapshots via `metaReducers`:

```typescript
export function logger(reducer) {
  return (state, action) => {
    console.log("action", action);
    const nextState = reducer(state, action);
    console.log("state", nextState);
    return nextState;
  };
}
```

### How do you test NgRx features?

- Actions: Verify action creators produce correct types and payloads.
- Reducers: Use input state and actions to assert new state.
- Effects: Use `provideMockActions` and mock services to test emitted actions.
- Selectors: Pass sample state and confirm selector output.

### Describe a common usage scenario for NgRx.

1. In an e-commerce app, share product lists, cart contents, and user profile
   across components.
2. Dispatch actions when adding to cart or loading product details.
3. Use effects for API calls to fetch products or submit orders.
4. Select slices for header cart icon, product page, and checkout form.

### How do you handle entity collections with NgRx?

1. Use NgRx Entity for CRUD operations on collections.
2. Define an EntityState for your model:

```typescript
export interface ProductState extends EntityState<Product> {
  selectedId: string | null;
}
```

3. Use `createEntityAdapter` to generate selectors and reducer helpers for
   standardized operations.

### How do you reset store on logout?

1. Define a root meta-reducer that intercepts logout:

```typescript
export function clearState(reducer) {
  return (state, action) =>
    action.type === logout.type
      ? reducer(undefined, action)
      : reducer(state, action);
}
```

2. Add it to `StoreModule.forRoot` via `metaReducers`.

---

## Setting Up NgRx in Angular

### Installation

You can add NgRx packages to your Angular project via the Angular CLI or npm. At
a minimum, you need @ngrx/store and @ngrx/effects; you may also add
@ngrx/entity, @ngrx/store-devtools, and others based on your needs:

```bash
ng add @ngrx/store
ng add @ngrx/effects
npm install @ngrx/entity @ngrx/store-devtools
```

Or directly with npm:

```bash
npm install @ngrx/store @ngrx/effects @ngrx/entity @ngrx/store-devtools --save
```

This setup ensures compatibility with Angular v17+ and the latest stable NgRx
releases.

### Project Structure

Organize your project's store-related code in a dedicated directory, often using
a **feature-based structure**:

```
src/
  store/
    actions/        # Action definitions
    reducers/       # Reducer functions
    effects/        # Effects for side-effects
    selectors/      # Selectors for state retrieval
    models/         # Typed models/interfaces
  features/
    user/
      store/
      ...etc.
```

This promotes modularity and maintainability, especially in large applications.

### Initial Store Setup

Register reducers and effects in your root module (or with Angular 16+
standalone bootstrap if using standalone components):

```typescript
@NgModule({
  imports: [
    StoreModule.forRoot(reducers, { metaReducers }),
    EffectsModule.forRoot([RootEffects]),
    !environment.production ? StoreDevtoolsModule.instrument({ maxAge: 25 }) : [],
  ],
  ...
})
export class AppModule {}
```

Or for standalone init (Angular 16+) in your `main.ts`:

```typescript
bootstrapApplication(AppComponent, {
  providers: [
    provideStore(reducers),
    provideEffects([RootEffects]),
    provideStoreDevtools({ maxAge: 25 }),
  ],
});
```

Reducer maps and meta-reducers can be composed to ensure a scalable entry point
for your app’s state management.

### Feature State Modules

For feature modules, register their feature reducer and effects in their
respective module:

```typescript
@NgModule({
  imports: [
    StoreModule.forFeature("user", userReducer),
    EffectsModule.forFeature([UserEffects]),
  ],
})
export class UserModule {}
```

This approach supports **lazy loading** and prevents the root store from
becoming monolithic.

### Signals-based Store Setup

To use SignalStore (signals-based state management):

```typescript
npm install @ngrx/signals

// counter.store.ts
export const CounterStore = signalStore(
  { providedIn: 'root' },
  withState({ count: 0 }),
  withMethods(store => ({
    increment() { patchState(store, { count: store.count() + 1 }); },
    decrement() { patchState(store, { count: store.count() - 1 }); }
  }))
);
```

Inject and use this store in components using Angular’s inject() API:

```typescript
@Component({...})
export class CounterComponent {
  readonly store = inject(CounterStore);
  // store.count() returns signal value
}
```

This pattern is well-suited to modern Angular development and standalone
components.

---

## Application Architecture with NgRx

### Modular State Organization

A robust NgRx application separates state into feature-based modules, each
managing a coherent slice of the state tree. Shared state might live at the
root, while feature state is encapsulated within modules like `UserState`,
`ProductState`, etc.

**Feature state interface example:**

```typescript
export interface FeatureState extends EntityState<MyEntity> {
  loading: boolean;
  error: string | null;
}
```

Reducers and selectors reside alongside the types they manage, improving
testability and navigation.

### Entity Management

For large collections, leverage **@ngrx/entity** to normalize state:

```typescript
export interface CoursesState extends EntityState<Course> {
  allCoursesLoaded: boolean;
}
export const adapter = createEntityAdapter<Course>();

export const initialCoursesState: CoursesState = adapter.getInitialState({
  allCoursesLoaded: false,
});

export const coursesReducer = createReducer(
  initialCoursesState,
  on(courseLoaded, (state, { course }) => adapter.addOne(course, state)),
  on(allCoursesLoaded, (state, { courses }) =>
    adapter.addAll(courses, { ...state, allCoursesLoaded: true })
  )
);
```

**Selectors with @ngrx/entity:**

```typescript
export const { selectAll, selectEntities } = adapter.getSelectors();
```

This provides well-optimized selectors and standardizes CRUD state updates.

### Component-Store Versus Global Store

Use `@ngrx/component-store` for stateful logic confined to a single component or
tightly coupled set of components. Otherwise, use the global store for truly
app-wide state.

SignalStore, with its intent-based API, also enables feature-based or
component-scoped state slices, fitting modern Angular’s move towards smaller,
standalone, reusable parts.

### Signals, Effects, and Lifecycle

In SignalStore:

- State is initialized with `withState`.
- State is updated with method functions, e.g., `increment`, `addEntity`, using
  `patchState`.
- Derived/memoized selectors are created using `withComputed`.
- Side-effects (effects) are handled via `rxMethod` or colocated observables.

This API reduces boilerplate and improves developer ergonomics, but Redux-style
event tracing and debugging still apply.

---

## Best Practices for NgRx Store Structure

A high-quality NgRx application embodies the following best practices:

### Separate and Modularized State

- Group files per feature: e.g., `user.actions.ts`, `user.reducer.ts`,
  `user.effects.ts`, `user.selectors.ts`.
- Prefix all files, selectors, and classes with their feature (e.g.,
  `userLogin`, `articleList`).
- Use descriptive action names with feature scoping: `[User Page] Load User`.
- Only store state in the root/global store if it is truly app-wide; otherwise
  place it in feature modules.

### Immutability and Pure Functions

- Reducers must avoid mutating state; always return new copies.
- Use meta-reducers like storeFreeze in development to enforce immutability.
- Avoid deeply nested state; flat state trees are more performant and easier to
  maintain.

### Strong Typing

- Use TypeScript interfaces for actions, state, entities, etc.
- Prefer createAction, createReducer, and createEffect APIs for type safety and
  concise code.

### Selectors and Memoization

- Define named selectors in feature files, not inline in components.
- Use selectors to compute derived data—never calculate directly in components
  or templates.
- Memoized selectors prevent unnecessary re-rendering and expensive
  computations.

### Effects and Side-Effects

- Each effect should have a single responsibility (e.g., `loadUsers$`,
  `saveUser$`).
- Handle both success and error actions for all side-effects.
- Avoid dispatching actions directly from components if the operation is
  asynchronous or has side effects—prefer effects.

### Testing

- Unit test all reducers (they are pure functions) and selectors.
- Test effects using marbles or observable mocking.
- Write integration tests that assert proper action dispatching and store update
  on user interaction.
- Mock the store in shallow component tests.

### Facades and Service Abstraction

- Optionally, use a facade pattern to abstract the store logic for complex
  domains, exposing observable streams and methods to components. This increases
  maintainability and improves testability.

### Linting and Automation

- Use linting tools (e.g., eslint-plugin-ngrx) and Angular schematics to enforce
  consistency and reduce boilerplate.

---

## Side-Effects Management with Effects

NgRx Effects address the problem of handling asynchronous or impure
operations—like API requests, persistent storage, or navigation.

### Basic Effect Structure

```typescript
@Injectable()
export class ExampleEffects {
  constructor(private actions$: Actions, private myService: MyService) {}

  loadItems$ = createEffect(() =>
    this.actions$.pipe(
      ofType(loadItems),
      switchMap(() =>
        this.myService.fetchItems().pipe(
          map((items) => loadItemsSuccess({ items })),
          catchError((error) => of(loadItemsFailure({ error })))
        )
      )
    )
  );
}
```

Effects should be registered either globally with `EffectsModule.forRoot()` or
per-feature with `EffectsModule.forFeature()` for lazy-loaded modules.

### Advanced Effects

- Use `@ngrx/entity` in effects for CRUD operations on large collections.
- Non-dispatching effects (with `{ dispatch: false }`) for actions like
  notifications that don't result in additional actions.
- Combine multiple actions or sequence them for composite side-effects.

### Best Practices for Effects

- Keep effects focused: one effect per action side-effect.
- Handle both positive and negative cases for each side-effect action.
- Use `Actions` stream piping, RxJS flattening operators (`switchMap`,
  `mergeMap`, `concatMap`, `exhaustMap`) appropriately based on concurrency
  needs.
- Test effects with marble testing, mocking dependent services and streams.

---

## Easing Debugging in NgRx

NgRx offers multiple powerful tools and techniques for debugging:

### Redux DevTools Integration

- Use @ngrx/store-devtools to integrate with Redux DevTools browser extension.
- Inspect actions and state changes, use time-travel debugging to rewind to
  previous state and trace application bugs.
- DevTools expose the state tree, enable replay, and expedite root-cause
  analysis of state bugs.

### Logging and Middleware

- Implement custom meta-reducers for logging every action/state change.
- Use logging only in development builds to avoid performance hit in production.

### Action and State Inspection

- Name action types precisely for traceability (`[Auth API] Login Success`).
- Use strong typing and consistent action structure to improve code navigation
  and traceability in debugging sessions.

### Component Inspection

- Use Angular DevTools to inspect component states and verify proper reactivity
  to store changes.
- Favor memoized selectors over inline selectors to reduce debugging cognitive
  load.

### Handling Common Debugging Issues

| Debugging Issue                       | Solution                                                                                    |
| ------------------------------------- | ------------------------------------------------------------------------------------------- |
| Action not triggering reducer         | Check that effect is registered and correct action creator is imported and dispatched       |
| Store slice undefined in selector     | Validate reducer is registered with correct feature key and selector refers to correct slot |
| Observable not completing             | Ensure effects terminate via RxJS completion/termination logic                              |
| Performance bottlenecks               | Use DevTools profiler, check subscription patterns and reset selectors if necessary         |
| Mutation errors or time-travel issues | Enable storeFreeze and ensure reducers are not mutating state directly                      |

Addressing these common issues proactively will make debugging in NgRx more
approachable.

---

## Testing Strategies for NgRx

A robust test suite increases confidence in state management logic:

### Reducer Unit Tests

Since reducers are pure functions, they are easily unit tested:

```typescript
describe("Counter Reducer", () => {
  it("should increment count", () => {
    const action = increment();
    const state = counterReducer({ count: 2 }, action);
    expect(state.count).toBe(3);
  });
});
```

Test reducers for all possible actions and edge cases (e.g., unknown actions,
empty payloads).

### Selector Tests

Selector tests should verify that selectors return the expected values from
given states:

```typescript
it("should select the user list", () => {
  const state = { users: [{ id: 1, name: "John" }] };
  expect(selectUserList(state)).toEqual([{ id: 1, name: "John" }]);
});
```

### Effect Tests

Test effects with mocked action streams and stubbed services using tools such as
`provideMockActions` and marble diagrams:

```typescript
describe("Task Effects", () => {
  it("should dispatch success action on successful load", () => {
    actions$ = of(loadTasks());
    taskService.getTasks.and.returnValue(of(tasks));
    effects.loadTasks$.subscribe((action) => {
      expect(action).toEqual(loadTasksSuccess({ tasks }));
    });
  });
});
```

Mock dependencies for isolation and determinism.

### Integration and E2E Testing

Test components by dispatching actions and checking corresponding UI/state
updates. Use Cypress or similar tools for true end-to-end validation.

---

## Common Functionalities: CRUD and Entity Management

NgRx excels at managing CRUD workflows for both primitive and complex entity
collections.

### Basic CRUD Patterns

- **Actions**: Define for load, create, update, delete, and their
  success/failure states.
- **Reducers**: Implement state transitions for all CRUD actions, returning new
  copies of entities.
- **Selectors**: Provide queries for `selectAll`, querying by ID, and filtered
  views.
- **Effects**: Orchestrate API interactions and dispatch success/failure
  actions.

**Sample pattern:**

```typescript
export const loadTodos = createAction("[Todo] Load Todos");
export const addTodo = createAction("[Todo] Add Todo", props<{ todo: Todo }>());

const todoReducer = createReducer(
  initialState,
  on(loadTodosSuccess, (state, { todos }) => ({ ...state, todos })),
  on(addTodo, (state, { todo }) => ({
    ...state,
    todos: [...state.todos, todo],
  }))
);
```

Selectors:

```typescript
export const selectTodos = createSelector(
  featureSelector,
  (state) => state.todos
);
```

Manage loading and error states for robust feedback in the UI. Use
`@ngrx/entity` to avoid repetitive reducer logic and automatic selector
generation.

### Entity Adapters

Entity adapters simplify collection management:

```typescript
const adapter = createEntityAdapter<Todo>();
const todoReducer = createReducer(
  adapter.getInitialState(),
  on(addTodoSuccess, (state, { todo }) => adapter.addOne(todo, state)),
  on(deleteTodoSuccess, (state, { id }) => adapter.removeOne(id, state))
);
```

This approach standardizes, optimizes, and future-proofs state management for
entity-heavy applications.

---

## Usage Scenarios with NgRx

NgRx is suitable for applications with:

- **Complex data flows:** e.g., dashboards, e-commerce shops (products, carts,
  profiles).
- **Multi-user collaboration:** Ensures shared state consistency across UI and
  connected clients.
- **Asynchronous operations:** Long-running workflows or heavy external API
  dependencies.
- **UI components requiring access to shared and reactive state.**
- **Large-scale enterprise Angular apps:** Enables modular, maintainable, and
  traceable architectures.

Specific scenarios include:

- Real-time chat systems (unread counters, messages)
- Domain entity management (products, users)
- Temporal features (undo/redo, optimistic updates)
- Form validation and cross-component forms (via @ngrx/forms)

---

## Sample Code Snippets and Patterns

### Action Creator Example

```typescript
export const login = createAction(
  "[Auth] Login",
  props<{ username: string; password: string }>()
);
```

### Reducer Example

```typescript
export const authReducer = createReducer(
  initialState,
  on(login, (state) => ({ ...state, isLoading: true })),
  on(loginSuccess, (state, { user }) => ({ ...state, isLoading: false, user }))
);
```

### Selector Example

```typescript
export const selectAuthState = createFeatureSelector<AuthState>("auth");
export const selectCurrentUser = createSelector(
  selectAuthState,
  (state) => state.user
);
```

### Effect Example

```typescript
@Injectable()
export class AuthEffects {
  constructor(private actions$: Actions, private authService: AuthService) {}

  login$ = createEffect(() =>
    this.actions$.pipe(
      ofType(login),
      switchMap(({ username, password }) =>
        this.authService.login(username, password).pipe(
          map((user) => loginSuccess({ user })),
          catchError((error) => of(loginFailure({ error })))
        )
      )
    )
  );
}
```

### Component with Store Usage

```typescript
@Component({...})
export class AuthComponent {
  user$ = this.store.select(selectCurrentUser);

  constructor(private store: Store) {}

  onLogin(credentials: { username: string, password: string }) {
    this.store.dispatch(login(credentials));
  }
}
```

### SignalStore Counter Example

```typescript
export const CounterStore = signalStore(
  { providedIn: "root" },
  withState({ count: 0 }),
  withMethods((store) => ({
    increment() {
      patchState(store, { count: store.count() + 1 });
    },
  }))
);
// In component: store.count(), store.increment()
```

SignalStore’s concise API is ideal for modern standalone Angular components and
efficient change detection.

---

## Performance Optimization in NgRx

Achieving high performance in NgRx applications involves:

- **Memoized selectors:** Prevent unnecessary recomputation and UI rerenders.
- **Normalized state:** Use entities with `ids` and `entities` maps for quick
  lookups.
- **Lazy loading feature modules and state:** Improves initial load performance.
- **Selective subscription:** Components should only subscribe to necessary
  slices of the store.
- **OnPush change detection:** Use with pure observables and selectors for
  minimal component updates.
- **Meta-reducers for development-only features (e.g., storeFreeze):** Enforce
  immutability without runtime performance penalties in production.
