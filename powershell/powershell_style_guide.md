# PowerShell Coding Style Guidelines

A comprehensive guide rooted in *The PowerShell Practice and Style Guide*
(PoshCode/PowerShellPracticeAndStyle), Microsoft's *Cmdlet Development
Guidelines*, the *PowerShell Scripting* documentation, and established
software engineering literature — notably *Design Patterns* (Gamma et al.,
1994) and *Clean Code* (Martin, 2008).

---

## Table of Contents

1. [Philosophy](#1-philosophy)
2. [Script Structure & Boilerplate](#2-script-structure--boilerplate)
3. [Code Layout & Formatting](#3-code-layout--formatting)
4. [Naming Conventions](#4-naming-conventions)
5. [Functions & Cmdlets](#5-functions--cmdlets)
6. [Variables & Data Types](#6-variables--data-types)
7. [Documentation & Comments](#7-documentation--comments)
8. [Error Handling](#8-error-handling)
9. [Modules & Imports](#9-modules--imports)
10. [Pipeline & Output](#10-pipeline--output)
11. [Design Patterns in PowerShell](#11-design-patterns-in-powershell)
12. [Testing](#12-testing)
13. [Database Access & ACID](#13-database-access--acid)
14. [Concurrency & Jobs](#14-concurrency--jobs)
15. [Performance & Idiomatic PowerShell](#15-performance--idiomatic-powershell)
16. [Security](#16-security)
17. [Defensive Programming & Input Validation](#17-defensive-programming--input-validation)
18. [Project Structure](#18-project-structure)
19. [Tooling](#19-tooling)
20. [SBOM Creation](#20-sbom-creation)
21. [References](#21-references)

---

## 1. Philosophy

### 1.1 PowerShell's Core Values

PowerShell is a **task automation** and **configuration management** shell
built on .NET. It passes **objects** through the pipeline — not text — making
it fundamentally different from POSIX shells.

### 1.2 Guiding Principles

| Principle | Source | Summary |
|---|---|---|
| **Verb-Noun consistency** | Cmdlet Development Guidelines | Every command follows `Verb-Noun` naming. No exceptions. |
| **Pipeline by design** | PowerShell in Action | Write functions that accept pipeline input and produce pipeline output. |
| **Readability over brevity** | PS Practice and Style Guide | Use full cmdlet and parameter names in scripts, not aliases. |
| **Fail explicitly** | *Clean Code*, defensive scripting | Use `$ErrorActionPreference = 'Stop'` so errors are never swallowed. |
| **Single Responsibility** | *Clean Code* Ch. 10, SOLID | Every function, module, and script does one thing. |
| **Idempotency** | Ops best practices | Running a script twice should produce the same result as running it once. |
| **YAGNI** | XP / *Clean Code* | Do not build for hypothetical future requirements. |

---

## 2. Script Structure & Boilerplate

### 2.1 File Extension

- Use `.ps1` for scripts, `.psm1` for modules, `.psd1` for module manifests.

### 2.2 Encoding

- PowerShell 5.1's default output encoding is **UTF-16 LE with BOM**
  (Unicode), which is incompatible with most cross-platform tools. Always
  specify `-Encoding utf8` (or `utf8BOM` / `utf8NoBOM` on PS 7+) when
  writing files. PowerShell 7+ defaults to **UTF-8 without BOM**.
- For source files (.ps1, .psm1, .psd1) save as **UTF-8 with BOM** so that
  Windows PowerShell 5.1 correctly handles non-ASCII characters; PS 7+
  reads either BOM and BOM-less UTF-8 without issue.

### 2.3 Strict Mode

Enable strict mode at the top of every script:

```powershell
Set-StrictMode -Version Latest
$ErrorActionPreference = 'Stop'
```

| Setting | Effect |
|---|---|
| `Set-StrictMode -Version Latest` | Disallows references to unset variables, uninitialised properties, and non-existent functions. |
| `$ErrorActionPreference = 'Stop'` | Promotes non-terminating errors to terminating — equivalent to `set -e` in Bash. |

> Note on `-Version Latest`: this binds to whatever strict-mode rules the
> *current* PowerShell engine considers "latest", which can introduce
> breaking checks when the script runs on a newer engine. For libraries
> intended to be portable across PowerShell 5.1 and 7+, pin to a known
> version such as `-Version 3.0`. Use `Latest` only when you control the
> execution environment.

### 2.4 Script Template

```powershell
#Requires -Version 5.1
#Requires -Modules @{ ModuleName = 'Pester'; ModuleVersion = '5.0' }

<#
.SYNOPSIS
    Brief one-line description.

.DESCRIPTION
    Detailed description of what the script does.

.PARAMETER Target
    The deployment target (staging or production).

.PARAMETER DryRun
    Preview actions without making changes.

.EXAMPLE
    .\Deploy-Application.ps1 -Target staging

.NOTES
    Author: Team Name
#>

[CmdletBinding()]
param(
    [Parameter(Mandatory)]
    [ValidateSet('staging', 'production')]
    [string]$Target,

    [switch]$DryRun
)

Set-StrictMode -Version Latest
$ErrorActionPreference = 'Stop'

function Deploy-Application {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$Target
    )

    Write-Verbose "Deploying to $Target..."
    # Deployment logic
}

# Entry point
Deploy-Application -Target $Target
```

### 2.5 `#Requires` Statements

Use `#Requires` to declare prerequisites at the top of the script — before
any code. PowerShell checks these before execution:

```powershell
#Requires -Version 7.0
#Requires -Modules Az.Accounts, Az.Storage
#Requires -RunAsAdministrator
```

---

## 3. Code Layout & Formatting

### 3.1 Indentation

- Use **4 spaces** per indentation level (PowerShell community default).
  Never use tabs.

### 3.2 Line Length

- **115 characters** is the recommended maximum (*PowerShell Practice and
  Style Guide*).
- Break long lines using **splatting** (preferred) or backtick (`` ` ``)
  continuation:

```powershell
# Preferred — splatting
$params = @{
    Path        = $sourcePath
    Destination = $destPath
    Recurse     = $true
    Force       = $true
}
Copy-Item @params

# Acceptable — backtick continuation (use sparingly)
Get-ChildItem -Path $sourcePath `
    -Filter '*.log' `
    -Recurse
```

- **Prefer splatting** over backtick continuation. Backticks are fragile —
  a trailing space after `` ` `` silently breaks the continuation.

### 3.3 Blank Lines (*PowerShell Practice and Style Guide*)

- **Two blank lines** between function definitions.
- **One blank line** between logical sections within a function.
- **No blank line** after the opening brace or before the closing brace of a
  function, `if`, `foreach`, `try`, or other block.
- **One blank line** after the `param()` block and before the function body.
- **One blank line** between the comment-based help block and `[CmdletBinding()]`.
- **No trailing blank lines** at the end of the file.

### 3.4 Braces

Opening brace on the **same line** as the statement (One True Brace Style):

```powershell
if ($condition) {
    # ...
} elseif ($other) {
    # ...
} else {
    # ...
}

foreach ($item in $collection) {
    Process-Item -Item $item
}

try {
    Invoke-RiskyOperation
} catch {
    Write-Error "Operation failed: $_"
} finally {
    Reset-State
}
```

### 3.5 Operators and Spacing

```powershell
# Yes — spaces around operators
$total = $price * $quantity
$isValid = ($age -ge 18) -and ($hasConsent -eq $true)

# No
$total=$price*$quantity
```

---

## 4. Naming Conventions

### 4.1 The Verb-Noun Rule

Every function and cmdlet **must** follow `Verb-Noun` naming using an
**approved verb** from `Get-Verb`:

| Entity | Convention | Example |
|---|---|---|
| Function / Cmdlet | `ApprovedVerb-SingularNoun` | `Get-User`, `Set-Configuration` |
| Script file | `Verb-Noun.ps1` | `Deploy-Application.ps1` |
| Module | `PascalCase` | `UserManagement`, `AzureTools` |
| Module manifest | Same as module + `.psd1` | `UserManagement.psd1` |
| Variable | `$PascalCase` | `$UserName`, `$OutputPath` |
| Parameter | `PascalCase` | `-FilePath`, `-ComputerName` |
| Constant | `$PascalCase` (with `Set-Variable -Option Constant`) | `$MaxRetries` |
| Private function | `Verb-Noun` (not exported from module) | `Resolve-InternalPath` |
| Class (PS 5+) | `PascalCase` | `UserAccount`, `ConfigEntry` |
| Enum (PS 5+) | `PascalCase`, members `PascalCase` | `[LogLevel]::Warning` |
| Boolean variable | Reads as assertion | `$IsActive`, `$HasPermission` |

### 4.2 Approved Verbs

Always use verbs from `Get-Verb`. Common mappings:

| Action | Approved Verb | NOT |
|---|---|---|
| Create a new resource | `New-` | ~~Create-~~, ~~Add-~~ (Add = append to collection) |
| Read / retrieve | `Get-` | ~~Fetch-~~, ~~Retrieve-~~, ~~Read-~~ |
| Modify in place | `Set-` | ~~Update-~~, ~~Modify-~~, ~~Change-~~ |
| Delete | `Remove-` | ~~Delete-~~, ~~Destroy-~~ |
| Check a condition | `Test-` | ~~Check-~~, ~~Validate-~~ |
| Start a process | `Start-` | ~~Run-~~, ~~Begin-~~ |
| Stop a process | `Stop-` | ~~Kill-~~, ~~End-~~ |
| Convert format | `ConvertTo-` / `ConvertFrom-` | ~~Transform-~~ |
| Send data | `Send-` | ~~Push-~~, ~~Transmit-~~ |
| Receive data | `Receive-` | ~~Pull-~~, ~~Fetch-~~ |

### 4.3 Naming Guidance (*Clean Code* Ch. 2)

- **Use full names.** `$ComputerName`, not `$cn`. Scripts are documentation.
- **No Hungarian notation.** The type system provides type information.
- **No aliases in scripts.** Use `Get-ChildItem`, not `gci` or `ls`. Aliases
  are for interactive use only.
- **No positional parameters in scripts.** Always name parameters explicitly:

```powershell
# Yes
Copy-Item -Path $source -Destination $dest -Force

# No — positional, unreadable
Copy-Item $source $dest -Force
```

---

## 5. Functions & Cmdlets

### 5.1 Size and Focus

- Functions should be **small** and do **one thing**.
- Extract complex logic into named helper functions.
- If a function exceeds ~40 lines, consider splitting it.

### 5.2 Advanced Functions

All functions in scripts and modules should be **advanced functions** with
`[CmdletBinding()]`:

```powershell
function Get-UserReport {
    <#
    .SYNOPSIS
        Generate a report of active users.

    .PARAMETER Domain
        The Active Directory domain to query.

    .PARAMETER IncludeDisabled
        Include disabled accounts in the report.

    .OUTPUTS
        [PSCustomObject] User report entries.

    .EXAMPLE
        Get-UserReport -Domain 'contoso.com'
    #>
    [CmdletBinding()]
    [OutputType([PSCustomObject])]
    param(
        [Parameter(Mandatory, ValueFromPipeline)]
        [string]$Domain,

        [switch]$IncludeDisabled
    )

    begin {
        Write-Verbose "Initialising report generator..."
        $results = [System.Collections.Generic.List[PSCustomObject]]::new()
    }

    process {
        $users = Get-ADUser -Filter * -Server $Domain
        if (-not $IncludeDisabled) {
            $users = $users | Where-Object { $_.Enabled }
        }
        foreach ($user in $users) {
            $results.Add([PSCustomObject]@{
                Name   = $user.Name
                Email  = $user.EmailAddress
                Domain = $Domain
            })
        }
    }

    end {
        $results
    }
}
```

### 5.3 `begin` / `process` / `end` Blocks

Use the three-block pattern when a function accepts **pipeline input**:

| Block | Runs | Use for |
|---|---|---|
| `begin` | Once, before the first pipeline object | Initialisation, resource acquisition |
| `process` | Once per pipeline object | Core per-item logic |
| `end` | Once, after the last pipeline object | Aggregation, cleanup, final output |

If a function does not accept pipeline input, a flat function body (no blocks)
is fine.

### 5.4 Parameter Validation

Use validation attributes to enforce constraints declaratively:

```powershell
param(
    [Parameter(Mandatory)]
    [ValidateNotNullOrEmpty()]
    [string]$Name,

    [ValidateRange(1, 65535)]
    [int]$Port = 8080,

    [ValidateSet('Debug', 'Info', 'Warning', 'Error')]
    [string]$LogLevel = 'Info',

    [ValidateScript({ Test-Path $_ -PathType Container })]
    [string]$OutputDirectory = '.',

    [ValidatePattern('^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')]
    [string]$Email
)
```

### 5.5 `SupportsShouldProcess` for Destructive Operations

Functions that modify state should support `-WhatIf` and `-Confirm`:

```powershell
function Remove-OldBackups {
    [CmdletBinding(SupportsShouldProcess, ConfirmImpact = 'High')]
    param(
        [Parameter(Mandatory)]
        [string]$BackupPath,

        [int]$RetentionDays = 30
    )

    $cutoff = (Get-Date).AddDays(-$RetentionDays)
    $oldFiles = Get-ChildItem -Path $BackupPath -File |
        Where-Object { $_.LastWriteTime -lt $cutoff }

    foreach ($file in $oldFiles) {
        if ($PSCmdlet.ShouldProcess($file.FullName, 'Remove')) {
            Remove-Item -Path $file.FullName -Force
        }
    }
}
```

### 5.6 Output Types

Declare `[OutputType()]` so callers and tools know what the function returns:

```powershell
[CmdletBinding()]
[OutputType([PSCustomObject])]
[OutputType([string], ParameterSetName = 'AsText')]
param( ... )
```

---

## 6. Variables & Data Types

### 6.1 Type Declarations

Use type accelerators for variable declarations when the type matters:

```powershell
[string]$Name = 'Alice'
[int]$Port = 8080
[datetime]$StartDate = Get-Date
[hashtable]$Config = @{ Timeout = 30; Retries = 3 }
[System.Collections.Generic.List[string]]$Items = @()
```

### 6.2 Constants

Use `Set-Variable` with `-Option Constant` or `-Option ReadOnly`:

```powershell
Set-Variable -Name MaxRetries -Value 3 -Option Constant
Set-Variable -Name DefaultTimeout -Value 30 -Option ReadOnly
```

- `Constant` — cannot be removed or changed (even with `-Force`).
- `ReadOnly` — cannot be changed but can be removed with
  `Remove-Variable -Force`.

### 6.3 Collections

Use typed .NET collections for performance-critical code:

```powershell
# Yes — typed list, fast Add()
$users = [System.Collections.Generic.List[PSCustomObject]]::new()
$users.Add($newUser)

# No — array concatenation, O(n) per append
$users = @()
$users += $newUser  # copies entire array every time
```

### 6.4 Custom Objects

Use `[PSCustomObject]` for structured data:

```powershell
$report = [PSCustomObject]@{
    ServerName = $env:COMPUTERNAME
    Status     = 'Healthy'
    CheckedAt  = Get-Date
    Uptime     = (Get-CimInstance Win32_OperatingSystem).LastBootUpTime
}
```

### 6.5 Splatting

Use splatting (`@params`) to pass parameters cleanly:

```powershell
$mailParams = @{
    From       = 'noreply@example.com'
    To         = $recipient
    Subject    = 'Deployment Complete'
    Body       = $body
    SmtpServer = 'mail.example.com'
}
Send-MailMessage @mailParams
```

### 6.6 Avoid `$global:` Scope

Minimise global variables. Pass data through parameters and return values.
If global state is needed, document it and scope it to the module with
`$script:`.

---

## 7. Documentation & Comments

### 7.1 Comment-Based Help

Every public function **must** have comment-based help in the `<# #>` block:

```powershell
function Get-ServerHealth {
    <#
    .SYNOPSIS
        Check the health of one or more servers.

    .DESCRIPTION
        Connects to each server via WinRM, checks CPU, memory, and disk
        usage, and returns a health report object.

    .PARAMETER ComputerName
        One or more server hostnames or IP addresses.

    .PARAMETER Credential
        Credentials for remote connection. Defaults to the current user.

    .OUTPUTS
        [PSCustomObject] A health report for each server.

    .EXAMPLE
        Get-ServerHealth -ComputerName 'web01', 'web02'

        Returns health reports for web01 and web02.

    .EXAMPLE
        'web01', 'web02' | Get-ServerHealth

        Pipeline input is supported.

    .NOTES
        Requires WinRM access to target servers.
    #>
    [CmdletBinding()]
    [OutputType([PSCustomObject])]
    param(
        [Parameter(Mandatory, ValueFromPipeline)]
        [string[]]$ComputerName,

        [PSCredential]$Credential
    )

    # ...
}
```

### 7.2 Help Keywords

| Keyword | Purpose | Required? |
|---|---|---|
| `.SYNOPSIS` | One-line description | Always |
| `.DESCRIPTION` | Detailed explanation | When synopsis is insufficient |
| `.PARAMETER Name` | Describe each parameter | For all parameters |
| `.OUTPUTS` | Type(s) returned | Always |
| `.EXAMPLE` | Usage example with explanation | At least one |
| `.NOTES` | Author, prerequisites, caveats | When applicable |
| `.LINK` | Related commands or documentation URLs | When helpful |
| `.INPUTS` | Types accepted from pipeline | When pipeline input is supported |

### 7.3 Inline Comments (*Clean Code* Ch. 4)

- Comments explain **why**, not what.
- Never commit commented-out code.
- Use `#` comments. Place them above the line they describe.
- Use `# TODO:` and `# FIXME:` sparingly.
- Write self-documenting code: meaningful variable and function names replace
  most comments.

---

## 8. Error Handling

### 8.1 Principles

- Set `$ErrorActionPreference = 'Stop'` so non-terminating errors become
  terminating — every error is caught or surfaced.
- **Never silently swallow errors.** Catch, handle, or propagate.
- Use `try`/`catch`/`finally` for structured error handling.

### 8.2 Terminating vs. Non-Terminating Errors

| Error Type | Default Behaviour | How to Catch |
|---|---|---|
| Terminating | Stops execution | `try`/`catch` |
| Non-terminating | Continues execution, writes to `$Error` | `-ErrorAction Stop` promotes it to terminating |

```powershell
# Promote non-terminating errors so they are caught
try {
    Get-Item -Path $path -ErrorAction Stop
} catch [System.Management.Automation.ItemNotFoundException] {
    Write-Warning "Path not found: $path"
} catch {
    Write-Error "Unexpected error: $_"
    throw
}
```

### 8.3 Custom Error Records

Create informative error records for reusable functions:

```powershell
function Get-UserById {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [int]$UserId
    )

    $user = $userStore[$UserId]
    if (-not $user) {
        $errorRecord = [System.Management.Automation.ErrorRecord]::new(
            [System.Exception]::new("User $UserId not found"),
            'UserNotFound',
            [System.Management.Automation.ErrorCategory]::ObjectNotFound,
            $UserId
        )
        $PSCmdlet.ThrowTerminatingError($errorRecord)
    }

    $user
}
```

### 8.4 `finally` for Cleanup

Use `finally` for resource cleanup — PowerShell's equivalent of context
managers / `defer` / `ensure`:

```powershell
$stream = $null
try {
    $stream = [System.IO.File]::OpenRead($path)
    # Process stream
} catch {
    Write-Error "Failed to read file: $_"
    throw
} finally {
    if ($stream) {
        $stream.Dispose()
    }
}
```

For .NET objects that implement `IDisposable`, always pair acquisition
with `try`/`finally` so `.Dispose()` runs on every exit path. PowerShell
has no `using`/`await using` statement (the `using` *keyword* is reserved
for `using namespace`, `using module`, and `using:` scope modifier — not
RAII):

```powershell
$reader = [System.IO.StreamReader]::new($path)
try {
    $content = $reader.ReadToEnd()
} finally {
    $reader.Dispose()
}
```

### 8.5 `Write-Error` vs. `throw` vs. `$PSCmdlet.ThrowTerminatingError`

| Method | Error Type | Use When |
|---|---|---|
| `Write-Error` | Non-terminating | Reporting but continuing (e.g. one item in a batch fails) |
| `throw` | Terminating (script-level) | Quick failure in scripts; not recommended in module functions |
| `$PSCmdlet.ThrowTerminatingError()` | Terminating (cmdlet-level) | Reusable functions in modules — proper error records, correct `$?` behaviour |

### 8.6 Logging

- Use `Write-Verbose` for diagnostic output (visible with `-Verbose`).
- Use `Write-Debug` for detailed debugging (visible with `-Debug`).
- Use `Write-Warning` for recoverable issues.
- Use `Write-Error` for non-fatal errors.
- Use `Write-Information` for structured informational output.
- **Never use `Write-Host`** for data output — it bypasses the pipeline and
  cannot be captured. Use it only for interactive user-facing display.

---

## 9. Modules & Imports

### 9.1 Module Design

- A module provides **one capability** — `UserManagement`, `Logging`,
  `DatabaseTools`.
- Export only public functions via `Export-ModuleMember` or the module
  manifest's `FunctionsToExport`.
- Keep internal helpers private (not exported).

### 9.2 Module Manifest (`.psd1`)

Always create a module manifest:

```powershell
@{
    RootModule        = 'UserManagement.psm1'
    ModuleVersion     = '1.0.0'
    GUID              = 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'
    Author            = 'Team Name'
    Description       = 'User account management cmdlets'
    PowerShellVersion = '5.1'
    FunctionsToExport = @('Get-User', 'New-User', 'Remove-User')
    CmdletsToExport   = @()
    VariablesToExport  = @()
    AliasesToExport    = @()
    RequiredModules    = @('ActiveDirectory')
}
```

- **Explicitly list exported functions.** Never use `'*'` — it defeats
  auto-discovery and slows import.
- Set `VariablesToExport` and `AliasesToExport` to empty arrays.

### 9.3 Import Ordering

Group `Import-Module` and `using module` statements:

1. **Built-in / Microsoft modules** (`Microsoft.PowerShell.Management`)
2. **Third-party modules** (`PSFramework`, `dbatools`)
3. **Internal / project modules**

```powershell
using module ActiveDirectory
using module Az.Accounts

using module .\Modules\InternalLogging
using module .\Modules\InternalConfig
```

### 9.4 One Module per Concern — No Redundant Dependencies

When multiple modules accomplish the same task, **pick one and use it
consistently**:

| Overlapping modules | Pick one |
|---|---|
| `Invoke-WebRequest` / `Invoke-RestMethod` | `Invoke-RestMethod` for APIs, `Invoke-WebRequest` for HTML |
| `dbatools` / `SqlServer` module | One SQL toolset per project |
| `PSReadLine` / custom input handling | `PSReadLine` (built-in) |
| `Pester` v4 / Pester v5 | Pester v5 exclusively |
| `Write-Host` / `Write-Information` | `Write-Information` for capturable output |

---

## 10. Pipeline & Output

### 10.1 Pipeline Design

PowerShell's pipeline passes **objects**, not text. Design functions to
participate in the pipeline:

```powershell
# Accepts pipeline input, produces pipeline output
function ConvertTo-Report {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory, ValueFromPipeline)]
        [PSCustomObject]$InputObject
    )

    process {
        [PSCustomObject]@{
            Name      = $InputObject.Name
            Status    = if ($InputObject.IsHealthy) { 'OK' } else { 'FAIL' }
            Timestamp = Get-Date -Format 'o'
        }
    }
}

# Usage
Get-Server | Test-Health | ConvertTo-Report | Export-Csv -Path report.csv
```

### 10.2 Output Rules

- **Return objects**, not formatted strings. Let the caller decide how to
  format.
- **Do not mix output types.** A function should return one type of object.
- **Do not use `Write-Host` for output** — it bypasses the pipeline.
- Use `Write-Output` or implicit output (preferred — just put the object on
  its own line).
- Use `[OutputType()]` to declare what the function returns.

### 10.3 Filtering Left, Formatting Right

Filter as early as possible in the pipeline. Format only at the end:

```powershell
# Yes — filter early, format last
Get-Process |
    Where-Object { $_.CPU -gt 100 } |
    Sort-Object CPU -Descending |
    Select-Object -First 10 |
    Format-Table Name, CPU, WorkingSet

# No — format in the middle (breaks the pipeline)
Get-Process |
    Format-Table Name, CPU |
    Where-Object { $_.CPU -gt 100 }  # too late — objects are format records
```

### 10.4 Avoid `Format-*` in Scripts

`Format-Table`, `Format-List`, etc. produce **format objects**, not data.
Never use them in scripts except as the very last command before display.
Use `Select-Object` and `Export-Csv` / `ConvertTo-Json` for structured output.

### 10.5 Object Passing & Pipeline Consistency

PowerShell passes **objects** through the pipeline, not text. This is the
fundamental characteristic that distinguishes PowerShell from POSIX shells
and must be understood to write correct, composable code.

#### Always Output Structured Objects

Functions must return `[PSCustomObject]`, typed .NET objects, or custom
classes — never raw strings for structured data. The consumer decides how
to display or format the result:

```powershell
# Good — structured object output
function Get-ServiceStatus {
    [CmdletBinding()]
    [OutputType([PSCustomObject])]
    param(
        [Parameter(Mandatory, ValueFromPipeline)]
        [string]$ServiceName
    )

    process {
        $svc = Get-Service -Name $ServiceName
        [PSCustomObject]@{
            Name   = $svc.Name
            Status = $svc.Status
            Host   = $env:COMPUTERNAME
        }
    }
}

# Bad — returning a formatted string
function Get-ServiceStatus {
    param([string]$ServiceName)
    $svc = Get-Service -Name $ServiceName
    "$($svc.Name) is $($svc.Status) on $env:COMPUTERNAME"
}
```

#### Declare Output Types

Use `[OutputType()]` on every function so callers and tooling know what
objects the function emits:

```powershell
[CmdletBinding()]
[OutputType([PSCustomObject])]
param( ... )
```

#### Never Convert to String Prematurely

Let the consuming command or the end user decide the display format. Do not
call `.ToString()`, `Out-String`, or `Format-*` inside functions that produce
data. Premature string conversion destroys the object's properties and makes
downstream filtering, sorting, and export impossible.

#### Parse External Tool Output at the Boundary

When calling external programs that return text (e.g. `git`, `kubectl`,
`netstat`), parse their output into objects **immediately** at the boundary
so the rest of the script works with structured data:

```powershell
# Parse text output from an external tool into objects
function Get-GitLog {
    [CmdletBinding()]
    [OutputType([PSCustomObject])]
    param([int]$Count = 10)

    git log --format='%H|%an|%ae|%s' -n $Count |
        ForEach-Object {
            $parts = $_ -split '\|', 4
            [PSCustomObject]@{
                Hash    = $parts[0]
                Author  = $parts[1]
                Email   = $parts[2]
                Subject = $parts[3]
            }
        }
}

# Now the rest of the script works with objects
Get-GitLog -Count 5 | Where-Object { $_.Author -eq 'Alice' }
```

#### Consistent Object Contracts

Consuming commands and functions must expect and handle objects consistently.
When a function accepts pipeline input, document and enforce the expected
object shape via parameter types and validation. When producing output,
always emit the same object shape — do not return different property sets
based on code paths.

---

## 11. Design Patterns in PowerShell

PowerShell is not class-based in the traditional sense, but its object
pipeline, modules, and .NET interop support common patterns.

### 11.1 Creational Patterns

#### Factory Function

```powershell
function New-Notification {
    <#
    .SYNOPSIS
        Create a notification appropriate to the event severity.

    .PARAMETER DomainEvent
        The domain event to build a notification from.

    .OUTPUTS
        [PSCustomObject] An urgent or standard notification.
    #>
    [CmdletBinding()]
    [OutputType([PSCustomObject])]
    param(
        # NOTE: do not name this parameter $Event — $Event is an automatic
        # variable used by the eventing subsystem and PSScriptAnalyzer flags
        # the assignment (PSAvoidAssignmentToAutomaticVariable).
        [Parameter(Mandatory)]
        [PSCustomObject]$DomainEvent
    )

    $type = if ($DomainEvent.Severity -eq 'Critical') { 'Urgent' } else { 'Standard' }

    [PSCustomObject]@{
        PSTypeName = "Notification.$type"
        Message    = $DomainEvent.Message
        Severity   = $DomainEvent.Severity
        CreatedAt  = Get-Date
    }
}
```

#### Builder (via Splatting)

Use hashtable construction for complex object building:

```powershell
function New-ServerConfig {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$Hostname,

        [int]$Port = 443,
        [switch]$EnableTls,
        [int]$Workers = 4
    )

    [PSCustomObject]@{
        Hostname = $Hostname
        Port     = $Port
        Tls      = [bool]$EnableTls
        Workers  = $Workers
    }
}
```

### 11.2 Structural Patterns

#### Adapter (Wrapper Function)

Wrap a complex or legacy interface behind a clean cmdlet:

```powershell
function Get-DiskUsage {
    <#
    .SYNOPSIS
        Return disk usage as a percentage for each drive.

    .OUTPUTS
        [PSCustomObject] Drive letter, total, free, and usage percentage.
    #>
    [CmdletBinding()]
    [OutputType([PSCustomObject])]
    param()

    Get-CimInstance -ClassName Win32_LogicalDisk -Filter "DriveType=3" |
        ForEach-Object {
            [PSCustomObject]@{
                Drive      = $_.DeviceID
                TotalGB    = [math]::Round($_.Size / 1GB, 2)
                FreeGB     = [math]::Round($_.FreeSpace / 1GB, 2)
                UsedPct    = [math]::Round((1 - $_.FreeSpace / $_.Size) * 100, 1)
            }
        }
}
```

#### Decorator (Wrapper Function with Pass-Through)

```powershell
function Invoke-WithRetry {
    <#
    .SYNOPSIS
        Invoke a script block with retry logic.

    .PARAMETER ScriptBlock
        The operation to execute.

    .PARAMETER MaxAttempts
        Maximum number of attempts.

    .PARAMETER DelaySeconds
        Base delay between retries (doubled each attempt).
    #>
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [scriptblock]$ScriptBlock,

        [int]$MaxAttempts = 3,
        [int]$DelaySeconds = 2
    )

    for ($attempt = 1; $attempt -le $MaxAttempts; $attempt++) {
        try {
            return & $ScriptBlock
        } catch {
            if ($attempt -eq $MaxAttempts) {
                throw
            }
            $delay = $DelaySeconds * [math]::Pow(2, $attempt - 1)
            Write-Warning "Attempt $attempt failed. Retrying in ${delay}s..."
            Start-Sleep -Seconds $delay
        }
    }
}

# Usage
$result = Invoke-WithRetry -MaxAttempts 5 -ScriptBlock {
    Invoke-RestMethod -Uri $apiUrl
}
```

### 11.3 Behavioural Patterns

#### Strategy (Script Blocks)

```powershell
function Export-Data {
    <#
    .SYNOPSIS
        Export data using a pluggable format strategy.

    .PARAMETER Data
        The data to export.

    .PARAMETER Formatter
        A script block that formats the data.
    #>
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [PSCustomObject[]]$Data,

        [Parameter(Mandatory)]
        [scriptblock]$Formatter
    )

    & $Formatter $Data
}

# Usage
$jsonFormatter = { param($d) $d | ConvertTo-Json -Depth 5 }
$csvFormatter = { param($d) $d | ConvertTo-Csv -NoTypeInformation }

Export-Data -Data $report -Formatter $jsonFormatter
```

---

## 12. Testing

### 12.1 Principles

- Use **Pester** (v5+) for all PowerShell testing.
- **Test behaviour, not implementation.** Tests must survive refactors.
- Follow the **testing pyramid**: unit > integration > e2e.
- Tests are first-class code — same quality standards apply.

### 12.2 Naming

Test files mirror source files with a `.Tests.ps1` suffix:

```
Module/
    Public/
        Get-User.ps1
    Tests/
        Get-User.Tests.ps1
```

### 12.3 Structure (Arrange-Act-Assert)

```powershell
Describe 'Get-UserReport' {
    Context 'when the domain has active users' {
        It 'returns a report with user names' {
            # Arrange
            Mock Get-ADUser { @(
                [PSCustomObject]@{ Name = 'Alice'; Enabled = $true }
                [PSCustomObject]@{ Name = 'Bob'; Enabled = $true }
            ) }

            # Act
            $result = Get-UserReport -Domain 'contoso.com'

            # Assert
            $result | Should -HaveCount 2
            $result[0].Name | Should -Be 'Alice'
        }
    }

    Context 'when IncludeDisabled is false' {
        It 'excludes disabled accounts' {
            Mock Get-ADUser { @(
                [PSCustomObject]@{ Name = 'Alice'; Enabled = $true }
                [PSCustomObject]@{ Name = 'Charlie'; Enabled = $false }
            ) }

            $result = Get-UserReport -Domain 'contoso.com'

            $result | Should -HaveCount 1
            $result[0].Name | Should -Be 'Alice'
        }
    }
}
```

### 12.4 Mocking

- Use Pester's `Mock` for external dependencies (Active Directory, REST APIs,
  file system).
- Mock at the **boundary** — cmdlets that talk to external systems.
- Use `Should -Invoke` to verify interactions.

### 12.5 Code Coverage

```powershell
# Pester v5 uses a configuration object
Invoke-Pester -Configuration @{
    Run          = @{ Path = '.\Tests' }
    CodeCoverage = @{ Enabled = $true; Path = '.\Module\Public\*.ps1' }
}
```

---

## 13. Database Access & ACID

When interacting with SQL databases from PowerShell, every query and
transaction must respect the **ACID** properties — **Atomicity**,
**Consistency**, **Isolation**, and **Durability**.

### 13.1 ACID at a Glance

| Property | Guarantee | Violation Example |
|---|---|---|
| **Atomicity** | A transaction either completes entirely or has no effect. | A transfer debits one account but the script crashes before crediting the other. |
| **Consistency** | A transaction moves the database from one valid state to another. | An insert succeeds despite violating a foreign key constraint. |
| **Isolation** | Concurrent transactions do not interfere with each other. | Two runspaces read the same balance, both withdraw, and the final balance is wrong. |
| **Durability** | Once committed, data survives crashes and power loss. | A commit returns successfully but the data is lost after a restart. |

### 13.2 Always Use Explicit Transactions

```powershell
$connectionString = "Server=localhost;Database=mydb;Integrated Security=True"
# Microsoft.Data.SqlClient is the supported replacement for the deprecated
# System.Data.SqlClient on .NET Core / PowerShell 7+.
$connection = [Microsoft.Data.SqlClient.SqlConnection]::new($connectionString)
$connection.Open()
$transaction = $connection.BeginTransaction()

try {
    $cmd = $connection.CreateCommand()
    $cmd.Transaction = $transaction

    # Prefer Parameters.Add(SqlParameter) over AddWithValue — AddWithValue
    # infers SqlDbType from the runtime value which can produce bad plans.
    $cmd.CommandText = "UPDATE accounts SET balance = balance - @Amount WHERE id = @FromId"
    $cmd.Parameters.Add('@Amount', [System.Data.SqlDbType]::Decimal).Value = $amount
    $cmd.Parameters.Add('@FromId', [System.Data.SqlDbType]::Int).Value = $fromId
    $cmd.ExecuteNonQuery() | Out-Null

    $cmd.Parameters.Clear()
    $cmd.CommandText = "UPDATE accounts SET balance = balance + @Amount WHERE id = @ToId"
    $cmd.Parameters.Add('@Amount', [System.Data.SqlDbType]::Decimal).Value = $amount
    $cmd.Parameters.Add('@ToId', [System.Data.SqlDbType]::Int).Value = $toId
    $cmd.ExecuteNonQuery() | Out-Null

    $transaction.Commit()
} catch {
    $transaction.Rollback()
    throw
} finally {
    $connection.Dispose()
}
```

With the `dbatools` module — pass parameters via `-SqlParameter`, never
interpolate them into the query string:

```powershell
$query = @'
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - @Amount WHERE id = @FromId;
UPDATE accounts SET balance = balance + @Amount WHERE id = @ToId;
COMMIT;
'@

Invoke-DbaQuery -SqlInstance $server -Database $db -Query $query `
    -SqlParameter @{ Amount = $amount; FromId = $fromId; ToId = $toId }
```

> WARNING: Using a double-quoted (`@"..."@`) here-string with `$amount` /
> `$fromId` interpolated directly is an SQL injection vulnerability —
> identical to string concatenation. Always use a single-quoted
> (`@'...'@`) here-string with `@Name` placeholders and `-SqlParameter`.

### 13.3 Use Parameterised Queries

```powershell
# Yes — parameterised with explicit SqlDbType (preferred; avoids type inference issues)
$cmd.CommandText = "SELECT * FROM users WHERE email = @Email"
$param = $cmd.Parameters.Add('@Email', [System.Data.SqlDbType]::NVarChar, 256)
$param.Value = $email

# No — string interpolation (SQL injection)
$cmd.CommandText = "SELECT * FROM users WHERE email = '$email'"

# Avoid — AddWithValue (inferred SqlDbType can cause bad query plans; see §13.2)
# $cmd.Parameters.AddWithValue('@Email', $email) | Out-Null
```

### 13.4 Connection Lifecycle

- **Dispose connections** in `finally` blocks — PowerShell has no
  automatic resource management for .NET objects in 5.1 (and the `using`
  *keyword* in PowerShell is not the C# RAII statement).
- Use connection pooling (enabled by default in `SqlClient`).
- Use `dbatools` or `SqlServer` module for high-level database operations
  rather than raw ADO.NET when possible.
- On PowerShell 7+ / .NET Core, prefer `Microsoft.Data.SqlClient` over
  the legacy `System.Data.SqlClient` (which is deprecated and frozen).

### 13.5 SQL Injection Protection

SQL injection is one of the most common and dangerous vulnerabilities in
database-interacting code. Every query that incorporates external input must
use parameterised queries — no exceptions.

#### Always Use Parameterised Queries

Use `@ParamName` placeholders and `SqlParameter` objects for all
user-supplied values. Never build SQL strings with concatenation or
interpolation:

```powershell
# BAD — SQL injection via string interpolation
$cmd.CommandText = "SELECT * FROM users WHERE name = '$userName'"

# BAD — SQL injection via string concatenation
$cmd.CommandText = "SELECT * FROM users WHERE name = '" + $userName + "'"

# GOOD — parameterised query with explicit SqlDbType
$cmd.CommandText = "SELECT * FROM users WHERE name = @UserName AND role = @Role"
$p1 = [Microsoft.Data.SqlClient.SqlParameter]::new('@UserName',
    [System.Data.SqlDbType]::NVarChar, 255)
$p1.Value = $userName
$cmd.Parameters.Add($p1) | Out-Null
$p2 = [Microsoft.Data.SqlClient.SqlParameter]::new('@Role',
    [System.Data.SqlDbType]::NVarChar, 50)
$p2.Value = $role
$cmd.Parameters.Add($p2) | Out-Null
$cmd.ExecuteReader()
```

#### Validate Input Before the Database Layer

Even with parameterised queries, validate and sanitise input before it
reaches the database. Use PowerShell's parameter validation attributes
(see section 5.4) to reject obviously invalid values at the function
boundary. This provides defence in depth — parameterisation prevents
injection, validation prevents nonsensical data.

#### Use Stored Procedures Where Appropriate

Stored procedures provide an additional layer of protection by separating
SQL logic from application code. They also improve performance through
query plan caching:

```powershell
$cmd.CommandText = 'usp_GetUserByEmail'
$cmd.CommandType = [System.Data.CommandType]::StoredProcedure
$cmd.Parameters.Add('@Email', [System.Data.SqlDbType]::NVarChar, 320).Value = $email
$cmd.ExecuteReader()
```

---

## 14. Concurrency & Jobs

### 14.1 Approaches

| Mechanism | Use When |
|---|---|
| `ForEach-Object -Parallel` (PS 7+) | I/O-bound work across many items |
| `Start-Job` / `Receive-Job` | Long-running background tasks |
| `Start-ThreadJob` (ThreadJob module) | Lightweight concurrency without process overhead |
| Runspaces (`[RunspacePool]`) | Maximum control and performance |
| `Invoke-Command -AsJob` | Parallel remote execution |

### 14.2 `ForEach-Object -Parallel` (PowerShell 7+)

```powershell
$servers | ForEach-Object -ThrottleLimit 10 -Parallel {
    $health = Test-Connection -ComputerName $_ -Count 1 -Quiet
    [PSCustomObject]@{
        Server  = $_
        Healthy = $health
    }
}
```

- Use `-ThrottleLimit` to control concurrency.
- Variables from the parent scope are **not automatically available** — use
  `$using:variable` to pass them in.
- Each parallel iteration runs in its own runspace — **no shared mutable
  state**.

### 14.3 Thread Safety

- **Avoid shared mutable state** between parallel executions.
- Use thread-safe collections (`[System.Collections.Concurrent.ConcurrentBag[T]]`)
  if aggregation is needed.
- Use `$using:` to pass read-only data into parallel blocks.

---

## 15. Performance & Idiomatic PowerShell

### 15.1 Pipeline vs. Loops

Use the pipeline for readability. Use `foreach` loops for performance-critical
paths:

```powershell
# Readable — pipeline
$activeUsers = Get-User | Where-Object { $_.IsActive }

# Faster for large collections — foreach loop
$activeUsers = foreach ($user in $allUsers) {
    if ($user.IsActive) { $user }
}
```

### 15.2 Avoid `+=` for Array Building

Array concatenation (`+=`) copies the entire array on every append:

```powershell
# Bad — O(n^2)
$results = @()
foreach ($item in $data) {
    $results += Process-Item -Item $item
}

# Good — collect from foreach output
$results = foreach ($item in $data) {
    Process-Item -Item $item
}

# Good — typed list
$results = [System.Collections.Generic.List[PSCustomObject]]::new()
foreach ($item in $data) {
    $results.Add((Process-Item -Item $item))
}
```

### 15.3 Use Full Cmdlet and Parameter Names

Aliases and positional parameters are for **interactive use only**:

```powershell
# Yes — scripts
Get-ChildItem -Path $dir -Filter '*.log' -Recurse |
    Where-Object { $_.Length -gt 1MB } |
    Remove-Item -Force

# No — aliases in scripts
gci $dir -fi '*.log' -r | ? { $_.Length -gt 1MB } | rm -fo
```

### 15.4 `.NET` Methods for Hot Paths

When performance matters, use .NET methods directly:

```powershell
# Fast — .NET string method
$lower = $value.ToLower()

# Slow — PowerShell operator (creates a new pipeline)
$lower = $value | ForEach-Object { $_.ToLower() }
```

### 15.5 Guard Clauses

Return early rather than nesting:

```powershell
function Process-Order {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [PSCustomObject]$Order
    )

    if ($Order.IsCancelled) {
        throw "Order $($Order.Id) is cancelled"
    }

    if ($Order.Items.Count -eq 0) {
        return [PSCustomObject]@{ Total = 0; Items = @() }
    }

    Build-Receipt -Order $Order
}
```

### 15.6 Copy Semantics

PowerShell variables hold **references** to .NET objects. Assignment does not
copy:

```powershell
$original = @{
    Name = 'Alice'
    Tags = [System.Collections.Generic.List[string]]@('admin')
}

# Reference — same object
$ref = $original
$ref.Name = 'Bob'
$original.Name  # 'Bob' — mutated!

# Shallow clone
$shallow = $original.Clone()
$shallow.Tags.Add('editor')
$original.Tags  # includes 'editor' — nested list is shared!

# Deep copy via serialization
$deep = [System.Management.Automation.PSSerializer]::Deserialize(
    [System.Management.Automation.PSSerializer]::Serialize($original)
)
```

| Operation | Outer | Nested | Use when |
|---|---|---|---|
| `$b = $a` | Shared | Shared | You want an alias |
| `$a.Clone()` | New | Shared | Flat hashtables, arrays of value types |
| Serialize/Deserialize round-trip | New | New | Deep copy of complex objects |

---

## 16. Security

### 16.1 Execution Policy

- Set to `RemoteSigned` or `AllSigned` on production systems.
- Never run `Set-ExecutionPolicy Unrestricted` in production.
- Scripts from the internet should be signed or explicitly unblocked.

### 16.2 Credential Handling

- **Never hardcode credentials** in scripts.
- Use `Get-Credential` for interactive input.
- Use `[PSCredential]` parameter types for passing credentials.
- Store secrets in Azure Key Vault, `SecretManagement` module, or encrypted
  files via `Export-Clixml`:

```powershell
# Store (interactive — prompts for credential)
Get-Credential | Export-Clixml -Path .\cred.xml

# Retrieve (only decryptable by the same user on the same machine)
$cred = Import-Clixml -Path .\cred.xml
```

### 16.3 Input Validation

Validate all external input via parameter validation attributes (see 5.4)
and explicit checks.

### 16.4 Avoid `Invoke-Expression`

`Invoke-Expression` is PowerShell's `eval` — it executes arbitrary code:

```powershell
# NEVER do this
Invoke-Expression $userInput

# Use the call operator with arrays instead.
# Note: do NOT name the splat variable $args — that is an automatic
# variable in PowerShell. Use any other name.
$cmd = 'Get-Process'
$cmdArgs = @('-Name', 'notepad')
& $cmd @cmdArgs
```

---

## 17. Defensive Programming & Input Validation

Defensive programming assumes that all external input is potentially
malicious, malformed, or unexpected. Validate early, validate thoroughly,
and never trust data you did not generate yourself.

### 17.1 Validate at Function Boundaries

Every function that accepts external input — user arguments, file content,
API responses, registry values, environment variables — must validate that
input before processing. PowerShell's parameter validation attributes
make this declarative and self-documenting:

```powershell
function New-UserAccount {
    [CmdletBinding(SupportsShouldProcess)]
    param(
        [Parameter(Mandatory)]
        [ValidateNotNullOrEmpty()]
        [ValidateLength(1, 255)]
        [string]$UserName,

        [Parameter(Mandatory)]
        [ValidatePattern('^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')]
        [string]$Email,

        [ValidateRange(1, 65535)]
        [int]$Port = 443,

        [ValidateSet('Reader', 'Contributor', 'Admin')]
        [string]$Role = 'Reader',

        [ValidateScript({ Test-Path $_ -PathType Container })]
        [string]$HomeDirectory
    )

    if ($PSCmdlet.ShouldProcess($UserName, 'Create user account')) {
        # Safe to proceed — all inputs are validated
    }
}
```

### 17.2 Parameter Validation Attributes

Use the full range of validation attributes to enforce constraints
declaratively:

| Attribute | Purpose | Example |
|---|---|---|
| `[ValidateNotNullOrEmpty()]` | Reject `$null` and empty strings | Required string parameters |
| `[ValidateLength(min, max)]` | Enforce string length bounds | Usernames, descriptions |
| `[ValidateRange(min, max)]` | Enforce numeric range | Port numbers, retry counts |
| `[ValidatePattern('regex')]` | Match against a regex | Email addresses, identifiers |
| `[ValidateSet('a','b','c')]` | Restrict to known values | Environment names, roles |
| `[ValidateScript({ ... })]` | Custom validation logic | Path existence, complex rules |
| `[ValidateCount(min, max)]` | Enforce array element count | Multi-value parameters |

### 17.3 Type-Constrain All Parameters

Always declare parameter types explicitly. Untyped parameters accept
anything, bypassing PowerShell's automatic type coercion and validation:

```powershell
# Good — type-constrained
param(
    [string]$Name,
    [int]$Port,
    [datetime]$StartDate,
    [System.IO.FileInfo]$LogFile
)

# Bad — untyped, accepts anything
param(
    $Name,
    $Port,
    $StartDate,
    $LogFile
)
```

### 17.4 Validate String Lengths

Always validate string lengths explicitly for data that will be stored in a
database, transmitted over a network, or used in file paths. Unbounded
strings can cause truncation errors, buffer issues, or denial-of-service
conditions:

```powershell
[ValidateLength(1, 255)]
[string]$DisplayName
```

### 17.5 Check Collection Bounds Before Indexing

Never index into an array or collection without first verifying that the
index is within bounds:

```powershell
if ($results.Count -gt 0) {
    $first = $results[0]
} else {
    Write-Warning "No results returned"
}
```

### 17.5.1 `$null` Comparisons: Put `$null` on the Left

PowerShell evaluates `-eq` and `-ne` element-wise when the **left** operand
is a collection. Writing `$x -eq $null` therefore behaves differently
depending on whether `$x` is a scalar or an array — for an array it returns
the filtered elements that are `$null`, not `$true`/`$false`.

Always put `$null` on the left so the comparison is unambiguous.
PSScriptAnalyzer enforces this via `PSPossibleIncorrectComparisonWithNull`:

```powershell
# Bad — wrong result when $value happens to be a collection
if ($value -eq $null) { ... }
if ($value -ne $null) { ... }

# Good — scalar comparison regardless of $value's shape
if ($null -eq $value) { ... }
if ($null -ne $value) { ... }
```

### 17.6 Validate File Paths Before Operations

Use `Test-Path` to verify that files and directories exist before
attempting to read, write, or delete them:

```powershell
if (-not (Test-Path -Path $ConfigFile -PathType Leaf)) {
    throw "Configuration file not found: $ConfigFile"
}
```

For parameters, use `[ValidateScript()]` to enforce this declaratively
(see 17.1).

### 17.7 Never Trust External Input

Treat all external data as untrusted, regardless of its source:

- **Registry values** — may be modified by other software or malware.
- **Environment variables** — may be set by parent processes or users.
- **File content** — may be corrupt, truncated, or tampered with.
- **API responses** — may return unexpected schemas or error payloads.
- **User input** — may be malicious, malformed, or simply wrong.

Sanitise input that will be used in file paths, registry paths, WMI
queries, or any context where special characters could alter behaviour.

### 17.8 Use `-WhatIf` and `-Confirm` as Defensive Measures

For any function that performs destructive or irreversible operations,
implement `SupportsShouldProcess`. This provides a built-in safety net
that allows callers to preview changes before committing them:

```powershell
[CmdletBinding(SupportsShouldProcess, ConfirmImpact = 'High')]
param( ... )

if ($PSCmdlet.ShouldProcess($target, 'Delete all records')) {
    Remove-AllRecords -Target $target
}
```

---

## 18. Project Structure

### 18.1 Module Layout

```
MyModule/
    MyModule.psd1               # module manifest
    MyModule.psm1               # root module (dot-sources public/private)
    Public/
        Get-User.ps1
        New-User.ps1
        Remove-User.ps1
    Private/
        Resolve-UserDn.ps1
        Test-LdapConnection.ps1
    Classes/
        UserAccount.ps1
    Tests/
        Get-User.Tests.ps1
        New-User.Tests.ps1
    en-US/
        about_MyModule.help.txt
```

### 18.2 Root Module Pattern

The `.psm1` dot-sources all public and private functions:

```powershell
# MyModule.psm1
# Use -ErrorAction SilentlyContinue so an empty Public/ or Private/
# directory does not abort module import under StrictMode.
$publicFunctions = @(Get-ChildItem -Path "$PSScriptRoot\Public\*.ps1" `
    -ErrorAction SilentlyContinue)
$privateFunctions = @(Get-ChildItem -Path "$PSScriptRoot\Private\*.ps1" `
    -ErrorAction SilentlyContinue)

foreach ($file in ($publicFunctions + $privateFunctions)) {
    . $file.FullName
}

# Force array semantics — $publicFunctions.BaseName on a single FileInfo
# returns a scalar string, on $null throws under StrictMode.
Export-ModuleMember -Function @($publicFunctions | ForEach-Object BaseName)
```

`$PSScriptRoot` is the directory of the *current script file*, set
automatically by PowerShell — prefer it to `(Get-Location).Path` or
`$PWD`, which reflect the caller's current directory and break when the
module is imported from elsewhere.

### 18.3 Script Layout

```
Scripts/
    Deploy-Application.ps1
    Invoke-BackupRotation.ps1
    Tests/
        Deploy-Application.Tests.ps1
```

### 18.4 Dependency Management

- Use `#Requires -Modules` for script dependencies.
- Use `RequiredModules` in module manifests.
- Use `PSResourceGet` (PowerShellGet v3) or `Install-Module` for gallery
  modules.
- Pin module versions in manifests:
  `@{ ModuleName = 'Az.Accounts'; ModuleVersion = '2.0' }`.

---

## 19. Tooling

### 19.1 Recommended Tool Chain

| Purpose | Tool | Notes |
|---|---|---|
| Linter | **PSScriptAnalyzer** | Community and Microsoft rules |
| Test framework | **Pester** (v5+) | BDD-style testing |
| Formatter | **PSScriptAnalyzer** (`Invoke-Formatter`) | Consistent formatting |
| Coverage | **Pester** `-CodeCoverage` | Line coverage |
| Documentation | **platyPS** | Generate MAML help from Markdown |
| Secret management | **SecretManagement** module | Vault-agnostic secret storage |
| Build automation | **InvokeBuild** or **psake** | Build/test/deploy pipelines |
| Module publishing | **PSResourceGet** | Publish to PSGallery or internal feed |

### 19.2 PSScriptAnalyzer Configuration

```powershell
# PSScriptAnalyzerSettings.psd1
# Pass via: Invoke-ScriptAnalyzer -Settings .\PSScriptAnalyzerSettings.psd1
@{
    Severity     = @('Error', 'Warning', 'Information')
    IncludeRules = @('*')
    ExcludeRules = @()
    Rules        = @{
        PSAvoidUsingCmdletAliases          = @{ Enable = $true }
        PSAvoidUsingPositionalParameters   = @{ Enable = $true }
        PSUseApprovedVerbs                 = @{ Enable = $true }
        PSUseDeclaredVarsMoreThanAssignments = @{ Enable = $true }
    }
}
```

### 19.3 CI Checks

```powershell
# Lint
Invoke-ScriptAnalyzer -Path . -Recurse -ReportSummary

# Format check
$files = Get-ChildItem -Recurse -Include '*.ps1', '*.psm1'
foreach ($file in $files) {
    $original = Get-Content -Path $file.FullName -Raw
    $formatted = Invoke-Formatter -ScriptDefinition $original
    if ($original -ne $formatted) {
        Write-Error "Formatting issue: $($file.FullName)"
    }
}

# Tests with coverage
Invoke-Pester -Configuration @{
    Run          = @{ Path = '.\Tests' }
    CodeCoverage = @{ Enabled = $true; Path = '.\Public\*.ps1' }
    Output       = @{ Verbosity = 'Detailed' }
}
```

---

## 20. SBOM Creation

### 20.1 What is an SBOM?

A Software Bill of Materials documents all PowerShell modules, .NET assemblies, and dependencies. Critical for compliance, vulnerability tracking, and supply chain security.

### 20.2 PowerShell Gallery Dependency Management

PowerShell modules are published on PowerShell Gallery. Declare dependencies in module manifest:
```powershell
# MyModule.psd1
@{
    RequiredModules = @(
        @{ ModuleName = 'Pester'; ModuleVersion = '5.4.0' },
        @{ ModuleName = 'PSScriptAnalyzer'; ModuleVersion = '1.21.0' }
    )
}
```

### 20.3 Listing Dependencies

```powershell
# Get all loaded modules
Get-Module

# Get module dependencies
$manifest = Import-PowerShellDataFile 'MyModule.psd1'
$manifest.RequiredModules

# Check for outdated modules
Get-InstalledModule | ForEach-Object {
    Find-Module -Name $_.Name |
        Where-Object { [version]$_.Version -gt [version]$_.InstalledVersion }
}
```

### 20.4 SBOM Generation

**Using PSResourceGet (PS 7.2+, install `Microsoft.PowerShell.PSResourceGet`)**:
```powershell
Find-PSResource -Name MyModule -IncludeDependencies |
    Select-Object Name, Version, Repository, Author |
    Export-Csv -Path sbom.csv -NoTypeInformation
```

For detailed inventory, create custom script:
```powershell
function New-ModuleSBOM {
    param([string]$ModuleName)
    
    $modules = @()
    $manifest = Import-PowerShellDataFile "$ModuleName.psd1"
    
    foreach ($req in $manifest.RequiredModules) {
        $modules += @{
            Name = $req.ModuleName
            Version = $req.ModuleVersion
            Repository = 'PowerShell Gallery'
        }
    }
    
    $modules | ConvertTo-Json | Out-File "sbom-$ModuleName.json"
}
```

### 20.5 Vulnerability Scanning

**Check for outdated modules**:
```powershell
# Compare installed vs latest
$installed = Get-InstalledModule
$installed | ForEach-Object {
    $latest = Find-Module -Name $_.Name
    if ([version]$latest.Version -gt [version]$_.Version) {
        Write-Warning "$($_.Name): $($_.Version) -> $($latest.Version)"
    }
}
```

Use GitHub Dependabot for automated scanning of `RequiredModules` in manifests.

### 20.6 Integration into CI/CD

- Pin module versions in `RequiredModules` (manifests)
- Run version comparison checks in CI
- Audit logs of installed modules
- Keep manifests in VCS for reproducibility
- Automatically update dependencies via Dependabot
- Scan for known module vulnerabilities from community sources

---

## 21. References

### Official Documentation

| Resource | URL |
|---|---|
| PowerShell Documentation | https://learn.microsoft.com/en-us/powershell/ |
| Cmdlet Development Guidelines | https://learn.microsoft.com/en-us/powershell/scripting/developer/cmdlet/cmdlet-development-guidelines |
| Approved Verbs | https://learn.microsoft.com/en-us/powershell/scripting/developer/cmdlet/approved-verbs-for-windows-powershell-commands |
| About Comment-Based Help | https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_comment_based_help |
| PSScriptAnalyzer Rules | https://learn.microsoft.com/en-us/powershell/utility-modules/psscriptanalyzer/rules/readme |
| PowerShell Practice and Style Guide | https://poshcode.gitbook.io/powershell-practice-and-style/ |
| Pester Documentation | https://pester.dev/docs/quick-start |

### Books

| Book | Authors | Key Takeaways for This Guide |
|---|---|---|
| *PowerShell in Action* | Bruce Payette & Richard Siddaway (2017) | Definitive deep dive: pipeline, type system, providers, remoting. |
| *Learn PowerShell in a Month of Lunches* | Travis Plunk, James Petty, et al. (2022) | Practical foundation: cmdlets, pipeline, formatting, remoting. |
| *The PowerShell Scripting & Toolmaking Book* | Don Jones & Jeff Hicks | Advanced functions, module development, toolmaking best practices. |
| *PowerShell for Sysadmins* | Adam Bertram (2020) | Real-world automation patterns for infrastructure management. |
| *Design Patterns: Elements of Reusable Object-Oriented Software* | Gamma, Helm, Johnson, Vlissides (1994) | Composition, strategy, decorator — applicable via script blocks and modules. |
| *Clean Code: A Handbook of Agile Software Craftsmanship* | Robert C. Martin (2008) | Small functions, meaningful names, SRP — applies to scripts and modules alike. |
