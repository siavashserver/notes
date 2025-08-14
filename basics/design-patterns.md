---
title: Design Patterns
---

## Design Pattern Categories

| Category   | What It Solves                               | Typical Patterns                       | When to Apply                                                  |
| ---------- | -------------------------------------------- | -------------------------------------- | -------------------------------------------------------------- |
| Creational | How objects are created                      | Singleton, Builder, Prototype, Factory | To make initialization flexible or delayed                     |
| Structural | How classes/objects are organized or related | Adapter, Decorator, Composite, Facade  | To alter or simplify class relationships or interfaces         |
| Behavioral | How objects communicate and coordinate       | Observer, Strategy, Command, Iterator  | To manage responsibility, decouple logic, or support undo/redo |

### Creational Patterns — _(Object construction mechanics)_

- **Intent**: Abstract and control how objects are created to boost flexibility
  and decouple clients from concrete types.
- **Use When**:

  - A system must remain independent of its object‐creation logic.
  - You need to build different representations of the same object step by step.
  - Only one (or a limited number of) instance(s) should be created (e.g.
    Singleton / Multiton).

### Structural Patterns — _(Class/object composition)_

- **Intent**: Organize and compose classes and objects to form larger structures
  that are flexible, efficient, and maintainable.
- **Use When**:

  - You want to hide the complexity of long dependency chains via a simplified
    facade.
  - You need to augment behavior dynamically without subclassing (e.g.
    Decorator).
  - You're managing many small, reusable objects in memory (e.g. Flyweight).

### Behavioral Patterns — _(Object interaction and communication)_

- **Intent**: Define how objects should collaborate and assign responsibilities
  clearly, enabling dynamic behavior and loose coupling.
- **Use When**:

  - Objects must react to events/state changes (Observer).
  - You want to encapsulate requests as objects to allow queuing or undo
    (Command).
  - You need to traverse elements in a collection without exposing its internal
    structure (Iterator).

---

## Creational Design Patterns

Creational patterns abstract the process of object creation, making your systems
more flexible regarding _what_ to build, _how_, _who_, and _when_. They reduce
coupling between client code and concrete types.

| Pattern          | Use Case                                         | Key Benefit                                | Key Drawback                      |
| ---------------- | ------------------------------------------------ | ------------------------------------------ | --------------------------------- |
| Singleton        | Unique shared resource (e.g. logger)             | Single instance, global access             | Hidden coupling, poor testability |
| Factory Method   | Family of objects instantiated by subclass       | Decouples creation from use                | Class explosion                   |
| Abstract Factory | Building related objects per “family” (themes)   | Ensures compatible product sets            | Boilerplate and rigidity          |
| Builder          | Stepwise assembly of complex objects             | Fluent, flexible, readable construction    | More classes; mutable builder     |
| Prototype        | Clone-and-transform rather than new from scratch | Efficient duplication, runtime flexibility | Needs careful clone logic         |

### Singleton

Ensures that a class has only one instance and provides a global access point to
it.

```csharp
public sealed class Logger
{
    private static readonly Lazy<Logger> _instance
      = new Lazy<Logger>(() => new Logger());

    public static Logger Instance => _instance.Value;

    private Logger() { /* private constructor */ }

    public void Log(string message)
        => Console.WriteLine($"[{DateTime.Now:HH:mm:ss}] {message}");
}
```

#### ✅ Advantage

- Single global point of access and a single instance guarantees efficient
  resource use.
- Supports _lazy initialization_—the instance isn’t created until it’s needed.

#### ❌ Drawbacks

- Introduces dependency hiding and tight coupling, making unit testing or
  subclassing difficult.
- Violates _Single Responsibility Principle_—it handles both logic and
  lifecycle.
- Risks global mutable state leading to side‑effects and concurrency issues if
  not carefully handled.

#### When to Use

- One-and-only service like logging, configuration, print spooler.
- Legacy systems where true DI isn’t possible.

#### Interview Questions

- **How would you implement a thread-safe, lazy Singleton in C#?** Use `Lazy<T>`
  or double-checked locking with a `private static readonly` instance and a
  private constructor.

- **Why is Singleton considered harmful in large systems?** It hides
  dependencies, causes tight coupling, breaks SRP, and is hard to mock or
  replace—especially in unit tests.

- **What’s a better alternative to writing your own Singleton in modern C#?**
  Use **Dependency Injection (DI)** and register the service with singleton
  lifetime: it’s testable, clear about dependencies, and still instanced just
  once.

### Factory Method

Defines an interface for creating objects, but lets subclasses decide which
concrete class to instantiate.

```csharp
public interface IVehicle
{
    void Drive();
}

public class Car : IVehicle
{
    public void Drive() => Console.WriteLine("Driving a car");
}

public class Bike : IVehicle
{
    public void Drive() => Console.WriteLine("Riding a bike");
}

// Factory Method
public abstract class VehicleFactory
{
    public abstract IVehicle Create();
}

public class CarFactory : VehicleFactory
{
    public override IVehicle Create() => new Car();
}

public class BikeFactory : VehicleFactory
{
    public override IVehicle Create() => new Bike();
}
```

#### ✅ Advantages

- Decouples client from specific types; adds new types without modifying the
  creator.

#### ❌ Disadvantages

- Adds complexity with many small creator classes.
- Needs subclassing for each new type, may lead to class explosion.

#### Use Cases

- When a superclass should defer instantiation to subclasses, e.g. GUI widgets
  across platforms.
- When clients register types dynamically (e.g. plugins).

#### Interview Questions

- **Difference between Factory Method and Abstract Factory?** Factory Method
  creates one kind of product via inheritance. Abstract Factory creates family
  of products via composition and often aggregates multiple factory methods.

- **Why avoid calling `new` directly in production code?** Direct `new` calls
  couple business logic to concrete classes; factories abstract away
  instantiation and improve testability.

### Abstract Factory

Provides an interface to create families of related products without specifying
their concrete classes.

```csharp
// Abstract Products
public interface IButton
{
    void Paint();
}

public interface ITextBox
{
    void Paint();
}

// Concrete Products for a Windows theme
public class WinButton : IButton => Console.WriteLine("WinButton painted");
public class WinTextBox : ITextBox => Console.WriteLine("WinTextBox painted");

// Concrete Products for Mac theme
public class MacButton : IButton => Console.WriteLine("MacButton painted");
public class MacTextBox : ITextBox => Console.WriteLine("MacTextBox painted");

// Abstract Factory
public interface IUIFactory
{
    IButton CreateButton();
    ITextBox CreateTextBox();
}

// Concrete Factory 1 (Windows)
public class WinFactory : IUIFactory
{
    public IButton CreateButton() => new WinButton();
    public ITextBox CreateTextBox() => new WinTextBox();
}

// Concrete Factory 2 (Mac)
public class MacFactory : IUIFactory
{
    public IButton CreateButton() => new MacButton();
    public ITextBox CreateTextBox() => new MacTextBox();
}

// Client
public class Application
{
    private readonly IButton _btn;
    private readonly ITextBox _txt;

    public Application(IUIFactory factory)
    {
        _btn = factory.CreateButton();
        _txt = factory.CreateTextBox();
    }

    public void Render()
    {
        _btn.Paint();
        _txt.Paint();
    }
}
```

#### ✅ Advantages

- Ensures product families are compatible, enforces consistency.
- Allows full swapping of families (e.g. themes or DB providers) without code
  change.

#### ❌ Disadvantages

- Adds boilerplate and layer of abstraction; hard to maintain if product lines
  evolve.

#### Use Cases

- UI libraries for multiple OS styles.
- Switching between implementations like SQL/NoSQL, or payment gateways.

#### Interview Questions

- **How would you extend Abstract Factory when a new product (e.g. `ISlider`) is
  added?** Add a method `CreateSlider()` to the interface and implement it in
  all concrete factories—violates Open/Closed principle unless planned.

- **When would Abstract Factory be overkill?** When only one product type is
  needed; simpler factory method or builder may suffice.

### Builder

Separates construction of a complex object from its representation so the same
construction process can create different representations.

```csharp
public class Report {
    public string Title { get; set; }
    public string Header { get; set; }
    public string Body { get; set; }
    public string Footer { get; set; }
}

public interface IReportBuilder {
    IReportBuilder SetTitle(string title);
    IReportBuilder SetHeader(string header);
    IReportBuilder SetBody(string body);
    IReportBuilder SetFooter(string footer);
    Report Build();
}

public class PdfReportBuilder : IReportBuilder {
    private Report _report = new Report();
    public IReportBuilder SetTitle(string t) { _report.Title = t; return this; }
    public IReportBuilder SetHeader(string h) { _report.Header = h; return this; }
    public IReportBuilder SetBody(string b) { _report.Body = b; return this; }
    public IReportBuilder SetFooter(string f) { _report.Footer = f; return this; }
    public Report Build() => _report;
}

public class ReportDirector {
    private readonly IReportBuilder _builder;
    public ReportDirector(IReportBuilder b) { _builder = b; }
    public void ConstructFullReport()
    {
        _builder
            .SetTitle("Monthly Sales")
            .SetHeader("Sales Header")
            .SetBody("Lots of numbers...")
            .SetFooter("Confidential");
    }
    public Report GetReport() => _builder.Build();
}

// Usage:
var director = new ReportDirector(new PdfReportBuilder());
director.ConstructFullReport();
Report r = director.GetReport();
```

#### ✅ Advantages

- Allows flexible and readable construction of objects with many optional parts.
- Cleanly separates _what_ parts are being built, and by whom.

#### ❌ Disadvantages

- Needing a `ConcreteBuilder` for each configuration leads to boilerplate and
  increased classes.
- Builder often needs to be mutable, which can lead to partially built states.

#### Use Cases

- Configuring domain objects with many optional parts, e.g. complex documents,
  UI components, HTTP requests.

#### Interview Questions

- **What's the difference between telescoping constructors and Builder?**
  Telescoping constructors use overloaded constructors to set various
  options—with many parameters, readability and maintenance suffer; Builder uses
  step-by-step fluent API.

- **Is Builder always needed in C# with optional and named args?** Not always;
  builder shines in scenarios where steps are conditional, cumulative, or
  order-dependent.

### Prototype

Creates new objects by copying an existing _prototype_, thereby avoiding costly
creation logic.

```csharp
public interface IEmployeePrototype
{
    IEmployeePrototype Clone();
}

public class EmployeeProfile : IEmployeePrototype
{
    public string Name { get; set; }
    public string Role { get; set; }
    public decimal Salary { get; set; }

    public IEmployeePrototype Clone()
        => new EmployeeProfile
        {
            Name = this.Name,
            Role = this.Role,
            Salary = this.Salary
        };
}

// Client:
var existing = new EmployeeProfile { Name = "Alice", Role = "Dev", Salary = 80000m };
var temp = (EmployeeProfile)existing.Clone();
temp.Name = "Bob"; temp.Role = "Contractor";
Console.WriteLine(temp.Name);
```

Or for more reusable setup, a `PrototypeRegistry` can store base instances to
clone later.

#### ✅ Advantages

- Avoids the cost of full initialization when an object is heavy to create.
- Promotes reuse and variability at runtime.
- No class hierarchy explosion; clients work with clones, not constructors.

#### ❌ Disadvantages

- Deep cloning is hard—problems with circular references, shared resources, and
  consistency.
- Prototype registry adds global state if misused.

#### Use Cases

- Creating many similar but slightly modified objects (UI elements, game
  characters, document templates).
- When object creation is resource-intensive and existing instances can serve as
  a baseline.

#### Interview Questions

- **How do you distinguish shallow vs deep clone? Why does it matter?** Shallow
  clone copies only top-level fields (reference IDs copy). Deep clone replicates
  entire object graph— essential when mutable sub-objects shouldn't be shared.

- **What’s a problem of using Prototype for objects that hold unmanaged
  resources or database handles?** Cloning those resources may create tricky
  bugs or performance issues; better to support resource initialization per
  clone.

---
