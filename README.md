# dotnet-library-checklist

A work-in-progress "living" checklist of things to do as a .NET library author.

## Deterministic Builds

From [Wikipedia](https://en.wikipedia.org/wiki/Reproducible_builds):
> Reproducible builds, also known as deterministic compilation, is a process of compiling software which ensures the resulting binary code can be reproduced. Source code compiled using deterministic compilation will always output the same binary.

More information: https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-options/code-generation#deterministic

## Source Link and PDBs

TODO

## Performance Considerations

- Use Span/Memory APIs where possible to reduce allocations
- Use ArrayPool where appropriate to avoid internal array allocations
- Expose Span/Memory APIs where appropriate to avoid consumers needing to call `ToArray` etc

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

## Binary Compatibility

[Binary compatibility refers to the ability of a consumer of an API to use the API on a newer version without recompilation.](https://docs.microsoft.com/en-us/dotnet/core/compatibility/categories#binary-compatibility)

- New methods don't typically break binary compatibility
- Adding optional parameters to an existing method breaks binary compatibility

TODO: Add other less-obvious examples of breaking binary compatibility

## Private Build Dependencies

TODO

## Async and ConfigureAwait

Read: https://github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md
