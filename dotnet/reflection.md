---
title: Reflection
---

## Core Reflection Scenarios

C# reflection is a robust runtime feature that empowers developers to inspect,
query, and manipulate assemblies, types, and their members within the .NET
ecosystem.

| Scenario                                      | Description                                                     |
| --------------------------------------------- | --------------------------------------------------------------- |
| Accessing Metadata                            | Inspect assemblies, types, and members at runtime               |
| Dynamic Method Invocation                     | Invoke methods with names or signatures unknown at compile time |
| Working with Attributes                       | Read custom and built-in attribute metadata                     |
| Inspecting Types at Runtime                   | Explore type details, determine members, runtime type logic     |
| Dynamic Object Creation                       | Instantiate types and objects dynamically                       |
| Invoking Constructors via Reflection          | Dynamically invoke constructors with/without parameters         |
| Reflection.Emit & Dynamic Assembly Generation | Generate code/assemblies/types for advanced scenarios           |
| Reflection Performance & Caching              | Optimize reflection by caching metadata, using delegates        |
| Reflection in Dependency Injection            | Used in DI frameworks for runtime resolution                    |
| Reflection in Serialization                   | Read/write and map object data at runtime                       |
| Reflection for Testing Frameworks             | Discover and execute tests dynamically                          |
| Plugin Architecture with Reflection           | Load and interact with plugins and external assemblies          |
| Security Considerations                       | Manage encapsulation, permissions, avoid abuse                  |
| Reflection and Generics                       | Handle and create generic types or generic methods              |
| Reflection Tools & Best Practices             | Employ correct patterns, flags, tools like ILSpy, Mono.Cecil    |

---

## Accessing Metadata via Reflection

Accessing and understanding the metadata associated with .NET types, assemblies,
members, and parameters is a foundational scenario for reflection. This scenario
is ubiquitous in frameworks (ORMs, serializers), code analyzers, and plugin
loaders, and frequently asked about in interviews to gauge baseline reflection
literacy.

```csharp
Type typeInfo = typeof(string);
Console.WriteLine("Namespace: " + typeInfo.Namespace);
Console.WriteLine("Assembly: " + typeInfo.Assembly.FullName);
Console.WriteLine("Methods:");
foreach (var method in typeInfo.GetMethods())
{
    Console.WriteLine(method.Name);
}
```

The above snippet lists the namespace, containing assembly, and all methods of
the `String` type. This pattern recurs across reflection-powered applications,
especially when interrogating unknown types at runtime.

**Detailed Analysis:**  
Reflection exposes an object graph of all loaded types and their metadata,
accessible via the `System.Type` class and related classes like `MethodInfo`,
`PropertyInfo`, and `FieldInfo`. Using reflection, it's possible to enumerate
not just public members, but—using appropriate `BindingFlags` such as
`BindingFlags.NonPublic` and `BindingFlags.Static`—to introspect private,
protected, and static members. Mastery of `BindingFlags` is crucial in interview
scenarios where you may need to access specific method overloads, properties, or
differentiate between accessibility levels.

Interviewers commonly check for understanding of the performance overhead and
restrictions when repeatedly accessing metadata and expect candidates to discuss
metadata caching, member information reuse, or tool support for analyzing types.

---

## Inspecting Types at Runtime

Being able to determine properties, methods, and characteristics of an object or
type at runtime is core to reflective programming. This forms the basis for
serialization, test discovery, code analyzers, and dynamic loading frameworks.

```csharp
object obj = "Test";
Type type = obj.GetType();
Console.WriteLine("Type Name: " + type.Name);
Console.WriteLine("Is Class: " + type.IsClass);
PropertyInfo[] properties = type.GetProperties();
foreach (var prop in properties)
{
    Console.WriteLine("Property: " + prop.Name);
}
```

This approach allows generic code to identify members, understand an object's
structure, and adapt logic accordingly—such as automatically mapping types to
database columns or UI fields.

**Detailed Analysis:**  
Type inspection is used everywhere: serializers like Newtonsoft.Json and
System.Text.Json use it extensively, logging frameworks auto-enumerate
properties, and code generators analyze type layouts. Key APIs include
`Type.GetMethods()`, `Type.GetProperties()`, `Type.GetFields()`, and
`Type.GetMembers()`. Candidates are often asked to implement utilities that dump
the structure of user-defined types or create logic that dynamically adapts to
unknown input data.

In interviews, expect follow-ups about accessing only non-public members (using
`BindingFlags`), differentiating between static and instance members, and
recursively traversing related types. Managing exceptions and edge cases—such as
inaccessible types or ambiguous matches—are also frequent test cases.

---

## Dynamic Method Invocation

Dynamic method invocation allows calling methods on objects (or static classes)
at runtime when the method name, signature, or even type is not known until
execution. This is fundamental for plugin activation, scripting, "late binding",
and meta-programming, and forms a core part of frameworks like NUnit/xUnit that
discover test methods at runtime.

```csharp
Type type = typeof(Math);
MethodInfo methodInfo = type.GetMethod("Pow", new[] { typeof(double), typeof(double) });
object result = methodInfo.Invoke(null, new object[] { 2.0, 3.0 });
Console.WriteLine("Pow(2,3): " + result);
```

This scenario demonstrates invoking the static `Pow` method on the `Math` class.
For instance methods, an object instance must be supplied as the first parameter
to `Invoke`.

**Detailed Analysis:**  
Key classes are `MethodInfo` (obtained via `Type.GetMethod` or `GetMethods`).
The `Invoke()` method accepts an object (or null for static methods) and an
array of arguments. A common interview pitfall is supplying the wrong target to
`Invoke`, such as attempting to invoke a static method on an instance, which
leads to runtime exceptions.

The necessity to distinguish between static and instance methods is highlighted
in frameworks that must support both styles. Handling method overloads (by
providing the right parameter types), variable argument methods (`params`),
generic method invocation (using `MakeGenericMethod`), and managing exceptions
(like `TargetInvocationException`) are subtle, practical reflection skills that
are often explicitly tested in interviews.

---

## Working with Attributes

Attributes in C# are a declarative mechanism for adding metadata to almost
anything in code: classes, methods, properties, assemblies, and more. Reflection
allows one to read and reason over these attributes at runtime, making it a
linchpin of frameworks (ASP.NET MVC/Web API route mapping, data validation, ORM
mapping, test discovery, and serialization customization).

```csharp
[AttributeUsage(AttributeTargets.Class)]
public class DeveloperInfoAttribute : Attribute
{
    public string Name { get; }
    public DeveloperInfoAttribute(string name) { Name = name; }
}

[DeveloperInfo("John")]
public class SampleClass { }

Type type = typeof(SampleClass);
object[] attrs = type.GetCustomAttributes(false);
foreach (DeveloperInfoAttribute attr in attrs)
{
    Console.WriteLine("Developer: " + attr.Name);
}
```

This pattern accesses custom attribute instances at runtime and is prevalent in
configuration-driven frameworks, dynamic model validators, and plugin
registration logic.

**Detailed Analysis:**  
The most common attribute reflection APIs are `Type.GetCustomAttributes`,
`MemberInfo.GetCustomAttributes`, and overloads that target a specific attribute
type and optionally include inherited attributes. Interviewers may ask for code
to filter or act upon classes or methods marked by specific attributes (e.g.,
finding all `TestMethod`-marked methods), and expect a grasp of attributes'
influence on serialization, validation, and behavioral modification in
frameworks.

A frequent deeper question is to write a utility that enumerates all classes
marked with a certain attribute in an assembly, or to discuss the performance
implications of attribute retrieval if performed repeatedly (hint: cache results
if possible).

---

## Dynamic Object Creation

Runtime instantiation of types—when the actual type is unknown until runtime—is
a pivotal reflection scenario. Commonly seen in plugin-based architectures,
editors, dependency injection systems, deserialization, and acceptance of user
input for object types.

```csharp
Type type = typeof(StringBuilder);
object instance = Activator.CreateInstance(type);
((StringBuilder)instance).Append("Hello Reflection");
Console.WriteLine(instance.ToString());
```

Or using strings to load external types:

```csharp
Type type = Type.GetType("Namespace.MyClass");
object obj = Activator.CreateInstance(type);
```

These techniques are central for decoupled system designs, such as plugin
runners, test frameworks, and serialization engines.

**Detailed Analysis:**  
`Activator.CreateInstance` is the main API for instantiating objects
dynamically. It has overloads for supplying constructor parameters, using
different assembly contexts, or using a string type name. In plugin systems,
assemblies might be loaded from disk with `Assembly.LoadFrom`, types retrieved
via `Assembly.GetType`, and instances created as above. Candidates should be
ready to discuss the handling of constructor selection, exceptions when
constructors are missing or ambiguous, and security (e.g., instantiating types
from untrusted sources).

Additionally, generic types require special attention: use `MakeGenericType` to
construct the appropriate `Type` before creation—mastery of this is crucial for
systems that deal with generics at runtime.

---

## Invoking Constructors via Reflection

In addition to `Activator.CreateInstance`, reflection exposes direct APIs for
inspecting and invoking specific constructors via the `ConstructorInfo` class.
This is central for advanced serialization, dependency injection frameworks, and
dynamic proxies.

```csharp
Type type = typeof(DateTime);
ConstructorInfo ctor = type.GetConstructor(new[] { typeof(long) });
object dateInstance = ctor.Invoke(new object[] { 637627163000000000L });
Console.WriteLine(dateInstance);
```

Directly invoking constructors is useful for handling non-default constructors
or those with complex parameter lists.

**Detailed Analysis:**  
`ConstructorInfo.Invoke` provides greater control over object construction,
supporting scenarios where parameter types, order, or accessibility (non-public
constructors) differ. Interviews may include exercises involving retrieval and
invocation of protected or private constructors (using suitable `BindingFlags`),
or handling value-types (which may lack explicit constructors).

Care must be taken with security: accessing non-public constructors is
restricted unless the executing code is fully trusted and has appropriate
permissions, a nuance that's especially crucial for secure plugin loading or
sandboxed environments.

---

## Reflection.Emit and Dynamic Assembly Generation

For advanced frameworks (ORMs like NHibernate, mock libraries such as Moq,
specialized serializers, or custom runtime compilers), C# exposes the
`System.Reflection.Emit` API for generating new types, members, and assemblies
at runtime.

```csharp
AssemblyName an = new AssemblyName("DynamicAssembly");
AssemblyBuilder ab = AppDomain.CurrentDomain.DefineDynamicAssembly(an, AssemblyBuilderAccess.Run);
ModuleBuilder mb = ab.DefineDynamicModule("MainModule");
TypeBuilder tb = mb.DefineType("MyDynamicType", TypeAttributes.Public);
MethodBuilder methodBuilder = tb.DefineMethod("SayHello", MethodAttributes.Public, null, null);
ILGenerator il = methodBuilder.GetILGenerator();
il.EmitWriteLine("Hello from dynamically generated method!");
il.Emit(OpCodes.Ret);
Type newType = tb.CreateType();
object obj = Activator.CreateInstance(newType);
newType.GetMethod("SayHello").Invoke(obj, null);
```

Reflection.Emit's main applications are in high-performance serializers, dynamic
proxies, AOP tools, and expression tree optimizers. In interviews, it's often
considered an advanced topic, but knowledge of its existence demonstrates
architectural acumen.

**Detailed Analysis:**  
Effective usage involves `AssemblyBuilder`, `ModuleBuilder`, `TypeBuilder`, and
`ILGenerator`, each representing increasing specificity for runtime code
generation. Interviewers may probe why reflection emit is faster for repeated
operations (no metadata traversal at runtime; emits IL directly), how it's used
in frameworks like Entity Framework (dynamic proxy generation), or for
on-the-fly compilation or code injection.

Practical pitfalls include debugging emitted code, security concerns, and
limitations in various .NET Core environments (such as Native AOT).

---

## Reflection Performance and Caching

Reflection is powerful but not free—intensive reflective operations are
significantly slower than compiled direct access. Performance considerations are
paramount for any serious usage of reflection, especially in loops, hot-path
code, or data serialization/deserialization scenarios.

```csharp
static Dictionary<string, PropertyInfo> propertyCache = new Dictionary<string, PropertyInfo>();
PropertyInfo GetCachedProperty(Type type, string name)
{
    if (!propertyCache.TryGetValue(name, out var prop))
    {
        prop = type.GetProperty(name);
        propertyCache[name] = prop;
    }
    return prop;
}
```

**Key Points:**

- Reflection lookup (e.g., via `GetProperty`, `GetMethod`, or `GetField`) is
  expensive; repeated invocations should cache relevant `PropertyInfo`,
  `MethodInfo`, etc.
- For repeated method/property access, generate and cache delegates (compiled
  expression trees) for up to 1000x speedup over basic reflection.
- Direct instantiation (`new`) is always fastest. For repeated,
  performance-critical operations, expression trees or even Reflection.Emit are
  optimal.

**Interview Guidance:**  
Expect to field questions about reflection costs, when they matter, how
frameworks mitigate them, and example caching strategies. Subtle nuances include
the caching of delegates vs. metadata and when source generators (C# 9+) are
preferable for serialization or codegen tasks.

---

## Reflection in Dependency Injection

Dependency injection frameworks resolve object graphs at runtime by inspecting
constructors, properties, or methods to provide required dependencies—by
necessity, this is usually implemented by reflection.

```csharp
Type serviceType = typeof(IMyService);
Type implementationType = typeof(MyService);
object instance = Activator.CreateInstance(implementationType);
```

Modern DI frameworks (Microsoft.Extensions.DependencyInjection, Autofac, Unity)
use reflection for constructor inspection, property injection, and sometimes
attribute discovery. With ASP.NET Core, this is now a default part of all web
projects.

**Interview Context:**  
Candidates are asked to explain the difference between constructor, property,
and method injection; how reflection discovers injectable members; what happens
when multiple constructors exist; and how service lifetimes are resolved.

Nuanced discussion may cover how to mitigate reflection costs in DI containers,
use of pre-caching, or advanced activation scenarios (e.g., using
`ActivatorUtilities.CreateInstance` to resolve dependencies).

---

## Reflection in Serialization

Serialization frameworks (e.g., Newtonsoft.Json, System.Text.Json, legacy
BinaryFormatter/XMLSerializer) extensively use reflection to read and write
object state at runtime.

```csharp
Type type = typeof(MyClass);
foreach (PropertyInfo prop in type.GetProperties())
{
    Console.WriteLine($"{prop.Name} = {prop.GetValue(myInstance)}");
}
```

Or for building dynamic object mappers:

```csharp
PropertyInfo prop = typeof(Person).GetProperty("Name");
string name = (string)prop.GetValue(personInstance);
```

**Detailed Analysis:**  
Serialization scenarios highlight reflection’s ability (and necessity) to
introspect object properties/fields in a type-agnostic fashion and link them to
serialization formats (e.g., JSON property names). Interview questions often
include writing a custom serializer with reflection, discussing attribute-based
customization, or handling cyclic references, private members, and performance
optimization (e.g., caching discovered members).

A modern trend is the shift toward source generation for improved performance
and trimming, as in .NET 8 and System.Text.Json, but reflection remains the
fallback for dynamic scenarios.

---

## Reflection in Testing Frameworks

Most testing frameworks (NUnit, xUnit, MSTest) use reflection to discover and
execute test methods, especially those marked with custom attributes (`[Test]`,
`[Fact]`, etc.):

```csharp
foreach (var method in typeof(MyTestClass).GetMethods())
{
    if (method.GetCustomAttributes(typeof(TestMethodAttribute), false).Length > 0)
    {
        method.Invoke(testInstance, null);
    }
}
```

The discovery and execution of tests without manual registration is only
feasible due to runtime metadata access. Reflection also allows private field
and method access necessary for advanced mocking or behavioral testing.

**Interview Context:**  
You may be asked to implement a simple test runner using reflection, or to
explain how and why test frameworks depend on attribute-driven discovery. More
advanced questions assess security implications (e.g., running code marked as
tests) and performance optimization strategies.

---

## Plugin Architecture with Reflection

Plugins extend application functionality via external, possibly user-created,
assemblies loaded at runtime. Reflection enables the host system to discover,
instantiate, and interact with unknown types via contracts/interfaces:

```csharp
Assembly pluginAssembly = Assembly.LoadFrom("MyPlugin.dll");
foreach (Type type in pluginAssembly.GetTypes())
{
    if (typeof(IPlugin).IsAssignableFrom(type))
    {
        IPlugin plugin = (IPlugin)Activator.CreateInstance(type);
        plugin.Execute();
    }
}
```

This is the backbone of extensible applications, game modding systems, and
modular enterprise software.

**Security Note:**  
Untrusted plugin code can compromise host integrity; sandboxing strategies or
strong input validation are required. Interviewers may ask you to discuss safe
assembly loading, use of custom `AssemblyLoadContext`, or plugin dependency
isolation.

---

## Security Considerations of Reflection

Reflection's ability to bypass normal access controls (e.g., accessing private
fields, internal methods, non-public constructors) introduces attack vectors
such as arbitrary code execution, confidential data leakage, and privilege
escalation.

```csharp
PropertyInfo prop = typeof(MyClass).GetProperty("Secret", BindingFlags.NonPublic | BindingFlags.Instance);
object value = prop.GetValue(instance);
```

Modern .NET frameworks restrict reflection-based access to nonpublic members for
partially trusted code (sandboxed context), and require explicit permissions
(e.g., `ReflectionPermission`) for full access.

**Interview Analysis:**  
Candidates are expected to be aware of the risks and mitigations—never perform
unchecked dynamic method calls with user input, always validate types when
loading plugins, and avoid reflection on security-critical paths. Proper
privilege separation, strong-naming and assembly verification, and minimal
access principles are key points.

---

## Reflection and Generics

Reflection supports not only non-generic types but also generic types and
methods. This enables advanced patterns such as dynamic creation of generic
containers and runtime invocation of generic methods:

```csharp
Type listType = typeof(List<>);
Type constructedType = listType.MakeGenericType(typeof(int));
object instance = Activator.CreateInstance(constructedType);
Console.WriteLine(instance.GetType()); // List<System.Int32>
```

Invoking generic methods via reflection also follows a similar pattern:

```csharp
MethodInfo genericMethod = type.GetMethod("GenericMethod");
MethodInfo constructedMethod = genericMethod.MakeGenericMethod(typeof(int));
constructedMethod.Invoke(instance, new object[] { 123 });
```

**Interview Context:**  
Candidates must explain the difference between open and closed generic types,
the process for specifying type arguments using
`MakeGenericType`/`MakeGenericMethod`, and practical applications such as
generic data mappers or service locators. A robust understanding is required to
pass advanced system design interviews.
