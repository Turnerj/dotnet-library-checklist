# dotnet-library-checklist

A work-in-progress "living" checklist of things to do as a .NET library author.

## Source Link, PDBs and Debugging

### Enable Source Link

From [MSDN](https://docs.microsoft.com/en-us/dotnet/standard/library-guidance/sourcelink):
> Source Link is a technology that enables source code debugging of .NET assemblies from NuGet by developers. Source Link executes when creating the NuGet package and embeds source control metadata inside assemblies and the package. Developers who download the package and have Source Link enabled in Visual Studio can step into its source code. Source Link provides source control metadata to create a great debugging experience.

📖 [How to add Source Link to your project](https://github.com/dotnet/sourcelink/blob/main/README.md#using-source-link-in-net-projects)

### Create Symbol Package (.snupkg)

From [MSDN](https://docs.microsoft.com/en-us/dotnet/standard/library-guidance/nuget#symbol-packages):
> Symbol files (*.pdb) are produced by the .NET compiler alongside assemblies. Symbol files map execution locations to the original source code so you can step through source code as it is running using a debugger. NuGet supports generating a separate symbol package (*.snupkg) containing symbol files alongside the main package containing .NET assemblies. The idea of symbol packages is they're hosted on a symbol server and are only downloaded by a tool like Visual Studio on demand.

📖 [How to create a Symbols Package](https://docs.microsoft.com/en-us/nuget/create-packages/symbol-packages-snupkg)

### Enable Deterministic Builds

From [MSDN](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-options/code-generation#deterministic):
> Causes the compiler to produce an assembly whose byte-for-byte output is identical across compilations for identical inputs.

Enabling Deterministic Builds by adding the following to your project:
```xml
<Deterministic>true</Deterministic>
```

📖 [Learn more about Reproducible Builds](https://en.wikipedia.org/wiki/Reproducible_builds)

## Performance Considerations

### Use Reduced-allocation APIs (Span/Memory)

From [MSDN](https://docs.microsoft.com/en-us/dotnet/standard/memory-and-spans/memory-t-usage-guidelines):
> .NET Core includes a number of types that represent an arbitrary contiguous region of memory. .NET Core 2.0 introduced `Span<T>` and `ReadOnlySpan<T>`, which are lightweight memory buffers that wrap references to managed or unmanaged memory. Because these types can only be stored on the stack, they are unsuitable for a number of scenarios, including asynchronous method calls. .NET Core 2.1 adds a number of additional types, including `Memory<T>`, `ReadOnlyMemory<T>`, `IMemoryOwner<T>`, and `MemoryPool<T>`. Like `Span<T>`, `Memory<T>` and its related types can be backed by both managed and unmanaged memory. Unlike `Span<T>`, `Memory<T>` can be stored on the managed heap.

Example (Getting a piece of a string)
```csharp
// Before
string result = myString.Substring(4, 8);

// After
ReadOnlySpan<char> result = myString.AsSpan().Slice(4, 8);
```

See also:
- [Adamn Sitnik's post on Span](https://adamsitnik.com/Span/)

### Expose your own Span/Memory APIs where appropriate

This helps avoid consumers needing to call `ToArray` unnecessarily if they are potentially already processing data in a span.

```csharp
// Before
public void DoWork(byte[] data) { }

// After
public void DoWork(Span<byte> data) { }
```

### Use ArrayPool where appropriate

From [MSDN](https://docs.microsoft.com/en-us/dotnet/api/system.buffers.arraypool-1):
> Using the `ArrayPool<T>` class to rent and return buffers (using the Rent and Return methods) can improve performance in situations where arrays are created and destroyed frequently, resulting in significant memory pressure on the garbage collector.

Example
```csharp
var buffer = ArrayPool<int>.Shared.Rent(1000);

// Do work...

ArrayPool<int>.Shared.Return(buffer);
```

See also:
- [Adam Sitnik's post on Array Pool](https://adamsitnik.com/Array-Pool/)

## Preparing for Trimming

1. Avoid reflection where possible
2. [**For .NET 6+**: If you need to use reflection, you can enable trim analyzers to help you find and resolve trim warnings](https://docs.microsoft.com/en-us/dotnet/core/deploying/trimming/prepare-libraries-for-trimming).

## Multi-platform Testing

- Run your tests on multi-platforms, particularly if you use APIs that may have nuances on different platforms (eg. networking, files etc)
- [GitHub Actions can make multi-platform testing pretty straight forward](https://github.com/TurnerSoftware/CacheTower/blob/082e9cd2b327c7b42d34068081f9105d31337703/.github/workflows/build.yml#L21-L61):
```yml
  build:
    name: Build ${{matrix.os}}
    runs-on: ${{matrix.os}}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macOS-latest]
```

## Consider Binary Compatibility

From [MSDN](https://docs.microsoft.com/en-us/dotnet/core/compatibility/categories#binary-compatibility):
> Binary compatibility refers to the ability of a consumer of an API to use the API on a newer version without recompilation. Such changes as adding methods or adding a new interface implementation to a type do not affect binary compatibility. However, removing or altering an assembly's public signatures so that consumers can no longer access the same interface exposed by the assembly does affect binary compatibility. A change of this kind is termed a binary incompatible change.

✔ Keeps Compatibility
```csharp
// Before
public void OldMethod() { }
// After
public void OldMethod() { }
public void NewMethod() { }
```

❌ Breaks Compatibility
```csharp
// Before
public void OldMethod() { }
// After
public void OldMethod(object obj = null) { }
```

👋 If you have more obscure examples of binary compatibility, please submit a PR!

## Private Build Dependencies

TODO

## Async and ConfigureAwait

Read: https://github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md
