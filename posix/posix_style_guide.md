# POSIX Shell Coding Style Guidelines

A comprehensive guide rooted in the POSIX shell specification (IEEE Std 1003.1), the Shell and Utilities volume, community best practices, and established software engineering literature (*Clean Code*, *The Pragmatic Programmer*).

---

## Table of Contents

1. [Philosophy](#1-philosophy)
2. [Code Layout & Formatting](#2-code-layout--formatting)
3. [Naming Conventions](#3-naming-conventions)
4. [Variables](#4-variables)
5. [Functions](#5-functions)
6. [Control Flow](#6-control-flow)
7. [I/O & Redirection](#7-io--redirection)
8. [Error Handling](#8-error-handling)
9. [Performance & Idioms](#9-performance--idioms)
10. [Security](#10-security)
11. [Testing](#11-testing)
12. [Defensive Programming & Input Validation](#12-defensive-programming--input-validation)
13. [Project Structure](#13-project-structure)
14. [Tooling](#14-tooling)
15. [SBOM Creation](#15-sbom-creation)
16. [References](#16-references)

---

## 1. Philosophy

### 1.1 What is POSIX Shell?

POSIX shell is the standardized shell specification defined in IEEE Std 1003.1. It represents the portable subset of shell features available on all Unix-like systems.

**Common POSIX-compliant shells:**
- `dash` (Debian/Ubuntu system shell, minimal)
- `ash` (BusyBox, embedded systems)
- `ksh` (Korn Shell, POSIX certified)
- `bash` (in POSIX mode, `/bin/sh` on many systems)
- `zsh` (in sh mode)

### 1.2 When to Use POSIX Shell

- **Portable scripts**: Must run on any Unix-like system (Linux, BSD, macOS, Solaris)
- **Embedded systems**: Systems with minimal shells (BusyBox, Alpine)
- **System initialization**: Init scripts, Docker entry points
- **Compatibility requirement**: Scripts shipped with libraries or frameworks
- **Hard limit**: ~200 lines. Beyond that, rewrite in Python or Go

### 1.3 POSIX Shell Trade-offs

**Advantages:**
- Maximum portability across Unix systems
- Minimal dependencies
- Small footprint (suitable for embedded systems)
- Stable, standardized behavior

**Disadvantages:**
- No arrays or associative arrays
- No extended pattern matching
- Limited string manipulation
- No `[[ ]]` (only `[ ]` or `test`)
- No `(( ))` C-style standalone arithmetic (use `$(( ))` arithmetic expansion instead)
- No process substitution

### 1.4 Guiding Principles

| Principle | Summary |
|---|---|
| **POSIX compliance** | Use only features defined in POSIX.1-2017 or earlier |
| **Portability first** | Assume the most minimal POSIX shell (dash) |
| **Explicit over implicit** | Use full command names; avoid shell-specific extensions |
| **Fail fast** | Check exit codes immediately; don't assume success |
| **Document deviations** | If shell-specific features required, document clearly |

---

## 2. Code Layout & Formatting

### 2.1 Shebang

**For POSIX-compliant scripts:**

```bash
#!/bin/sh
# Script description: one-line summary
# Usage: ./script.sh [options] arguments
# Requirements: POSIX shell, standard Unix tools

set -eu
```

**Why `#!/bin/sh`:**
- On POSIX systems, `/bin/sh` is guaranteed to be a POSIX shell
- Portable across all Unix-like systems
- System shells are optimized for init scripts

**Alternative (if using `#!/usr/bin/env sh`):**
- More portable on macOS/BSD where `/bin/sh` might vary
- But not standard practice for system scripts

### 2.2 Indentation & Spacing

- **2 spaces** indentation. Never tabs.
- **80 characters** hard line limit. Break with `\` (backslash-newline).
- Indent continuations by 4 spaces.
- Two blank lines between top-level function definitions.
- One blank line between logical sections within a function.
- No blank line at start/end of function body.
- Opening braces on same line: `func() {`, `if [ ... ]; then`.

### 2.3 Line Breaking

Break lines after:
- Pipes (`|`)
- Logical operators (`&&`, `||`)
- Redirections (`>`, `<`, `>>`)

```bash
# Good
grep pattern file \
  | sort \
  | uniq

# Bad (if exceeds 80 chars)
grep pattern file | sort | uniq
```

### 2.4 Comments

- **`#`** is the only POSIX comment character — everything from `#` to end of line is ignored
- Note: `:` is a no-op builtin (not a comment); using `: "string"` as documentation is non-idiomatic
- Explain *why*, not *what*

```bash
# Bad
x=5  # Set x to 5

# Good
x=$(echo "$data" | cut -d: -f1)  # Extract field before colon
```

---

## 3. Naming Conventions

### 3.1 Script Names

- **kebab-case.sh** or **snake_case.sh** describing the action
- Always include `.sh` extension for clarity

```bash
# Good
build-release.sh
check-dependencies.sh
deploy-app.sh

# Bad
script
MyScript.sh
run
```

### 3.2 Functions

- **`snake_case`** (e.g., `process_file`, `validate_email`)
- Prefix private functions with `_`: `_helper_function`
- No spaces around parentheses: `func() {`

```bash
process_file() { ... }
_private_helper() { ... }
validate_input() { ... }
```

### 3.3 Variables

- **`UPPER_SNAKE_CASE`** for global constants: `readonly MAX_RETRIES=3`
- **`snake_case`** for function-scoped variables: `count=0`
- **`snake_case`** for exported variables: `export DEPLOY_TARGET="prod"`
- Prefix booleans with `is_`, `has_`, `can_`: `is_valid=1`

```bash
readonly DB_HOST="localhost"
result=""
is_root=0
```

**Note:** Strictly POSIX shells do not support the `local` keyword. While widely supported by `dash`, `bash`, `ksh`, and most modern POSIX shells, `local` is a de facto standard rather than a POSIX standard. Document if your script relies on it.

---

## 4. Variables

### 4.1 Declaration & Initialization

```bash
# Function-scoped variables (POSIX has no built-in scoping)
count=0

# Read-only constants (POSIX standard)
readonly CONSTANT="fixed"

# Global variables
GLOBAL_VAR="value"

# Exported (POSIX standard)
export PATH="/usr/local/bin:$PATH"
```

**Note:** Strict POSIX has no variable scoping primitive. The `local` keyword exists in most modern POSIX shells (dash, bash, ksh, ash) but is not in the POSIX standard. For maximum portability, use unique variable naming or unset variables at function end.

### 4.2 Quoting

**Always double-quote variable expansions:**

```bash
# Good
echo "$filename"
result="$(command "$arg")"
if [ -f "$config_file" ]; then

# Bad
echo $filename              # Word splitting risk
result=`command $arg`       # Backticks, unquoted
if [ -f $config_file ]; then  # Unquoted
```

### 4.3 Parameter Expansion

Use POSIX-compliant expansions only:

```bash
${variable}              # Variable value
${variable:-default}     # Default if unset or null
${variable:=default}     # Assign default if unset or null
${variable:?error}       # Exit with error if unset or null
${variable#pattern}      # Remove shortest prefix match
${variable##pattern}     # Remove longest prefix match
${variable%pattern}      # Remove shortest suffix match
${variable%%pattern}     # Remove longest suffix match
```

**Additional POSIX expansion:**
- `${variable:+alternate}` — Use `alternate` if `variable` is set and non-null (POSIX standard)

**Not POSIX (avoid):**
- `${variable//old/new}` (Bash/Zsh extension)
- `${variable,,}` (Bash extension)
- `${variable^^}` (Bash extension)
- `${variable:offset:length}` (substring; Bash extension, not in POSIX)
- Array syntax (no arrays in POSIX)

---

## 5. Functions

### 5.1 Structure

```bash
# Function definition (positional args via $1, $2)
process_data() {
    # Note: 'local' is widely supported but not strictly POSIX.
    # Use 'local' for clarity in modern shells (dash, bash, ksh).
    input="$1"
    output="$2"
    
    # Validate parameters
    [ -n "$input" ] || { echo "ERROR: input required" >&2; return 1; }
    [ -n "$output" ] || { echo "ERROR: output required" >&2; return 1; }
    
    # Function logic
    echo "Processing: $input"
    
    return 0
}

# Function call
if ! process_data "infile" "outfile"; then
    echo "ERROR: processing failed" >&2
    exit 1
fi
```

### 5.2 Parameter Passing & Return Values

POSIX shell functions receive positional arguments via `$1`, `$2`, etc.:

```bash
calculate_sum() {
    num1="$1"
    num2="$2"
    
    # Validation
    if ! echo "$num1" | grep -q '^[0-9][0-9]*$'; then
        echo "ERROR: num1 not numeric" >&2
        return 1
    fi
    
    if ! echo "$num2" | grep -q '^[0-9][0-9]*$'; then
        echo "ERROR: num2 not numeric" >&2
        return 1
    fi
    
    # POSIX arithmetic expansion
    result=$((num1 + num2))
    echo "$result"
    return 0
}

# Call and capture result
if sum=$(calculate_sum 5 10); then
    echo "Sum: $sum"
else
    exit 1
fi
```

### 5.3 Return Codes

```bash
return 0  # Success
return 1  # Failure
```

**Check immediately after function call:**

```bash
if my_function arg1 arg2; then
    echo "Success"
else
    echo "Failed with code $?"
    exit 1
fi
```

---

## 6. Control Flow

### 6.1 Conditionals with `[ ]` and `test`

**Always use `[ ]` (POSIX `test`)**, never `[[ ]]` (Bash extension):

```bash
# File tests
[ -f "$file" ]       # Regular file exists
[ -d "$dir" ]        # Directory exists
[ -r "$file" ]       # File is readable
[ -w "$file" ]       # File is writable
[ -x "$file" ]       # File is executable
[ -e "$file" ]       # File exists (any type)

# String tests
[ -n "$var" ]        # Non-empty string
[ -z "$var" ]        # Empty string
[ "$str1" = "$str2" ]   # String equality
[ "$str1" != "$str2" ]  # String inequality

# Numeric tests
[ "$num1" -eq "$num2" ]  # Equal
[ "$num1" -ne "$num2" ]  # Not equal
[ "$num1" -lt "$num2" ]  # Less than
[ "$num1" -le "$num2" ]  # Less or equal
[ "$num1" -gt "$num2" ]  # Greater than
[ "$num1" -ge "$num2" ]  # Greater or equal

# Logical operators
[ "$a" = "x" ] && [ "$b" = "y" ]   # AND
[ "$a" = "x" ] || [ "$b" = "y" ]   # OR
[ ! -f "$file" ]                    # NOT
```

### 6.2 If Statements

```bash
if [ -f "$file" ]; then
    echo "File exists"
elif [ -d "$file" ]; then
    echo "Directory exists"
else
    echo "Not found"
fi

# One-liner — AVOID this idiom: if the first echo fails, the second still
# runs because of how && and || chain. Use a full if/else instead.
# [ -f "$file" ] && echo "File exists" || echo "Not found"  # buggy
if [ -f "$file" ]; then echo "File exists"; else echo "Not found"; fi
```

### 6.3 Case Statements

```bash
case "$action" in
    start)
        start_service
        ;;
    stop)
        stop_service
        ;;
    restart)
        stop_service
        start_service
        ;;
    *)
        echo "ERROR: unknown action '$action'" >&2
        return 1
        ;;
esac
```

### 6.4 Loops

```bash
# While loop for line processing
while IFS= read -r line; do
    process "$line"
done < "$file"

# For loop over positional arguments
for arg in "$@"; do
    process "$arg"
done

# For loop over glob expansion
for file in *.txt; do
    process "$file"
done
```

**Never:** `for line in $(cat file)` — use `while read` or `< file` instead.

---

## 7. I/O & Redirection

### 7.1 Here-Documents

```bash
cat << 'EOF'
This is a here-document.
Variables like $VAR are literal (single quotes prevent expansion).
EOF

cat << EOF
This expands variables: $VAR
EOF
```

### 7.2 Input Redirection

```bash
# Read from file
while IFS= read -r line; do
    process "$line"
done < input.txt

# Safe: variable persists after loop
```

### 7.3 Output Redirection

```bash
# Redirect stdout
command > output.txt

# Append to file
command >> output.txt

# Redirect stderr
command 2> error.log

# Redirect both
command > output.txt 2>&1

# Suppress output
command > /dev/null 2>&1
```

### 7.4 Temporary Files

**Always use `mktemp`:**

```bash
temp_file=$(mktemp)
trap 'rm -f "$temp_file"' EXIT

echo "data" > "$temp_file"
process "$temp_file"
# Cleanup happens automatically on exit
```

Never use predictable paths like `/tmp/script.$$` — security risk.

---

## 8. Error Handling

### 8.1 Strict Mode

```bash
#!/bin/sh
set -eu
```

- **`set -e`**: Exit immediately if any command fails
- **`set -u`**: Exit if an undefined variable is used

### 8.2 Error Reporting

```bash
die() {
    echo "ERROR: $*" >&2
    exit 1
}

[ -f "$config_file" ] || die "Config file not found"
```

### 8.3 Cleanup on Exit

```bash
# Initialize before `trap` so `set -u` does not error if the trap fires
# before assignment. Use `${var:-}` defensively inside the handler.
temp_dir=""

cleanup() {
    # Capture exit status first; any command in the handler overwrites $?.
    # The script's exit status is preserved automatically when the trap
    # returns, so no explicit `exit` is needed (and `return` inside an
    # EXIT trap does NOT change the script's exit code).
    [ -n "${temp_dir:-}" ] && [ -d "$temp_dir" ] && rm -rf "$temp_dir"
}

trap cleanup EXIT

temp_dir=$(mktemp -d)
# Use $temp_dir
# Cleanup happens automatically on exit
```

**Trap inheritance:** EXIT/ERR traps do **not** survive `exec` (the new program replaces the shell). Subshells (`( ... )`, command substitution `$(...)`, pipelines) inherit EXIT traps only in some shells (bash resets them; dash and ksh behaviour varies). Do not rely on EXIT traps firing in subshells — perform cleanup in the parent shell only.

**Note:** `mktemp` is a de facto standard but is not part of POSIX.1. It is available on all modern Linux, BSD, and macOS systems. For strict POSIX environments without `mktemp`, use `tmpdir=${TMPDIR:-/tmp}/script.$$-$(awk 'BEGIN{srand();print int(rand()*100000)}')` and verify uniqueness.

### 8.4 Handling Specific Failures

```bash
# Expected failure: use || true
optional_command || true

# Explicit check
if ! command; then
    echo "Command failed but continuing..."
fi

# Temporarily disable error exit
set +e
optional_command
set -e
```

---

## 9. Performance & Idioms

### 9.1 Prefer Built-ins

Use POSIX shell built-ins instead of external commands:

```bash
# Good (built-in parameter expansion)
basename="${var##*/}"           # Equivalent to basename
dirname="${var%/*}"              # Equivalent to dirname

# Avoid (external commands)
basename "$var"
dirname "$var"
```

### 9.2 Avoid Useless Pipes

```bash
# Good
grep pattern < file

# Bad
cat file | grep pattern
```

### 9.3 Arithmetic

Use `$((expression))` arithmetic expansion (POSIX.1-2001+ standard):

```bash
result=$((5 + 3))
count=$((count + 1))

# Comparison
if [ $((count % 2)) -eq 0 ]; then
    echo "Even"
fi
```

**Note:** `$((arithmetic))` is the modern POSIX standard. Avoid `expr` (slower, requires external process). Use `expr` only for shells older than POSIX.1-2001 (very rare in practice).

### 9.4 String Manipulation

```bash
# Remove shortest suffix matching pattern (POSIX)
filename="${path%.txt}"

# Remove longest suffix
basename="${path%%.*}"

# Remove shortest prefix matching pattern (POSIX)
without_leading_slash="${path#/}"

# Remove longest prefix
extension="${file##*.}"

# Substring extraction is NOT POSIX — use cut, awk, or sed:
field=$(echo "$line" | cut -d: -f2)
```

---

## 10. Security

### 10.1 Input Validation

Validate all external input:

```bash
validate_number() {
    value="$1"
    
    # Check if numeric (one or more digits)
    if ! echo "$value" | grep -q '^[0-9][0-9]*$'; then
        echo "ERROR: not a number" >&2
        return 1
    fi
    
    # Check range
    if [ "$value" -lt 0 ] || [ "$value" -gt 100 ]; then
        echo "ERROR: out of range" >&2
        return 1
    fi
    
    return 0
}
```

### 10.2 Prevent Command Injection

**Never use `eval`:**

```bash
# Bad: command injection
eval "process $input"

# Good: use command with arguments
command "$input"
```

### 10.3 Secure Temporary Files

```bash
# Good: unpredictable name
temp_file=$(mktemp)

# Bad: predictable
temp_file="/tmp/script.$$"
```

### 10.4 No Hardcoded Secrets

```bash
# Bad
API_KEY="sk-1234567890abcdef"

# Good
API_KEY="${API_KEY:-}"
[ -n "$API_KEY" ] || die "API_KEY not set"
```

---

## 11. Testing

### 11.1 Manual Testing Checklist

- [ ] Test with missing arguments
- [ ] Test with invalid input (special characters, empty strings)
- [ ] Test on different shells (dash, ash, ksh, bash in sh mode)
- [ ] Test on different Unix systems (Linux, BSD, macOS)
- [ ] Verify no temporary files are left
- [ ] Check all error paths

### 11.2 Shell Compatibility Check

```bash
#!/bin/sh
# Check shell compatibility
for shell in sh dash bash ksh; do
    if command -v "$shell" >/dev/null 2>&1; then
        echo "Testing with $shell..."
        "$shell" script.sh test-arg
    fi
done
```

### 11.3 ShellCheck for POSIX

Use ShellCheck with POSIX mode:

```bash
shellcheck -S info script.sh  # Show all issues
```

---

## 12. Defensive Programming & Input Validation

### 12.1 Boundary Validation

```bash
process_file() {
    file="$1"
    
    # Presence check
    [ -n "$file" ] || die "file parameter required"
    
    # Existence and readability
    [ -r "$file" ] || die "cannot read file: $file"
    
    # Regular file (not directory)
    [ -f "$file" ] || die "not a file: $file"
    
    # Process
    while IFS= read -r line; do
        process_line "$line"
    done < "$file"
}
```

### 12.2 Type and Format Validation

```bash
validate_email() {
    email="$1"
    
    # Very basic validation (use a stricter pattern for production)
    if echo "$email" | grep -q '^[^@][^@]*@[^@][^@]*\.[^@][^@]*$'; then
        return 0
    else
        echo "ERROR: invalid email format" >&2
        return 1
    fi
}
```

### 12.3 Range Validation

```bash
validate_port() {
    port="$1"
    
    # Must be numeric (one or more digits)
    if ! echo "$port" | grep -q '^[0-9][0-9]*$'; then
        echo "ERROR: port must be numeric" >&2
        return 1
    fi
    
    # Range check
    if [ "$port" -lt 1 ] || [ "$port" -gt 65535 ]; then
        echo "ERROR: port must be 1-65535" >&2
        return 1
    fi
    
    return 0
}
```

---

## 13. Project Structure

```
project/
├── script.sh              # Main executable
├── lib/
│   ├── helpers.sh         # Shared functions
│   ├── validation.sh      # Input validation
│   └── config.sh          # Configuration
├── tests/
│   └── test_script.sh     # Test suite
└── README.md              # Usage documentation
```

---

## 14. Tooling

- **ShellCheck** (linter): `shellcheck script.sh`
- **Built-in help**: `help` or man pages
- **POSIX specs**: [IEEE 1003.1](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html)

---

## 15. SBOM Creation & Dependency Management

### 15.1 Document Dependencies

```bash
# External tools required:
# - grep (POSIX standard)
# - sed (POSIX standard)
# - awk (POSIX standard)
# - curl (optional, for remote operations)

# Check for required tools
for cmd in grep sed awk; do
    command -v "$cmd" >/dev/null || die "Required tool not found: $cmd"
done
```

### 15.2 Version Constraints

Document POSIX shell version requirement:

```bash
#!/bin/sh
# Requires: POSIX shell (sh)
# Tested on: dash, bash, ksh
# Not compatible with: csh, tcsh (csh family)
```

---

## 16. References

### Official Standards
- **[IEEE Std 1003.1-2017](https://pubs.opengroup.org/onlinepubs/9699919799/)** — POSIX specification
- **[Shell & Utilities](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html)** — Shell command language

### Community & Best Practices
- **[POSIX Shell Pitfalls](https://mywiki.wooledge.org/BashGuide/Practices)** — Common mistakes
- **[Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html)** — General shell practices
- **[ShellCheck](https://www.shellcheck.net/)** — Static analysis for shell scripts

### Tools & Resources
- **[Dash](http://gondor.apana.org.au/~herbert/dash/)** — Minimal POSIX shell (reference implementation)
- **[SS64 sh Reference](https://ss64.com/bash/)** — Command reference

### Books & Articles
- **"The Pragmatic Programmer"** (Hunt & Thomas, 2019) — General principles
- **"Clean Code"** (Martin, 2008) — Code quality principles
- **[Portable Shell Programming](https://www.in-ulm.de/~mascheck/various/sh-implementation/)** — Deep dive into portability

### When to Use Bash/Zsh Instead

If your script needs:
- Arrays or associative arrays
- Extended pattern matching
- Complex string manipulation
- `[[ ]]` or process substitution
- Shell-specific features

→ **Switch to Bash or Zsh, or rewrite in Python/Go.**
