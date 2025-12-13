# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a modernized port of Microsoft Patterns and Practices Enterprise Library Common module to .NET Standard 2.0. The original EntLib was discontinued by Microsoft, but this project maintains the core functionality for applications still relying on EntLib patterns.

**Targets**: .NET Framework 4.6.1+, .NET Standard 2.0+, .NET Core 2.0+
**NuGet Package**: `Rsft.EntLib.Common.Standard`

## Repository Structure

```
src/
├── Common.sln                    # Main solution file
├── Components/
│   └── Common.csproj            # Main library project (generates NuGet package)
│       ├── Configuration/       # Configuration infrastructure (file-based, hierarchical, merging)
│       ├── Instrumentation/     # Performance counters and event logging
│       ├── Utility/             # Guard, type utilities, extensions, string resolvers
│       └── GlobalSuppressions.cs
└── Tests/
    ├── Common/                  # Unit tests (VSTS format, .NET 4.6.1 only)
    │   └── Common.Tests.VSTS.csproj
    └── Common.TestSupport/      # Shared testing utilities
        └── Common.TestSupport.csproj
```

## Key Architecture

### Configuration System
The configuration module (`Configuration/`) provides a flexible, extensible configuration management system:
- **FileConfigurationSource**: Loads configuration from .config files
- **HierarchicalConfigurationSource**: Merges configuration from multiple sources
- **ConfigurationSourceFactory**: Factory pattern for creating configuration sources
- Custom type converters (e.g., `AssemblyQualifiedTypeNameConverter`, `ByteArrayTypeConverter`)
- Event-driven: Sources can track changes and notify listeners

Note: `EnterpriseLibraryContainer.cs` and `SectionChangedEventArgs.cs` were removed due to .NET Standard compatibility constraints.

### Instrumentation System
The instrumentation module (`Instrumentation/`) provides performance counters and event logging:
- **EnterpriseLibraryPerformanceCounter**: Wrapper around System.Diagnostics performance counters
- **EnterpriseLibraryPerformanceCounterFactory**: Creates and manages counters
- **EventLogEntryFormatter**: Formats messages for Windows event log
- Attribute-based configuration: `PerformanceCounterAttribute`, `EventLogDefinitionAttribute`

Note: Installer builders were removed due to missing .NET Standard APIs (System.Configuration.Install limitations).

### Utility Modules
- **Guard.cs**: Parameter validation (NotNull, NotNullOrEmpty, TypeIs, etc.)
- **IStringResolver/ResourceStringResolver**: String resource resolution patterns
- **TypeExtensions**: Reflection utilities
- **EnumerableExtensions**: LINQ-style utilities
- **StaticReflection**: Compile-time type/member detection patterns
- **ReplacementFormatter.cs**: Template-based string formatting for messages with named placeholders
- **ResourceStringLoader.cs**: Loads and caches resource strings from embedded resources

### Top-Level Utility Files
- **GlobalSuppressions.cs**: Code analysis rule suppressions (FxCop, StyleCop). Documents why specific warnings are suppressed globally.

## Design Patterns

This library uses several key design patterns that are worth understanding:

### Fluent Interfaces (IFluentInterface)
The `IFluentInterface` interface hides object methods (GetType, GetHashCode, ToString, Equals) from IntelliSense when building fluent APIs. This keeps the API surface clean for users building method chains.

### Factory Pattern
Multiple factory classes manage object creation:
- **ConfigurationSourceFactory**: Creates configuration sources by type name (supports polymorphic instantiation)
- **EnterpriseLibraryPerformanceCounterFactory**: Creates and manages performance counter instances

### Attribute-Based Configuration
Metadata is declared via attributes that drive runtime behavior:
- **PerformanceCounterAttribute**, **PerformanceCountersDefinitionAttribute**: Declare counters on classes
- **EventLogDefinitionAttribute**: Declare event log sources
- **HandlesSectionAttribute**: Map configuration sections to handlers
- **RegisterAsMetadataTypeAttribute**: Associate metadata types with data types

The system reflects over these attributes to discover and configure components.

### String Resolution Pattern (IStringResolver)
The `IStringResolver` interface allows flexible string/resource resolution:
- **ConstantStringResolver**: Returns a fixed string
- **DelegateStringResolver**: Resolves strings via a delegate
- **ResourceStringResolver**: Loads strings from resource files
This pattern lets code pass string resolution logic without hard-coding resource lookups.

## Build Commands

**Restore dependencies:**
```bash
dotnet restore src/Common.sln
```

**Build the solution:**
```bash
dotnet build src/Common.sln
```

**Build in Release mode:**
```bash
dotnet build src/Common.sln -c Release
```

**Build only the main library:**
```bash
dotnet build src/Components/Common.csproj
```

**Run all tests:**
```bash
dotnet test src/Common.sln
```

**Run a specific test class:**
```bash
dotnet test src/Common.sln --filter "ClassName=YourTestClass"
```

**Run a single test method:**
```bash
dotnet test src/Common.sln --filter "Name~YourTestMethod"
```

**Pack NuGet package:**
```bash
dotnet pack src/Components/Common.csproj -c Release
```

The package will be generated in `src/Components/bin/Release/` and includes both .NET Standard 2.0 and .NET Framework 4.6.1 assemblies.

## Important Notes

### Assembly Signing
The library is signed with `Rolosoft.Practices.EnterpriseLibrary.Common.snk`. The project file references this key. Do not modify the signing configuration.

### .NET Standard Compatibility Constraints
Several original EntLib features were removed due to missing APIs in .NET Standard:
- Configuration container initialization (`EnterpriseLibraryContainer`)
- Event handling for section changes
- Performance counter installer builders
- Some event log installation features

If you need these features, they may need to be re-implemented using modern .NET approaches.

### Testing
- **Test Framework**: Uses Visual Studio Unit Test Framework (MSTest)
- **Mocking**: Moq 4.8.2
- **Castle.Core**: Used by Moq for dynamic proxy generation
- Tests are .NET Framework 4.6.1 only (some compatibility issues prevent migration to .NET Standard)
- Configuration tests use `.config` files in the test directory (copied at build time via `CopyToOutputDirectory`)

**Common.TestSupport** provides shared testing utilities:
- Base classes for fixtures
- Mock/stub implementations of configuration interfaces
- Helper methods for test object construction
- Resource loading helpers for test configuration files

**Configuration Files in Tests**: Many configuration tests validate round-trip serialization/deserialization using `.config` files:
- Files are physically located in `Tests/Common/Configuration/` and other test subdirectories
- They are copied to the output directory at build time
- Tests instantiate `FileConfigurationSource` or `HierarchicalConfigurationSource` with paths to these files
- See `MergedConfigurationFile.config` for examples of hierarchical merging
- See `ComposedConfigurationFile.config` for examples of composite sources

**Debugging Tests**: Run with verbose output using:
```bash
dotnet test src/Common.sln --verbosity detailed
```

### Documentation
Original Microsoft documentation: https://msdn.microsoft.com/en-us/library/ff648951.aspx

### Version Management
The project uses semantic versioning. Current version is **7.0.0** (set in `Common.csproj`):

- **AssemblyVersion**: `7.0.0.0` (increment major version for breaking changes)
- **FileVersion**: `7.0.0.0` (should match AssemblyVersion)
- **Version**: `7.0.0` (NuGet package version, should match AssemblyVersion without the trailing .0)
- **PackageReleaseNotes**: Update with summary of changes before packing release

These should always be updated together for releases. The NuGet package metadata includes these versions so users can see what they're getting.

### Configuration Merging and Hierarchies
The configuration system supports sophisticated source composition:

- **HierarchicalConfigurationSource**: Merges configuration from multiple sources with a parent/child relationship. Child sources override parent settings.
- **CompositeConfigurationSource**: Chains multiple sources together (simpler than hierarchical, no merging).
- **Merging Behavior**: See test files `MergedConfigurationFile.config` and `HierarchicalConfigurationSourceHandlerFixture.cs` for how sections and elements are merged, especially the Events.cs variant which demonstrates change notifications through hierarchies.

This is useful when you want configuration to come from multiple files or sources, with override capabilities.

## Troubleshooting

### Build Issues

**Multi-targeting failures** (building for both .NET Standard 2.0 and .NET Framework 4.6.1):
- Ensure the `TargetFrameworks` property is correctly set in Common.csproj
- .NET Framework 4.6.1 tests cannot be built on .NET Core/.NET 5+ toolchains; use a Windows machine with .NET Framework installed, or use Docker

**Configuration file not found in tests**:
- Verify `.config` files have `CopyToOutputDirectory` set to `Always` or `PreserveNewest` in the test project
- Check that test files are in the correct directory structure
- Paths in test code may be relative to the output directory, not the source directory

**Assembly signing failures**:
- The signing key file `Rolosoft.Practices.EnterpriseLibrary.Common.snk` must exist in `src/Components/`
- If it's missing or corrupted, the build will fail with a signing error
- Do not regenerate the key as it changes the assembly identity (old code can't reference the new assembly)

### Runtime Issues

**Type loading failures with Configuration**:
- Custom configuration element types must be public and have parameterless constructors
- Type names in configuration must be fully assembly-qualified (e.g., `"MyNamespace.MyType, MyAssembly"`)
- Use `AssemblyQualifiedTypeNameConverter` when defining custom converters

**Performance Counter access denied**:
- Performance counters may require elevated privileges; tests may fail if run without admin rights on Windows
- Some systems don't have the required counter categories; see `EnterpriseLibraryPerformanceCounterFactory`

## Development Tips

### Quick Build Validation
To quickly validate changes without running all tests:
```bash
dotnet build src/Components/Common.csproj
```

This builds only the main library and is faster than a full solution build.

### Testing Workflow
1. **Quick run**: `dotnet test src/Tests/Common/Common.Tests.VSTS.csproj` (test project only)
2. **With coverage**: Add coverage tools via NuGet and run tests with coverage
3. **Specific failure**: Use `--filter` to isolate and debug single tests

### Before Publishing a Release
1. Update version numbers in `Common.csproj` (AssemblyVersion, FileVersion, Version, PackageReleaseNotes)
2. Run full test suite: `dotnet test src/Common.sln`
3. Build release package: `dotnet pack src/Components/Common.csproj -c Release`
4. Inspect the generated `.nupkg` file to ensure both .NET Standard 2.0 and .NET Framework 4.6.1 assemblies are included
5. Push to NuGet: `dotnet nuget push path/to/package.nupkg --api-key <key>`

### Debug vs Release Builds
- **Debug**: Includes symbols and debug info. Good for development and troubleshooting. Larger binaries.
- **Release**: Optimized, stripped of debug info. Generates XML documentation files (see `DocumentationFile` in csproj). Use for shipping.

The project is configured to generate documentation files in Release builds, which are included in the NuGet package.

## Common Development Tasks

**Modifying Configuration System**: Changes to configuration classes typically require corresponding test fixtures in `Tests/Common/Configuration/`. Many fixtures demonstrate round-trip serialization/deserialization.

**Adding Performance Counters**: Add attributes to classes (`PerformanceCounterAttribute`), then update factory (`EnterpriseLibraryPerformanceCounterFactory`) if needed.

**Extending Utility Classes**: Keep utility classes in `Components/Utility/`. Follow the pattern used by existing utilities (e.g., `Guard` for parameter validation, extension methods for framework types).

**Publishing Updates**: NuGet package is auto-generated via `GeneratePackageOnBuild` in the project file. Update version numbers in `Common.csproj` before building Release packages.
