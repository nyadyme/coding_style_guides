# PowerShell — AI Coding Rules

Apply these rules when generating or reviewing PowerShell code.

## Boilerplate (every script)

- `Set-StrictMode -Version Latest` and `$ErrorActionPreference = 'Stop'`.
- `#Requires` statements for version, modules, and admin elevation.
- Comment-based help (`.SYNOPSIS`, `.PARAMETER`, `.OUTPUTS`, `.EXAMPLE`).
- `[CmdletBinding()]` on all functions.

## Formatting

- 4 spaces indentation. Never tabs.
- 115 characters recommended line limit.
- Break long lines with splatting (preferred) or backtick (sparingly).
- Two blank lines between function definitions.
- One blank line after `param()` block.
- Opening brace on same line (One True Brace Style).

## Naming

- `Verb-Noun` for all functions using approved verbs from `Get-Verb`.
- `$PascalCase` for variables and parameters.
- No aliases in scripts: `Get-ChildItem`, not `gci`. No positional parameters.
- Always name parameters explicitly: `Copy-Item -Path $src -Destination $dst`.
- Approved verb mappings: `New-` (create), `Get-` (read), `Set-` (modify), `Remove-` (delete), `Test-` (check), `Start-`/`Stop-` (process), `ConvertTo-`/`ConvertFrom-` (format).

## Functions

- Small, one thing. Max ~40 lines.
- All functions must be advanced functions with `[CmdletBinding()]`.
- `[OutputType()]` declaration.
- Parameter validation attributes: `[ValidateNotNullOrEmpty()]`, `[ValidateRange()]`, `[ValidateSet()]`, `[ValidateScript()]`, `[ValidatePattern()]`.
- `[Parameter(Mandatory, ValueFromPipeline)]` for pipeline input.
- `begin`/`process`/`end` blocks for pipeline-accepting functions.
- `SupportsShouldProcess` with `$PSCmdlet.ShouldProcess()` for destructive operations.

## Pipeline & Output

- Return objects, not formatted strings. Let caller decide formatting.
- No `Write-Host` for data output (bypasses pipeline). Use implicit output.
- `[OutputType()]` to declare return types.
- Filter left, format right. `Format-*` only as the very last command.
- No `Format-*` in scripts except for final display.
- PowerShell passes **objects** through the pipeline, not text. This is fundamental.
- Always output `[PSCustomObject]`, typed .NET objects, or custom classes — never raw strings for structured data.
- Never convert to string prematurely (no `.ToString()`, `Out-String`, or `Format-*` inside data-producing functions).
- Parse external tool text output into objects immediately at the boundary.
- Consuming functions must expect and handle objects consistently — emit the same object shape from all code paths.

## Variables & Data

- Type accelerators: `[string]$Name`, `[int]$Port`.
- `Set-Variable -Option Constant` for true constants.
- `[System.Collections.Generic.List[T]]` for collection building. Never `$arr += $item`.
- `[PSCustomObject]@{ ... }` for structured data.
- Splatting (`@params`) over backtick continuation.
- Minimise `$global:` scope. Use `$script:` for module state.

## Documentation (Comment-Based Help)

- `.SYNOPSIS` (always), `.DESCRIPTION`, `.PARAMETER` (all params), `.OUTPUTS` (always), `.EXAMPLE` (at least one).
- Place help block inside function, before `[CmdletBinding()]`.

## Error Handling

- `$ErrorActionPreference = 'Stop'` so non-terminating errors are caught.
- `try`/`catch`/`finally` for structured handling.
- Catch specific exception types: `catch [System.IO.FileNotFoundException]`.
- `$PSCmdlet.ThrowTerminatingError()` in module functions (not `throw`).
- `Write-Error` for non-terminating (one item in batch fails).
- `finally` for resource cleanup (`.Dispose()` on IDisposable objects).
- `Write-Verbose` for diagnostics, `Write-Warning` for recoverable issues.
- Never `Write-Host` for data. `Write-Information` for structured info.

## Modules

- One module per capability. Module manifest (`.psd1`) always.
- Explicitly list `FunctionsToExport` — never `'*'`.
- `VariablesToExport = @()`, `AliasesToExport = @()`.
- One module per concern: one SQL toolset, one logging approach.

## Concurrency

- `ForEach-Object -Parallel` (PS 7+) with `-ThrottleLimit`.
- `$using:variable` to pass parent-scope data.
- No shared mutable state. Thread-safe collections for aggregation.
- `Start-Job`/`Start-ThreadJob` for background work.

## Database

- Explicit transactions via `SqlConnection.BeginTransaction()`.
- Parameterised queries: `@Email` parameters, never string interpolation.
- `finally { $connection.Dispose() }` for connection cleanup.
- Use `dbatools` or `SqlServer` module over raw ADO.NET when possible.
- **SQL injection protection:** always use `@ParamName` placeholders with `SqlParameter` objects.
- Never build SQL with string concatenation or `"... WHERE col = '$var'"` interpolation.
- Validate and sanitise input before it reaches the database layer — defence in depth.
- Use stored procedures where appropriate for additional protection and query plan caching.

## Security

- Execution policy: `RemoteSigned` or `AllSigned` in production.
- Never `Invoke-Expression` (PowerShell's `eval`).
- `[PSCredential]` for credentials. `SecretManagement` module for secrets.
- Never hardcode credentials. `Export-Clixml` for encrypted storage.
- Parameter validation for all external input.

## Copy Semantics

- Variables hold references. `.Clone()` for shallow copy of hashtables/arrays.
- `PSSerializer.Serialize`/`Deserialize` round-trip for deep copy.

## Testing

- Pester v5+ for all testing. `Describe`/`Context`/`It` structure.
- `.Tests.ps1` suffix mirroring source files.
- Arrange-Act-Assert. `Should` assertions.
- `Mock` at boundaries. `Should -Invoke` for verification.

## Patterns

- Factory: `New-` functions returning `[PSCustomObject]`.
- Builder: splatting hashtables. Decorator: wrapper functions with retry/logging.
- Strategy: script blocks as parameters.

## Defensive Programming

- Validate all external input at function boundaries using parameter validation attributes.
- Use `[ValidateNotNullOrEmpty()]`, `[ValidateLength()]`, `[ValidateRange()]`, `[ValidatePattern()]`, `[ValidateSet()]`, `[ValidateScript()]`.
- Type-constrain all parameters: `[string]$Name`, `[int]$Port` — never untyped.
- Validate string lengths explicitly for data that will be stored or transmitted.
- Check array/collection bounds before indexing.
- Validate file paths exist with `Test-Path` before operations.
- Never trust external input: registry values, environment variables, file content, API responses.
- Sanitise input used in file paths, registry paths, or WMI queries.
- `SupportsShouldProcess` with `-WhatIf`/`-Confirm` for destructive operations as a defensive measure.
- `$null` comparisons: put `$null` on the LEFT (`$null -eq $x`, not `$x -eq $null`). When the left operand is a collection, `-eq` filters element-wise — PSScriptAnalyzer enforces this via `PSPossibleIncorrectComparisonWithNull`.
- Do not name parameters/variables after automatic variables: `$args`, `$input`, `$matches`, `$error`, `$event`, `$host`, `$home`, `$pwd`, `$psitem`, `$this`, `$_`. PSScriptAnalyzer flags via `PSAvoidAssignmentToAutomaticVariable`.

## Project Structure

- Module: `Public/`, `Private/`, `Classes/`, `Tests/` directories.
- `.psm1` dot-sources `Public/*.ps1` and `Private/*.ps1`.
- `Export-ModuleMember -Function $publicFunctions.BaseName`.

## Tooling

- PSScriptAnalyzer (lint/format), Pester (test), platyPS (docs).
- InvokeBuild/psake (build automation), SecretManagement (secrets).

## SBOM Creation

- Declare module dependencies in `RequiredModules` in .psd1 manifest.
- Pin module versions: `@{ ModuleName = 'Pester'; ModuleVersion = '5.4.0' }`.
- Use `Get-InstalledModule` to inventory current modules.
- Check for outdated modules and updates regularly.
- Document external dependencies and .NET assembly requirements.
- Gate CI/CD on version compatibility and security checks.
