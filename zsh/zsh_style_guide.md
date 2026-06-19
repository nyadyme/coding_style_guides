# Zsh Coding Style Guidelines

A comprehensive guide rooted in the Zsh manual, the Advanced Bash-Scripting Guide, Google Shell Style Guide, ShellCheck, and established software engineering literature (*Clean Code*, *The Pragmatic Programmer*).

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

### 1.1 Zsh's Strengths

Zsh is a powerful, interactive shell with extensive scripting capabilities. Key strengths:
- **Interactive features**: powerful completion system, globbing, history expansion
- **Scripting power**: associative arrays, floating-point arithmetic, extended pattern matching
- **POSIX compatibility**: can be run as sh-compatible shell when needed

### 1.2 When to Use Zsh

- **Interactive configuration**: .zshrc and .zprofile for shell customization
- **Complex automation scripts**: when features justify Zsh-only dependencies
- **System administration**: advanced file operations, conditional logic
- **Hard limit**: ~500 lines. Beyond that, rewrite in Python or Go

### 1.3 Guiding Principles

| Principle | Summary |
|---|---|
| **Explicit over implicit** | Use explicit option flags and clear variable expansion |
| **Fail fast** | Use `set -euo pipefail` equivalent in Zsh mode |
| **Safety first** | Quote all variables and command substitutions |
| **POSIX where possible** | Code to Zsh but avoid Zsh-only features if POSIX shells suffice |
| **Keep it simple** | Avoid complex nested constructs; prefer clarity |

---

## 2. Code Layout & Formatting

### 2.1 Shebang and Headers

```bash
#!/usr/bin/env zsh
# Script description: one-line summary
# Usage: ./script.zsh [options] arguments

set -euo pipefail

# External dependencies
# Depends on: jq, curl
```

**Why `#!/usr/bin/env zsh`:** Finds zsh in PATH rather than hard-coding `/bin/zsh` or `/usr/bin/zsh`.

### 2.2 Indentation & Spacing

- **2 spaces** indentation. Never tabs.
- **80 characters** hard line limit. Break with `\` (backslash-newline).
- Indent continuations by 4 spaces.
- Two blank lines between top-level function definitions.
- One blank line between logical sections within a function.
- No blank lines at start/end of function body.
- Opening braces on same line: `func() {`, `if [[ ... ]]; then`.

### 2.3 Line Breaking

Break lines after:
- Pipes (`|`)
- Logical operators (`&&`, `||`)
- Flags and arguments in long commands
- Redirections (`>`, `<`, `>>`)

```bash
# Good
grep pattern file \
  | sort \
  | uniq

curl -s https://example.com \
  -H "Authorization: Bearer $token" \
  -d @payload.json

# Bad
grep pattern file | sort | uniq  # if line exceeds 80 chars
```

### 2.4 Comments

- **Inline comments**: Explain *why*, not *what*. Code explains what.

```bash
# Bad
x=5  # Set x to 5

# Good
x=${data#*:}  # Extract substring after colon
```

- **Function header comments**: Always include for non-trivial functions
- **SAFETY comments**: Required above every dangerous operation

---

## 3. Naming Conventions

### 3.1 Functions

- **Zsh convention:** `snake_case` (e.g., `process_file`)
- **Prefix for private:** `_private_function` (convention, not enforced)
- **Booleans:** Prefix with `is_`, `has_`, `can_` (e.g., `is_valid_email`)
- **Script names:** `kebab-case.zsh` or `snake_case.zsh` describing the action

```bash
# Good
start_service() { ... }
is_root() { ... }
check_dependencies() { ... }

# Bad
StartService() { ... }
root() { ... }  # Ambiguous
check() { ... }  # Too generic
```

### 3.2 Variables

- **Local variables:** `snake_case`
- **Global constants:** `UPPER_SNAKE_CASE` with `readonly` or `typeset -r`
- **Environment variables:** `UPPER_SNAKE_CASE`
- **Temporary/scratch:** `_temp`, `_result` (indicate internal use)

```bash
readonly DB_HOST="localhost"
readonly MAX_RETRIES=3
local config_file="${HOME}/.config/app/config"
```

### 3.3 Zsh-Specific Naming

- **Array variables:** Use plural or descriptive names: `files`, `user_ids`, `options`
- **Associative arrays:** Descriptive: `declare -A config_map`, `declare -A error_codes`

---

## 4. Variables

### 4.1 Variable Declarations

**Always declare variables with `local` inside functions:**

```bash
process_data() {
    local input="$1"
    local output_dir="$2"
    local -r max_size=1000
    local -i count=0
    
    # ...
}
```

**Variable typing in Zsh:**

```bash
local -i count=0          # Integer
local -r readonly_var="x" # Read-only
local -a array_var=()     # Array
local -A hash_var=()      # Associative array
local -F float_var=3.14   # Float
```

### 4.2 Quoting and Expansion

**Always double-quote variable expansions and command substitutions:**

```bash
# Good
echo "${filename}"
result="$(command "$arg")"
if [[ -f "${config_file}" ]]; then

# Bad
echo $filename              # Word splitting risk
result=`command $arg`       # Backticks, unquoted
if [[ -f $config_file ]]; # Unquoted variable
```

**Exceptions to quoting:**

- Inside `(( ))` arithmetic: `(( count++ ))`
- Left side of `=~` in `[[ ]]`: `[[ $str =~ ^[0-9]+$ ]]`
- **Deliberate word splitting:** `local -a args=$options` (rare; prefer arrays)

### 4.3 Arrays and Associative Arrays

Use arrays instead of space-delimited strings:

```bash
# Good
local -a files=("file1.txt" "file2.txt" "file with spaces.txt")
for file in "${files[@]}"; do
    process "$file"
done

# Bad
local files="file1.txt file2.txt file with spaces.txt"
for file in $files; do  # Breaks on "file with spaces.txt"
    process "$file"
done
```

**Associative arrays:**

```bash
local -A config_map=(
    [host]="localhost"
    [port]="5432"
    [user]="admin"
)

echo "${config_map[host]}"  # Print specific key
```

### 4.4 Parameter Expansion

Use `${ }` form with braces:

```bash
${variable}           # Default: variable expansion
${variable:-default}  # Default if unset or null
${variable:=default}  # Assign default if unset or null
${variable:?error}    # Exit with error if unset or null
${variable:+value}    # Use value if variable is set and non-null
${variable#pattern}   # Remove shortest matching prefix
${variable##pattern}  # Remove longest matching prefix
${variable%pattern}   # Remove shortest matching suffix
${variable%%pattern}  # Remove longest matching suffix
${variable/old/new}   # Replace first occurrence (Zsh/Bash)
${variable//old/new}  # Replace all occurrences (Zsh/Bash)
${(L)variable}        # Lowercase (Zsh-specific)
${(U)variable}        # Uppercase (Zsh-specific)
```

---

## 5. Functions

### 5.1 Structure

Every function must be small, focused, and testable:

```bash
# Good: small, single responsibility
read_config() {
    local -r config_file="$1"
    [[ -r "$config_file" ]] || return 1
    source "$config_file"
}

validate_email() {
    local -r email="$1"
    [[ "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]
}

# Bad: too long, multiple responsibilities
process_everything() {
    # 100+ lines mixing validation, processing, and output
}
```

### 5.2 Parameter Validation

Validate parameters at the top of the function:

```bash
copy_file() {
    local -r src="$1"
    local -r dst="$2"
    
    [[ -n "$src" ]] || { echo "ERROR: src required" >&2; return 1; }
    [[ -n "$dst" ]] || { echo "ERROR: dst required" >&2; return 1; }
    [[ -f "$src" ]] || { echo "ERROR: src not a file" >&2; return 1; }
    [[ -d "${dst%/*}" ]] || { echo "ERROR: dst parent not a directory" >&2; return 1; }
    
    cp "$src" "$dst"
}
```

### 5.3 Return Values

```bash
is_valid() {
    [[ $# -eq 1 ]] && [[ -n "$1" ]] && [[ "$1" =~ ^[0-9]+$ ]]
}

if is_valid "42"; then
    echo "Valid number"
fi
```

**Return data via stdout:**

```bash
get_hostname() {
    hostname  # Data to stdout
}

local -r hostname="$(get_hostname)"
```

---

## 6. Control Flow

### 6.1 Use `[[ ]]` for Conditionals

Always use `[[ ]]`, never `[ ]` or `test`:

```bash
# Good
if [[ -f "$file" ]]; then
    process "$file"
fi

if [[ "$count" -gt 10 ]]; then
    echo "Count is large"
fi

if [[ "$string" =~ ^[A-Z] ]]; then
    echo "Starts with uppercase"
fi

# Bad
if [ -f "$file" ]; then
if test -f "$file"; then
if [ $count -gt 10 ]; then
```

### 6.2 Arithmetic

Use `(( ))` for arithmetic:

```bash
# Good
(( count++ ))
(( total += amount ))
if (( count > max )); then
    echo "Exceeded limit"
fi

# Bad
count=$((count + 1))
if [ $count -gt $max ]; then
```

### 6.3 Case Statements

Prefer `case` over long `if/elif` chains:

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

# For loop over array
for item in "${items[@]}"; do
    process "$item"
done

# C-style loop (Zsh supports this)
for (( i=0; i < 10; i++ )); do
    process "$i"
done
```

**Never:** `for line in $(cat file)` — use `while read` or `< <(command)` instead.

---

## 7. I/O & Redirection

### 7.1 Here-Documents

Use for multi-line output:

```bash
cat << 'EOF'
This is a here-document.
Variables like $VAR are literal (single quotes).
EOF

cat << EOF
This expands variables: $VAR
EOF
```

### 7.2 Process Substitution

Avoid subshell variable loss:

```bash
# Good: variables persist
while IFS= read -r line; do
    count=$((count + 1))
    echo "$line"
done < <(grep pattern file)

# Bad: subshell loses $count
grep pattern file | while IFS= read -r line; do
    count=$((count + 1))  # Lost after loop
done
echo "$count"  # Empty
```

### 7.3 Temporary Files

**Always use `mktemp`:**

```bash
local -r temp_file="$(mktemp)"
trap 'rm -f "$temp_file"' EXIT

# ... use $temp_file ...
```

Never use predictable paths like `/tmp/script.$$` — security risk.

### 7.4 Pipe Data Carefully

Zsh passes **text through pipes**, not typed objects:

```bash
# Pipe text to jq for structured parsing
curl -s "$url" | jq '.data[]'

# Never assume format stability
# Parse at boundaries and validate
if ! data=$(curl -s "$url" | jq -r '.name' 2>/dev/null); then
    echo "ERROR: failed to parse response" >&2
    return 1
fi
```

---

## 8. Error Handling

### 8.1 Strict Mode

```bash
#!/usr/bin/env zsh
set -euo pipefail
```

- **`-e`**: Exit on error
- **`-u`**: Exit on undefined variable
- **`-o pipefail`**: Pipe fails if any command fails

### 8.2 Error Reporting

```bash
die() {
    echo "ERROR: $*" >&2
    exit 1
}

[[ -f "$config_file" ]] || die "Config file not found: $config_file"
```

### 8.3 Cleanup on Exit

```bash
main() {
    local -r temp_dir="$(mktemp -d)"
    trap 'rm -rf "$temp_dir"' EXIT
    
    # Use $temp_dir
    # Cleanup happens automatically on exit
}

main "$@"
```

### 8.4 Handling Expected Failures

```bash
# Option 1: || true
command || true

# Option 2: Explicit check
if ! command; then
    echo "Command failed but continuing..."
fi

# Option 3: Disable error exit temporarily
set +e
optional_command
set -e
```

---

## 9. Performance & Idioms

### 9.1 Prefer Built-ins

Use Zsh/Bash built-ins instead of external commands:

```bash
# Good (built-in parameter expansion)
${variable##*/}     # Like basename
${variable%/*}      # Like dirname
${(L)variable}      # Lowercase (Zsh-native)
${(U)variable}      # Uppercase (Zsh-native)

# Avoid (external commands)
basename "$variable"
dirname "$variable"
tr '[:upper:]' '[:lower:]' <<<"$variable"
```

Note: `${var,,}` / `${var^^}` are Bash syntax. Use Zsh's native `${(L)var}` /
`${(U)var}` for case conversion.

### 9.2 Avoid Useless `cat`

```bash
# Good
grep pattern < file

# Bad
cat file | grep pattern
```

### 9.3 Brace Expansion

Create multiple similar items:

```bash
mkdir -p project/{src,tests,docs,config}
for dir in src tests docs config; do
    mkdir -p "project/$dir"
done
```

### 9.4 Zsh-Specific Optimizations

```bash
# Zsh arithmetic (no need for $((..)))
(( count++ ))

# Zsh glob qualifiers (in parentheses after the pattern)
print -l *.txt(.)      # (.) regular files only
print -l *.txt(@)      # (@) symbolic links only
print -l *(/)          # (/) directories only
print -l *(.x)         # combined: regular files that are executable

# Zsh functions are faster than subshells
myfunc() { true; }
```

---

## 10. Security

### 10.1 Input Validation

Validate all external input:

```bash
validate_port() {
    local -r port="$1"
    [[ "$port" =~ ^[0-9]+$ ]] || return 1
    (( port >= 1 && port <= 65535 )) || return 1
}

validate_hostname() {
    local -r host="$1"
    [[ "$host" =~ ^[a-zA-Z0-9]([a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(\.[a-zA-Z0-9]([a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$ ]]
}
```

### 10.2 Command Injection Prevention

**Never `eval`:**

```bash
# Bad: command injection
eval "process $input"

# Good: use array for dynamic commands
local -a cmd=(curl -s "$url")
[[ -n "$token" ]] && cmd+=(-H "Authorization: Bearer $token")
"${cmd[@]}"
```

### 10.3 Safe Temporary Files

```bash
local -r tmpfile="$(mktemp)"
trap 'rm -f "$tmpfile"' EXIT

# Never
local tmpfile="/tmp/script.$$"  # Predictable, vulnerable
```

### 10.4 No Hardcoded Secrets

```bash
# Bad
API_KEY="sk-1234567890abcdef"

# Good
API_KEY="${API_KEY:-}"
[[ -n "$API_KEY" ]] || { echo "ERROR: API_KEY not set" >&2; return 1; }
```

---

## 11. Testing

### 11.1 ShellCheck

All scripts **must** pass ShellCheck:

```bash
shellcheck script.zsh
```

Enable in CI/CD. Fix all warnings, not just errors.

### 11.2 Bats (Bash Automated Testing System)

Use for unit and integration tests:

```bash
@test "validate_email accepts valid addresses" {
    source script.zsh
    validate_email "user@example.com"
}

@test "validate_email rejects invalid addresses" {
    source script.zsh
    ! validate_email "invalid"
}
```

### 11.3 Manual Testing

- Test with different input sizes
- Test edge cases (empty input, special characters)
- Test on different shells (bash, zsh, dash) if claiming POSIX compatibility
- Use `--dry-run` flag for destructive scripts

---

## 12. Defensive Programming & Input Validation

Zsh passes text through pipes. Validate at every boundary — CLI args, files, external commands, environment.

### 12.1 Boundary Validation

```bash
process_user_input() {
    local -r input="$1"
    
    # Validate presence
    [[ -n "$input" ]] || { echo "ERROR: input required" >&2; return 1; }
    
    # Validate format
    [[ "$input" =~ ^[a-zA-Z0-9_-]+$ ]] || { echo "ERROR: invalid format" >&2; return 1; }
    
    # Validate length
    [[ ${#input} -le 255 ]] || { echo "ERROR: input too long" >&2; return 1; }
    
    # Process
    process "$input"
}
```

### 12.2 Type and Range Validation

```bash
process_number() {
    local -r value="$1"
    
    # Validate numeric
    [[ "$value" =~ ^[0-9]+$ ]] || { echo "ERROR: not a number" >&2; return 1; }
    
    # Validate range
    (( value >= 0 && value <= 100 )) || { echo "ERROR: out of range [0,100]" >&2; return 1; }
    
    process "$value"
}
```

### 12.3 File Path Validation

```bash
read_config() {
    local -r config_file="$1"
    
    # Exists and readable
    [[ -r "$config_file" ]] || { echo "ERROR: cannot read $config_file" >&2; return 1; }
    
    # Is a regular file (not symlink or directory)
    [[ -f "$config_file" ]] || { echo "ERROR: not a file: $config_file" >&2; return 1; }
    
    source "$config_file"
}
```

### 12.4 External Command Output Parsing

```bash
# Parse JSON safely
if ! data=$(curl -s "$url" | jq -r '.field' 2>/dev/null); then
    echo "ERROR: failed to parse response" >&2
    return 1
fi

# Validate parsed result
[[ -n "$data" ]] || { echo "ERROR: missing field in response" >&2; return 1; }
```

---

## 13. Project Structure

```
project/
├── script.zsh          # Main executable
├── lib/
│   ├── helpers.zsh     # Shared functions
│   ├── validation.zsh  # Input validation
│   └── config.zsh      # Configuration defaults
├── tests/
│   ├── test_helpers.bats
│   └── test_validation.bats
├── .shellcheckrc       # ShellCheck config
└── README.md           # Usage and documentation
```

---

## 14. Tooling

- **ShellCheck** (linter): `shellcheck script.zsh`
- **Bats** (testing): `bats tests/*.bats`
- **Shfmt** (formatter): Not applicable; use Zsh conventions
- **Zsh built-in help**: `help function_name` or online manual

---

## 15. SBOM Creation & Dependency Management

### 15.1 Document External Dependencies

```bash
# At the top of the script
# External dependencies:
# - jq (JSON processor)
# - curl (HTTP client)
# - grep, sort, uniq (standard Unix tools)

# Validate dependencies
for cmd in jq curl grep sort uniq; do
    command -v "$cmd" &>/dev/null || { echo "ERROR: $cmd not found" >&2; exit 1; }
done
```

### 15.2 Version Constraints

Document minimum versions if applicable:

```bash
# Minimum versions:
# - zsh >= 5.0
# - jq >= 1.5

# Zsh exposes its version via $ZSH_VERSION (string) and $ZSH_VERSINFO (array).
# Use the bundled `is-at-least` autoload for portable version checks.
autoload -Uz is-at-least
is-at-least 5.0 "$ZSH_VERSION" || {
    echo "ERROR: zsh 5.0+ required (found $ZSH_VERSION)" >&2
    exit 1
}
```

### 15.3 Container Deployment

Include dependencies in Dockerfile:

```dockerfile
RUN apt-get update && apt-get install -y \
    zsh \
    jq \
    curl \
    && rm -rf /var/lib/apt/lists/*

COPY script.zsh /usr/local/bin/
RUN chmod +x /usr/local/bin/script.zsh
```

---

## 16. References

### Official Documentation
- **[Zsh Manual](http://zsh.sourceforge.net/Doc/)** — Complete reference
- **[POSIX Shell Command Language](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html)** — Standard

### Community & Best Practices
- **[Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html)** — Industry standard
- **[Advanced Bash-Scripting Guide](https://www.tldp.org/LDP/abs/html/)** — Comprehensive tutorial
- **[ShellCheck](https://www.shellcheck.net/)** — Static analysis

### Books
- **"The Pragmatic Programmer"** (Hunt & Thomas, 2019) — General principles
- **"Clean Code"** (Martin, 2008) — Code quality and naming
