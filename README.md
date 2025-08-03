# Gnobbi.DebuggerTools

**Working State not yet complete**

## Overview

Gnobbi.DebuggerTools is a set of .NET libraries and source generators designed to provide advanced debugging and diagnostics capabilities using the decorator pattern. The solution includes abstractions, source code generators, and test projects to help instrument and analyze method calls, execution times, and other runtime data.

## Key Features

- **Decorator Pattern via Source Generation:**  
  Automatically generates diagnostic decorators for interfaces using Roslyn source generators.

- **Diagnostics Collection:**  
  Captures method execution details such as process/thread IDs, method/class names, start/finish times, elapsed milliseconds and in/out parameters of the method.

- **Flexible Configuration:**  
  Supports configuration via `Gnobbi.DebugTools.Decorator.Config.json` to control which interfaces/namespaces are decorated.

- **Dependency Injection Support:**  
  Integrates with `Microsoft.Extensions.DependencyInjection` for easy registration and usage.

## Usage
1. Project Setup
* add package `Gnobbi.DebugTools.Decorator.Abstractions` to your project
* add package `Gnobbi.DebugTools.Decorator.SourceCodeGenration` to your project
* add to project.csprj:

```xml
  <ItemGroup>
    <ProjectReference Include="Gnobbi.DebugTools.Decorator.Abstractions" Version="1.0.0"/>
    <ProjectReference Include="Gnobbi.DebugTools.Decorator.SourceCodeGenerator" Version="1.0.0" OutputItemType="Analyzer" ReferenceOutputAssembly="false" />
  </ItemGroup>
	<ItemGroup>
		<AdditionalFiles Include="Gnobbi.DebugTools.Decorator.Config.json" />
	</ItemGroup>
```

2. Implement Diagnostic Entry Handler
```csharp
using Gnobbi.DebugTools.Decorator;
namespace YourNameSpace;

public class FakeDiagnosticEntryHandler : IDiagnosticEntryHandler
{
    public List<DiagnosticEntiyBase> Entries { get; } = [];

    public Task HandleDiagnosticEntryAsync(DiagnosticEntiyBase entity)
    {
        Entries.Add(entity);
        return Task.CompletedTask;
    }
}
```

3 register decorators in your `Startup.cs` or `Program.cs`:
```csharp
using Gnobbi.DebugTools.Decorator;
...
    serviceCollection.AddSingleton<IDiagnosticEntryHandler, FakeDiagnosticEntryHandler>();
    ServiceDecoratorRegistration.RegisterDecorators(serviceCollection); // this registers decorators only if interfaces are decorated
```

4. **Mark Interfaces for Decoration:**
```csharp
[Decorate(DecoratingType.Diagnostics)] 
public interface IProjectRepository { ... }
```
Attribute Decorate always win over the configuration file, so if you want to decorate an interface, you can use the attribute.

5. **Or Configure Diagnostics:**
Add `Gnobbi.DebugTools.Decorator.Config.json` to your project and specify which interfaces/namespaces to include or exclude.

```json
{
  "Type": "Namespaces"`// or All or Attributes`,
  "IncludeNamespaces": [ "Gnobbi.DebugTools.Decorator.Decorator.Test.DomainTestClasses" ], // decorates interfaces of the namespace list
  "ExcludeInterfaces": [ "IProjectRepository" ] // excludes specific interfaces from decoration
}
```

## Example Diagnostic Entity

```csharp
public class DiagnosticEntiyBase { 
	public int ProcessId { get; set; } 
	public int ThreadId { get; set; } 
	public string MethodName { get; set; } 
	public string ClassName { get; set; } 
	public DateTime StartedAt { get; set; } 
	public DateTime FinshedAt { get; set; } 
	public int ElapsedMilliseconds { get; set; } 
}
```

Foreach decorated method, it creates an instance of `DiagnosticEntiyBase` with input and result data of the method

## Projects

- **Gnobbi.DebugTools.Decorator.Abstractions**  
  Contains core abstractions, attributes, and base types for diagnostics and decoration.

- **Gnobbi.DebugTools.Decorator.SourceCodeGenerator**  
  Implements Roslyn-based source generators to automatically create decorators for interfaces marked with `[Decorate]` attributes.

- **Gnobbi.DebugTools.Decorator.Test**  
  Provides sample domain classes, repositories, and NUnit-based tests to demonstrate and validate the decorator and diagnostics functionality.

## Requirements

- .NET 9.0 (for test and abstractions projects)
- .NET Standard 2.0 (for source generator)
- Visual Studio 2022 or later

## Getting Started

1. Clone the repository.
2. Open the solution in Visual Studio.
3. Build all projects.
4. Run the tests in `Gnobbi.DebugTools.Decorator.Test` to verify functionality.

## Status

**Work in progress.**  
The project is in an early working state and not yet feature-complete.

## License

MIT License.