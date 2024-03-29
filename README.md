# dotnet-library-checklist

A work-in-progress "living" checklist of things to do as a .NET library author.

## Source Link, PDBs and Debugging

### Enable Source Link

From [MSDN](https://docs.microsoft.com/en-us/dotnet/standard/library-guidance/sourcelink):
> Source Link is a technology that enables source code debugging of .NET assemblies from NuGet by developers. Source Link executes when creating the NuGet package and embeds source control metadata inside assemblies and the package. Developers who download the package and have Source Link enabled in Visual Studio can step into its source code. Source Link provides source control metadata to create a great debugging experience.

Configure Source Link by adding the follow to your project:
```xml
<PropertyGroup>
    <!-- Optional: Include the repository URL in the NuGet package -->
    <PublishRepositoryUrl>true</PublishRepositoryUrl>
    <!-- Optional: Embed untracked source files in the PDB -->
    <EmbedUntrackedSources>true</EmbedUntrackedSources>
</PropertyGroup>
<ItemGroup>
    <PackageReference Include="Microsoft.SourceLink.GitHub" Version="1.1.1" PrivateAssets="All"/>
</ItemGroup>
```

If you're not using GitHub, there are alternative Source Link packages to reference instead.
There are packages for GitHub, GitLab, Azure Repos, Bitbucket and others.

📖 [Source Link and .NET libraries](https://docs.microsoft.com/en-us/dotnet/standard/library-guidance/sourcelink)

### Embedding Symbols

Symbol files map execution locations to the original source code so you can step through source code as it is running using a debugger.

While not documented as the recommended solution [on MSDN](https://docs.microsoft.com/en-us/dotnet/standard/library-guidance/nuget#symbol-packages), it has been [discussed in the .NET SDK repository](https://github.com/dotnet/sdk/issues/2679) and later [in a Microsoft blog post](https://devblogs.microsoft.com/dotnet/producing-packages-with-source-link/) that embedded symbols may be the best way to provide symbols to consumers.

Embed symbols in your packages by adding the following to your project:
```xml
<DebugType>embedded</DebugType>
```

📖 [Understanding symbol files and Visual Studio's symbol settings](https://devblogs.microsoft.com/devops/understanding-symbol-files-and-visual-studios-symbol-settings/)

### Enable Deterministic Builds

From [MSDN](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-options/code-generation#deterministic):
> Causes the compiler to produce an assembly whose byte-for-byte output is identical across compilations for identical inputs.

Enabling Deterministic builds by adding the following to your project:
```xml
<ContinuousIntegrationBuild>true</ContinuousIntegrationBuild>
```

As noted in [a Microsoft blog post about deterministic builds](https://devblogs.microsoft.com/dotnet/producing-packages-with-source-link/#deterministic-builds), it should only be enabled on your CI otherwise it can affect local debugging. This can be accomplished by adding a condition to the surrounding `PropertyGroup` like:

```xml
<PropertyGroup Condition="'$(GITHUB_ACTIONS)' == 'true'">
    <ContinuousIntegrationBuild>true</ContinuousIntegrationBuild>
</PropertyGroup>
```

Or alternatively as an MSBuild flag in your specific CI build script:
```
dotnet pack /p:ContinuousIntegrationBuild=true
```

Note: The official MSDN docs may be incorrect as it specifies `<Deterministic>true</Deterministic>`. This doesn't seem to do anything and doesn't satisfy the "Deterministic" flag in NuGet Package Manager.

📖 [Learn more about Reproducible Builds](https://en.wikipedia.org/wiki/Reproducible_builds)

### Write and Generate XML Docs

From [MSDN](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/xmldoc/):
> C# source files can have structured comments that produce API documentation for the types defined in those files. The C# compiler produces an XML file that contains structured data representing the comments and the API signatures. Other tools can process that XML output to create human-readable documentation in the form of web pages or PDF files, for example.

Example of XML documentation for a class:
```csharp
/// <summary>
///  This class performs an important function.
/// </summary>
public class MyClass {}
```

Additionally, add this to your project for an XML docs file to be generated when you package up your project.
This will allows others to access your documentation from inside their code editor.

```xml
<GenerateDocumentationFile>true</GenerateDocumentationFile>
```

#### Tips when writing XML docs

Beyond some of the automatically generated tags that appear when you type `///` in different spots, there are a few tips to help improve your documentation.

1. Use `<see cref="MyTypeOrProperty"/>` to point to a specific type or property. Depending on the preview tool (eg. Visual Studio), it will display a link to jump to that reference. It will also be automatically updated in Visual Studio if you rename the reference.
2. Use `<inheritdoc/>` to avoid repeating yourself from base types, interfaces or overridden properties. You can use it inconjunction with other tags like `<remarks>` to inherit most of the docs but selectively override different parts.
3. You can use `<paramref name="parameterName"/>` inside other blocks like `<summary>` or `<remarks>`. Similar to `<see>` blocks, it provides a reference rather than a constant string. There is also `<typeparamref name="TMyTypeParam"/>`.
4. You can use `<para></para>` to define paragraphs within your documentation. You can alternatively use `<br>` but support might very on tools that process the documentation file.
5. You can link out to websites with `<see href="https://example.org/">My Link</see>`. Alternatively you can use `<a href=""></a>` though support may vary between tools.
6. Use `<c>My.Piece.OfCode();</c>` to show a single line example of code.
7. Use the `langword` attribute (eg. `<see langword="while" />`) to reference a language keyword in the docs.

📖 [XML documentation comments - document APIs using `///` comments](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/xmldoc/)

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
- [Adam Sitnik's post on Span](https://adamsitnik.com/Span/)

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
public void MyMethod() { }
// After
public void MyMethod() { }
public void MyMethod(object obj) { }
```

❌ Breaks Compatibility
```csharp
// Before
public void MyMethod() { }
// After
public void MyMethod(object obj = null) { }
```

👋 If you have more obscure examples of binary compatibility, please submit a PR!

## Private Build Dependencies

TODO

## Async and ConfigureAwait

Read: https://github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md
