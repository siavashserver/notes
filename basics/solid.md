---
title: SOLID Principles
---

## Single Responsibility

Each class has only one responsibility and therefore has only one reason to
change.

### Before (bad)

```csharp
class Report {
  void Generate() { … }
  void SaveToFile() { … }
}
```

### After (good)

```csharp
class ReportGenerator { void Generate() { … } }
class ReportSaver     { void Save(Report r) { … } }
```

## Open-closed

Open-closed principle states that every atomic unit of code, such as class,
module or function, should be open for extension, but closed for modification.

What it means is that, once written, the unit of code should be unchangeable,
unless some errors are detected in it. However, it should also be written in
such a way that additional functionality can be attached to it in the future if
requirements are changed or expanded. This can be achieved by common features of
object oriented programming, such as inheritance and abstraction.

```csharp
abstract class Shape { abstract double Area(); }
class Circle : Shape { … override Area() => …; }
class Rectangle : Shape { … override Area() => …; }

double TotalArea(IEnumerable<Shape> shapes) {
  return shapes.Sum(s => s.Area());
}
```

You can add more shapes (e.g. Triangle) without modifying `TotalArea` or other
classes.

## Liskov Substitution

When extending a class, you should be able to pass objects of the subclass in
place of objects of the parent class without breaking the client code.

### Problematic

```csharp
class Bird { virtual void Fly() { … } }
class Ostrich : Bird { override void Fly() => throw new NotImplementedException(); }
```

### Better design

```csharp
abstract class Bird { }
abstract class FlyingBird : Bird { abstract void Fly(); }
class Eagle : FlyingBird { override void Fly() { … } }
class Ostrich : Bird { /* no Fly method */ }
```

Ostrich no longer presents a broken contract.

## Interface Segregation

Clients shouldn’t be forced to implement methods they do not use.

### Too Broad

```csharp
interface IWorker {
  void Work();
  void Eat();
}
class Robot : IWorker { void Work() { … } void Eat() { throw … } }
```

### Refactored

```csharp
interface IWork { void Work(); }
interface IFeed { void Eat(); }
class Human : IWork, IFeed { … }
class Robot : IWork { … }
```

## Dependency Inversion

High-level classes shouldn’t depend on low-level classes. Both should depend on
abstractions.

### Bad (tight coupling)

```csharp
class LightSwitch {
  LightBulb bulb = new LightBulb();
  void Operate() { bulb.TurnOn(); }
}
```

### Good (depends on interface)

```csharp
interface ISwitchable { void TurnOn(); void TurnOff(); }
class LightBulb : ISwitchable { … }
class Fan      : ISwitchable { … }
class LightSwitch {
  private ISwitchable device;
  public LightSwitch(ISwitchable device) { this.device = device; }
  void Operate() { device.TurnOn(); }
}
```

Switch works with any `ISwitchable`, improving flexibility and enabling mocks in
tests.

---

## Inversion Of Control

Core business and higher level systems should not directly depend on concrete
implementation of lower level systems. Instead they should work with
abstractions, which encourages decoupling.

IoC implementations:

- Dependency Injection
- Service Locator (anti-pattern)

### Dependency Injection (DI)

**Dependency Injection** is a design pattern where an object’s required
dependencies are provided from the outside rather than created internally. This
ensures dependencies are **explicit**, **type-safe**, and visible in the class
API via constructor parameters.

Advantages of DI:

- Dependencies declared in constructor signatures → clear intent and contract.
- Compile‑time safety: missing dependencies trigger build errors.
- Easily testable: replace real dependencies with mocks in unit tests.

```csharp
public interface ILogger
{
    void Log(string message);
}

public class ConsoleLogger : ILogger
{
    public void Log(string message) => Console.WriteLine(message);
}

public class OrderService
{
    private readonly ILogger _logger;
    public OrderService(ILogger logger)
    {
        _logger = logger;
    }
    public void Process(int orderId)
    {
        // business logic...
        _logger.Log($"Processed order {orderId}");
    }
}
```

At composition root:

```csharp
var logger = new ConsoleLogger();
var service = new OrderService(logger);
service.Process(123);
```

### Service Locator Pattern

**Service Locator** provides a static/global registry. Classes call into this
locator to **resolve** their dependencies at runtime rather than having them
passed in.

```csharp
public static class ServiceLocator
{
    static readonly Dictionary<Type, object> _services = new();

    public static void Register<T>(T impl) => _services[typeof(T)] = impl;
    public static T Get<T>() => (T)_services[typeof(T)];
}

public class OrderServiceWithLocator
{
    public void Process(int orderId)
    {
        var logger = ServiceLocator.Get<ILogger>();
        logger.Log($"Processed order {orderId}");
    }
}
```

At startup:

```csharp
ServiceLocator.Register<ILogger>(new ConsoleLogger());
var svc = new OrderServiceWithLocator();
svc.Process(456);
```

#### Why Service Locator Is Considered an Anti‑Pattern?

- Hidden Dependencies: Service Locator conceals the true dependencies of a
  class—nothing in the constructor signals what services are needed.
  Pre-conditions become invisible until runtime.

- Runtime Errors, Not Compile‑Time Errors: Missing registrations or
  misconfigurations cause runtime failures rather than compilation failures,
  increasing fragility.

- Poor Testability: In tests, you must configure the global locator—often as
  static. That leads to fragile, hard-to-isolate tests and can break parallel
  test execution.

- Violates Encapsulation and Interface Segregation: Clients gain access to more
  services than necessary. It violates encapsulation and may breach Interface
  Segregation Principle, since the locator exposes a broad API beyond the
  consumer’s needs.

- Reduces Composability: With DI, different consumers can receive different
  configurations or decorated instances. With Service Locator, switching
  implementations per consumer is cumbersome or impossible.

---

## Sample IoC Implementations

### Minimal C# DI Container

A minimal IoC container that:

- Registers interface‑to‑implementation mappings
- Resolves types by constructor injection
- Handles singletons or transient instances

```csharp
public enum Lifetime { Singleton, Transient }

public class DiContainer {
    private readonly Dictionary<Type, (Type impl, Lifetime lifetime)> _registrations = new();
    private readonly Dictionary<Type, object> _singletons = new();

    public void Register<TService, TImpl>(Lifetime lifetime = Lifetime.Transient)
        where TImpl : TService {
        _registrations[typeof(TService)] = (typeof(TImpl), lifetime);
    }

    public TService Resolve<TService>() => (TService)Resolve(typeof(TService));

    private object Resolve(Type serviceType) {
        if (_singletons.TryGetValue(serviceType, out var instance))
            return instance;

        if (!_registrations.TryGetValue(serviceType, out var reg))
            throw new InvalidOperationException($"Service {serviceType.Name} not registered");

        var implType = reg.impl;
        var ctor = implType.GetConstructors().OrderByDescending(c => c.GetParameters().Length).First();
        var args = ctor.GetParameters().Select(p => Resolve(p.ParameterType)).ToArray();
        var obj = Activator.CreateInstance(implType, args);

        if (reg.lifetime == Lifetime.Singleton)
            _singletons[serviceType] = obj;

        return obj;
    }
}
```

```csharp
public interface IService { void Execute(); }
public class ServiceImpl : IService { public void Execute() => Console.WriteLine("Running"); }
public class Client { private readonly IService _svc; public Client(IService svc) => _svc = svc; public void Run() => _svc.Execute(); }

var container = new DiContainer();
container.Register<IService, ServiceImpl>(Lifetime.Singleton);
var client = container.Resolve<Client>();
client.Run();
```

### Simple Service Locator (Anti‑Pattern)

A centralized registry that hides dependencies and encourages runtime
resolution:

```csharp
public static class ServiceLocator {
    private static readonly Dictionary<Type, object> _services = new();

    public static void Register<T>(T service) => _services[typeof(T)] = service;
    public static T Get<T>() {
        if (_services.TryGetValue(typeof(T), out var svc)) return (T)svc;
        throw new InvalidOperationException($"Service {typeof(T).Name} not registered");
    }
}
```

```csharp
public interface ILogger { void Log(string msg); }
public class ConsoleLogger : ILogger { public void Log(string msg) => Console.WriteLine(msg); }

ServiceLocator.Register<ILogger>(new ConsoleLogger());

public class Worker {
    public void DoWork() {
        var logger = ServiceLocator.Get<ILogger>();
        logger.Log("Working...");
    }
}
```
