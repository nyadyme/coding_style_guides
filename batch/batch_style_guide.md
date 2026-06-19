# Windows Batch Coding Style Guidelines

A practical guide for Windows batch scripting, grounded in Microsoft documentation, community best practices, and established software engineering literature (*Clean Code*, *The Pragmatic Programmer*).

---

## Table of Contents

1. [Philosophy](#1-philosophy)
2. [Code Layout & Formatting](#2-code-layout--formatting)
3. [Naming Conventions](#3-naming-conventions)
4. [Variables & Environment](#4-variables--environment)
5. [Labels & Control Flow](#5-labels--control-flow)
6. [Subroutines (CALL :label)](#6-subroutines-call-label)
7. [Error Handling](#7-error-handling)
8. [Input Validation & Defense](#8-input-validation--defense)
9. [File & Directory Operations](#9-file--directory-operations)
10. [Performance & Idioms](#10-performance--idioms)
11. [Security](#11-security)
12. [Testing](#12-testing)
13. [Project Structure](#13-project-structure)
14. [Tooling](#14-tooling)
15. [SBOM Creation](#15-sbom-creation)
16. [References](#16-references)

---

## 1. Philosophy

### 1.1 When to Use Batch

- **Thin wrapper scripts**: Calling existing tools or batch jobs
- **Windows system administration**: Registry manipulation, service management, scheduled tasks
- **Windows CI/CD integration**: AppVeyor, GitHub Actions runners, Azure Pipelines
- **Hard limit**: ~100 lines. Beyond that, rewrite in PowerShell or Python

### 1.2 Batch Limitations

Batch is a legacy language with significant constraints:
- **No arrays or complex data structures**
- **Limited string manipulation** (delayed expansion workarounds)
- **No proper functions** (subroutines via labels; parameter passing via variables)
- **Poor error handling** (ERRORLEVEL checks are clunky)
- **Minimal regex support**
- **Difficult testing and debugging**

**When NOT to use batch:**
- Complex logic or algorithms → PowerShell or Python
- Data transformation → Python or Go
- System architecture changes → PowerShell
- Anything over ~100 lines → PowerShell

### 1.3 Guiding Principles

| Principle | Summary |
|---|---|
| **Explicit over implicit** | Use full command names; avoid shortcuts that reduce clarity |
| **Fail fast** | Check return codes immediately; don't assume success |
| **Keep it simple** | Avoid complex nesting and delayed expansion tricks |
| **Document non-obvious code** | Batch syntax is cryptic; explain why, not what |
| **Use PowerShell for complexity** | Batch is not suitable for sophisticated logic |

---

## 2. Code Layout & Formatting

### 2.1 Boilerplate

```batch
@echo off
REM Script description: one-line summary
REM Usage: script.bat [options] arguments
REM Requires: Windows 7+ (or specify version)

setlocal enabledelayedexpansion

REM External dependencies
REM Depends on: certutil, curl (or equivalent)
```

**Key elements:**
- `@echo off` — Suppress command echoing (always use)
- `REM` — Comments (never use `::`; it has issues with parentheses)
- `setlocal enabledelayedexpansion` — Enable delayed variable expansion
- `endlocal` — Clean up local scope (see section 4.4)

### 2.2 Indentation & Spacing

- **4 spaces** indentation (or 2 if preferred consistently)
- **Blank line** between logical sections
- **No blank lines** inside blocks (if/for/setlocal)
- Indent code inside `if`, `for`, `setlocal` blocks

```batch
REM Good
if exist "file.txt" (
    echo File found
    call :process_file "file.txt"
)

REM Bad
if exist "file.txt" (
echo File found
call :process_file "file.txt"
)
```

### 2.3 Line Length

Keep lines under **100 characters**. Batch doesn't have a clean continuation character for all contexts, so break logically:

```batch
REM Good: break before arguments
set "query=SELECT * FROM users"
pushd "%APPDATA%\MyApp" ^
  && dir /b *.log ^
  || echo Directory not found
```

**Note:** `^` continues lines, but use sparingly — it's error-prone.

### 2.4 Comments

- **Inline comments**: Explain *why*, not *what*. The code shows what.

```batch
REM Bad
set "count=0"  REM Initialize count

REM Good
REM Check if user already exists before creating
if exist "%TEMP%\user_%USERNAME%.lock" goto :user_exists
```

---

## 3. Naming Conventions

### 3.1 Batch Filenames

- **kebab-case.bat** or **snake_case.bat** describing the action
- Examples: `build-release.bat`, `check_dependencies.bat`

```batch
REM Good
deploy-app.bat
rebuild-solution.bat

REM Bad
script.bat
MyScript.BAT
```

### 3.2 Labels (Subroutines)

- **`:lowercase_with_underscores`** for subroutines
- **`:UPPERCASE_FOR_ERROR_HANDLERS`** (optional, for distinction)
- Prefix private labels with underscore: `:_helper_function`

```batch
@echo off
setlocal enabledelayedexpansion

call :main
exit /b %errorlevel%

:main
    echo Starting application
    call :validate_config
    if errorlevel 1 (
        echo ERROR: validate_config failed
        exit /b 1
    )
    exit /b 0

:validate_config
    REM Validation logic
    exit /b 0
```

**Critical:** Without the leading `call :main` + top-level `exit /b`,
execution would fall through label boundaries. Labels do not stop
control flow — only `goto`, `exit /b`, or `exit` do.

### 3.3 Variables

- **ALL_CAPS_WITH_UNDERSCORES** for user-visible variables
- **camelCase_or_snake_case** for internal temporary variables
- Prefix with context: `LOCAL_temp_count`, `GLOBAL_config_path`

```batch
set "OUTPUT_DIR=%CD%\build"
set "temp_file=%TEMP%\%RANDOM%.txt"
set "app_version=1.0.0"
```

---

## 4. Variables & Environment

### 4.1 Variable Declaration & Scoping

**Always use `setlocal` to isolate scope:**

```batch
@echo off
setlocal enabledelayedexpansion

set "GLOBAL_VAR=value"
call :my_function
echo %GLOBAL_VAR%  REM Still "value"

endlocal
```

**Inside functions (labels), use local scope:**

```batch
:my_function
    setlocal
    set "LOCAL_VAR=local_value"
    echo %LOCAL_VAR%
    endlocal
    exit /b 0
```

### 4.2 Delayed Expansion

Use `!variable!` when variable is set and read inside the same block:

```batch
REM Problem: loop counter not updating
for /l %%i in (1,1,10) do (
    set "count=%%i"
    echo %count%  REM Prints blank or old value
)

REM Solution: delayed expansion
setlocal enabledelayedexpansion
for /l %%i in (1,1,10) do (
    set "count=%%i"
    echo !count!  REM Correct: prints 1, 2, 3...
)
endlocal
```

**Rule:** Inside `(...)` blocks after `for` or `if`, use `!var!` instead of `%var%`.

### 4.3 Parameter Passing

Batch has no function parameters. Pass data via variables:

```batch
:process_file
    set "file_path=%~1"           REM %~1 strips quotes
    set "output_file=%~2"
    
    if not exist "!file_path!" (
        echo ERROR: File not found: !file_path!
        exit /b 1
    )
    
    echo Processing: !file_path!
    REM ... processing logic ...
    exit /b 0

REM Call:
call :process_file "C:\path\to\file.txt" "output.txt"
```

**Tips:**
- `%~1` — Removes surrounding quotes from %1
- `%~d1` — Drive letter only
- `%~p1` — Path only
- `%~n1` — Filename only
- `%~x1` — Extension only

### 4.4 Return Values

Batch subroutines use `exit /b code`:

```batch
:validate_email
    set "email=%~1"
    
    REM Very basic validation
    if not "!email!"=="" (
        if "!email:@=!" neq "!email!" (
            exit /b 0  REM Success
        )
    )
    exit /b 1  REM Failure

REM Call and check:
call :validate_email "user@example.com"
if errorlevel 1 (
    echo Invalid email
) else (
    echo Valid email
)
```

---

## 5. Labels & Control Flow

### 5.1 If Statements

```batch
REM Existence checks
if exist "file.txt" (
    echo File exists
)

if not exist "directory\" (
    mkdir "directory"
)

REM String comparison (case-sensitive by default)
if "!var!"=="value" (
    echo Match
)

REM Case-insensitive string comparison (use /i flag)
if /i "!var!"=="VALUE" (
    echo Match (case-insensitive)
)

REM Numeric comparison
if %count% gtr 10 (
    echo Count is greater than 10
)

if %count% geq 10 echo Count is >= 10
if %count% lss 10 echo Count is < 10
if %count% leq 10 echo Count is <= 10
if %count% equ 10 echo Count equals 10
if %count% neq 10 echo Count not equal 10
```

**Comparison operators:** `equ` (=), `neq` (≠), `lss` (<), `leq` (≤), `gtr` (>), `geq` (≥)

### 5.2 For Loops

```batch
REM Iterate over files
for %%f in (*.txt) do (
    echo Processing: %%f
    call :process_file "%%f"
)

REM Numeric loop
for /l %%i in (1,1,100) do (
    echo Number: %%i
)

REM Directory recursion
for /r "path" %%d in (.) do (
    echo Directory: %%d
)

REM Parse command output
for /f "tokens=1,2" %%a in ('dir /b') do (
    echo Name: %%a  Size: %%b
)
```

### 5.3 Goto & Labels

Use `goto` for error handling and flow control:

```batch
if not exist "config.ini" goto :config_missing
if errorlevel 1 goto :error

REM ... normal flow ...
goto :end

:config_missing
    echo ERROR: config.ini not found
    exit /b 1

:error
    echo ERROR: Command failed with code %errorlevel%
    exit /b 1

:end
    echo Success
    exit /b 0
```

---

## 6. Subroutines (CALL :label)

### 6.1 Structure

```batch
@echo off
setlocal enabledelayedexpansion

call :main
exit /b %errorlevel%

:main
    REM `goto` inside a CALLed subroutine escapes the call entirely.
    REM Inside subroutines, use `exit /b N` to return to the caller.
    echo Starting main routine

    call :validate_environment
    if errorlevel 1 (
        echo An error occurred in validate_environment
        exit /b 1
    )

    call :process_data
    if errorlevel 1 (
        echo An error occurred in process_data
        exit /b 1
    )

    echo All tasks completed successfully
    exit /b 0

:validate_environment
    REM Check for required tools
    where /q powershell
    if errorlevel 1 (
        echo ERROR: PowerShell not found
        exit /b 1
    )
    exit /b 0

:process_data
    REM Data processing logic
    exit /b 0
```

### 6.2 Passing Parameters

```batch
:copy_file_safe
    set "source=%~1"
    set "destination=%~2"
    
    if not exist "!source!" (
        echo ERROR: Source file not found: !source!
        exit /b 1
    )
    
    copy "!source!" "!destination!" >nul 2>&1
    if errorlevel 1 (
        echo ERROR: Failed to copy file
        exit /b 1
    )
    
    exit /b 0

REM Call with arguments
call :copy_file_safe "C:\file.txt" "C:\backup\file.txt"
if errorlevel 1 goto :error
```

---

## 7. Error Handling

### 7.1 Checking Return Codes

**CRITICAL: Understanding `if errorlevel`**

In batch, `if errorlevel n` means "if errorlevel is **greater than or equal to** n":
- `if errorlevel 1` — True if errorlevel >= 1 (i.e., failure)
- `if not errorlevel 1` — True if errorlevel < 1 (i.e., success, errorlevel = 0)
- `if errorlevel 0` — Always true (errorlevel is always >= 0)

For explicit equality, use `if %errorlevel% equ 0` or `if %errorlevel% neq 0`.

```batch
REM Immediate check
del "%file%"
if errorlevel 1 (
    echo ERROR: Failed to delete file
    exit /b 1
)

REM After external command
curl -s "https://example.com" -o response.json
if errorlevel 1 (
    echo ERROR: Download failed
    exit /b 1
)

REM Subroutine result check
call :validate_input
if errorlevel 1 (
    echo ERROR: Validation failed
    goto :error
)
```

**Rule:** Check `errorlevel` immediately after commands that can fail.

### 7.2 Cleanup with Traps

Batch has no `trap` like shell scripts. Use labels and capture the exit
code explicitly — labels do **not** stop control flow, and a naive
`goto :cleanup` from an error path will lose the failure code if
`:cleanup` ends with `exit /b 0`.

```batch
:main
    setlocal enabledelayedexpansion

    set "temp_file=%TEMP%\batch_temp_%RANDOM%_%RANDOM%.txt"
    set "rc=0"

    REM ... processing ...
    REM Capture errorlevel into a named variable so cleanup can preserve it.
    call :do_work
    set "rc=!errorlevel!"

    REM Fall through to cleanup; do NOT use `goto :cleanup` here, because
    REM labels are not flow-control barriers — execution would reach
    REM :cleanup either way. Explicit `goto` is only needed to skip code.

:cleanup
    if exist "!temp_file!" del "!temp_file!" >nul 2>&1
    endlocal & exit /b %rc%
```

Key points:
- Save `errorlevel` into a named variable immediately after the risky
  call — subsequent commands (including the cleanup `del`) will clobber it.
- `endlocal & exit /b %rc%` — `%rc%` is expanded **before** `endlocal`
  runs (single-line parsing), so the value crosses the scope boundary.
  Without this trick, `endlocal` would discard `rc` before `exit /b`
  could read it.
- **`errorlevel` and `CALL`:** the variable `%errorlevel%` is persisted
  across `call :label` boundaries — the called subroutine's `exit /b N`
  sets `errorlevel` in the caller's scope. However, `setlocal`/`endlocal`
  inside the subroutine restores variables but does **not** discard the
  subroutine's final `errorlevel`.

### 7.3 Nested Error Handling

```batch
for %%f in (*.txt) do (
    call :process_file "%%f"
    if errorlevel 1 (
        echo ERROR: Processing failed for %%f
        goto :error
    )
)

goto :end

:error
    exit /b 1

:end
    exit /b 0
```

---

## 8. Input Validation & Defense

### 8.1 Argument Validation

```batch
:main
    if "%~1"=="" (
        echo Usage: %0 input_file output_file
        exit /b 1
    )
    
    if "%~2"=="" (
        echo ERROR: output_file required
        exit /b 1
    )
    
    call :process "%~1" "%~2"
    exit /b %errorlevel%

:process
    set "input=%~1"
    set "output=%~2"
    
    if not exist "!input!" (
        echo ERROR: Input file not found: !input!
        exit /b 1
    )
    
    exit /b 0
```

### 8.2 String Validation

```batch
:validate_port
    set "port=%~1"
    
    REM Check if numeric using findstr regex
    echo !port!| findstr /r "^[0-9][0-9]*$" >nul
    if errorlevel 1 (
        echo ERROR: Port must be numeric
        exit /b 1
    )
    
    REM Now safe to compare numerically
    if !port! lss 1 (
        echo ERROR: Port must be ^>= 1
        exit /b 1
    )
    
    if !port! gtr 65535 (
        echo ERROR: Port must be ^<= 65535
        exit /b 1
    )
    
    exit /b 0

REM Call
call :validate_port "8080"
if errorlevel 1 goto :error
```

### 8.3 Path Validation

```batch
:validate_path
    REM CAUTION: Never name a batch variable `path` — it shadows the
    REM critical %PATH% environment variable used to locate executables.
    REM Use `dir_path`, `target_path`, etc.
    set "dir_path=%~1"

    REM Remove trailing backslash
    if "!dir_path:~-1!"=="\" set "dir_path=!dir_path:~0,-1!"

    REM Check if exists and is directory
    if not exist "!dir_path!\" (
        echo ERROR: Directory not found: !dir_path!
        exit /b 1
    )

    exit /b 0
```

---

## 9. File & Directory Operations

### 9.1 Robust File Handling

```batch
REM Create directory if it doesn't exist
if not exist "build\" mkdir "build"

REM Safe file deletion
if exist "old_file.txt" (
    del "old_file.txt"
    if errorlevel 1 (
        echo WARNING: Could not delete old file
    )
)

REM Copy with error checking
copy /y "source.txt" "destination.txt" >nul
if errorlevel 1 (
    echo ERROR: Copy failed
    exit /b 1
)

REM Safe directory deletion
if exist "temp_dir" (
    rmdir /s /q "temp_dir"
    if errorlevel 1 (
        echo ERROR: Failed to remove directory
        exit /b 1
    )
)
```

### 9.2 Temporary Files

```batch
REM Create a unique temp file. Avoid %DATE% and %TIME% — their format is
REM locale-dependent (slashes, colons, and AM/PM markers vary). Combine
REM %RANDOM% with the PID-equivalent counter for low collision risk.
set "temp_file=%TEMP%\batch_%RANDOM%_%RANDOM%.tmp"
if exist "!temp_file!" (
    REM Extremely unlikely collision — regenerate
    set "temp_file=%TEMP%\batch_%RANDOM%_%RANDOM%_%RANDOM%.tmp"
)

REM Use temp file
echo data > "!temp_file!"

REM Clean up
if exist "!temp_file!" del "!temp_file!"
```

For higher-assurance unique names, shell out to PowerShell:
`for /f "usebackq delims=" %%t in (\`powershell -NoProfile -Command "[System.IO.Path]::GetTempFileName()"\`) do set "temp_file=%%t"`

---

## 10. Performance & Idioms

### 10.1 Avoid Unnecessary Commands

```batch
REM Bad: uses external command
for /f %%a in ('echo test') do set "var=%%a"

REM Good: use native batch
set "var=test"
```

### 10.2 Suppress Output

```batch
REM Suppress normal output
dir >nul

REM Suppress error output
where /q powershell
if errorlevel 1 echo Not found

REM Suppress both
call :some_command >nul 2>&1
```

### 10.3 Batch Optimizations

```batch
REM Use if exist for simple checks (faster than testing errorlevel)
if exist "file.txt" (
    REM Process file
)

REM Use findstr for simple pattern matching
findstr /c:"ERROR" logfile.txt >nul
if errorlevel 1 (
    echo No errors found
)
```

---

## 11. Security

### 11.1 Command Injection Prevention

**Never use user input in commands without validation:**

```batch
REM Bad: command injection risk
set "user_input=%~1"
call %user_input%  REM User could pass "del /s /q C:\*"

REM Good: validate before using
set "command=%~1"
if "!command!"=="start_app" (
    call :start_app
) else if "!command!"=="stop_app" (
    call :stop_app
) else (
    echo ERROR: Unknown command
    exit /b 1
)
```

### 11.2 No Hardcoded Credentials

```batch
REM Bad
set "password=SecureP@ssw0rd"

REM Good
set "password=%API_PASSWORD%"
if "!password!"=="" (
    echo ERROR: API_PASSWORD environment variable not set
    exit /b 1
)
```

### 11.3 File Path Safety

```batch
REM Use quotes around all file paths
del "%file_path%"

REM Use !var! in blocks where needed
if exist "!file_path!" (
    copy "!file_path!" "backup\"
)
```

---

## 12. Testing

### 12.1 Manual Testing Checklist

- [ ] Test with missing arguments
- [ ] Test with invalid file paths
- [ ] Test with special characters in filenames
- [ ] Test with very long paths (>260 characters)
- [ ] Test on different Windows versions (7, 10, 11, Server)
- [ ] Test in different code pages and locales
- [ ] Verify cleanup happens (no temp files left)
- [ ] Check all error paths

### 12.2 Batch Test Example

```batch
@echo off
setlocal enabledelayedexpansion

echo Testing validate_port function...

call :test_valid_port
call :test_invalid_port

goto :end

:test_valid_port
    echo Test: Valid port 8080
    call :validate_port "8080"
    if errorlevel 1 (
        echo FAIL: Should accept 8080
        exit /b 1
    )
    echo PASS
    exit /b 0

:test_invalid_port
    echo Test: Invalid port abc
    call :validate_port "abc"
    if not errorlevel 1 (
        echo FAIL: Should reject abc
        exit /b 1
    )
    echo PASS
    exit /b 0

:validate_port
    REM Reject non-numeric input via findstr regex, then range-check.
    set "input=%~1"
    echo !input!| findstr /r "^[0-9][0-9]*$" >nul || exit /b 1
    set /a "port=!input!" 2>nul
    if !port! lss 1 exit /b 1
    if !port! gtr 65535 exit /b 1
    exit /b 0

:end
    echo All tests completed
    endlocal
    exit /b 0
```

---

## 13. Project Structure

```
project\
├── build.bat             # Main build script
├── lib\
│   ├── helpers.bat       # Shared utilities
│   ├── validation.bat    # Input validation
│   └── config.bat        # Configuration
├── tests\
│   └── test_build.bat    # Test suite
└── README.md             # Usage documentation
```

---

## 14. Tooling

- **Built-in commands**: `cmd.exe`, `powershell.exe` (for validation)
- **Utilities**: `findstr`, `for`, `if`, `set`, `call`
- **Debugging**: `@echo on` for tracing, `pause` to stop for inspection
- **External tools**: certutil, curl, git, etc.

---

## 15. SBOM Creation & Dependency Management

### 15.1 Document Dependencies

```batch
REM External tools required:
REM - certutil (Windows built-in)
REM - curl (Windows 10+ or must be installed)
REM - git (must be installed)

REM Check for required tools
where /q curl
if errorlevel 1 (
    echo ERROR: curl is required but not found
    echo Please install curl from https://curl.se
    exit /b 1
)

where /q git
if errorlevel 1 (
    echo ERROR: git is required but not found
    exit /b 1
)
```

### 15.2 Version Constraints

```batch
REM Minimum Windows version check
for /f "tokens=2 delims=[]" %%a in ('ver ^| find /i "version"') do (
    set "version=%%a"
)

REM Windows 7 is NT 6.1, Windows 10 is 10.0, etc.
REM This is approximate; detailed checking requires WMI
```

---

## 16. References

### Official Documentation
- **[Microsoft Batch File Documentation](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands-glossary)** — Official reference
- **[WMIC (Windows Management Instrumentation)](https://docs.microsoft.com/en-us/windows/win32/winrm/wmi-cmdlets)** — System information
- **[Advanced Batch Scripting](https://www.robvanderwoude.com/batchfiles.html)** — Community guide (very detailed)

### Community & Best Practices
- **[SS64 Batch Reference](https://ss64.com/nt/)** — Comprehensive command reference
- **[Stack Overflow Batch Tag](https://stackoverflow.com/questions/tagged/batch-file)** — Q&A resource

### Books & Articles
- **"The Pragmatic Programmer"** (Hunt & Thomas, 2019) — General principles
- **"Clean Code"** (Martin, 2008) — Code quality (applies to batch too)

### When to Migrate

If your batch script is:
- Growing beyond 100 lines
- Handling complex data structures
- Requiring sophisticated error recovery
- Needing regular expression support
- Running on both Windows and Unix

→ **Rewrite in PowerShell or Python instead.**
