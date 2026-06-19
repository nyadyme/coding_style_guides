# Windows Batch — AI Coding Rules

Apply these rules when generating or reviewing Windows batch scripts (.bat, .cmd).

## When to Use Batch

- Thin wrappers calling existing tools on Windows
- Windows system administration (registry, services, scheduled tasks)
- Windows CI/CD integration (AppVeyor, GitHub Actions, Azure Pipelines)
- Hard limit: ~100 lines. Beyond that, use PowerShell or Python.

## Boilerplate (every script)

```batch
@echo off
REM Script description: one-line summary
REM Usage: script.bat [options] arguments

setlocal enabledelayedexpansion
```

- `@echo off` — Suppress command echoing
- `REM` — Comments (never `::`; breaks in parentheses)
- `setlocal enabledelayedexpansion` — Enable delayed variable expansion
- `endlocal` — Clean up scope before exit

## Formatting

- 4 spaces indentation. Never tabs.
- Blank lines between logical sections.
- Keep lines under 100 characters.
- No blank lines inside `if`, `for`, `setlocal` blocks.

## Naming

- Script files: `kebab-case.bat` or `snake_case.bat` describing action
- Labels (subroutines): `:lowercase_with_underscores`
- Variables: `ALL_CAPS_WITH_UNDERSCORES` for user-visible; `lowercase_for_temp`

## Variables & Scoping

- Always `setlocal` to isolate scope. Call `endlocal` before exit.
- Inside blocks (if/for), use `!variable!` not `%variable%` (delayed expansion).
- Pass data via variables; batch has no function parameters.
- Use `%~1` to strip quotes from arguments; `%~d1` drive, `%~n1` name, `%~x1` extension.

## Control Flow

- `if exist "file"` for file checks
- `if "!var!"=="value"` for string comparison (case-sensitive by default; use `if /i` for case-insensitive)
- `if %count% gtr 10` for numeric comparison (`equ`, `neq`, `lss`, `leq`, `gtr`, `geq`)
- `for %%f in (*.txt) do (...)` for file iteration
- `for /l %%i in (1,1,10) do (...)` for numeric loops
- `goto :label` for error handling and flow control

## Subroutines & Return Codes

```batch
:my_subroutine
    REM Subroutine logic
    exit /b 0  REM Success, or /b 1 for failure

REM Call and check
call :my_subroutine
if errorlevel 1 goto :error
```

- Check `errorlevel` immediately after commands that can fail
- Return `exit /b 0` (success) or `exit /b 1` (failure)
- Always check return codes from called subroutines

## CRITICAL: `if errorlevel` Semantics

- `if errorlevel n` means "errorlevel is **>= n**", NOT equal to n
- `if errorlevel 1` — True on failure (any non-zero exit code)
- `if not errorlevel 1` — True on success (errorlevel = 0)
- `if errorlevel 0` — ALWAYS TRUE (common bug)
- For explicit equality: `if %errorlevel% equ 0` or `if %errorlevel% neq 0`

## CRITICAL: Control Flow Pitfalls

- **Labels do NOT stop execution.** Without `goto`/`exit /b` between labels,
  control falls through into the next labeled block. Always end each
  subroutine with `exit /b N`.
- **`goto` inside a CALLed subroutine escapes the call.** Use `exit /b N`
  to return to the caller; reserve `goto :error` for the top-level script.
- **Never name a variable `path`.** It shadows the `%PATH%` environment
  variable and breaks command resolution. Use `dir_path`, `file_path`, etc.
- **Avoid `%DATE%`/`%TIME%` for filename uniqueness** — both are
  locale-dependent (separators, AM/PM markers vary). Combine multiple
  `%RANDOM%` expansions instead.

## Error Handling

- Check `errorlevel` after every risky command
- Suppress output: `command >nul`, `command 2>&1`
- Suppress both: `command >nul 2>&1`
- Create cleanup logic before `goto :error`; use labels for error paths

## Input Validation

- Validate argument presence: `if "%~1"=="" (echo ERROR: arg required & exit /b 1)`
- Validate string format with `findstr` for simple patterns
- Validate numeric: `set /a "result=%input% + 0" 2>nul` then check if empty
- Validate file existence: `if not exist "!path!" (echo ERROR: not found & exit /b 1)`
- Validate path permissions: `if not exist "!path!\." (echo ERROR: not directory & exit /b 1)`

## File Operations

- Always quote file paths: `del "%file%"` not `del %file%`
- Use delayed expansion in blocks: `if exist "!file!" (...)`
- Create temp files safely: `set "temp=%TEMP%\batch_%RANDOM%.txt"` then clean up
- Delete with error check: `del "!file!" && exit /b 0 || exit /b 1`

## Security

- Never pass unsanitized user input to commands or subroutines
- Validate against whitelist (case statement) for command routing
- Never hardcode secrets; use environment variables with fallback checking
- Always quote file paths to prevent injection
- Use `>nul 2>&1` to hide sensitive tool output

## Testing

- Test with missing arguments
- Test with special characters in filenames (`&`, `|`, `(`, `)`, etc.)
- Test error paths (file not found, permission denied, etc.)
- Test cleanup (verify no temp files remain)
- Test on target Windows version (7, 10, 11, Server)

## Defensive Programming

- Validate arguments at top of each subroutine
- Check file existence before operations
- Check command return codes immediately
- Use late binding (%%var in loops) carefully; understand scope
- Avoid complex nesting; prefer early `goto` for error exit
- Comment non-obvious delayed expansion usage (`!var!`)
- Never assume success; always validate results

## Tooling

- Use `where /q cmd` to check if tool exists
- Use `findstr` for simple pattern matching
- Use `for /f` to parse command output
- Minimize external dependencies; prefer built-in tools
- Document all external tool requirements at script top

## SBOM & Dependency Management

- List external tools (curl, git, etc.) in header comments
- Check tool availability: `where /q curl || echo ERROR: curl required & exit /b 1`
- Document minimum Windows version requirement
- Include dependencies in CI/CD environment setup
- Use container images with dependencies pre-installed

## When to Migrate Away from Batch

If your script:
- Exceeds ~100 lines
- Uses complex data structures or algorithms
- Requires regex or sophisticated string handling
- Needs robust error recovery
- Must run on non-Windows systems

→ **Rewrite in PowerShell or Python instead.**
