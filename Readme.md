# CSharp.SourceGen.Scriban

A C# source generator that renders [Scriban](https://github.com/scriban/scriban) templates.

- [Installation](#installation)
- [Source generation](#source-generation)
- [Saving generated files](#saving-generated-files)
- [Limitations](#limitations)
- [Example](#example)
- [See also](#see-also)


## Installation

```text
dotnet add package Dartk.CSharp.SourceGen.Scriban
```

To avoid propagating dependency on the package set the option `PrivateAssets="all"` in the project file:

```xml
<ItemGroup>
    <PackageReference Include="Dartk.CSharp.SourceGen.Scriban"
                      Version="0.1.0-alpha1"
                      PrivateAssets="All" />
</ItemGroup>
```

## Source generation

Include scriban template files with `.scriban` extension to the project as `AdditionalFiles`. For example, to render all scriban templates in the `ScribanTemplates` folder add this to the project file:

```xml
<ItemGroup>
    <AdditionalFiles Include="ScribanTemplates/**" />
</ItemGroup>
```

A [complete example](#example) is presented below.

If a `.scriban` file's name starts with an underscore '_', then it will not be rendered. But it can be included in other templates using the [`include`](https://github.com/scriban/scriban/blob/master/doc/language.md#911-include-name-arg1argn) statement.

> **Warning**: Microsoft Visual Studio 22 (tested on version 17.4.3 on Windows OS) will call a source generator on every edit of the files that are being cached by the generator. Thus, every character insertion or deletion in a `.scriban` template will cause the template rendering. Therefore, edit those files in an external editor for better performance. JetBrains Rider does not have this issue, it will invoke source generators only when changes to the cached files are saved.


## Saving generated files

To save the generated source files set properties `EmitCompilerGeneratedFiles` and `CompilerGeneratedFilesOutputPath` in the project file:

```xml
<PropertyGroup>
    <EmitCompilerGeneratedFiles>true</EmitCompilerGeneratedFiles>
    <!--Generated files will be saved to 'obj\GeneratedFiles' folder-->
    <CompilerGeneratedFilesOutputPath>$(BaseIntermediateOutputPath)\GeneratedFiles</CompilerGeneratedFilesOutputPath>
</PropertyGroup>
```


## Limitations

Passing parameters to scriban templates is not supported.


## Example

Create a new console C# project:

```text
dotnet new console Example.SourceGenScriban
```

Install the package `Dartk.CSharp.SourceGen.Scriban` and set the property `PrivateAssets="All"` by editing the project file `Example.SourceGenScriban.csproj`:

```xml
<ItemGroup>
    <PackageReference Include="Dartk.CSharp.SourceGen.Scriban"
                      Version="0.1.0-alpha1"
                      PrivateAssets="All"/>
</ItemGroup>
```

Create a `ScribanTemplates` folder in the project directory and include it's content as `AdditionalFiles`:

```xml
<ItemGroup>
    <AdditionalFiles Include="ScribanTemplates/**" />
</ItemGroup>
```

Add the following files to the `ScribanTemplates` folder:

`_global.scriban`

```liquid
{{
{{
# File name starts with an underscore, therefore will not be rendered.
# But it can be included in other templates.

numbers = [ "one", "two", "three" ]
}}
```

`generated-values.scriban`

```liquid
{{- include '_numbers.scriban'    # include statement is supported -}}

namespace Generated;

public record Number(int Int, string String) {
    {{- i = 0 }}
    {{- for number in numbers    # using variable 'numbers' from '_numbers.scriban' }}
    public static readonly Number {{ string.capitalize number }} = new ({{ ++i }}, "{{ number }}");
    {{- end }}
}
```

The generator will skip `_numbers.scriban` but render `Number.scriban`:

```c#
// Generated by Dartk.SourceGen.Scriban from 'Number.scriban'
namespace Generated;

public record Number(int Int, string String) {
    public static readonly Number One = new (1, "one");
    public static readonly Number Two = new (2, "two");
    public static readonly Number Three = new (3, "three");
}
```

Now `Generated.Number` class can be used in your code.

Put this in the `Program.cs`:

```c#
using static System.Console;

WriteLine(Generated.Number.One);
WriteLine(Generated.Number.Two);
WriteLine(Generated.Number.Three);
```

Code above will write the following:

```text
Number { Int = 1, String = one }
Number { Int = 2, String = two }
Number { Int = 3, String = three }
```


## See also

* [Scriban](https://github.com/scriban/scriban) - A fast, powerful, safe and lightweight scripting language and engine for .NET
* [CSharp.SourceGen.Csx](https://github.com/dartk/csharp-sourcegen-csx) - Generate C# code from C# scripts
* [CSharp.SourceGen.Fsx](https://github.com/dartk/csharp-sourcegen-fsx) - Generate C# code from F# scripts
* [CSharp.SourceGen.Examples](https://github.com/dartk/csharp-sourcegen-csx) - Examples that demonstrate how to use `CSharp.SourceGen.*` code generators
