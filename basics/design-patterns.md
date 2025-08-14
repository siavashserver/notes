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

#### Basic Singleton

```csharp
public class Singleton
{
    private static Singleton _instance;

    public static Singleton Instance
    {
        get
        {
            if (_instance == null)
                _instance = new Singleton();
            return _instance;
        }
    }

    private Singleton() { }
}
```

#### Thread-Safe with `lock` (Eager Locking)

```csharp
public class Singleton
{
    private static Singleton _instance;
    private static readonly object _lock = new object();

    public static Singleton Instance
    {
        get
        {
            lock (_lock)
            {
                if (_instance == null)
                    _instance = new Singleton();
            }
            return _instance;
        }
    }

    private Singleton() { }
}
```

#### Double-Checked Locking (Lazy and Efficient)

```csharp
public class Singleton
{
    private static Singleton _instance;
    private static readonly object _lock = new object();

    public static Singleton Instance
    {
        get
        {
            if (_instance == null)
            {
                lock (_lock)
                {
                    if (_instance == null)
                        _instance = new Singleton();
                }
            }
            return _instance;
        }
    }

    private Singleton() { }
}
```

#### Static Initialization (Eager and Thread-Safe)

```csharp
public class Singleton
{
    private static readonly Singleton _instance = new Singleton();

    public static Singleton Instance => _instance;

    private Singleton() { }
}
```

#### `Lazy<T>` Singleton (Best Practice in Modern C#)

```csharp
public class Singleton
{
    private static readonly Lazy<Singleton> _instance =
        new Lazy<Singleton>(() => new Singleton());

    public static Singleton Instance => _instance.Value;

    private Singleton() { }
}
```

#### Singleton Using Static Class

```csharp
public static class Singleton
{
    public static void Log(string message)
    {
        Console.WriteLine($"[{DateTime.Now}] {message}");
    }
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

## Structural Patterns

Structural design patterns tackle class and object composition: they help
organize components and interfaces in ways that improve **modularity,
flexibility, and maintainability**.

| Pattern   | Purpose                               | Pros                                     | Cons                                     | Typical Use-Case                  |
| --------- | ------------------------------------- | ---------------------------------------- | ---------------------------------------- | --------------------------------- |
| Adapter   | Match two incompatible interfaces     | Allows reuse, no changes to legacy code  | Extra layer of abstraction               | Legacy API integration            |
| Bridge    | Separate abstraction & implementation | Supports independent extension (SRP/OCP) | More classes, adds complexity            | Cross-platform UI or backend      |
| Composite | Treat parts and wholes uniformly      | Simplifies hierarchical object handling  | Hard to unify divergent leaf types       | File systems, UI trees            |
| Decorator | Dynamically add behavior              | Flexible runtime composition             | Many small classes, complex chains       | Logging, formatting, decoration   |
| Facade    | Simplify complex subsystems           | Clear and simple API for clients         | May become too central, duplicated logic | Wrapping subsystems or libraries  |
| Flyweight | Share common window of data           | Huge memory savings                      | Complexity in sharing and state mgmt     | High-volume identical objects     |
| Proxy     | Control access, lazy load, protection | Access control, efficient object usage   | Can duplicate target interface logic     | Remote services, caches, security |

### Adapter Pattern

Enable incompatible interfaces to work together by wrapping one class into
another expected interface.

```csharp
public interface IBird { void Quack(); }

public class Duck : IBird { public void Quack() => Console.WriteLine("Quack!"); }

public class Turkey { public void Gobble() => Console.WriteLine("Gobble!"); }

public class TurkeyAdapter : IBird {
    private readonly Turkey _turkey;
    public TurkeyAdapter(Turkey turkey) => _turkey = turkey;
    public void Quack() => _turkey.Gobble();
}
```

#### ✅ Advantages

- Decouples client code from legacy or third‑party APIs.
- Allows reuse without modifying existing classes.

#### ❌ Disadvantages

- Adds an extra layer of abstraction.
- May become cluttered if many adaptations are needed.

#### Use Cases

- Interfacing new code with legacy components or third‑party APIs.

#### Interview Questions

- **When should you choose Adapter over Facade?** Use Adapter when you need an
  object to satisfy a specific interface; Facade provides a simplified overall
  interface to a subsystem.

### Bridge Pattern

Decouple an abstraction from its implementation so both can evolve
independently.

```csharp
// Abstraction
public abstract class Remote {
  protected IDevice _device;
  protected Remote(IDevice device)=> _device = device;
  public abstract void TogglePower();
}

// Implementations
public interface IDevice { bool IsEnabled(); void Enable(); void Disable(); }
public class TvDevice : IDevice { /*...*/ }

// Extensions
public class BasicRemote : Remote {
  public BasicRemote(IDevice dev): base(dev){}
  public override void TogglePower(){
    _device.IsEnabled() ? _device.Disable() : _device.Enable();
  }
}
```

#### ✅ Advantages

- Supports independent variation and extensibility.
- Follows SRP and OCP cleanly.

#### ❌ Disadvantages

- More classes and interfaces to manage.
- Can feel heavyweight if unnecessary.

#### Use Cases

- When both abstraction and implementation layers need separate evolution, e.g.
  GUI toolkit vs OS.

#### Interview Questions

- **What problem does Bridge solve?** It prevents combinatorial explosion when
  both abstraction and implementation can vary (e.g. platform and UI type).

### Composite Pattern

Treat individual and composed objects uniformly using a recursive tree
structure.

```csharp
public interface IGraphic { void Draw(); }

public class Circle : IGraphic { public void Draw() => Console.WriteLine("Circle"); }

public class CompositeGraphic : IGraphic {
  private readonly List<IGraphic> _items = new();
  public void Add(IGraphic g) => _items.Add(g);
  public void Draw() { foreach(var g in _items) g.Draw(); }
}
```

#### ✅ Advantages

- Simplifies handling of part–whole hierarchies.
- Uniform interface for leaves and composites.

#### ❌ Disadvantages

- Difficult to design a common interface when leaf and composite differ greatly.
- May permit misuse or overly general APIs.

#### Use Cases

- Graphic editors, file systems, UI hierarchy, document structure.

#### Interview Questions

- **How does Composite help with client code?** Client code can treat leaves and
  aggregations the same, without type checks.

### Decorator Pattern

Add responsibilities to objects dynamically by wrapping them without
subclassing.

```csharp
public interface IComponent { void Operation(); }

public class ConcreteComponent : IComponent {
  public void Operation() => Console.WriteLine("Core Operation");
}

public abstract class Decorator : IComponent {
  protected IComponent _component;
  protected Decorator(IComponent c) => _component = c;
  public virtual void Operation() => _component.Operation();
}

public class DecoratorA : Decorator {
  public DecoratorA(IComponent c): base(c){}
  public override void Operation() { Console.WriteLine("Before"); base.Operation(); }
}
```

#### ✅ Advantages

- Extends behavior flexibly at runtime.
- Composable “slices” of behavior.

#### ❌ Disadvantages

- Many small decorator classes can clutter the design.
- Hard to trace through layers of wrapping.

#### Use Cases

- Logging, UI embellishments, runtime feature toggles.

#### Interview Questions

- **When to use Decorator vs inheritance?** Use Decorator when you want dynamic
  and combinable behavior additions rather than static subclassing.

### Facade Pattern

Provide a unified, simple interface to a complex subsystem.

```csharp
public class Sub1 { public void Op1() => Console.WriteLine("Sub1 ready"); }

public class Sub2 { public void Op2() => Console.WriteLine("Sub2 go"); }

public class Facade {
  private readonly Sub1 _s1; private readonly Sub2 _s2;
  public Facade(Sub1 s1, Sub2 s2){ _s1=s1; _s2=s2;}
  public void Operation() {
    _s1.Op1(); _s2.Op2(); Console.WriteLine("Facade complete");
  }
}
```

#### ✅ Advantages

- Simplifies client interaction.
- Reduces dependencies on subsystem internals.

#### ❌ Disadvantages

- Can become a monolithic god-class if poorly designed.
- Masks subsystem flexibility if not carefully planned.

#### Use Cases

- Simplifying access to complex libraries or legacy subsystems, or exposing a
  clean API boundary.

#### Interview Questions

- **When would you avoid Facade?** When client needs fine‑grained control or
  needs to access many subsystem APIs; a facade may obscure too much.

Absolutely! Let's dive into the **Flyweight** and **Proxy** structural patterns
with detailed explanations, real-world use cases, sample C# code, pros & cons,
and interview-oriented Q\&A.

### Flyweight Pattern

Minimize memory usage by sharing as much data as possible between many objects.
It's ideal when you have **lots of similar objects**.

```csharp
// Shared data: shape, texture, etc.
public class TreeType
{
    public string Name;
    public string Color;
    public string Texture;

    public TreeType(string name, string color, string texture)
    {
        Name = name; Color = color; Texture = texture;
    }

    public void Draw(int x, int y)
    {
        Console.WriteLine($"Drawing {Name} tree at ({x},{y}) with {Texture}");
    }
}

// Factory to manage flyweights
public class TreeFactory
{
    private static Dictionary<string, TreeType> _types = new();

    public static TreeType GetTreeType(string name, string color, string texture)
    {
        string key = $"{name}_{color}_{texture}";
        if (!_types.ContainsKey(key))
            _types[key] = new TreeType(name, color, texture);
        return _types[key];
    }
}

// Context object (extrinsic state)
public class Tree
{
    private int _x, _y;
    private TreeType _type;

    public Tree(int x, int y, TreeType type)
    {
        _x = x; _y = y; _type = type;
    }

    public void Draw() => _type.Draw(_x, _y);
}
```

#### ✅ Advantages

- Huge memory savings when dealing with many similar objects.
- Can improve performance in rendering, simulations, caching.

#### ❌ Disadvantages

- Makes code more complex (you must manage extrinsic/intrinsic split).
- Not useful if object data is too unique.

#### Use Cases

- Large-scale UI elements (buttons, icons).
- Document editors (glyphs/characters).
- Game objects (bullets, trees, tiles).
- Font/character rendering.

#### Interview Questions

- **What problem does the Flyweight pattern solve?** It minimizes memory by
  sharing common state (intrinsic) between many objects and separating
  per-instance state (extrinsic).

- **How do you identify extrinsic vs intrinsic state?** Intrinsic is shared and
  immutable (e.g., texture, shape). Extrinsic is unique per object and passed
  externally (e.g., position).

### Proxy Pattern

Provide a placeholder or surrogate for another object to **control access**, add
**functionality**, or **delay** object creation. Proxy is a structural pattern
that lets you substitute a real object with a special placeholder that controls
access to it.

- **Key Variants:**
  - **Virtual Proxy**: lazy initialization
  - **Remote Proxy**: access object on another machine
  - **Protection Proxy**: restrict access (permissions)
  - **Logging/Smart Proxy**: add extra behavior like caching, audit

```csharp
// Real subject
public interface IImage
{
    void Display();
}

public class RealImage : IImage
{
    private string _filename;
    public RealImage(string filename)
    {
        _filename = filename;
        LoadFromDisk();
    }

    private void LoadFromDisk()
    {
        Console.WriteLine($"Loading {_filename} from disk...");
    }

    public void Display()
    {
        Console.WriteLine($"Displaying {_filename}");
    }
}

// Proxy
public class ImageProxy : IImage
{
    private RealImage _realImage;
    private string _filename;

    public ImageProxy(string filename)
    {
        _filename = filename;
    }

    public void Display()
    {
        if (_realImage == null)
            _realImage = new RealImage(_filename);
        _realImage.Display();
    }
}
```

#### ✅ Advantages

- Control access to expensive or sensitive resources.
- Adds functionality like lazy loading, logging, caching.
- Remote invocation over network (e.g., gRPC stub, WCF proxy).

#### ❌ Disadvantages

- Adds complexity.
- Can hide the actual behavior or cause unexpected side effects if not
  transparent.

#### Use Cases

- Lazy loading large objects (e.g., images, files).
- Database connection or object-relational mappers.
- Authorization checks (security proxy).
- API rate limiting or logging wrapper.

#### Interview Questions

- **When would you use Proxy instead of Decorator?** Decorator _adds behavior_
  to objects. Proxy _controls access_ to the original object—e.g., lazy loading,
  security, or remote access.

- **How does a proxy differ from a facade?** Proxy acts on behalf of a specific
  object, forwarding requests. Facade provides a new interface to a whole
  subsystem.

- **What is a virtual proxy?** A proxy that defers the creation of a heavy
  object until it is actually needed.

---

## Behavioral Patterns

Behavioral patterns deal with **how objects interact and share responsibility**,
aiming to make communication flexible and loosely coupled.

| Pattern         | Use Case                    | Pros                    | Cons                       |
| --------------- | --------------------------- | ----------------------- | -------------------------- |
| Chain of Resp.  | Approval chains, middleware | Loose coupling          | May not handle requests    |
| Command         | UI buttons, undo/redo       | Decoupling, undoable    | Many classes               |
| Interpreter     | DSLs, calculators           | Easy to add grammar     | Hard to scale              |
| Iterator        | Collections                 | Encapsulation           | Performance on big sets    |
| Mediator        | UI coordination, chat       | Simplifies interactions | Can grow complex           |
| Memento         | Undo/redo                   | Preserves encapsulation | Memory overhead            |
| State           | UI states, workflows        | Cleaner state handling  | Many state classes         |
| Strategy        | Algorithms, payment, sort   | Swap at runtime         | Strategy selection logic   |
| Observer        | Event systems, pub/sub      | Dynamic updates         | Memory leaks               |
| Template Method | Processing pipelines        | Code reuse              | Inflexibility              |
| Visitor         | AST traversal, reporting    | Adds behavior easily    | Breaks on structure change |

### Observer Pattern

Define a one-to-many dependency so that when one object changes its state, its
dependents are notified automatically and updated.

```csharp
public interface IObserver { void Update(string message); }

public class Subject {
    private readonly List<IObserver> observers = new();
    public void Attach(IObserver o) => observers.Add(o);
    public void Detach(IObserver o) => observers.Remove(o);
    public void Notify(string msg) { foreach(var o in observers) o.Update(msg); }
}

public class ConcreteObserver : IObserver {
    private readonly string name;
    public ConcreteObserver(string n) => name = n;
    public void Update(string message) => Console.WriteLine($"{name} received: {message}");
}
```

#### ✅ Advantages

- Loose coupling: Subject doesn’t know concrete observers.
- Dynamic subscription/unsubscription.

#### ❌ Disadvantages

- Requires careful lifecycle management (prevent memory leaks).
- Ordering and concurrency can be tricky.

#### Use Cases

- GUI event listeners, publishing state changes (e.g., stock price updates).

#### Interview Questions

- **How does Observer maintain loose coupling?** Subject only knows observers
  via interface, not concrete types.
- **How to avoid memory leaks in Observer?** Observers should detach themselves
  or use weak references.

### Strategy Pattern

Define a family of interchangeable algorithms or behaviors and select one at
runtime.

```csharp
public interface ICompressionStrategy { void Compress(string file); }

public class ZipCompression : ICompressionStrategy {
    public void Compress(string file) => Console.WriteLine($"Compressing {file} as .zip");
}

public class RarCompression : ICompressionStrategy {
    public void Compress(string file) => Console.WriteLine($"Compressing {file} as .rar");
}

public class CompressionContext {
    private ICompressionStrategy strategy;
    public void SetStrategy(ICompressionStrategy strat) => strategy = strat;
    public void Archive(string file) => strategy.Compress(file);
}
```

#### ✅ Advantages

- Swap behavior dynamically.
- Supports open/closed principle — new strategies without modifying context.

#### ❌ Disadvantages

- Increases number of strategy classes.
- Strategy selection logic must be maintained externally.

#### Use Cases

- Payment methods, serialization options, sorting algorithms.

#### Interview Questions

- **When to use Strategy vs. State?** Strategy encapsulates interchangeable
  algorithms; State encapsulates state-based behavior.
- **Why composition over inheritance here?** It allows runtime flexibility and
  reduces subclass explosion.

### Template Method Pattern

Define the skeleton of an algorithm in a base class, deferring some steps to
subclasses.

```csharp
public abstract class DataProcessor {
    public void Process() {
        Read();
        ProcessCore();
        Save();
    }
    protected abstract void Read();
    protected abstract void ProcessCore();
    protected abstract void Save();
}

public class XmlProcessor : DataProcessor {
    protected override void Read() => Console.WriteLine("Read XML");
    protected override void ProcessCore() => Console.WriteLine("Process XML");
    protected override void Save() => Console.WriteLine("Save XML");
}

public class JsonProcessor : DataProcessor {
    protected override void Read() => Console.WriteLine("Read JSON");
    protected override void ProcessCore() => Console.WriteLine("Process JSON");
    protected override void Save() => Console.WriteLine("Save JSON");
}
```

#### ✅ Advantages

- Enforces algorithm structure and code reuse.
- Reduces duplication across similar workflows.

#### ❌ Disadvantages

- Inflexible: algorithm order fixed in base class.
- Adding new steps may require changing base class.

#### Use Cases

- File processing pipelines, report generation flows, setup procedures.

#### Interview Questions

- **What happens if XML requires an extra validation step?** You’d override or
  extend in subclass—but base structure stays intact.
- **How does Template support code reuse?** Common steps reside in the abstract
  class; subclasses only implement variable steps.

## Chain of Responsibility

Pass a request along a chain of handlers. Each handler decides either to handle
it or pass it to the next.

```csharp
abstract class Handler {
    protected Handler next;
    public void SetNext(Handler handler) => next = handler;
    public abstract void Handle(string request);
}

class Manager : Handler {
    public override void Handle(string request) {
        if (request == "low") Console.WriteLine("Manager approved low-level request");
        else next?.Handle(request);
    }
}

class Director : Handler {
    public override void Handle(string request) {
        if (request == "medium") Console.WriteLine("Director approved medium-level request");
        else next?.Handle(request);
    }
}
```

#### ✅ Advantages

- Reduces coupling between sender and receiver.
- Flexible and dynamic handler chain.

#### ❌ Disadvantages

- Can be hard to debug.
- No guarantee that the request will be handled.

#### Use Cases

- Logging, approval workflows, middleware pipelines.

#### Interview Questions

- **What if no handler processes the request?** It may fail silently unless
  explicitly handled.

## Command

Encapsulate a request as an object to allow undo, logging, and queuing.

```csharp
interface ICommand { void Execute(); }

class LightOnCommand : ICommand {
    public void Execute() => Console.WriteLine("Light turned ON");
}

class RemoteControl {
    private ICommand command;
    public void SetCommand(ICommand cmd) => command = cmd;
    public void PressButton() => command.Execute();
}
```

#### ✅ Advantages

- Supports undo/redo.
- Decouples sender from receiver.

#### ❌ Disadvantages

- Increases class count.

#### Use Cases

- UI actions, transaction management, undo stacks.

#### Interview Questions

- **How is Command used in a UI?** Button clicks trigger command objects.

## Interpreter

Defines a grammar and interpreter for a domain-specific language (DSL).

```csharp
interface IExpression { int Interpret(); }

class Number : IExpression {
    private int number;
    public Number(int num) => number = num;
    public int Interpret() => number;
}

class Add : IExpression {
    private IExpression left, right;
    public Add(IExpression l, IExpression r) { left = l; right = r; }
    public int Interpret() => left.Interpret() + right.Interpret();
}
```

#### ✅ Advantages

- Great for small DSLs.
- Easy to extend.

#### ❌ Disadvantages

- Inefficient for large grammars.
- Hard to maintain as it grows.

#### Use Cases

- Expression evaluators, simple compilers, regex.

#### Interview Questions

- **When is Interpreter not appropriate?** For complex grammars—better use
  parser generators.

## Iterator

Access elements of an aggregate object without exposing its representation.

```csharp
class NameRepository {
    private string[] names = { "Alice", "Bob", "Charlie" };
    public IEnumerator<string> GetEnumerator() {
        foreach (var name in names) yield return name;
    }
}
```

#### ✅ Advantages

- Hides internal structure.
- Supports multiple traversals.

#### ❌ Disadvantages

- May complicate iteration for dynamic structures.

#### Use Cases

- Collections, tree traversal.

#### Interview Questions

- **How is IEnumerator different from IEnumerable?** `IEnumerator` moves over
  data; `IEnumerable` returns enumerators.

## Mediator

Centralize complex communications between objects to promote loose coupling.

```csharp
// Mediator Interface
public interface IMediator
{
    void SendMessage(string message, string from, string to);
}

// Concrete Mediator
public class ModuleMediator : IMediator
{
    private Dictionary<string, IModule> _modules = new();

    public void Register(string name, IModule module)
    {
        _modules[name] = module;
    }

    public void SendMessage(string message, string from, string to)
    {
        if (_modules.ContainsKey(to))
        {
            _modules[to].Receive(message, from);
        }
        else
        {
            Console.WriteLine($"[Mediator] Receiver '{to}' not found.");
        }
    }
}

// Colleague Interface
public interface IModule
{
    void Receive(string message, string from);
    void Send(string message, string to);
}

// Concrete Colleague
public class UserModule : IModule
{
    private string _name;
    private IMediator _mediator;

    public UserModule(string name, IMediator mediator)
    {
        _name = name;
        _mediator = mediator;
    }

    public void Receive(string message, string from)
    {
        Console.WriteLine($"{_name} received from {from}: {message}");
    }

    public void Send(string message, string to)
    {
        _mediator.SendMessage(message, _name, to);
    }
}

// Demo
class Program
{
    static void Main()
    {
        var mediator = new ModuleMediator();

        var authModule = new UserModule("AuthService", mediator);
        var orderModule = new UserModule("OrderService", mediator);
        var notificationModule = new UserModule("NotificationService", mediator);

        mediator.Register("AuthService", authModule);
        mediator.Register("OrderService", orderModule);
        mediator.Register("NotificationService", notificationModule);

        authModule.Send("User authenticated successfully", "OrderService");
        orderModule.Send("Order placed for user", "NotificationService");
    }
}
```

#### ✅ Advantages

- Reduces coupling.
- Simplifies object interaction.

#### ❌ Disadvantages

- Can become overly complex mediator.

#### Use Cases

- UI coordination, chat apps, Air Traffic Control.

#### Interview Questions

- **How is Mediator different from Observer?** Mediator coordinates multiple
  components; Observer broadcasts to subscribers.

## Memento

Capture an object’s state so it can be restored later.

```csharp
class Editor {
    public string Content { get; set; }
    public Memento Save() => new Memento(Content);
    public void Restore(Memento m) => Content = m.Content;
}

class Memento {
    public string Content { get; }
    public Memento(string content) => Content = content;
}
```

#### ✅ Advantages

- Preserves encapsulation.
- Supports undo.

#### ❌ Disadvantages

- Can be memory-heavy.

#### Use Cases

- Undo/redo, backup systems.

#### Interview Questions

- **How is Memento different from Prototype?** Memento is for restoring past
  state; Prototype is for cloning.

## State

Allow an object to alter its behavior when its internal state changes.

```csharp
// State Interface
public interface IOrderState
{
    void Handle(OrderContext context);
}

// Concrete States
public class NewOrderState : IOrderState
{
    public void Handle(OrderContext context)
    {
        Console.WriteLine("Order is new. Moving to processing...");
        context.SetState(new ProcessingState());
    }
}

public class ProcessingState : IOrderState
{
    public void Handle(OrderContext context)
    {
        Console.WriteLine("Order is processing. Moving to shipped...");
        context.SetState(new ShippedState());
    }
}

public class ShippedState : IOrderState
{
    public void Handle(OrderContext context)
    {
        Console.WriteLine("Order shipped. Process complete.");
    }
}

// Context
public class OrderContext
{
    private IOrderState _state;

    public OrderContext(IOrderState state)
    {
        _state = state;
    }

    public void SetState(IOrderState state)
    {
        _state = state;
    }

    public void Process()
    {
        _state.Handle(this);
    }
}

// Demo
class Program
{
    static void Main()
    {
        var order = new OrderContext(new NewOrderState());
        order.Process();
        order.Process();
        order.Process();
    }
}
```

#### ✅ Advantages

- Eliminates conditional logic.
- Encapsulates state-specific behavior.

#### ❌ Disadvantages

- Increases class count.

#### Use Cases

- UI controls, game states, workflow engines.

#### Interview Questions

- **When to use State vs. Strategy?** State depends on context; Strategy is
  chosen externally.

## Visitor

Separate algorithms from object structures, adding operations without modifying
the objects.

```csharp
interface IVisitor { void Visit(ElementA a); void Visit(ElementB b); }
interface IElement { void Accept(IVisitor visitor); }

class ElementA : IElement {
    public void Accept(IVisitor visitor) => visitor.Visit(this);
}

class ConcreteVisitor : IVisitor {
    public void Visit(ElementA a) => Console.WriteLine("Visited A");
    public void Visit(ElementB b) => Console.WriteLine("Visited B");
}
```

#### ✅ Advantages

- Add new operations without changing object structure.

#### ❌ Disadvantages

- Modifying element structure breaks visitors.

#### Use Cases

- AST traversal, compilers, reporting engines.

#### Interview Questions

- **What’s a drawback of Visitor?** A: Fragile to structural changes in the
  elements.

---

## Gang of Four (GoF) Design Patterns Cheatsheet

### Creational Patterns

| Pattern              | Intent                                                                 | Example (C#)                                                   | Pros                            | Cons                        | Follows SOLID | Reason                                                                                                                                          |
| -------------------- | ---------------------------------------------------------------------- | -------------------------------------------------------------- | ------------------------------- | --------------------------- | ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Singleton**        | Ensures a class has only one instance                                  | `static` class or `private constructor` with `static instance` | Controlled access, lazy loading | Global state, hard to test  | SRP, OCP      | SRP: Manages single responsibility of controlling instance creation; OCP: With abstraction, new variations can be added without modifying usage |
| **Factory Method**   | Define interface for object creation, let subclasses decide            | `IFactory` with `CreateProduct()`                              | Loose coupling                  | Proliferation of classes    | OCP, DIP      | OCP: Add new product types without modifying factory; DIP: Clients depend on abstractions, not concrete classes                                 |
| **Abstract Factory** | Create families of related objects without specifying concrete classes | `IUIFactory` with `CreateButton()` & `CreateTextBox()`         | Enforces consistency            | Complex structure           | SRP, DIP      | SRP: Factory handles only creation logic; DIP: Depends on abstract factories instead of concrete implementations                                |
| **Builder**          | Step-by-step construction of complex objects                           | `Builder pattern for Pizza/House`                              | Clear, flexible object creation | May require director class  | SRP           | SRP: Builder encapsulates construction steps, separate from product logic                                                                       |
| **Prototype**        | Clone an object instead of creating                                    | `ICloneable`, `.Clone()` method                                | Reduces cost of creation        | Deep vs shallow copy issues | SRP, OCP      | SRP: Object cloning separated from main logic; OCP: New cloneable types added without modifying client code                                     |

### Structural Patterns

| Pattern       | Intent                                          | Example (C#)                                  | Pros                    | Cons                          | Follows SOLID | Reason                                                                                                                                        |
| ------------- | ----------------------------------------------- | --------------------------------------------- | ----------------------- | ----------------------------- | ------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| **Adapter**   | Convert interface of one class to another       | `IUsbDevice` adapter for `LegacySerialDevice` | Reuse legacy code       | Added layer                   | OCP, DIP      | OCP: Can add new adapters without altering clients; DIP: Clients depend on target abstraction, not legacy types                               |
| **Bridge**    | Separate abstraction from implementation        | `RemoteControl` & `TV`                        | Independent variation   | Complex hierarchy             | SRP, OCP, DIP | SRP: Separates abstraction and implementation; OCP: Change abstraction or implementation independently; DIP: Abstraction depends on interface |
| **Composite** | Treat individual and group of objects uniformly | `Graphic`, `Circle`, `Group : Graphic`        | Hierarchical structures | Hard to restrict tree depth   | SRP, OCP      | SRP: Composite manages children, leaf handles logic; OCP: Add new components without changing structure                                       |
| **Decorator** | Add responsibilities dynamically                | `Stream`, `BufferedStream`, `EncryptedStream` | No subclass explosion   | Many small classes            | OCP, SRP      | SRP: Each class adds one responsibility; OCP: Add new behavior without modifying base class                                                   |
| **Facade**    | Provide simplified interface to a system        | `CompilerFacade.Compile()`                    | Simplifies use          | Hides power of subsystem      | SRP, OCP      | SRP: Facade is only responsible for coordination; OCP: Subsystems can change without affecting facade users                                   |
| **Flyweight** | Reduce object count via sharing                 | `CharacterFactory` in text rendering          | Saves memory            | Complexity with mutable state | SRP           | SRP: Separates shared and unique state                                                                                                        |
| **Proxy**     | Control access to real object                   | `VirtualProxy`, `SecurityProxy`               | Lazy loading, control   | Adds indirection              | SRP, OCP      | SRP: Proxy handles access control; OCP: Can add proxies without modifying subject or client                                                   |

### Behavioral Patterns

| Pattern                     | Intent                                 | Example (C#)                               | Pros                | Cons                             | Follows SOLID | Reason                                                                                                                   |
| --------------------------- | -------------------------------------- | ------------------------------------------ | ------------------- | -------------------------------- | ------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **Chain of Responsibility** | Pass request along handlers            | `Logger -> EmailNotifier -> SMSNotifier`   | Flexible, decoupled | Unhandled requests               | SRP, OCP      | SRP: Each handler has one job; OCP: Add new handlers without changing the chain                                          |
| **Command**                 | Encapsulate request as an object       | `ICommand`, `LightOnCommand`, `Invoker`    | Undo/redo support   | Class explosion                  | SRP, OCP      | SRP: Encapsulates action separately from invoker; OCP: New commands don’t affect invoker                                 |
| **Interpreter**             | Define grammar for language            | Expression trees for math parsing          | Easy to extend      | Hard to scale                    | SRP           | SRP: Each class represents one grammar rule/responsibility                                                               |
| **Iterator**                | Sequential access to collection        | `IEnumerator`, `foreach`                   | Hides structure     | Inefficient in large sets        | SRP           | SRP: Iterator class only handles traversal                                                                               |
| **Mediator**                | Centralize communication               | `DialogMediator`, `Button`, `TextBox`      | Reduces coupling    | Mediator grows complex           | SRP, OCP      | SRP: Mediator handles interactions only; OCP: Add new components without changing others                                 |
| **Memento**                 | Capture and restore object state       | `Editor.Save()`, `Restore()`               | Supports undo       | Memory overhead                  | SRP           | SRP: State saving is separated from business logic                                                                       |
| **Observer**                | One-to-many notification               | `IObservable`/`IObserver`, event system    | Loose coupling      | Memory leaks if not unsubscribed | SRP, OCP      | SRP: Observer handles notification only; OCP: Add new observers without changing subject                                 |
| **State**                   | Change behavior by state               | `OnState`, `OffState`, `Context`           | Avoids if-else      | State explosion                  | SRP, OCP      | SRP: State logic in separate classes; OCP: Add states without changing context                                           |
| **Strategy**                | Encapsulate interchangeable algorithms | `ISortStrategy`, `QuickSort`, `BubbleSort` | Runtime flexibility | Requires interface setup         | SRP, OCP, DIP | SRP: Each strategy class has one algorithm; OCP: Add new strategies easily; DIP: Context depends on strategy abstraction |
| **Template Method**         | Define skeleton with variable steps    | `AbstractClass`, `DoStep1()`, `DoStep2()`  | Code reuse          | Inflexibility                    | SRP, OCP      | SRP: Common logic centralized; OCP: Override steps without modifying base                                                |
| **Visitor**                 | Add new behavior to existing classes   | `IVisitor.Visit(ElementA)`                 | Open for operations | Breaks if structure changes      | OCP           | OCP: Add operations without changing data structure                                                                      |

---
