---
title: ASP.NET
---

### Middleware

#### Key points

- Middleware executes in the order registered and can short-circuit the pipeline
  by not calling the next delegate.
- Middleware works before MVC selects a controller/action, so it cannot rely on
  model binding or ActionContext.

#### Minimal custom middleware (ASP.NET Core)

```csharp
// RequestLoggingMiddleware.cs
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;
    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }
    public async Task InvokeAsync(HttpContext context)
    {
        _logger.LogInformation("Incoming {Method} {Path}", context.Request.Method, context.Request.Path);
        await _next(context); // call next middleware
        _logger.LogInformation("Outgoing {StatusCode}", context.Response.StatusCode);
    }
}
```

Register in Program.cs:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseMiddleware<RequestLoggingMiddleware>();

app.MapControllers();
app.Run();
```

#### Usage scenarios (when to use middleware)

- Authentication schemes that apply globally (custom API key validation before
  MVC).
- Global exception handling or centralized logging that must run for static
  files, endpoints, and MVC routes.
- Request/response transformation, CORS, static file serving, rate limiting,
  compression.

---

### Filters

Filters run inside the MVC/Razor pipeline. Types include: AuthorizationFilter,
ResourceFilter, ActionFilter, ExceptionFilter, ResultFilter, and PageFilter
(Razor Pages). Use filters when you need MVC context, model/state, or
per-controller/action control.

#### 1. Authorization filter

- Runs early; used to allow/deny access to actions.
- Can short-circuit by setting context.Result.

Sample:

```csharp
public class RequireHeaderAttribute : Attribute, IAuthorizationFilter
{
    private readonly string _header;
    public RequireHeaderAttribute(string header) => _header = header;
    public void OnAuthorization(AuthorizationFilterContext context)
    {
        if (!context.HttpContext.Request.Headers.ContainsKey(_header))
            context.Result = new UnauthorizedResult();
    }
}
```

Usage:

```csharp
[RequireHeader("X-Client-Id")]
public class PaymentsController : ControllerBase { ... }
```

When to use: action-level access checks that depend on routing, user identity,
or action metadata.

---

#### 2. Resource filter

- Wraps execution before and after model binding and action selection; good for
  caching resources or short-circuiting before model binding.
- Executes once per request for the resource.

Sample:

```csharp
public class SimpleCacheResourceFilter : Attribute, IResourceFilter
{
    public void OnResourceExecuting(ResourceExecutingContext context)
    {
        // Example: check in-memory cache and short-circuit
        var key = context.HttpContext.Request.Path;
        if (MemoryCache.TryGetValue(key, out var cached))
            context.Result = new ContentResult { Content = (string)cached };
    }
    public void OnResourceExecuted(ResourceExecutedContext context) { }
}
```

When to use: cheap pre-processing that should run before model binding (e.g.,
output caching).

---

#### 3. Action filter

- Runs before and after the action method and has access to action arguments and
  model state.
- Good for validation, logging around action execution, or modifying action
  arguments.

Sample:

```csharp
public class ValidateModelAttribute : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        if (!context.ModelState.IsValid)
            context.Result = new BadRequestObjectResult(context.ModelState);
    }
}
```

When to use: model validation, timing an action, modifying action inputs.

---

#### 4. Exception filter

- Catches exceptions thrown by actions or later filters and converts them to
  results.
- Note: middleware-level exception handling runs earlier/later and affects
  non-MVC requests too.

Sample:

```csharp
public class ApiExceptionFilter : IExceptionFilter
{
    private readonly ILogger<ApiExceptionFilter> _logger;
    public ApiExceptionFilter(ILogger<ApiExceptionFilter> logger) => _logger = logger;
    public void OnException(ExceptionContext context)
    {
        _logger.LogError(context.Exception, "Unhandled");
        context.Result = new ObjectResult(new { error = "server_error" }) { StatusCode = 500 };
    }
}
```

When to use: translate MVC-specific exceptions to HTTP responses; prefer
middleware for truly global error handling across all endpoints.

---

#### 5. Result filter

- Runs before and after the action result executes (response writing).
- Useful for modifying the result or adding headers.

Sample:

```csharp
public class AddResponseHeaderAttribute : ResultFilterAttribute
{
    private readonly string _name;
    private readonly string _value;
    public AddResponseHeaderAttribute(string name, string value) { _name = name; _value = value; }
    public override void OnResultExecuting(ResultExecutingContext context)
    {
        context.HttpContext.Response.Headers[_name] = _value;
    }
}
```

When to use: add per-action headers, wrap result content for specific
controllers.

---

#### 6. Page filters (Razor Pages)

- Similar to action/resource/result filters but for Razor Pages
  (OnPageHandlerExecuting, OnPageHandlerExecuted, etc.).
- Use when you need page-handler level hooks in Razor Pages.

Sample:

```csharp
public class LogPageFilter : Attribute, IPageFilter
{
    public void OnPageHandlerExecuting(PageHandlerExecutingContext context) { /* before handler */ }
    public void OnPageHandlerExecuted(PageHandlerExecutedContext context) { /* after handler */ }
    public void OnPageHandlerSelected(PageHandlerSelectedContext context) { }
}
```

When to use: page-specific concerns in Razor Pages.

---

### How to register filters

- Per-action / per-controller: add attribute to controller or action (examples
  above).
- Globally: in Startup/Program add to MvcOptions.Filters:

```csharp
builder.Services.AddControllers(options =>
{
    options.Filters.Add<ValidateModelAttribute>(); // global
});
```

---

### Middleware vs Filters — guidance and decision matrix

| Concern                                                            |           Use Middleware | Use Filter           |
| ------------------------------------------------------------------ | -----------------------: | -------------------- |
| Needs to run for static files, non-MVC endpoints                   |                      Yes | No                   |
| Requires access to ActionContext, model state, or action arguments |                       No | Yes                  |
| Global cross-cutting (logging, CORS, compression, global auth)     |                      Yes | Rarely               |
| Per-controller or per-action behavior                              |                       No | Yes                  |
| Centralized exception handling for entire host                     |                      Yes | No                   |
| Short-circuit before model binding but only for MVC resources      | Possibly resource filter | Yes (ResourceFilter) |

Sources: general middleware behavior and examples; differences and when filters
can access MVC constructs vs middleware scope.

---

### Practical examples — combined usage

- Put global exception handling, compression, static file, and CORS middleware
  in Program.cs (middleware level).
- Use AuthorizationFilter for fine-grained, action-level policies and
  ActionFilter for model validation and action timing.
- Use ResourceFilter for output caching that should run before model binding.

Fact: middleware runs before the MVC filter pipeline and does not have access to
ActionExecutingContext, so choose based on needed context and scope.
