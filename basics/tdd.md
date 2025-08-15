---
title: Test Driven Development (TDD)
---

## Fundamentals of Test-Driven Development (TDD)

### What is TDD?

**Test-Driven Development (TDD)** is a software engineering discipline where
tests are written before production code. The methodology follows a repeatable,
disciplined cycle, succinctly described as "Red–Green–Refactor":

1. **Red:** Write a new test for desired functionality—this test should fail at
   first.
2. **Green:** Write the minimal amount of code needed to pass the test.
3. **Refactor:** Improve the code, structure, and test without changing
   functionality, ensuring all tests still pass.

### TDD Cycle Example

A typical TDD workflow looks like this:

1. List scenarios for a feature.
2. Pick one scenario and write a failing test.
3. Run all tests—the new test fails.
4. Write the simplest code to make the test pass.
5. Repeat for each scenario, refactoring as needed to improve design.

**Example in C#:**

```csharp
// Test--fails initially
[Fact]
public void Add_OneAndOne_ReturnsTwo()
{
    var calc = new Calculator();
    var result = calc.Add(1, 1);
    Assert.Equal(2, result);
}

// Production Code--writes only enough to pass test
public int Add(int a, int b)
{
    return a + b;
}
```

Here, the test defines desired behavior _first_, and the code is only expanded
enough to fulfill that contract. Refactors can follow if safe (with tests
green).

---

## Advantages and Disadvantages of TDD

### Key Advantages

The benefits of Test-Driven Development are widely recognized in professional
software engineering:

- **Requirement Clarity and Focus:** Writing tests first forces developers to
  think clearly about requirements and expected behaviors.
- **Only Necessary Code:** Every line of production code is written with an
  explicit, verified need—reducing overengineering and bloat.
- **High Test Coverage:** Automated tests cover each feature, resulting in
  confidence for continuous integration, refactoring, and quick bug
  localization.
- **Refactor Friendly:** Comprehensive tests make code safe to refactor,
  upgrade, or optimize without breaking functionality.
- **Modular, Decoupled Design:** TDD encourages small units and clear
  interfaces, supporting high cohesion and loose coupling between system parts.
- **Regression Safety:** Test suites catch regressions early, supporting rapid,
  reliable changes.
- **Documentation:** Tests serve as executable documentation, helping onboard
  new developers and providing living usage examples.
- **Less Debugging:** Defects are caught early, minimizing laborious debugging
  later in SDLC.

#### TDD Advantages Summary Table

| Advantage                     | Result/Explanation                                 |
| ----------------------------- | -------------------------------------------------- |
| Requirement clarity           | Forces upfront, detailed requirement analysis      |
| Only code that’s needed       | No excess code; all code is tested and necessary   |
| Modular design                | Clear interfaces, easy-to-test (and maintain) code |
| Easier to maintain & refactor | Decoupled components, promoting safe changes       |
| High coverage                 | Nearly every behavior has tests, boosting quality  |
| Living documentation          | Tests show how code should be used                 |
| Less debugging                | Bugs caught early, before production               |
| Confidence for change         | Can refactor boldly due to regression safety       |

### Notable Disadvantages

Despite its strengths, TDD comes with important caveats and challenges:

- **Time Investment:** Writing tests first, especially for simple features or
  during requirements churn, may slow initial development.
- **Learning Curve/Discipline:** Effective TDD, especially with mocks, stubs,
  and dependency injection, demands significant skill and discipline.
- **Not a Silver Bullet:** TDD finds only the bugs the tests imagine;
  developer-created tests may miss edge cases or share the same
  misunderstandings as the code.
- **Test Maintenance Overhead:** When requirements shift, tests must evolve with
  the code, doubling change management effort.
- **False Sense of Security:** Passing tests can lead to missed issues if tests
  are superficial or miss integration, performance, or compliance concerns.
- **Complex Test Code:** Mocking extensive systems (UIs, databases) can
  introduce complexity or brittle tests.
- **Team/Cultural Resistance:** Organizational buy-in and consistent process are
  critical—partial adoption can fracture design philosophy and QA.

#### Disadvantages Table

| Disadvantage                                 | Explanation                                      |
| -------------------------------------------- | ------------------------------------------------ |
| Increased upfront time                       | Test-writing slows initial pace                  |
| Requires skilled discipline                  | TDD is hard to master; poor tests undo benefits  |
| Tests need maintenance                       | Any code/test change must be synchronized        |
| Can complicate architecture                  | Requires good design, dependency inversion, etc. |
| Ongoing cultural buy-in needed               | Works best with total-team adoption              |
| Not all bugs (esp. integration) found by TDD | Only as thorough as the tests themselves         |

---

## Software Testing Strategies Overview

| Level                | Scope/Description                                | Speed         | Who runs it            |
| -------------------- | ------------------------------------------------ | ------------- | ---------------------- |
| **Unit Testing**     | Isolated functions/classes/modules               | Very Fast     | Developers             |
| **Integration Test** | Interactions between multiple components/modules | Fast–Moderate | Devs & Testers         |
| **System Test**      | Full system as a whole; all integrated modules   | Slower        | QA (independent)       |
| **Acceptance Test**  | Validate against user/business requirements      | Slowest       | Customers/stakeholders |

### 1. Unit Testing

**Definition:** Testing the smallest testable parts of an application
independently (e.g., methods, classes).

- **Focus:** Fast, isolated, deterministic verification—no external network,
  databases, or I/O.
- **Tools:** xUnit (.NET), Jest/Jasmine (JS/TS), NUnit, etc.
- **Scope:** All possible execution paths, including edge cases.

**Best Practices:**

- Fast, repeatable, no external dependencies.
- Name tests clearly (following AAA: Arrange, Act, Assert).
- Each test covers a single responsibility and scenario.
- Use mocks/stubs to replace dependencies.

### 2. Integration Testing

**Definition:** Assess interactions between integrated modules/components (e.g.,
services, databases, APIs).

- **Focus:** Validates correct interface usage, data flow, and workflow between
  units.
- **Types:** Incremental (top-down, bottom-up, sandwich) or Big Bang.
- **Tools:** xUnit (.NET), Karma (Angular), HTTP clients, in-memory servers.

**Best Practices:**

- Use realistic environments/test data.
- Isolate real from fake dependencies (use in-memory stores or containers).
- Test critical paths and error cases across modules.
- Automate test runs in CI/CD.

### 3. System Testing

**Definition:** Testing the fully integrated application as a complete system
against specifications.

- **Focus:** End-to-end workflows, validating complete features and
  behaviors—often via the UI or public API.
- **Types:** Functional, performance, security, usability, compatibility, and
  regression.
- **Responsibility:** Usually independent QA or dedicated testers.

**Best Practices:**

- Use production-like environments.
- Simulate real user flows and edge cases.
- Automate repetitive flows; use manual checks for exploratory and usability
  scenarios.
- Manage defects and regression retesting cycles closely.

### 4. Acceptance Testing

**Definition:** Final verification that the system meets user/business
requirements before release.

- **Focus:** User-centric validation, often by end users or their
  representatives.
- **Types:** UAT (User Acceptance Testing), Alpha, Beta, Contract,
  Regulatory/Compliance.
- **Automation:** Can be partially automated (BDD, Selenium, Cucumber); but many
  acceptance tests are still manual.

**Best Practices:**

- Derive tests from user stories and acceptance criteria.
- Use production-like data and environments.
- Validate not only correctness but also usability and user satisfaction.
- Clearly document scenarios, expected results, and user feedback.

#### Acceptance vs. System Testing Table

| Aspect    | Acceptance Testing                | System Testing                    |
| --------- | --------------------------------- | --------------------------------- |
| Who       | Users, customers, stakeholders    | QA team                           |
| Data/Env  | Real/prod-like data, prod env     | Test data, test/stage env         |
| Main Goal | Verify business/user requirements | Verify all system specs/behaviors |
| Outcome   | Release Go/No-Go, User sign-off   | Defects noted, product readiness  |

---

## Test Doubles: Fake, Mock, and Stub Objects

When writing isolated tests, especially unit tests, it is necessary to **replace
dependencies** (databases, APIs, external services) with stand-ins—known
collectively as **test doubles**. There are important distinctions among
**fakes, mocks, and stubs**:

| Double Type | Purpose                                             | Characteristics                                            |
| ----------- | --------------------------------------------------- | ---------------------------------------------------------- |
| **Fake**    | Functional but substitute version (in-memory impl.) | Works, but takes shortcuts (e.g., in-memory DB, fake SMTP) |
| **Stub**    | Supplies hardcoded/canned responses                 | Does not verify method calls—only state                    |
| **Mock**    | Verifies interactions and expectations              | Pre-programmed, checks calls/parameters for assertions     |

- **Fake**: Used when a working but simpler drop-in is suitable—e.g., in-memory
  repo, fake payment processor.
- **Stub**: Used when test needs controlled outputs from dependencies, without
  verifying how the code uses them.
- **Mock**: Used when you need to assert that certain methods are called (and
  with which parameters), possibly in a specific order.

**Key differences:**

- **Mocks** allow verification and set expectations on how they’re used;
  **stubs** simply return canned data; **fakes** implement simplified real
  logic.
- Usage is often determined by whether you need to check indirect outputs
  (state) or indirect interactions (behavior).

#### Test Doubles Comparison Table

| Fake                                  | Stub                        | Mock                           |
| ------------------------------------- | --------------------------- | ------------------------------ |
| Working impl., simplified for testing | Returns specified responses | Checks if/when/how called      |
| E.g., in-memory repo, fake mailer     | Hardcodes “getUser” return  | Verifies “send” was called     |
| Used for integration-like tests       | Used for state verification | Used for behavior verification |

**In TDD/Unit Testing:**

- Use fakes or stubs for dependencies where behavior is irrelevant.
- Use mocks if you must verify contract/interactions with dependencies (e.g.,
  did the order system send an email?).

---

## White Box vs. Black Box Testing: Methodologies and Use Cases

Software testing strategies fall into two broad categories—**white box** and
**black box**—each with different scopes, strengths, and practitioners.

| Attribute          | **White Box Testing**                   | **Black Box Testing**                        |
| ------------------ | --------------------------------------- | -------------------------------------------- |
| Tester’s Knowledge | Full access to source code              | No access to source code                     |
| Focus              | Internal logic, structure, code paths   | External behavior, requirements              |
| Techniques         | Control flow, code coverage, assertions | Input/Output validation, UI/API, regression  |
| Who Performs       | Developers (often)                      | QA, end users, non-developers                |
| Examples           | Unit, integration, static analysis      | System, acceptance, functional, UI tests     |
| When Used          | During build and code refinement        | Before release or when testing from user POV |

- **White Box Testing:** Testers design cases to cover all code paths, edge
  cases, and internal error handling. They use code coverage tools and often run
  tests as part of CI/CD pipelines. Applicable to unit and lower-level
  integration tests.
- **Black Box Testing:** Testers verify application behavior, API contracts, and
  workflows without knowing implementation. Used heavily for acceptance, system,
  and functional testing. Focuses on user experience, security, performance, and
  external correctness.

**Together**, these methodologies ensure both internal robustness and external
user-value alignment.

---

## xUnit Core Features

### xUnit Attributes and Assertions

xUnit provides several **core attributes**:

- `[Fact]`: For standard, parameterless tests. Run once per method; best for
  classic unit tests.
- `[Theory]`: For data-driven tests. Executes the same test logic against
  multiple input values.
- `[InlineData]`: Supplies parameters for a `[Theory]` with simple data
  (primitives, strings, etc.).
- `[MemberData]` and `[ClassData]`: Feed in parameter sets, supporting complex
  or external data.

**Key assertion methods include**:

- `Assert.Equal(expected, actual)`
- `Assert.True(condition)`
- `Assert.False(condition)`
- `Assert.NotNull(object)`
- `Assert.Throws<T>(Action)` (for exception checking)
- `Assert.Collection`, `Assert.Contains`, `Assert.DoesNotContain`, etc., for
  collection checks

```csharp
[Fact]
public void Add_TwoNumbers_ReturnsSum()
{
    Assert.Equal(5, 2 + 3); // Passes
}

[Theory]
[InlineData(1, 2, 3)]
[InlineData(5, 7, 12)]
public void Add_Theory_Test(int a, int b, int expected)
{
    Assert.Equal(expected, a + b);
}
```

---

### Data-Driven Testing: Theory, InlineData, MemberData, and ClassData

**`[Theory]`** enables parameterized tests, drastically reducing duplication and
improving coverage.

- **`[InlineData]`**: Best for simple literals (numbers, strings)
- **`[MemberData]`**: Use class/static methods or properties for more
  flexible/complex data (including objects)
- **`[ClassData]`**: For reusable data providers, e.g., from CSV, XML, large
  sets

#### InlineData

```csharp
[Theory]
[InlineData(3, 2, 1)]
[InlineData(6, 4, 2)]
public void Subtract_Test(int result, int a, int b)
{
    Assert.Equal(result, a - b);
}
```

#### MemberData

```csharp
public static IEnumerable<object[]> Numbers =>
    new List<object[]>
    {
        new object[] {2, 2, 0},
        new object[] {4, 2, 2}
    };

[Theory]
[MemberData(nameof(Numbers))]
public void Subtract_Test(int expected, int a, int b)
{
    Assert.Equal(expected, a - b);
}
```

#### ClassData

```csharp
public class NumberData : IEnumerable<object[]>
{
    public IEnumerator<object[]> GetEnumerator()
    {
        yield return new object[] { 3, 1, 2 };
        yield return new object[] { 7, 3, 4 };
    }
    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}

[Theory]
[ClassData(typeof(NumberData))]
public void Add_Test(int expected, int a, int b)
{
    Assert.Equal(expected, a + b);
}
```

For strong typing and better intellisense, consider using `TheoryData<T1, T2,
...>`, especially with xUnit v3.

---

### Test Fixtures and Shared Context: IDisposable, IAsyncLifetime, IClassFixture, ICollectionFixture

#### Per-Test Setup/Teardown

xUnit creates a new instance of each test class per test, so place setup/new
object creation in the **constructor** and shared clean-up (e.g., resource
disposal) in the `Dispose()` method via `IDisposable`. For async initialization,
implement `IAsyncLifetime` (`InitializeAsync()`/`DisposeAsync()`).

```csharp
public class MyTests: IDisposable
{
    private readonly Resource _res;
    public MyTests() { _res = new Resource(); }
    public void Dispose() { _res.Dispose(); }

    [Fact]
    public void FooTest() { /* ... */ }
}
```

#### Class Fixture

Use `IClassFixture<TFixture>` to **share expensive setup** (e.g., DB, http
server) across all tests in a class:

```csharp
public class DatabaseFixture : IDisposable
{
    public DbConnection Connection { get; }
    public DatabaseFixture() { /* Connect, seed DB, etc. */ }
    public void Dispose() { /* Cleanup */ }
}

public class ServiceTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;
    public ServiceTests(DatabaseFixture fixture) { _fixture = fixture; }
}
```

#### Collection Fixture

Sharing a fixture instance _across multiple test classes_ (e.g., integration
tests sharing the same server):

```csharp
[CollectionDefinition("DatabaseCollection")]
public class DatabaseCollection : ICollectionFixture<DatabaseFixture> { }

[Collection("DatabaseCollection")]
public class FirstClass
{
    DatabaseFixture _fixture;
    public FirstClass(DatabaseFixture fixture) { _fixture = fixture; }
}
[Collection("DatabaseCollection")]
public class SecondClass { /*...*/ }
```

**Best practices**:

- Only use shared state for _read-only_ or test-isolated resources
- Implement `IDisposable` or `IAsyncLifetime` for cleanup
- Prefer class fixture per test class to avoid shared state issues unless you
  need cross-class coordination (integration seed DB, etc.)

---

### xUnit Test Naming, Organization, and Best Practices

- **Descriptive test names**: Use `MethodUnderTest_Scenario_ExpectedResult`
  style or Given_When_Then for instant readability (optionally customized using
  `[Fact(DisplayName="My custom test name")]`)
- **Arrange-Act-Assert (AAA) pattern**: Every test should clearly distinguish
  setup, execution, and assertion phases for clarity and maintainability
- **One assertion per test (ideally)**: Each test should check one
  behavior/consequence only. Multiple asserts in one test can obscure failures
- **Isolated tests**: No test should depend on another’s outcome or execution
  order
- **Naming configuration**: In `xunit.runner.json`, set `"methodDisplay":
"method"`, and optionally `"replaceUnderscoreWithSpace"` for display purposes
- **Helper methods over inheritance**: Favor composition for setup code to make
  tests easier to follow

**Example of clear, intention-revealing naming**:

```csharp
[Fact(DisplayName = "CalculateTotal returns correct total without discount")]
public void CalculateTotal_ShouldReturnCorrectTotal_WithoutDiscount() { /* ... */ }
```

---

## Unit Testing ASP.NET Core Controllers and Services

### Mocking Dependencies with Moq

Unit tests must isolate the code under test from its dependencies (e.g., DB,
external APIs). **Moq** is the de facto standard .NET mocking library, providing
a fluent API to set up expected behaviors, return values, and verify
interactions.

**Install**:

```bash
dotnet add package Moq
```

**Basic Service Test Example**:

Suppose you have a repository interface and a service:

```csharp
public interface IEmployeeRepository
{
    Task<Employee> GetByIdAsync(int id);
}
public class EmployeeService
{
    private readonly IEmployeeRepository _repo;
    public EmployeeService(IEmployeeRepository repo) { _repo = repo; }
    public Task<Employee> FetchEmployee(int id) => _repo.GetByIdAsync(id);
}
```

**Unit Test**:

```csharp
public class EmployeeServiceTests
{
    private readonly Mock<IEmployeeRepository> _repoMock;
    private readonly EmployeeService _service;

    public EmployeeServiceTests()
    {
        _repoMock = new Mock<IEmployeeRepository>();
        _service = new EmployeeService(_repoMock.Object);
    }

    [Fact]
    public async Task FetchEmployee_ReturnsEmployee()
    {
        var mockEmployee = new Employee { Id = 1, Name = "Alice" };
        _repoMock.Setup(r => r.GetByIdAsync(1))
          .ReturnsAsync(mockEmployee);

        var result = await _service.FetchEmployee(1);

        Assert.Equal("Alice", result.Name);
        _repoMock.Verify(r => r.GetByIdAsync(1), Times.Once);
    }
}
```

**Common Patterns**:

- Setup mock returns via `.Setup(…)`
- Verify calls to dependencies with `.Verify(…)`
- Use `[Theory]` + Moq for scenario coverage

---

## Integration Testing in ASP.NET Core with xUnit

### WebApplicationFactory & TestServer

**Integration tests** in ASP.NET Core exercise app components (controllers,
filters, middleware) _with real_ (in-memory) HTTP requests, but without external
process overhead.

Leverage `WebApplicationFactory<TStartup>` (from
`Microsoft.AspNetCore.Mvc.Testing`):

**Test Project Setup**

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.Mvc.Testing" />
    <!-- ...other packages, e.g., xunit, Moq, EF provider... -->
  </ItemGroup>
</Project>
```

**Sample Test Class**:

```csharp
public class MyControllerIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;
    public MyControllerIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task GetWeather_ReturnsSuccessAndCorrectContentType()
    {
        var response = await _client.GetAsync("/weatherforecast");
        response.EnsureSuccessStatusCode();
        Assert.Equal("application/json; charset=utf-8", response.Content.Headers.ContentType.ToString());
    }
}
```

#### Customization Tips

- **Database setup**: Override `ConfigureWebHost()` to replace real DB with EF
  Core InMemory/SQLite.
- **Auth/Role testing**: Inject custom authentication middleware for tests
- **Test database seeding**: Provide hooks for seeding or resetting DB between
  tests

---

### EF Core InMemory and SQLite Providers for Integration Tests

Options for database-backed integration testing:

- **EF Core InMemory provider**: Fast, easy, but not a true relational database
  (lacks FK constraints, transactions).
- **EF Core SQLite in-memory mode**: Closest possible simulation to real
  production with support for relational constraints and transactions.

**Example**:

```csharp
services.AddDbContext<ApplicationDbContext>(options =>
    options.UseInMemoryDatabase("TestDb"));
```

or

```csharp
services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlite("DataSource=:memory:"));
```

When using SQLite in-memory, ensure the connection is _opened_ and not disposed
between context instances.

---

### Authentication and Authorization in Integration Tests

To simulate authenticated users or specific role claims:

1. **Custom Authentication Handlers**: Add a fake handler to the DI container
   only in test mode:

```csharp
public class TestAuthHandler : AuthenticationHandler<AuthenticationSchemeOptions>
{
    public TestAuthHandler(IOptionsMonitor<AuthenticationSchemeOptions> options, ILoggerFactory logger, UrlEncoder encoder, ISystemClock clock)
        : base(options, logger, encoder, clock) { }

    protected override Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        var claims = new[] { new Claim(ClaimTypes.Name, "TestUser") };
        var identity = new ClaimsIdentity(claims, "TestScheme");
        var principal = new ClaimsPrincipal(identity);
        var ticket = new AuthenticationTicket(principal, "TestScheme");
        return Task.FromResult(AuthenticateResult.Success(ticket));
    }
}
```

2. **Inject in test factory**:

```csharp
factory.WithWebHostBuilder(builder => {
    builder.ConfigureTestServices(services =>
        services.AddAuthentication("TestScheme")
                .AddScheme<AuthenticationSchemeOptions, TestAuthHandler>("TestScheme", o => { }));
});
```

3. **Set the Authorization header in test**:

```csharp
client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("TestScheme");
```

---

### Shared Context and Database Seeding Across Tests

Integration tests often require seeding a known database state before execution.
This is commonly addressed by registering a shared fixture that initializes or
resets the database for classes or collections:

```csharp
public static void InitializeDbForTests(ApplicationDbContext db)
{
    db.Messages.AddRange(new [] {
        new Message { Text = "Hello" },
        new Message { Text = "World" }
    });
    db.SaveChanges();
}
```

Call this method within test setup or collection fixture setup.

---

## System and End-to-End Testing in ASP.NET Core with Playwright

### Playwright Overview and Setup

**Playwright** (official site:
[playwright.dev/dotnet](https://playwright.dev/dotnet/)) is a Microsoft-designed
end-to-end browser automation library supporting Chrome, Firefox, and WebKit. It
can simulate user actions, validate DOM/UI, and cover real user interaction
patterns.

**Install Playwright for .NET**:

```bash
dotnet add package Microsoft.Playwright
npx playwright install
```

or, for the CLI:

```bash
dotnet tool install --global Microsoft.Playwright.CLI
playwright install
```

Integrate with test runner via `Microsoft.Playwright.Xunit` for tight xUnit
integration.

---

### Playwright xUnit Test Sample

An xUnit-based Playwright test project might look like:

```csharp
using Microsoft.Playwright;
using Xunit;

public class BrowserTests : IAsyncLifetime
{
    private IPage _page;
    private IBrowser _browser;

    public async Task InitializeAsync()
    {
        var playwright = await Playwright.CreateAsync();
        _browser = await playwright.Chromium.LaunchAsync();
        _page = await _browser.NewPageAsync();
    }

    public async Task DisposeAsync()
    {
        await _browser?.CloseAsync();
    }

    [Fact]
    public async Task HomePage_LoadsSuccessfully()
    {
        await _page.GotoAsync("http://localhost:5000");
        var content = await _page.ContentAsync();
        Assert.Contains("Welcome", content);
    }
}
```

Key points:

- Use `IAsyncLifetime` to control asynchronous setup and clean-up across test
  classes.
- Best practice: Each test uses a fresh BrowserContext for test isolation and
  reliability.

**Common Patterns**:

- Use `[Fact]` for one-off UI cases, `[Theory]` with URL or state data for
  scenario/table tests.
- Use Playwright's selectors to check for button clicks, field entries,
  navigation, AJAX waits, etc.

---

### Playwright Versus Integration Testing

- **Integration testing** (with WebApplicationFactory): No real browser, only
  in-memory HTTP roundtrips.
- **System/E2E testing** (with Playwright): Real browsers, JS executed,
  CSS/layout/SPA logic validated.

**Playwright in CI**: Can be scripted to run on headless browsers in pipelines,
validating releases at UX level.

---

## Best Practices, Patterns, and Test Organization in xUnit

### Project Structure

- **Separate test projects**: `MyProject.Tests` suiting both unit and
  integration tests, avoid mingling with production code.
- **Organize test files by feature or SUT**:
  `Controllers/OrderControllerTests.cs`, `Services/OrderServiceTests.cs`
- **Data in test sources**: Use `[Theory]`, external data files, or generator
  code to isolate data from assertion logic.

---

### Arrange-Act-Assert (AAA) Pattern

Each test should have clear phases:

- **Arrange**: Set up test data, mocks, and context.
- **Act**: Execute the method under test.
- **Assert**: Verify results.

```csharp
[Fact]
public void Calculate_Discounted_Price()
{
    // Arrange
    var service = new PricingService();
    var product = new Product { Price = 100, Discount = 20 };

    // Act
    var result = service.GetDiscountedPrice(product);

    // Assert
    Assert.Equal(80, result);
}
```

---

### Test Isolation and Shared Setup

- Use `IClassFixture` for class-scoped shared context, `ICollectionFixture` for
  cross-class shared state.
- Clean up shared resources with `IDisposable`/`IAsyncLifetime` implementations.
- Use per-test randomization or test-specific names for test DBs to avoid test
  collisions.

---

### Test Naming, Display, and Reporting

- Name tests to tell a story: `CalculateTotal_Should_ApplyDiscount_WhenProvided`
- For enhanced readability, configure `"replaceUnderscoreWithSpace"` in
  `xunit.runner.json`
- Consider `[Fact(DisplayName="...")]` for documentation-rich reporting when
  running in large pipelines

---

### When to Use Unit, Integration, or System Tests

#### Unit Test

- Isolated logic: business rules, helpers, models, services with only in-memory
  data/manipulation.
- All dependencies are mocked.

#### Integration Test

- Exercise app layers: controller → service → external API/DB (with test DB) as
  a unit.
- Use `WebApplicationFactory` + in-memory servers, optionally swap DB with EF
  Core InMemory/SQLite.

#### System/E2E Test

- Simulate real user; verify routing, cookies, authentication, SPA navigation,
  full backend/frontend stack.
- Use Playwright (or Selenium for legacy).

---

## Full Sample: Layered ASP.NET Core Test Suite

**Example**: Product API

- **Controllers**: Use Moq for repo/service, `[Fact]` and `[Theory]`.
- **Services**: Pure xUnit, heavy on Theory and InlineData.
- **Integration tests**: Use `WebApplicationFactory<Program>`, swap DB provider
  for InMemory/SQLite, seed before runs.
- **System/E2E with Playwright**: Automate product creation, validation, and UI
  flows.

### Sample: Unit Test for Service Method

```csharp
public class DiscountServiceTests
{
    [Theory]
    [InlineData(100, 10, 90)]
    [InlineData(200, 25, 150)]
    public void CalculateDiscountedPrice_ReturnsExpected(decimal price, int discountPercent, decimal expected)
    {
        var service = new DiscountService();
        var result = service.CalculateDiscountedPrice(price, discountPercent);
        Assert.Equal(expected, result);
    }
}
```

### Sample: Controller Unit Test (with Moq)

```csharp
public class ProductsControllerTests
{
    private readonly Mock<IProductService> _serviceMock = new();
    private readonly ProductsController _controller;

    public ProductsControllerTests()
    {
        _controller = new ProductsController(_serviceMock.Object);
    }

    [Fact]
    public async Task GetProduct_ReturnsProduct_WhenExists()
    {
        _serviceMock.Setup(s => s.FindByIdAsync(1)).ReturnsAsync(new Product { Id = 1, Name = "Test" });

        var result = await _controller.Get(1);

        var ok = Assert.IsType<OkObjectResult>(result);
        var product = Assert.IsType<Product>(ok.Value);

        Assert.Equal("Test", product.Name);
    }
}
```

### Sample: Integration Test (WebApplicationFactory, In-Memory DB)

```csharp
public class ProductApiIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;
    public ProductApiIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task PostProduct_CreatesProduct_ReturnsCreated()
    {
        var newProduct = new { Name = "Keyboard", Price = 49.99M };
        var response = await _client.PostAsJsonAsync("/api/products", newProduct);

        Assert.Equal(HttpStatusCode.Created, response.StatusCode);

        var content = await response.Content.ReadFromJsonAsync<Product>();
        Assert.Equal("Keyboard", content.Name);
    }
}
```

### Sample: Playwright System Test

```csharp
public class ShopE2ETests : IAsyncLifetime
{
    private IBrowser _browser;
    private IPage _page;
    public async Task InitializeAsync()
    {
        var playwright = await Playwright.CreateAsync();
        _browser = await playwright.Chromium.LaunchAsync(new BrowserTypeLaunchOptions { Headless = true });
        _page = await _browser.NewPageAsync();
    }
    public async Task DisposeAsync() => await _browser.DisposeAsync();

    [Fact]
    public async Task HomePage_ShowsWelcome()
    {
        await _page.GotoAsync("http://localhost:5000");
        var text = await _page.InnerTextAsync("h1");
        Assert.Equal("Welcome to Shop!", text);
    }
}
```

---

## Jasmine Core Features

### Overview

**Jasmine** is a widely-adopted framework for BDD-style testing in JavaScript.
With its expressive syntax, out-of-the-box assertions (matchers), support for
test doubles (spies), and easy-to-use asynchronous test utilities, Jasmine makes
writing readable, maintainable tests straightforward for Angular developers.

#### Key Features Table

| Feature                | Description                                               | Sample Usage                                  |
| ---------------------- | --------------------------------------------------------- | --------------------------------------------- |
| describe / it          | Group and define test suites / individual cases           | `describe('suite', ()=>{ it('case', ... ) })` |
| beforeEach / afterEach | Setup/teardown hooks for test suites                      | `beforeEach(() => {}), afterEach(() => {})`   |
| expect / Matchers      | Assertions on code behavior                               | `expect(val).toBe(42)`                        |
| Spies                  | Mocking and tracking functions and methods                | `spyOn(obj, 'fn')` or `createSpyObj()`        |
| Async tools            | Handles async code with done, async/await, fakeAsync/tick | See below                                     |

---

### Jasmine Syntax Essentials

#### describe and it

```typescript
describe("MathOperations", () => {
  it("should add two numbers correctly", () => {
    const result = 1 + 2;
    expect(result).toBe(3);
  });
});
```

The `describe` function declares a test suite; `it` defines an individual test
(spec). This structure helps organize your testing by feature, module, or
component.

#### Setup and Teardown: beforeEach and afterEach

```typescript
beforeEach(() => {
  obj = new SomeClass();
});

afterEach(() => {
  obj.cleanup();
});
```

These hooks initialize or clear state before/after each test, improving
isolation and repeatability.

#### Expectations and Matchers

```typescript
expect(value).toBeDefined();
expect(arr).toContain(5);
expect(obj).toEqual({ a: 1, b: 2 });
```

Jasmine includes a broad set of _matchers_:

- `toBe` (strict equality)
- `toEqual` (deep equality)
- `toBeTruthy`, `toBeFalsy`, `toBeNull`, `toBeUndefined`
- `toContain`, `toMatch`, `toHaveBeenCalledWith`, etc.

A more exhaustive list is available [in the Jasmine
docs](https://jasmine.github.io/api/edge/matchers.html).

##### Example Custom Matcher

```typescript
beforeEach(() => {
  jasmine.addMatchers({
    toBeDivisibleBy: function () {
      return {
        compare: function (actual, expected) {
          return { pass: actual % expected === 0 };
        },
      };
    },
  });
});

it("is divisible by 2", function () {
  expect(4).toBeDivisibleBy(2);
});
```

Custom matchers help make tests more expressive.

---

### Spies and Mocking Functions/Objects

Jasmine provides mechanisms for stubbing, spying, or fully faking dependencies.
This is critical for unit testing components or services in isolation.

#### spyOn, createSpy, and createSpyObj

```typescript
const obj = { fetchData: () => {} };

spyOn(obj, "fetchData").and.returnValue("mock data");
obj.fetchData();
expect(obj.fetchData).toHaveBeenCalled();
```

Or, for standalone functions/objects:

```typescript
const fetchSpy = jasmine.createSpy("fetch").and.returnValue("mock");
fetchSpy("url");
expect(fetchSpy).toHaveBeenCalledWith("url");
```

To create a full fake with multiple methods:

```typescript
const apiMock = jasmine.createSpyObj("ApiService", ["save", "remove"]);
apiMock.save.and.returnValue("ok");
```

Spies allow you to:

- Track call count and arguments (`toHaveBeenCalledWith`,
  `toHaveBeenCalledTimes`)
- Substitute return values or throw errors
- Observe side effects and interactions

**Best Practice:** Always keep mock/fake objects’ shape equivalent to the
original; TypeScript types help enforce this.

---

### Asynchronous Testing in Jasmine

Angular code often uses observables, promises, and timeouts. Jasmine supports
asynchronous patterns using:

- The `done` callback for manual async completion;
- Angular’s `async`, `fakeAsync`, `tick`, and `waitForAsync` utilities for
  better control.

#### Example: Promises with done

```typescript
it("fetches data asynchronously", (done) => {
  apiService.getData().then((data) => {
    expect(data).toEqual(expected);
    done();
  });
});
```

#### Angular’s waitForAsync and fakeAsync

For testing Angular code, use:

```typescript
import { fakeAsync, tick, waitForAsync } from "@angular/core/testing";

it("should update after timeout", fakeAsync(() => {
  component.delayedAction();
  tick(1000);
  expect(component.status).toBe("done");
}));
```

With `waitForAsync`, all async actions must resolve before the test completes:

```typescript
it("should render after promise resolved", waitForAsync(() => {
  fixture.whenStable().then(() => {
    expect(fixture.nativeElement.querySelector("p").textContent).toContain(
      "Done"
    );
  });
}));
```

**Tip:** Use `fakeAsync` and `tick` to synchronously flush timers in tests,
avoiding brittle waits.

---

## Karma Core Features

### Overview

**Karma** is the test runner for JavaScript applications, originally created by
the Angular team. It executes Jasmine (or Mocha, QUnit, etc.) tests in real
browsers (Chrome, Firefox, Edge, Safari) or headless environments like
ChromeHeadless, reporting results interactively or in CI pipelines.

#### Key Features Table

| Feature                | Description                                                       |
| ---------------------- | ----------------------------------------------------------------- |
| Real browser execution | Runs tests in actual browsers, including headless/CI environments |
| Automatic watching     | Re-runs tests on file changes during development                  |
| Plugins                | Extensible via reporters, coverage, launchers                     |
| Code coverage          | Integrates with Istanbul for coverage reports                     |
| Multi-browser support  | Run tests across multiple browsers/devices in parallel            |
| CI/CD integration      | Supports single-run, headless, and reporting for automation       |

#### Example Karma Configuration (`karma.conf.js`)

```javascript
module.exports = function (config) {
  config.set({
    basePath: "",
    frameworks: ["jasmine", "@angular-devkit/build-angular"],
    plugins: [
      require("karma-jasmine"),
      require("karma-chrome-launcher"),
      require("karma-coverage"),
      require("karma-jasmine-html-reporter"),
      require("@angular-devkit/build-angular/plugins/karma"),
    ],
    browsers: ["Chrome", "ChromeHeadless"],
    reporters: ["progress", "kjhtml", "coverage"],
    coverageReporter: { type: "html", dir: "coverage/" },
    autoWatch: true,
    singleRun: false,
  });
};
```

This sample config shows Jasmine as the framework, Chrome/ChromeHeadless as
launch browsers, and reporter configuration. Coverage can be output as HTML and
text-summary.

#### Headless and CI Usage

In CI, use headless mode and singleRun:

```bash
ng test --browsers=ChromeHeadless --watch=false
```

Or, for npm:

```json
"scripts": {
  "test": "ng test --browsers=ChromeHeadless --watch=false"
}
```

**Tip:** For GitHub Actions or similar, ensure the appropriate browser launchers
and dependencies are installed.

---

### Common Karma Plugins Table

| Plugin                      | Purpose                                |
| --------------------------- | -------------------------------------- |
| karma-jasmine               | Jasmine framework integration          |
| karma-chrome-launcher       | Launch Chrome/ChromeHeadless browsers  |
| karma-coverage              | Generates code coverage reports        |
| karma-jasmine-html-reporter | Human-friendly in-browser results      |
| karma-teamcity-reporter     | Integration with TeamCity CI system    |
| karma-firefox-launcher      | Launch Firefox for cross-browser tests |

---

## Configuring Karma and Jasmine

### Angular CLI Default Configuration

Angular CLI scaffolds projects with Jasmine and Karma configured out of the box.

- `.spec.ts` files are generated alongside each component, service, pipe, or
  directive.
- The default Karma configuration (`karma.conf.js`) is managed internally, but
  can be customized.

To generate a config:

```bash
ng generate config karma
```

Or create `karma.conf.js` manually.

### Key Config Options Table (Summary)

| Setting            | Purpose                               | Typical Value                  |
| ------------------ | ------------------------------------- | ------------------------------ |
| `frameworks`       | Test frameworks used                  | `['jasmine']`                  |
| `files`            | Files to load, glob patterns          | `['src/**/*.spec.ts']`         |
| `browsers`         | Browsers for running tests            | `['ChromeHeadless']`           |
| `reporters`        | How results are displayed             | `['progress', 'coverage']`     |
| `coverageReporter` | Paths and formats for coverage output | `type: 'html', ...`            |
| `autoWatch`        | Re-run tests on file changes          | `true` for development         |
| `singleRun`        | Run once and exit (CI)                | `false` for dev, `true` for CI |
| `plugins`          | List of plugin modules                | e.g., `karma-jasmine`          |
| `customLaunchers`  | Custom browser launch settings        | ChromeHeadless config          |

---

### Jasmine Configuration

Usually handled via `jasmine.json` or by options set in `karma.conf.js`. Jasmine
is also configured for random test order, stopping on first failure, etc. For
TypeScript, use `tsconfig.spec.json` for spec compilation settings.

**Minimal `tsconfig.spec.json`:**

```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "outDir": "./out-tsc/spec",
    "module": "commonjs",
    "types": ["jasmine", "node"]
  },
  "files": ["test.ts"],
  "include": ["**/*.spec.ts"]
}
```

---

## Angular Testing Utilities

### TestBed, ComponentFixture, and Helpers

Angular provides specialized testing utilities for isolating and instantiating
modules, components, and services.

#### Common Utilities Table

| Utility                           | Description                                           |
| --------------------------------- | ----------------------------------------------------- |
| TestBed                           | Configures and initializes Angular testing module     |
| ComponentFixture                  | Fixture for debugging/testing a component instance    |
| waitForAsync, fakeAsync           | Async helpers for test timing and zone management     |
| tick, flush, discardPeriodicTasks | Async and timer manipulation in tests                 |
| DebugElement                      | Access to internal DOM and component tree             |
| By                                | Helper for querying DebugElements by CSS or directive |
| HttpClientTestingModule           | Fixture for mocking HTTP requests                     |
| inject                            | Retrieve service instance from testing injector       |

#### Example: TestBed and ComponentFixture

```typescript
import { TestBed, ComponentFixture } from "@angular/core/testing";
import { MyComponent } from "./my.component";

describe("MyComponent", () => {
  let fixture: ComponentFixture<MyComponent>;
  let component: MyComponent;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [MyComponent],
    }).compileComponents();
    fixture = TestBed.createComponent(MyComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it("should create", () => {
    expect(component).toBeTruthy();
  });
});
```

TestBed “bootstraps” the component in an isolated module for each test or suite.

---

### Handling Asynchronous Tests

Angular provides:

- **fakeAsync/tick:** Control time in tests using timers (setTimeout, debounce).
- **waitForAsync:** Automatically resolves all pending async code.
- **whenStable:** Waits for fixture/component stability (all async events
  settled).

#### Example: Async Data Fetch

```typescript
it("should display data after fetch", fakeAsync(() => {
  component.fetchData();
  tick(2000); // simulate delay
  fixture.detectChanges();
  expect(fixture.nativeElement.textContent).toContain("result data");
}));
```

**Best Practice:** Prefer `fakeAsync` where possible for maximum determinism.

---

## Unit Testing Angular Components

### Testing Component Classes and Templates

Tests should verify:

- Correct behavior of component methods and state
- Interaction with template (via fixture, DebugElement, By)
- Event handlers and Output emission

#### Sample Counter Component and its Test

`counter.component.ts`

```typescript
import { Component, Input, Output, EventEmitter } from "@angular/core";
@Component({
  selector: "app-counter",
  template: `
    <div>
      <p>Count: {{ count }}</p>
      <button (click)="increment()">Increment</button>
      <button (click)="decrement()">Decrement</button>
    </div>
  `,
})
export class CounterComponent {
  @Input() count = 0;
  @Output() countChange = new EventEmitter<number>();
  increment() {
    this.count++;
    this.countChange.emit(this.count);
  }
  decrement() {
    this.count--;
    this.countChange.emit(this.count);
  }
}
```

`counter.component.spec.ts`

```typescript
import { ComponentFixture, TestBed } from "@angular/core/testing";
import { By } from "@angular/platform-browser";
import { CounterComponent } from "./counter.component";

describe("CounterComponent", () => {
  let fixture: ComponentFixture<CounterComponent>;
  let component: CounterComponent;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [CounterComponent],
    }).compileComponents();

    fixture = TestBed.createComponent(CounterComponent);
    component = fixture.componentInstance;
    component.count = 0;
    fixture.detectChanges();
  });

  it("should create", () => {
    expect(component).toBeTruthy();
  });

  it("should increment on button click", () => {
    const incBtn = fixture.debugElement.query(By.css("button:first-of-type"));
    incBtn.triggerEventHandler("click", null);
    fixture.detectChanges();
    expect(component.count).toBe(1);
  });

  it("should emit countChange when incremented", () => {
    let emitted: number;
    component.countChange.subscribe((value) => (emitted = value));
    component.increment();
    expect(emitted).toBe(1);
  });
});
```

This test covers state changes, user actions, and Output events.

---

### Testing Components in Relative Isolation

You can test component classes _without_ TestBed for pure logic, but won’t cover
template/DOM interaction. For true unit tests, instantiate the class directly:

```typescript
it("should calculate bonus without TestBed", () => {
  const comp = new BonusComponent();
  expect(comp.calculateBonus(2)).toBe(20);
});
```

**Note:** This approach skips dependency injection, provider, and template
logic.

---

## Unit Testing Angular Services

Services generally contain business logic and interact with external APIs.

#### Sample Service

`greeting.service.ts`

```typescript
import { Injectable } from "@angular/core";
@Injectable({ providedIn: "root" })
export class GreetingService {
  getGreeting(name: string): string {
    return `Hello, ${name}!`;
  }
}
```

`greeting.service.spec.ts`

```typescript
import { GreetingService } from "./greeting.service";
describe("GreetingService", () => {
  it("returns greeting", () => {
    const service = new GreetingService();
    expect(service.getGreeting("World")).toBe("Hello, World!");
  });
});
```

**Best Practice:** When possible, instantiate stateless services directly.

#### Testing Services with Angular’s Dependency Injection

```typescript
import { TestBed } from "@angular/core/testing";
import { GreetingService } from "./greeting.service";
describe("GreetingService with DI", () => {
  let service: GreetingService;
  beforeEach(() => {
    TestBed.configureTestingModule({ providers: [GreetingService] });
    service = TestBed.inject(GreetingService);
  });
  it("should be created", () => {
    expect(service).toBeTruthy();
  });
});
```

Use the TestBed to get the correct DI setup and mock providers.

---

## Angular Integration Testing

Integration tests verify correct interaction between 2 or more units (e.g.,
component + service, parent + child), often using mocked dependencies to isolate
third-party or infrastructure layers, but keeping internal Angular logic
present.

### Example: Component and Service

Assume `GreetingComponent` depends on `GreetingService`:

`greeting.component.ts`

```typescript
import { Component, OnInit } from "@angular/core";
import { GreetingService } from "./greeting.service";

@Component({
  selector: "app-greeting",
  template: `<p>{{ greeting }}</p>`,
})
export class GreetingComponent implements OnInit {
  greeting: string;
  constructor(private greetingService: GreetingService) {}
  ngOnInit(): void {
    this.greeting = this.greetingService.getGreeting("World");
  }
}
```

`greeting.component.spec.ts`

```typescript
import { render } from "@angular/testing-library";
import { GreetingComponent } from "./greeting.component";
import { GreetingService } from "./greeting.service";

describe("GreetingComponent", () => {
  it("should display the greeting", async () => {
    const component = await render(GreetingComponent, {
      providers: [GreetingService],
    });
    expect(component.getByText("Hello, World!")).toBeTruthy();
  });
});
```

This test uses the real service, thus integrating component and service logic.

---

### Integration Testing with Mocked Service

```typescript
const greetingServiceMock = {
  getGreeting: (name: string) => "Bye, " + name,
};

beforeEach(async () => {
  await TestBed.configureTestingModule({
    declarations: [GreetingComponent],
    providers: [{ provide: GreetingService, useValue: greetingServiceMock }],
  }).compileComponents();
});

it("should show mocked greeting", () => {
  const fixture = TestBed.createComponent(GreetingComponent);
  fixture.detectChanges();
  expect(fixture.nativeElement.textContent).toContain("Bye, World");
});
```

This proves interaction, but isolates external dependencies.

---

## System Testing (High-level/End-to-End Simulation) with Jasmine and Karma

System tests (sometimes called e2e or acceptance tests) simulate full
application behavior—checking UI, user flows, and network interactions as users
do. Though Protractor or Cypress is used for robust e2e, you can perform shallow
system-level checks in component/unit tests using Jasmine/Karma.

#### Simulating System Flow in TestBed

Suppose a user logs in, fetches data, and performs a workflow. Mock all HTTP and
service dependencies to keep tests deterministic and fast.

```typescript
import { TestBed, fakeAsync, tick } from "@angular/core/testing";
import {
  HttpClientTestingModule,
  HttpTestingController,
} from "@angular/common/http/testing";
import { AppComponent } from "./app.component";
import { AuthService } from "./auth.service";

describe("AppComponent System Test", () => {
  let fixture;
  let httpMock: HttpTestingController;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [AppComponent],
      imports: [HttpClientTestingModule],
      providers: [AuthService],
    }).compileComponents();
    fixture = TestBed.createComponent(AppComponent);
    httpMock = TestBed.inject(HttpTestingController);
  });

  it("should perform login and data fetch workflow", fakeAsync(() => {
    fixture.componentInstance.login("user", "pass");
    const loginReq = httpMock.expectOne("/api/login");
    loginReq.flush({ token: "abc" });

    fixture.componentInstance.fetchData();
    const dataReq = httpMock.expectOne("/api/data");
    dataReq.flush([{ id: 1, name: "Test" }]);
    tick();
    fixture.detectChanges();

    expect(fixture.nativeElement.textContent).toContain("Test");
  }));
});
```

This checks the integration flow, including real events, but all backends are
mocked.

---

## Mocking Dependencies in Angular Unit Tests

### Jasmine Spies and Angular TestBed for Mocking

For unit isolation, always mock dependencies that perform:

- HTTP/REST API calls
- Routing/navigation
- External libraries/utilities
- State, store, or services not under test

#### Spying on Methods

```typescript
const loggerSpy = jasmine.createSpyObj("LoggerService", ["log"]);
service = new SomeService(loggerSpy);
// Now loggerSpy.log can be tracked for calls, arguments, etc.
```

#### Providing Mocks in TestBed

```typescript
TestBed.configureTestingModule({
  providers: [{ provide: AuthService, useValue: authServiceMock }],
});
```

#### Faking Observables and Promises

```typescript
spyOn(service, "fetchData").and.returnValue(of([])); // RxJS Observable
spyOn(service, "getValue").and.returnValue(Promise.resolve("mocked"));
```

#### HTTP Mock Example (HttpClientTestingModule)

```typescript
import { HttpTestingController } from "@angular/common/http/testing";

let httpMock: HttpTestingController;
beforeEach(() => {
  TestBed.configureTestingModule({
    imports: [HttpClientTestingModule],
  });
  httpMock = TestBed.inject(HttpTestingController);
});

it("should mock GET request", () => {
  service.getData().subscribe((data) => {
    expect(data).toEqual([{ id: 1 }]);
  });
  const req = httpMock.expectOne("/api/data");
  req.flush([{ id: 1 }]); // Responds with mock
});
```

**Always call `httpMock.verify()` in `afterEach` to prevent memory leaks or
unclosed connections.**

---

## Testing HTTP Calls

### Mock HTTP Calls—Best Practice

Use Angular’s `HttpClientTestingModule` and `HttpTestingController` for
unit/integration tests involving HTTP:

```typescript
import {
  HttpClientTestingModule,
  HttpTestingController,
} from "@angular/common/http/testing";
import { TestBed } from "@angular/core/testing";

beforeEach(() => {
  TestBed.configureTestingModule({
    imports: [HttpClientTestingModule],
  });
});

it("should intercept HTTP calls", () => {
  const service = TestBed.inject(UserService);
  const httpMock = TestBed.inject(HttpTestingController);

  service.getUsers().subscribe((users) => {
    expect(users).toEqual([{ id: 1, name: "Alice" }]);
  });

  const req = httpMock.expectOne("/api/users");
  expect(req.request.method).toBe("GET");
  req.flush([{ id: 1, name: "Alice" }]);
});
```

Covers GET, POST, PUT, DELETE, and custom headers/query params.

---

### Error Handling in HTTP Tests

```typescript
it("should handle HTTP error gracefully", () => {
  service.getUsers().subscribe({
    next: () => fail("Should have failed"),
    error: (err) => expect(err.status).toBe(404),
  });

  const req = httpMock.expectOne("/api/users");
  req.flush("Not Found", { status: 404, statusText: "Not Found" });
});
```

Always test both success and failure paths.

---

## Structuring and Organizing Tests in Angular

### Directory and File Structure Best Practices

A clean, consistent test file structure helps maintainability and
discoverability.

#### Recommended Directory Structure

```
src/
  app/
    components/
      my-component/
        component.ts
        component.html
        component.spec.ts
    services/
      my-service/
        service.ts
        service.spec.ts
    pipes/
      my-pipe/
        pipe.ts
        pipe.spec.ts
    directives/
      my-directive/
        directive.ts
        directive.spec.ts
  shared/
    ... (same pattern)
  core/
    ...
```

- Place `.spec.ts` files _next to_ their implementations.
- Integration or system tests go in a top-level `tests/` or feature-specific
  folder.

#### File Naming

| Purpose   | Implementation File | Test File            |
| --------- | ------------------- | -------------------- |
| Component | my-component.ts     | my-component.spec.ts |
| Service   | my-service.ts       | my-service.spec.ts   |
| Pipe      | my-pipe.ts          | my-pipe.spec.ts      |
| Guard     | auth-guard.ts       | auth-guard.spec.ts   |

#### Test Organization

- Use `describe` blocks to group related specs.
- Test one feature/unit per file for clarity.
- For shared/test utility helpers, place in `/src/test-utils/` or similar.

### Sample Organization Table

| Folder      | Content                                  | Convention               |
| ----------- | ---------------------------------------- | ------------------------ |
| components/ | UI Components & their tests              | One folder per component |
| services/   | Services & their tests                   | One folder per service   |
| pipes/      | Pipes & their tests                      | One folder per pipe      |
| directives/ | Directives & their tests                 | One folder per directive |
| shared/     | Reusable (pure) components, pipes, utils | As above, test colocated |
| tests/      | System/integration/e2e test glue code    | By feature or test type  |

**Best Practice:** Organize by feature domain when possible (“feature modules”),
not technical type.

---

## Advanced and Best Practices for Angular Testing

### Best Practice Summary Table

| Practice                                | Reason/Benefit                                               |
| --------------------------------------- | ------------------------------------------------------------ |
| Test only your code, not Angular itself | Avoid redundant/costly coverage; focus on unique logic       |
| Mock dependencies                       | Isolate unit under test, control side effects/behavior       |
| Don’t over-mock                         | Use real objects where possible, mock only for isolation     |
| Keep tests atomic                       | One behavior/assertion per test for clarity, maintainability |
| Test both happy and unhappy paths       | Catch regressions/edge cases                                 |
| Use helpers for repeated setup          | DRY—avoid copy/paste fatigue in test files                   |
| Avoid testing private methods           | Focus on public API/use, not internals                       |
| Favor fakeAsync over real timers        | Determinism, flakiness reduction                             |
| Organize by feature/domain              | Easier scaling, onboarding, code tracking                    |
| Integrate coverage in CI pipelines      | Maintain high, actionable coverage; catch missing cases      |
| Run tests in headless mode on CI        | Fast, scalable, no manual browser dependency                 |

### Example: Setting up Headless CI Testing

Update `package.json` as follows:

```json
"scripts": {
  "test": "ng test --browsers=ChromeHeadless --watch=false --code-coverage",
  "test:prod": "ng test --browsers=ChromeHeadless --watch=false --code-coverage"
}
```

Or configure `karma.conf.js`:

```javascript
browsers: ['Chrome', 'ChromeHeadless'],
customLaunchers: {
  ChromeHeadlessCI: {
    base: 'ChromeHeadless',
    flags: ['--no-sandbox']
  }
}
```

And in your CI workflow:

```sh
ng test --browsers=ChromeHeadless --watch=false --no-progress
```

### Integrating and Viewing Code Coverage

- After running `ng test`, view the HTML report at `coverage/index.html`.
- Set thresholds in `karma.conf.js` to fail builds when insufficient areas are
  covered.

---

## Summary Table: Jasmine and Karma Feature Comparison

| Feature                | Jasmine                            | Karma                                              |
| ---------------------- | ---------------------------------- | -------------------------------------------------- |
| Test suite declaration | describe, it                       | Runs Jasmine suites in browser(s)                  |
| Assertions             | expect(), toBe(), toEqual(), etc.  | Evaluates and reports via plugins                  |
| Spies                  | spyOn(), createSpyObj()            | Used within Jasmine; tracked/reported in browser   |
| Async testing          | done, async/await, fakeAsync, tick | Integrates seamlessly with Jasmine's async methods |
| Code coverage          | -                                  | Via karma-coverage plugin                          |
| Plugin architecture    | -                                  | Reporters, launchers, preprocessors                |
| CI support             | -                                  | Headless launch, singleRun for automation          |
| Test file auto-run     | -                                  | Auto-watches files/dependencies                    |

---

## Sample Complete Unit, Integration, and System Tests

### Unit Test (Isolated)

Service logic, no Angular involved:

```typescript
describe("CalculatorService (Isolated Test)", () => {
  let calculator: CalculatorService;
  beforeEach(() => (calculator = new CalculatorService()));
  it("adds numbers", () => expect(calculator.add(2, 2)).toBe(4));
});
```

### Unit Test (with Angular/DI)

Component tested with Angular DI and template:

```typescript
describe("MyComponent", () => {
  let fixture: ComponentFixture<MyComponent>;
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [MyComponent],
    }).compileComponents();
    fixture = TestBed.createComponent(MyComponent);
    fixture.detectChanges();
  });
  it("should show correct title", () => {
    expect(fixture.nativeElement.querySelector("h1").textContent).toContain(
      "My Title"
    );
  });
});
```

### Integration Test

Component + Service with/sans mocks:

```typescript
describe("LoginComponent", () => {
  let fixture: ComponentFixture<LoginComponent>;
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [LoginComponent],
      providers: [AuthService],
    }).compileComponents();
    fixture = TestBed.createComponent(LoginComponent);
  });
  it("should login and show welcome", () => {
    // Optionally, spy on AuthService.login
    jest
      .spyOn(fixture.componentInstance.authService, "login")
      .mockReturnValue(of(true));
    fixture.componentInstance.login("user", "pass");
    fixture.detectChanges();
    expect(fixture.nativeElement.textContent).toContain("Welcome user");
  });
});
```

### System Test (Full Flow)

Simulate HTTP call, user event, and multi-unit integration:

```typescript
describe("CompleteAppFlow", () => {
  let fixture: ComponentFixture<AppComponent>;
  let httpMock: HttpTestingController;
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [AppComponent],
      imports: [HttpClientTestingModule],
      providers: [UserService],
    }).compileComponents();
    fixture = TestBed.createComponent(AppComponent);
    httpMock = TestBed.inject(HttpTestingController);
  });
  it("loads user profile on start", () => {
    fixture.detectChanges();
    const req = httpMock.expectOne("/api/profile");
    req.flush({ name: "Alice" });
    fixture.detectChanges();
    expect(fixture.nativeElement.textContent).toContain("Alice");
  });
  afterEach(() => httpMock.verify());
});
```
