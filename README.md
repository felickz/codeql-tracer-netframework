# codeql-tracer-netframework
Generates a CodeQL custom tracing configuration for ASP.NET to make MVCBuildViews and other compiler options conditional.  This action overrides the default lua tracing configuration for csharp found [here](https://github.com/github/codeql/blob/7e1dd38623f822046bda8e7d7652cb41f638d417/csharp/tools/tracing-config.lua#L149-L167) using the [--extra-tracing-config](https://docs.github.com/en/code-security/codeql-cli/codeql-cli-manual/database-init#--extra-tracing-configtracing-configlua) CodeQL option.

> [!WARNING]
>Overriding the default CodeQL Csharp Tracer configuration can increase the potential for false positives / false negatives in security analysis.

## Usage

Register the action before any of the CodeQL Actions:

```
...

    strategy:
      fail-fast: false
      matrix:
        language: [ 'csharp' ]

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Custom Lua Config Tracer for ASPNET MVC
      uses: felickz/codeql-tracer-netframework@main
      with:
        MvcBuildViews: false

    # Initializes the CodeQL tools for scanning.
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v3
      with:
        languages: ${{ matrix.language }}

    # Tracer configuration applies to either autobuild OR custom build steps as the options are injected automatically into the underlying compiler invocations.
    - name: Autobuild
      uses: github/codeql-action/autobuild@v3

    #   If the Autobuild fails above, remove it and uncomment the following three lines.
    #   modify them (or add more) to build your code if your project, please refer to the EXAMPLE below for guidance.

    # - run: |
    #     echo "Run, Build Application using script"
    #     ./location_of_script_within_repo/buildscript.sh


...

```

## Options

### MvcBuildViews

`/p:MvcBuildViews=true`: This flag is used to enable the precompilation of ASP.NET MVC views. Precompiling views can help catch errors at build time that would otherwise only be caught at runtime, and can also result in faster startup times for your application, since the views don't need to be compiled on the fly the first time they're requested.  This is required for security scanning as the CodeQL engine requires source code files to be compiled so that they can be fully extracted into the CodeQL database.  Disabling this flag can increase the potential for false negatives in security analysis as the view engine is a major location for vulnerability sources and sinks.

### EmitCompilerGeneratedFiles

`/p:EmitCompilerGeneratedFiles=true`: This flag is used to instruct the compiler to output additional files during the build process that show what code was generated by the compiler. This is required to instruct the compiler write generated files to disk (such as .cshtml.g.cs), so that they can be extracted by CodeQL.   Disabling this flag can increase the potential for false negatives in security analysis as the view engine is a major location for vulnerability sources and sinks.

### DisableUseSharedCompilation
`/p:UseSharedCompilation=false`: This flag is used to disable the reuse of the same compiler process for multiple compilations, rather than having to start up a new process for each build. Setting this flag to false can be useful in certain scenarios where shared compilation causes issues such as the case with CodeQL where the tracer needs full visibility into every call of the compiler process.   Disabling this flag can increase the potential for false positives/negatives in security analysis as source code files may not be extracted into the CodeQL database and therefore will be excluded from analysis.


