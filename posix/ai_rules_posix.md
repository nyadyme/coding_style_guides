# POSIX Shell — AI Coding Rules

Apply these rules when generating or reviewing POSIX shell scripts (.sh).

## When to Use POSIX Shell

- Portable scripts that must run on any Unix-like system
- Embedded systems with minimal shells (BusyBox, Alpine, dash)
- System initialization scripts and entry points
- Scripts shipped with libraries or frameworks
- Hard limit: ~200 lines. Beyond that, use Python or Go.

## Boilerplate (every script)

```bash
#!/bin/sh
# Script description: one-line summary
# Usage: ./script.sh [options] arguments
# Requirements: POSIX shell, standard Unix tools

set -eu
```

- `#!/bin/sh` — Portable shebang (guaranteed POSIX on any Unix system)
- `set -eu` — Exit on error, exit on undefined variable
- Comments with `#` only. (`:` is a no-op builtin, not a comment — avoid `: "string"` as documentation.)

## Formatting

- 2 spaces indentation. Never tabs.
- 80 characters hard line limit. Break with `\` (backslash-newline).
- Two blank lines between top-level function definitions.
- No blank lines inside `if`, `for`, or `while` blocks.
- Opening brace on same line: `func() {`, `if [ ... ]; then`.

## Naming

- Script files: `kebab-case.sh` or `snake_case.sh`
- Functions: `snake_case` (e.g., `process_file`, `validate_input`)
- Variables: `UPPER_SNAKE_CASE` for constants; `snake_case` for locals
- Globals: `GLOBAL_VAR=value`; locals: `local var=value`

## Variables & Scoping

- Always double-quote expansions: `"$var"` not `$var`
- Always use braces in ambiguous contexts: `"${var}_suffix"`
- `local` is NOT POSIX standard but widely supported (dash, bash, ksh, ash). Use it for clarity in modern shells; document if portability requires strict POSIX.
- No arrays or associative arrays (POSIX doesn't support them)
- Use parameter expansion only: `${var:-default}`, `${var#pattern}`
- `readonly` and `export` ARE POSIX standard.

## Control Flow

- Use `[ ]` (POSIX test), **never** `[[ ]]` (Bash extension)
- File tests: `[ -f "$file" ]`, `[ -d "$dir" ]`, `[ -r "$file" ]`
- String tests: `[ -n "$var" ]`, `[ -z "$var" ]`, `[ "$a" = "$b" ]`
- Numeric tests: `[ "$a" -eq "$b" ]`, `[ "$a" -lt "$b" ]` (use `-eq`, `-ne`, `-lt`, `-le`, `-gt`, `-ge`)
- Logical: `[ "$a" = "x" ] && [ "$b" = "y" ]` (use && and ||, never `[[ ]]` conditions)
- Case statements over long if/elif chains

## Functions

```bash
function_name() {
    local arg1="$1"
    local arg2="$2"
    
    [ -n "$arg1" ] || return 1  # Validate
    
    # Function body
    
    return 0
}
```

- Small, under 20 lines, one thing
- `name() { }` form only (no `function` keyword; not POSIX)
- Return status via `return 0`/`return 1`
- Return data via stdout: `echo "result"` and capture with `$(function)`
- Validate parameters immediately

## Error Handling

- `set -eu` is the foundation
- Check exit codes immediately: `if ! command; then`
- `trap cleanup EXIT` for guaranteed cleanup
- Errors to stderr: `echo "ERROR: ..." >&2`
- `die()` function: `echo "ERROR: $*" >&2; exit 1`
- Handle expected failures explicitly: `cmd || true`

## I/O & Redirection

- Here-documents for multi-line output
- Break long pipelines across lines (after `|`)
- Always `mktemp` for temp files + `trap 'rm -f "$tmp"' EXIT`
- POSIX shell passes **text** through pipes, not objects
- Use structured formats (JSON via `jq`, CSV, TSV) for machine consumption
- Parse external command output carefully; don't assume format stability

## Arithmetic

- Use `$((expression))` arithmetic expansion (POSIX.1-2001+ standard)
- Example: `result=$((a + b))`, `count=$((count + 1))`
- Comparison: `if [ $((x % 2)) -eq 0 ]; then`
- Avoid `expr` (external process, slower) — use only for shells older than POSIX.1-2001 (very rare)

## Avoid (Not POSIX Compliant)

- **NO** `[[` — use `[` instead (Bash/Ksh extension)
- **NO** arrays or associative arrays
- **NO** `${var//old/new}` substitution — Bash extension
- **NO** `${var,,}` or `${var^^}` case conversion — Bash extension
- **NO** `(( arithmetic ))` C-style — use `$(( ))` instead (the double-paren `(( ))` standalone form is Bash)
- **NO** `function` keyword — use `name() {` (POSIX form)
- **NO** process substitution `< <(cmd)` — Bash/Ksh extension
- **NO** `+=` operator for string concatenation — Bash extension
- **NO** `local -i`, `local -r`, `declare` — Bash extensions
- **NO** `set -o pipefail` — not POSIX (Bash/Ksh extension)
- **NO** `&>` redirection — Bash extension (use `>file 2>&1`)
- Note: `readonly` and `export` ARE POSIX standard
- Note: `$(())` arithmetic expansion IS POSIX (since POSIX.1-2001)
- Note: `${var:+alternate}` IS POSIX (parameter expansion)
- Note: `local` is widely supported (dash, bash, ksh, ash) but NOT in POSIX standard

## Security

- Validate all external input: check format, length, range
- Never `eval` with user input
- Never parse `ls` output
- Never hardcode secrets; use environment variables
- Always use `mktemp` (never predictable `/tmp/` paths)
- Always quote variables: `"$var"` not `$var`
- Use `--` to separate options from arguments when passing user input

## Testing

- ShellCheck: `shellcheck -S info script.sh`
- Test on multiple shells: dash, bash, ksh, ash
- Test on multiple Unix systems: Linux, BSD, macOS
- Manual test: missing args, invalid input, special characters
- Verify cleanup: no temp files left behind

## Defensive Programming

- Validate all script arguments at entry
- Check argument presence: `[ -n "$1" ] || die "arg required"`
- Validate format with `grep -q 'pattern'` for matching
- Validate file existence: `[ -f "$file" ] || die "file not found"`
- Validate ranges: `[ "$port" -ge 1 ] && [ "$port" -le 65535 ]` || die
- Validate string lengths: `[ ${#input} -le 255 ]` || die
- Sanitize input in file paths, command construction
- Never pass unsanitized input to `rm`, `mv`, or external commands
- Use `--` separator when passing user input to commands

## Portability

- Document minimum POSIX version (POSIX.1-2008 assumed)
- Document which shells tested (dash, bash in sh mode, ksh, etc.)
- Assume most minimal shell (dash); test accordingly
- Avoid shell-specific extensions unless documented
- Check tool availability: `command -v tool >/dev/null || die "tool required"`

## Tooling

- ShellCheck (lint with portable mode)
- POSIX specification and test suites
- Multiple shell versions for testing

## SBOM & Dependency Management

- Document external tool dependencies in header
- Validate tool existence: `command -v curl >/dev/null || die "curl required"`
- Use version-agnostic tool calls when possible
- Container deployments: include tools in image
- Pin versions in dependency declarations

## When to Migrate

If you need:
- Arrays or complex data structures → Use Bash, Zsh, or Python
- Extended pattern matching or globs → Use Bash or Zsh
- `[[ ]]` conditional syntax → Use Bash or Zsh
- Process substitution `< <(cmd)` → Use Bash or Zsh
- Complex logic over 200 lines → Use Python or Go

Stay POSIX if you need portability across all Unix systems and minimal shells.
