# Introduction

This project consists of a MSBuild targets file that provider greater control over NuGet dependencies when using SDK-style projects.

When packing SDK-style projects, there is no control over the resulting NuGet package's dependencies. The resulting package merely specifies the minimum version that is accepted. This is a problem because a later version of the dependency may contain breaking changes.

This project allows to specify stricter version requirements for each referenced NuGet package.

The following version range types are supported:

| Type | Description | Resulting [version range](https://docs.microsoft.com/en-us/nuget/concepts/package-versioning#version-ranges-and-wildcards) from `1.2.3` |
|-|-|-|
| Exact | Exact version match. Only this specific version will be allowed. | `[1.2.3]` |
| SemVer | Allow any minor version increment, but require the same major version. | `[1.2.3, 2.0.0)` |
| Minimum | Minimum version, inclusive. Any superior version will be allowed. | `1.2.3` |

# Installation

Simply install this package:

```
Install-Package BetterPackageReferences
```

By default, the following version range types are assumed:

| Reference type | Default value | Control Variable |
|-|-|-|
| ProjectReference | Exact | `DefaultVersionRangeTypeForProjectReferences` |
| PackageReference | SemVer | `DefaultVersionRangeTypeForPackageReferences` |

The defaults can be changed by setting the corresponding variable:

```xml
<PropertyGroup>
  <DefaultVersionRangeTypeForProjectReferences>SemVer</DefaultVersionRangeTypeForProjectReferences>
  <DefaultVersionRangeTypeForPackageReferences>Minimum</DefaultVersionRangeTypeForPackageReferences>
</PropertyGroup>
```
The version range type can also be configured for a specific reference by adding the `VersionRangeType` metadata:

```xml
<ProjectReference Include="..\OtherPackage\OtherPackage.csproj">
  <VersionRangeType>SemVer</VersionRangeType>
</ProjectReference>
```

```xml
<PackageReference Include="Microsoft.Extensions.Logging.Abstractions" Version="3.0.1">
  <VersionRangeType>Exact</VersionRangeType>
</PackageReference>
```

# Example

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="BetterPackageReferences" Version="1.0.0">
      <PrivateAssets>all</PrivateAssets>
    </PackageReference>

    <PackageReference Include="AutoMapper" Version="7.0.1" />
    <PackageReference Include="Microsoft.Extensions.Logging.Abstractions" Version="3.0.1">
      <VersionRangeType>Exact</VersionRangeType>
    </PackageReference>
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\OtherPackage\OtherPackage.csproj" />
  </ItemGroup>

</Project>
```

Packing the above project will result in a NuGet package with the following metadata:

```xml
<?xml version="1.0" encoding="utf-8"?>
<package xmlns="http://schemas.microsoft.com/packaging/2013/05/nuspec.xsd">
  <metadata>
    <id>MyPackage</id>
    <version>1.2.3</version>
    <dependencies>
      <group targetFramework=".NETStandard2.0">
        <dependency id="AutoMapper" version="[7.0.1, 8.0.0)" />
        <dependency id="Microsoft.Extensions.Logging.Abstractions" version="[3.0.1]" />
        <dependency id="OtherPackage" version="[1.2.3]" />
      </group>
    </dependencies>
  </metadata>
</package>
```