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

**For .NET 6+**

- Avoid reflection where possible

Read more: https://docs.microsoft.com/en-us/dotnet/core/deploying/trimming/prepare-libraries-for-trimming

## Multi-platform Testing

- Run your tests on multi-platforms, particularly if you use APIs that may have nuances on different platforms (eg. networking, files etc)

## Binary Compatibility

TODO: describe it, how to avoid issues etc

- Adding optional parameters to a method breaks binary compatibility

Read more: https://docs.microsoft.com/en-us/dotnet/core/compatibility/categories#binary-compatibility

## Private Build Dependencies

TODO

## Async and ConfigureAwait

Read: https://github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md
