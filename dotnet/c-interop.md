---
title: C Interop
---

## LibraryImport (modern)

- Source generator emits compile-time wrapper code
- Faster, AOT-friendly, inlinable, debuggable

```csharp
class NativeMethods
{
    [LibraryImport("user32.dll",
        EntryPoint = "MessageBoxW",
        SetLastError = true,
        StringMarshalling = StringMarshalling.Utf16)]
    public static partial int MessageBoxW(
        IntPtr hWnd, string lpText, string lpCaption, uint uType);
}

class Program
{
    static void Main()
    {
        NativeMethods.MessageBoxW(IntPtr.Zero,
            "Hello from LibraryImport!", "Hi", 0);
    }
}
```

## DllImport (old)

- Generates IL marshalling code at runtime (via IL stub)
- Slight runtime overhead, needs runtime IL & JIT

```csharp
class NativeMethods
{
    [DllImport("user32.dll", EntryPoint = "MessageBoxW",
        CharSet = CharSet.Unicode, SetLastError = true)]
    public static extern int MessageBoxW(
        IntPtr hWnd, string lpText, string lpCaption, uint uType);
}

class Program
{
    static void Main()
    {
        NativeMethods.MessageBoxW(IntPtr.Zero,
            "Hello from DllImport!", "Hi", 0);
    }
}
```
