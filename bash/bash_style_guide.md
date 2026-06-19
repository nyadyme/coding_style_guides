# Bash Coding Style Guidelines

A comprehensive guide rooted in the *Advanced Bash-Scripting Guide* (Mendel
Cooper), the GNU Bash Reference Manual, Google's Shell Style Guide,
ShellCheck's rule set, and established software engineering principles from
*Clean Code* (Martin, 2008).

---

## Table of Contents

1. [Philosophy](#1-philosophy)
2. [Script Structure & Boilerplate](#2-script-structure--boilerplate)
3. [Code Layout & Formatting](#3-code-layout--formatting)
4. [Naming Conventions](#4-naming-conventions)
5. [Functions](#5-functions)
6. [Variables & Data Types](#6-variables--data-types)
7. [Quoting & Word Splitting](#7-quoting--word-splitting)
8. [Control Flow](#8-control-flow)
9. [Error Handling](#9-error-handling)
10. [Input / Output & Redirection](#10-input--output--redirection)
11. [Process Management](#11-process-management)
12. [Testing & Validation](#12-testing--validation)
13. [Performance & Idiomatic Bash](#13-performance--idiomatic-bash)
14. [Security](#14-security)
15. [Defensive Programming & Input Validation](#15-defensive-programming--input-validation)
16. [Portability](#16-portability)
17. [Tooling](#17-tooling)
18. [SBOM Creation & Dependency Management](#18-sbom-creation--dependency-management)
19. [References](#19-references)

---

## 1. Philosophy

### 1.1 When to Use Bash

Bash is the right tool for **glue code** — orchestrating other programs,
automating repetitive tasks, and writing thin wrappers. If the script
exceeds ~200 lines, needs data structures beyond arrays, or performs complex
string manipulation, **rewrite it in Python, Go, or another general-purpose
language**.

### 1.2 Guiding Principles

| Principle | Source | Summary |
|---|---|---|
| **Fail early, fail loud** | Defensive scripting | Use `set -euo pipefail` so errors are never swallowed. |
| **Quote everything** | ShellCheck, *ABSG* Ch. 5 | Unquoted variables are the #1 source of Bash bugs. |
| **Readability over cleverness** | *Clean Code*, Google Shell Style Guide | Shell syntax is dense enough. Favour clarity. |
| **Prefer built-ins over external commands** | *ABSG* Ch. 15 | `[[ ]]`, parameter expansion, and arithmetic beat `sed`/`awk` for simple tasks. |
| **One script, one purpose** | *Clean Code* Ch. 10 | A script does one job. Compose scripts via pipes and arguments. |
| **Idempotency** | Ops best practices | Running a script twice should produce the same result as running it once. |

---

## 2. Script Structure & Boilerplate

### 2.1 Shebang

Every script **must** start with a shebang. Use the `env` form for
portability:

```bash
#!/usr/bin/env bash
```

- Use `#!/usr/bin/env bash`, not `#!/bin/bash` — the latter fails on systems
  where Bash is installed elsewhere (NixOS, Homebrew, FreeBSD).
- Never use `#!/bin/sh` for scripts that rely on Bash-specific features.

### 2.2 Strict Mode

Immediately after the shebang, enable strict mode:

```bash
#!/usr/bin/env bash
set -euo pipefail
```

| Flag | Effect |
|---|---|
| `-e` (`errexit`) | Exit immediately if any command returns non-zero. |
| `-u` (`nounset`) | Treat unset variables as an error. |
| `-o pipefail` | A pipeline fails if *any* stage fails, not just the last. |

Add `IFS=$'\n\t'` if you process newline/tab-delimited data and want to avoid
space-splitting surprises:

```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'
```

### 2.3 Script Template

```bash
#!/usr/bin/env bash
set -euo pipefail

# Description: Brief one-line summary of what this script does.
# Usage:       script_name.sh <arg1> [arg2]

readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "${BASH_SOURCE[0]}")"

usage() {
    cat <<EOF
Usage: ${SCRIPT_NAME} <target> [options]

Arguments:
    target    The deployment target (staging|production)

Options:
    -h, --help    Show this help message
    -v, --verbose Enable verbose output
EOF
    exit 1
}

main() {
    local target="${1:-}"
    if [[ -z "${target}" ]]; then
        usage
    fi

    # Script logic here
    deploy "${target}"
}

deploy() {
    local target="$1"
    echo "Deploying to ${target}..."
}

main "$@"
```

### 2.4 The `main` Function Pattern

**Always** wrap script logic in a `main` function and call it at the end of
the file:

```bash
main() {
    parse_args "$@"
    validate_environment
    do_work
}

main "$@"
```

**Why:**

- Functions are defined before they are called — no ordering surprises.
- `set -e` behaves more predictably inside functions.
- The script can be sourced in tests without immediately executing.

---

## 3. Code Layout & Formatting

### 3.1 Indentation

- Use **2 spaces** per indentation level (Google Shell Style Guide). Never use
  tabs.

### 3.2 Line Length

- **80 characters** is the ideal maximum. Treat it as a hard limit and break
  lines proactively rather than letting them grow.
- **Always break with `\`** (backslash-newline) whenever a line approaches or
  exceeds 80 characters. Place the `\` at the end of the line with no
  trailing whitespace, and indent the continuation by **4 spaces**:

```bash
curl \
    --silent \
    --fail \
    --header "Authorization: Bearer ${TOKEN}" \
    --output "${output_file}" \
    "${url}"
```

- **Where to break** — choose logical points that keep the continuation
  readable:
  - After pipe operators (`|`), before the next command.
  - After logical operators (`&&`, `||`).
  - After each flag/option of a command.
  - After each argument in a function call.
  - After redirection operators (`>`, `>>`, `2>&1`).

```bash
# Pipes — break after |
find "${src_dir}" -name "*.log" -mtime +30 \
    | sort \
    | head -n 20

# Logical operators — break after && or ||
[[ -f "${config}" ]] \
    && source "${config}" \
    || die "Config not found: ${config}"

# Long conditionals — break after ||, &&
if [[ -n "${host}" ]] \
    && [[ -n "${port}" ]] \
    && [[ "${port}" -gt 0 ]]; then
    connect "${host}" "${port}"
fi

# Long strings — use heredocs instead of \
cat <<EOF
This is a long message that would exceed 80 characters
if written on a single line with echo.
EOF
```

- **Do not break** inside quoted strings with `\` — it embeds a literal
  newline. Use concatenation or heredocs for long strings.
- Configure `shfmt` to match: `shfmt -i 2 -ci` (the `-ci` flag indents
  switch cases, and `shfmt` preserves `\` continuation indentation).

### 3.3 Blank Lines (Google Shell Style Guide, `shfmt`)

- **Two blank lines** between top-level function definitions (Google Shell
  Style Guide recommendation for visual separation in large scripts).
- **One blank line** between logical sections within a function.
- **No blank line** between a function name and its opening brace `{`.
- **No blank line** at the start or end of a function body.
- **One blank line** after the script header comment block before the first
  code.
- **One blank line** between global variable/constant declarations and
  function definitions.
- **No trailing blank lines** at the end of the file (a single newline
  terminator is required).

### 3.4 Braces and Keywords

- Opening brace on the **same line** as the function name:

```bash
my_function() {
    # ...
}
```

- `then`, `do` on the **same line** as `if`, `for`, `while`:

```bash
if [[ -f "${file}" ]]; then
    process "${file}"
fi

for item in "${items[@]}"; do
    echo "${item}"
done
```

### 3.5 Comments

- Use `#` comments. Place them **above** the line they describe, not inline
  (except for short annotations).
- Start every script with a header comment: description, usage, author
  (optional).
- **Comments explain why**, not what. If the code needs a "what" comment,
  rewrite the code (extract a function, use better names).
- Never commit commented-out code.

---

## 4. Naming Conventions

| Entity | Convention | Example |
|---|---|---|
| Script file | `kebab-case` or `snake_case` | `deploy-app.sh`, `run_tests.sh` |
| Function | `snake_case` | `parse_args`, `validate_input` |
| Local variable | `snake_case` | `file_count`, `output_dir` |
| Global/env variable | `UPPER_SNAKE_CASE` | `DATABASE_URL`, `LOG_LEVEL` |
| Constant | `UPPER_SNAKE_CASE` with `readonly` | `readonly MAX_RETRIES=3` |
| Loop variable | Short, descriptive | `file`, `line`, `item` |

### 4.1 Naming Guidance

- **Use intention-revealing names.** `backup_dir` beats `d`.
- **Prefix booleans** with `is_`, `has_`, `should_`: `is_verbose`, `has_error`.
- **No single-letter variables** except in trivial loops (`i`, `n`).
- **Script names** should describe the action: `deploy-app.sh`, not
  `script1.sh`.

---

## 5. Functions

### 5.1 Size and Focus

- Functions should be **small** — ideally under 20 lines.
- Each function does **one thing**.
- Extract complex logic into named functions rather than inlining.

### 5.2 Function Declaration

Use the `name() { }` form (POSIX-compatible), not `function name { }`:

```bash
# Yes
my_function() {
    local arg="$1"
    echo "${arg}"
}

# No — "function" keyword is a Bashism with no benefit
function my_function {
    # ...
}
```

### 5.3 Local Variables

**Always** declare function-scoped variables with `local`:

```bash
process_file() {
    local file="$1"
    local line_count

    line_count="$(wc -l < "${file}")"
    echo "File ${file} has ${line_count} lines"
}
```

- Declare and assign on **separate lines** when the assignment involves a
  command substitution — otherwise `local` masks the exit code:

```bash
# Bad — local masks the exit code of the command
local output="$(failing_command)"  # $? is always 0

# Good — exit code is preserved
local output
output="$(failing_command)"
```

### 5.4 Return Values

- Use **return codes** (`return 0` for success, `return 1` for failure) for
  status.
- Use **stdout** for data output.
- Never use global variables to return data from functions — use command
  substitution:

```bash
get_timestamp() {
    date +%Y%m%d_%H%M%S
}

timestamp="$(get_timestamp)"
```

### 5.5 Argument Validation

Validate arguments at the top of each function:

```bash
deploy() {
    local target="${1:?Error: target is required}"
    local version="${2:-latest}"

    # ...
}
```

- `${var:?message}` exits with an error if the variable is unset or empty.
- `${var:-default}` provides a default value.

---

## 6. Variables & Data Types

### 6.1 Declaration

- Use `readonly` for constants:

```bash
readonly CONFIG_FILE="/etc/myapp/config.yml"
readonly MAX_RETRIES=3
```

- Use `local` inside functions (see 5.3).
- Use `declare -a` for arrays, `declare -A` for associative arrays:

```bash
declare -a files=("a.txt" "b.txt" "c.txt")
declare -A config=(
    [host]="localhost"
    [port]="5432"
)
```

### 6.2 Arrays (*ABSG* Ch. 27)

Arrays are one of Bash's most useful features. Use them instead of
space-delimited strings:

```bash
# Yes — array, safe with spaces in filenames
files=("file one.txt" "file two.txt")
for file in "${files[@]}"; do
    process "${file}"
done

# No — word splitting breaks on spaces
files="file one.txt file two.txt"
for file in ${files}; do
    process "${file}"
done
```

- `"${array[@]}"` expands each element as a separate word.
- `"${array[*]}"` expands all elements as a single word (rarely wanted).
- `"${#array[@]}"` gives the array length.

### 6.3 Associative Arrays (Bash 4+)

> macOS ships Bash 3.2 as `/bin/bash` for licensing reasons. Scripts that
> use associative arrays, `${var,,}` / `${var^^}` case conversion, or
> `${BASH_VERSINFO[0]} >= 4` features will fail under the default macOS
> shell. Use the `env` shebang and require users to install a modern
> Bash via Homebrew (`brew install bash`), or add a version guard at the
> top of the script (see 16.1).

```bash
declare -A colours=(
    [error]="red"
    [warning]="yellow"
    [info]="green"
)

echo "Errors are shown in ${colours[error]}"
```

### 6.4 Avoid Global Variables

Minimise global mutable state. Pass data through function arguments and
return values (stdout). If a global is necessary, declare it with `readonly`
at the top of the script.

---

## 7. Quoting & Word Splitting

### 7.1 The Cardinal Rule

> **Always double-quote variable expansions and command substitutions.**

```bash
# Yes
echo "${name}"
cp "${source}" "${dest}"
result="$(some_command "${arg}")"

# No — word splitting and globbing
echo $name
cp $source $dest
result=$(some_command $arg)
```

### 7.2 When Not to Quote

The **only** exceptions:

- Integer arithmetic inside `(( ))`: `(( count++ ))`.
- Assignments where the right side is a simple literal: `x=hello` (but
  quoting is never wrong: `x="hello"`).
- Inside `[[ ]]` on the left side of `=~` (the regex must be unquoted).

### 7.3 Quoting Types (*ABSG* Ch. 5)

| Quote | Behaviour | Use for |
|---|---|---|
| `"double"` | Expands `$variables`, `$(commands)`, `\escapes` | **Default — use this** |
| `'single'` | Literal — no expansion | Fixed strings with `$`, backticks |
| `$'...'` | ANSI-C quoting (`\n`, `\t`, `\\`) | Strings with special characters |
| `<<'EOF'` | Here-doc without expansion | Template content with literal `$` |

### 7.4 Brace Your Variables

Always use `${var}`, not `$var`, to prevent ambiguity when a variable is
followed by characters that could be part of an identifier:

```bash
# Ambiguous — Bash tries to expand $filename, not $file followed by "name"
echo "$file" "name"        # works, but two arguments
echo "$filename_backup"    # expands $filename_backup, probably unintended

# Clear — braces delimit the variable name
echo "${file}name"
echo "${filename}_backup"
```

---

## 8. Control Flow

### 8.1 Conditionals

Use `[[ ]]` (Bash built-in), not `[ ]` or `test`:

```bash
# Yes — [[ ]] supports pattern matching, &&, ||, regex
if [[ -f "${file}" && "${file}" == *.txt ]]; then
    process "${file}"
fi

# No — [ ] requires quoting gymnastics and has no &&
if [ -f "$file" ] && [ "$file" = "*.txt" ]; then  # broken glob
    process "$file"
fi
```

### 8.2 Common Test Operators (*ABSG* Ch. 7)

| Test | Meaning |
|---|---|
| `-f file` | File exists and is a regular file |
| `-d dir` | Directory exists |
| `-e path` | Path exists (any type) |
| `-r file` | File is readable |
| `-w file` | File is writable |
| `-x file` | File is executable |
| `-z string` | String is empty |
| `-n string` | String is non-empty |
| `str1 == str2` | String equality (inside `[[ ]]`) |
| `str1 =~ regex` | Regex match (inside `[[ ]]`) |
| `-eq`, `-ne`, `-lt`, `-gt` | Integer comparison |

### 8.3 `case` Statements

Use `case` instead of long `if`/`elif` chains:

```bash
case "${action}" in
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
        echo "Unknown action: ${action}" >&2
        exit 1
        ;;
esac
```

### 8.4 Loops

```bash
# Iterate over array elements
for file in "${files[@]}"; do
    process "${file}"
done

# C-style for loop
for (( i = 0; i < count; i++ )); do
    echo "Item ${i}"
done

# Process lines from a file or command
while IFS= read -r line; do
    echo "${line}"
done < "${input_file}"
```

- **Never use `for line in $(cat file)`** — it breaks on whitespace and
  performs globbing.
- Use `while IFS= read -r line` for safe line-by-line processing.

### 8.5 Arithmetic

Use `(( ))` for arithmetic, not `expr` or `let`:

```bash
(( count++ ))
(( total = a + b * c ))

if (( count > 10 )); then
    echo "Limit exceeded"
fi
```

---

## 9. Error Handling

### 9.1 Strict Mode Recap

`set -euo pipefail` is the foundation. Beyond that:

### 9.2 Trap for Cleanup

Use `trap` to ensure cleanup on exit, error, or signal — Bash's equivalent
of context managers / `defer`:

```bash
tmp_file=""  # initialise so the trap can run even if mktemp fails

cleanup() {
    # Use an `if` block, not `[[ ]] && rm`. The short-circuit form returns
    # the exit status of the last command in the chain; if the condition is
    # false, the function exits non-zero, which under `set -e` can corrupt
    # the script's final exit code when the trap runs on success.
    if [[ -n "${tmp_file}" && -f "${tmp_file}" ]]; then
        rm -f "${tmp_file}"
    fi
    echo "Cleaned up" >&2
}
trap cleanup EXIT

tmp_file="$(mktemp)"
# tmp_file is guaranteed to be cleaned up on any exit path
```

Order matters: declare `tmp_file=""` *before* installing the trap so the
trap handler can safely reference it under `set -u`, then assign the
real value after the trap is in place.

- `trap ... EXIT` runs on **any** exit (success, error, signal).
- `trap ... ERR` runs only on errors (useful for logging).
- `trap ... INT TERM` catches Ctrl-C and kill signals.

Stack multiple concerns:

```bash
tmp_dir="$(mktemp -d)"
trap 'rm -rf "${tmp_dir}"' EXIT
```

### 9.3 Error Messages

Write errors to **stderr**, not stdout:

```bash
die() {
    echo "ERROR: $*" >&2
    exit 1
}

[[ -f "${config_file}" ]] || die "Config file not found: ${config_file}"
```

### 9.4 Checking Command Availability

```bash
require_command() {
    command -v "$1" >/dev/null 2>&1 || die "Required command not found: $1"
}

require_command curl
require_command jq
```

### 9.5 Handling Expected Failures

When a command is **expected** to fail (e.g. checking if a process is
running), explicitly handle it so `set -e` does not exit:

```bash
# Option 1 — || true
grep -q "pattern" file.txt || true

# Option 2 — if block
if ! grep -q "pattern" file.txt; then
    echo "Pattern not found, continuing..."
fi

# Option 3 — explicit if/else. Prefer this over `cmd && success || fail`,
# because if `success` itself returns non-zero the `fail` branch also
# runs (the classic "ternary trap" in shell).
if pgrep -x myprocess >/dev/null 2>&1; then
    echo "Running"
else
    echo "Not running"
fi
```

---

## 10. Input / Output & Redirection

### 10.1 Redirection Basics (*ABSG* Ch. 20)

| Syntax | Meaning |
|---|---|
| `> file` | Redirect stdout, truncate |
| `>> file` | Redirect stdout, append |
| `2> file` | Redirect stderr |
| `2>&1` | Redirect stderr to stdout |
| `&> file` | Redirect both stdout and stderr (Bash) |
| `< file` | Redirect stdin from file |
| `<<EOF` | Here-document |
| `<<<string` | Here-string |

### 10.2 Here-Documents

Use here-documents for multi-line output or input:

```bash
cat <<EOF
Usage: ${SCRIPT_NAME} <command>

Commands:
    deploy    Deploy the application
    rollback  Roll back to the previous version
EOF
```

Use `<<-EOF` (with tab indentation) or `<<'EOF'` (no variable expansion) as
needed.

### 10.3 Pipes

- **Keep pipelines readable.** Break long pipelines across lines:

```bash
find "${src_dir}" -name "*.log" -mtime +30 \
    | sort \
    | while IFS= read -r file; do
        archive "${file}"
    done
```

- Remember that each pipeline stage runs in a **subshell** — variable
  assignments inside a pipe do not affect the parent shell. Use process
  substitution (`< <(command)`) to avoid this.

### 10.4 Process Substitution (*ABSG* Ch. 23)

```bash
# Compare two command outputs
diff <(sort file1.txt) <(sort file2.txt)

# Read lines without a subshell
count=0  # initialise before the loop — (( count++ )) returns 1 the first
         # time count is unset, which aborts the script under `set -e`.
while IFS= read -r line; do
    (( ++count ))  # pre-increment so the arithmetic result is non-zero
done < <(find . -name "*.txt")
```

### 10.5 Temporary Files

Always use `mktemp` and clean up with `trap`:

```bash
tmp_file="$(mktemp)"
trap 'rm -f "${tmp_file}"' EXIT

curl -sS "${url}" > "${tmp_file}"
process "${tmp_file}"
```

Never use predictable temporary file paths (security risk — symlink attacks).

### 10.6 Data Passing & Structured Output

Bash passes **text** through pipes, not typed objects. This is the opposite
of PowerShell and has important implications for how data flows between
commands.

#### Use Structured Formats for Machine Consumption

When output will be consumed by another script or program, use a structured
format rather than free-form text:

```bash
# Good — JSON output, parseable by jq
get_service_status() {
  local name="$1"
  local status
  status="$(systemctl is-active "${name}" 2>/dev/null || echo "unknown")"

  printf '{"name":"%s","status":"%s","host":"%s"}\n' \
      "${name}" "${status}" "$(hostname)"
}

# Good — TSV for simple tabular data (GNU find; on BSD use stat -f)
list_large_files() {
  local dir="$1"
  find "${dir}" -type f -size +100M -printf '%s\t%p\n' \
      | sort -rn
}
```

#### Prefer `jq` for JSON Processing

Use `jq` for parsing and manipulating JSON — never `grep`, `sed`, or `awk`
on JSON data. JSON's structure (nested objects, escaping, multiline values)
makes regex-based parsing unreliable:

```bash
# Good — jq understands JSON structure
api_response="$(curl -sS "${api_url}")"
name="$(echo "${api_response}" | jq -r '.user.name')"
count="$(echo "${api_response}" | jq '.items | length')"

# Bad — fragile regex on JSON
name="$(echo "${api_response}" | grep -o '"name":"[^"]*"' | cut -d'"' -f4)"
```

#### Parse External Command Output Carefully

Do not assume the format of external command output is stable across
versions or platforms. When parsing output from tools like `ps`, `df`, or
`ip`, use well-defined output flags where available, and document format
assumptions:

```bash
# Good — use -o for explicit column selection
pids="$(ps -eo pid= -o comm= | awk '$2 == "nginx" {print $1}')"

# Bad — relies on default column positions that vary across systems
pids="$(ps aux | grep nginx | awk '{print $2}')"
```

#### Clear Variable Contracts Between Functions

When passing data between functions, use clear naming conventions and
document expected formats. Functions return data via stdout — callers
capture it with command substitution:

```bash
# Function documents its output format (GNU coreutils; BSD df differs)
get_disk_usage() {
  # Output: one line per mount, format: "usage_pct<TAB>mount_point"
  df --output=pcent,target | tail -n +2 | sed 's/^ *//'
}

# Caller knows the contract
while IFS=$'\t' read -r pct mount; do
  if (( ${pct%\%} > 90 )); then
    echo "WARNING: ${mount} at ${pct}" >&2
  fi
done < <(get_disk_usage)
```

---

## 11. Process Management

### 11.1 Background Processes

```bash
long_running_task &
pid=$!

# Wait for completion
wait "${pid}"
echo "Task finished with exit code $?"
```

### 11.2 Parallel Execution

Use `xargs -P` or GNU `parallel` for controlled concurrency:

```bash
# Process 4 files at a time
find . -name "*.gz" -print0 \
    | xargs -0 -P 4 -I{} gunzip {}
```

### 11.3 Signal Handling (*ABSG* Ch. 32)

```bash
shutdown() {
    echo "Shutting down gracefully..." >&2
    # Kill background jobs. `jobs -p` may be empty; collect into an array
    # so we do not call `kill` with no arguments (a usage error on most
    # systems). The unquoted expansion below is intentional — we want
    # word splitting on each PID.
    local pids
    pids=$(jobs -p)
    if [[ -n "${pids}" ]]; then
        kill ${pids} 2>/dev/null || true
    fi
    exit 0
}

# INT and TERM are the standard shutdown signals; do NOT trap KILL (9) or
# STOP (17) — they cannot be trapped by design.
trap shutdown INT TERM
```

### 11.4 Subshells

Be aware that parentheses `( )` create a subshell — variable changes inside
do not propagate:

```bash
x=1
( x=2; echo "${x}" )  # prints 2
echo "${x}"            # prints 1 — subshell change is lost
```

Use `{ }` for grouping without a subshell when you need side effects.

---

## 12. Testing & Validation

### 12.1 ShellCheck

**Every script must pass ShellCheck.** It is a comprehensive static analyzer
that catches quoting, globbing, command substitution, and logic bugs (Bash has
no type system to check):

```bash
shellcheck my_script.sh
```

- Fix every finding. If a suppression is necessary, annotate with a
  comment explaining why: `# shellcheck disable=SC2086`.
- Run ShellCheck in CI on every PR.

### 12.2 Testing Frameworks

Use **Bats** (Bash Automated Testing System) for unit and integration tests:

```bash
#!/usr/bin/env bats

@test "greet returns hello message" {
    result="$(./greet.sh "World")"
    [[ "${result}" == "Hello, World!" ]]
}

@test "deploy fails without target argument" {
    run ./deploy.sh
    [[ "${status}" -ne 0 ]]
    [[ "${output}" == *"Usage"* ]]
}
```

### 12.3 Testing Practices

- Test the **script's public interface** — its arguments, stdout, stderr, and
  exit code.
- Use `setup()` and `teardown()` in Bats for fixture management.
- Test **edge cases**: empty input, missing files, permission errors.
- Use `tmp_dir` fixtures (via `mktemp -d`) to isolate filesystem tests.

### 12.4 Dry-Run Mode

For destructive scripts, implement a `--dry-run` flag:

```bash
DRY_RUN="${DRY_RUN:-false}"

run_cmd() {
    if [[ "${DRY_RUN}" == "true" ]]; then
        echo "[DRY RUN] $*" >&2
    else
        "$@"
    fi
}

run_cmd rm -rf "${old_backups}"
```

---

## 13. Performance & Idiomatic Bash

### 13.1 Prefer Built-ins Over External Commands (*ABSG* Ch. 15)

Every external command forks a new process. For operations on small data,
use Bash built-ins:

```bash
# Yes — parameter expansion (no fork)
filename="${path##*/}"           # basename
dirname="${path%/*}"             # dirname
lower="${string,,}"              # lowercase (Bash 4+)
upper="${string^^}"              # uppercase (Bash 4+)
trimmed="${string#"${string%%[![:space:]]*}"}"  # leading whitespace

# No — external commands for trivial operations
filename="$(basename "${path}")"
dirname="$(dirname "${path}")"
lower="$(echo "${string}" | tr '[:upper:]' '[:lower:]')"
```

### 13.2 Parameter Expansion (*ABSG* Ch. 10)

| Expansion | Result |
|---|---|
| `${var:-default}` | Use `default` if `var` is unset or empty |
| `${var:=default}` | Assign `default` if `var` is unset or empty |
| `${var:?error}` | Exit with `error` if `var` is unset or empty |
| `${var:+alt}` | Use `alt` if `var` is set and non-empty |
| `${#var}` | Length of `var` |
| `${var#pattern}` | Remove shortest prefix match |
| `${var##pattern}` | Remove longest prefix match |
| `${var%pattern}` | Remove shortest suffix match |
| `${var%%pattern}` | Remove longest suffix match |
| `${var/old/new}` | Replace first match |
| `${var//old/new}` | Replace all matches |
| `${var^}` | Uppercase first character (Bash 4+) |
| `${var^^}` | Uppercase all (Bash 4+) |
| `${var,}` | Lowercase first character (Bash 4+) |
| `${var,,}` | Lowercase all (Bash 4+) |

### 13.3 Avoid Useless Use of `cat`

```bash
# Yes — redirect directly
while IFS= read -r line; do
    echo "${line}"
done < file.txt

grep "pattern" < file.txt

# No — unnecessary cat
cat file.txt | while IFS= read -r line; do
    echo "${line}"
done

cat file.txt | grep "pattern"
```

### 13.4 Avoid Useless Use of `echo` with `grep`

```bash
# Yes — use [[ ]] or case for string matching
if [[ "${string}" == *pattern* ]]; then
    echo "Match"
fi

# No — spawning two processes for a string check
if echo "${string}" | grep -q "pattern"; then
    echo "Match"
fi
```

### 13.5 Brace Expansion

Use brace expansion for generating sequences:

```bash
mkdir -p project/{src,tests,docs}
touch file_{01..10}.txt
cp config.yml{,.bak}
```

### 13.6 Command Grouping

Use `{ }` to group commands without a subshell:

```bash
{
    echo "Header"
    process_data
    echo "Footer"
} > output.txt
```

---

## 14. Security

### 14.1 Input Validation

**Never trust user input.** Validate and sanitise all external input:

```bash
# Validate that input is a number
if [[ ! "${port}" =~ ^[0-9]+$ ]]; then
    die "Invalid port: ${port}"
fi

# Validate that input is a known value
case "${env}" in
    staging|production) ;;
    *) die "Invalid environment: ${env}" ;;
esac
```

### 14.2 Avoid `eval`

`eval` is almost always a security vulnerability. It executes arbitrary code:

```bash
# NEVER do this — code injection
eval "${user_input}"

# If you absolutely must construct commands dynamically, use arrays:
cmd=(curl --silent --fail "${url}")
"${cmd[@]}"
```

### 14.3 Temporary Files

- Always use `mktemp`, never hardcoded paths in `/tmp`.
- Set restrictive permissions: `umask 077` or `mktemp` (which defaults to
  `0600`).
- Clean up with `trap ... EXIT`.

### 14.4 Credential Handling

- **Never hardcode secrets** in scripts.
- Read from environment variables or files with restrictive permissions.
- Use `read -rs` for interactive password input (no echo).
- Clear sensitive variables when done: `unset password`.

### 14.5 Avoid Parsing `ls`

`ls` output is not safe to parse — filenames can contain newlines, spaces,
and special characters:

```bash
# Yes — use globs or find
for file in "${dir}"/*.txt; do
    [[ -f "${file}" ]] || continue
    process "${file}"
done

# No — breaks on spaces, newlines, special chars
for file in $(ls "${dir}"); do
    process "${file}"
done
```

### 14.6 Command Injection

Shell injection is Bash's equivalent of SQL injection. It occurs when
unvalidated input is interpolated into commands, allowing an attacker to
execute arbitrary code.

#### Quote All Variables in Commands

Unquoted variables undergo word splitting and globbing, which can alter
command behaviour in dangerous ways:

```bash
# DANGEROUS — if filename contains spaces or glob chars
rm ${user_file}

# SAFE — quoted
rm "${user_file}"
```

#### Never Use `eval` or Backticks with User Input

`eval` executes arbitrary strings as code. Backtick command substitution
(`` `...` ``) is harder to read and nest. Use arrays for dynamic command
construction:

```bash
# BAD — command injection via eval
eval "grep '${user_pattern}' ${user_file}"

# GOOD — arrays prevent injection
cmd=(grep -- "${user_pattern}" "${user_file}")
"${cmd[@]}"
```

#### Use `--` to Separate Options from Arguments

When passing user-supplied values to commands, use `--` to prevent the
values from being interpreted as flags:

```bash
# User input starting with "-" won't be treated as a flag
grep -- "${pattern}" "${file}"
rm -- "${filename}"
```

---

## 15. Defensive Programming & Input Validation

Defensive programming assumes that all external input — script arguments,
file content, environment variables, command output — is potentially
malicious or malformed. Validate early, validate thoroughly, and fail
loudly on invalid data.

### 15.1 Always Quote Variables

The single most important defensive practice in Bash. Unquoted variables
are subject to word splitting and pathname expansion, which causes subtle
bugs and security vulnerabilities:

```bash
# Always
echo "${var}"
cp "${source}" "${dest}"

# Never
echo $var
cp $source $dest
```

### 15.2 Validate All Script Arguments

Check argument count and values at the top of `main()`:

```bash
main() {
  if [[ $# -lt 2 ]]; then
    echo "Usage: ${SCRIPT_NAME} <host> <port>" >&2
    exit 1
  fi

  local host="$1"
  local port="$2"

  validate_host "${host}"
  validate_port "${port}"

  connect "${host}" "${port}"
}
```

### 15.3 Validate Types and Ranges

Bash has no type system, so validate manually with pattern matching and
arithmetic comparisons:

```bash
# Validate integer
if [[ ! "${port}" =~ ^[0-9]+$ ]]; then
  die "Port must be a number: ${port}"
fi

# Validate range
if [[ "${port}" -lt 1 || "${port}" -gt 65535 ]]; then
  die "Port out of range (1-65535): ${port}"
fi

# Validate string length
if [[ ${#username} -gt 255 ]]; then
  die "Username too long (max 255 characters)"
fi
if [[ -z "${username}" ]]; then
  die "Username cannot be empty"
fi
```

### 15.4 Validate File Existence and Permissions

Check that files and directories exist and are accessible before operating
on them:

```bash
[[ -f "${config_file}" ]] || die "Config file not found: ${config_file}"
[[ -r "${config_file}" ]] || die "Config file not readable: ${config_file}"
[[ -d "${output_dir}" ]] || die "Output directory not found: ${output_dir}"
[[ -w "${output_dir}" ]] || die "Output directory not writable: ${output_dir}"
```

### 15.5 Use Strict Mode

`set -euo pipefail` is the foundation of defensive Bash scripting:

- `-e` — exit on any command failure.
- `-u` — treat unset variables as errors.
- `-o pipefail` — a pipeline fails if any stage fails.

### 15.6 Never Use `eval` with User Input

`eval` executes arbitrary strings as shell commands. It is almost never
necessary and always dangerous with external input:

```bash
# NEVER
eval "${user_command}"

# Use arrays for dynamic commands
cmd=(curl --silent --fail "${url}")
"${cmd[@]}"
```

### 15.7 Never Pass Unvalidated Input to Destructive Commands

Validate all input before passing it to `rm`, `mv`, `chmod`, or other
commands that modify the filesystem:

```bash
# Validate before deleting
if [[ -z "${target_dir}" ]]; then
  die "Target directory is empty — refusing to delete"
fi

if [[ "${target_dir}" == "/" ]]; then
  die "Refusing to delete root directory"
fi

rm -rf "${target_dir}"
```

### 15.8 Sanitise Input for Paths and Commands

Strip or reject characters that have special meaning in file paths or
shell syntax:

```bash
# Remove path traversal sequences
clean_name="${user_input//\.\./}"

# Reject input containing shell metacharacters
if [[ "${user_input}" =~ [[:space:]\;\|\&\$\`\\] ]]; then
  die "Input contains invalid characters: ${user_input}"
fi
```

### 15.9 Use `--` to Separate Options from Arguments

When passing user input to commands, use `--` to prevent values starting
with `-` from being interpreted as flags:

```bash
grep -- "${user_pattern}" "${file}"
rm -- "${user_filename}"
```

---

## 16. Portability

### 16.1 Bash Version Awareness

| Feature | Minimum Bash |
|---|---|
| Associative arrays (`declare -A`) | 4.0 |
| `${var,,}` / `${var^^}` case conversion | 4.0 |
| `|&` (pipe stderr) | 4.0 |
| `coproc` | 4.0 |
| Negative array indices | 4.2 |
| `${var@Q}` (quote for re-input) | 4.4 |
| `${var@a}` (attribute query) | 4.4 |
| Nameref (`declare -n`) | 4.3 |
| `BASH_ARGV0` | 5.0 |
| `wait -n` (any child) | 4.3 |

- Document the minimum Bash version at the top of the script.
- Check the version at runtime if using features that may be missing:

```bash
if (( BASH_VERSINFO[0] < 4 )); then
    die "This script requires Bash 4.0+"
fi
```

### 16.2 GNU vs. BSD Utilities

macOS ships BSD `sed`, `grep`, `date`, etc., which differ from GNU versions.
For portable scripts:

- Use `sed -i.bak` instead of `sed -i` (BSD requires a backup extension).
- Use `date -u +%s` (portable) instead of `date -d` (GNU-only).
- `find -printf` is GNU-only — not available on BSD `find`. Use `-exec`
  with explicit formatting for portability.
- `find -regex` syntax differs between GNU and BSD (GNU uses Emacs regex
  by default; BSD uses basic regex).
- Document which platform the script targets, or test for the variant:

```bash
if date --version &>/dev/null; then
    # GNU date
    yesterday="$(date -d 'yesterday' +%Y-%m-%d)"
else
    # BSD date (macOS)
    yesterday="$(date -v-1d +%Y-%m-%d)"
fi
```

### 16.3 POSIX Compatibility

If the script must run on `/bin/sh` (not Bash):

- No `[[ ]]`, no `(( ))`, no arrays, no process substitution.
- Use `[ ]` / `test`, `expr`, and POSIX parameter expansion only.
- Set the shebang to `#!/bin/sh`.
- Generally, prefer writing Bash-specific scripts and documenting the
  requirement rather than writing lowest-common-denominator POSIX sh.

---

## 17. Tooling

### 17.1 Recommended Tool Chain

| Purpose | Tool | Notes |
|---|---|---|
| Linter | **ShellCheck** | Non-negotiable — catches quoting, globbing, and logic bugs |
| Formatter | **shfmt** | Consistent formatting (`shfmt -i 2 -ci`) |
| Test framework | **Bats** (bats-core) | Bash unit/integration testing |
| Static analysis | **ShellCheck** + **bashate** | bashate for style checks (OpenStack style) |
| Debugging | `set -x` / `PS4='+(${BASH_SOURCE}:${LINENO}): '` | Trace execution with file/line info |
| Profiling | `time`, `bash -x` with timestamps | Identify slow commands |

### 17.2 Debugging

Enable tracing with `set -x` and a detailed prompt:

```bash
export PS4='+(${BASH_SOURCE[0]}:${LINENO}): ${FUNCNAME[0]:+${FUNCNAME[0]}(): }'
set -x
```

Use `trap 'echo "Line ${LINENO}: exit ${?}" >&2' ERR` for error-line
reporting.

### 17.3 CI Checks

```bash
# Lint all shell scripts — use find so we do not depend on `shopt -s
# globstar`, and so the script works even when no .sh files exist at the
# top level.
find . -type f -name '*.sh' -print0 | xargs -0 shellcheck

# Format check
shfmt -d -i 2 -ci .

# Run tests
bats tests/
```

---

## 18. SBOM Creation & Dependency Management

### 18.1 What is an SBOM?

Bash scripts may have system-level dependencies (binaries, libraries) rather than package-manager dependencies. Document these explicitly.

### 18.2 Documenting External Dependencies

In comments at the top of the script, list all required commands/binaries:
```bash
#!/usr/bin/env bash
# External dependencies:
#   - curl (for HTTP requests)
#   - jq (for JSON parsing)
#   - git (for VCS operations)

set -euo pipefail

command -v curl &>/dev/null || { echo "curl required"; exit 1; }
command -v jq &>/dev/null || { echo "jq required"; exit 1; }
```

### 18.3 Generating Dependency Lists

For scripts using system packages (apt, brew, yum):
```bash
# Document in comment block or separate file
# DEPENDENCIES.txt:
# curl
# jq
# git
```

For container-based scripts, use Dockerfile to document:
```dockerfile
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y \
    curl \
    jq \
    git
```

### 18.4 Vulnerability Scanning for System Dependencies

If using Docker containers, scan images:
```bash
# Using Trivy (container image scanner)
trivy image ubuntu:22.04
```

### 18.5 License Compliance

Bash scripts themselves need license headers (e.g., MIT, Apache-2.0):
```bash
#!/usr/bin/env bash
# Copyright (c) 2025 YourName
# Licensed under MIT License
# https://opensource.org/licenses/MIT
```

Document licenses of dependencies in a LICENSE or COPYING file.

### 18.6 Best Practice: Pin Versions

```bash
# Instead of relying on "latest", pin versions:
# requirements-ubuntu.txt
curl:8.0.0
jq:1.6
git:2.34
```

Or use container images with pinned OS version:
```dockerfile
FROM ubuntu:22.04  # Specific version, not "latest"
```

---

## 19. References

### Official & Community Documentation

| Resource | URL |
|---|---|
| GNU Bash Reference Manual | https://www.gnu.org/software/bash/manual/ |
| *Advanced Bash-Scripting Guide* | https://tldp.org/LDP/abs/html/ |
| Google Shell Style Guide | https://google.github.io/styleguide/shellguide.html |
| ShellCheck Wiki | https://www.shellcheck.net/wiki/ |
| Bash Hackers Wiki | https://wiki.bash-hackers.org |
| POSIX Shell Specification | https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html |
| Bats (Testing Framework) | https://github.com/bats-core/bats-core |
| shfmt (Formatter) | https://github.com/mvdan/sh |

### Books

| Book | Authors | Key Takeaways for This Guide |
|---|---|---|
| *Advanced Bash-Scripting Guide* | Mendel Cooper | Comprehensive Bash reference: arrays, parameter expansion, process substitution, signal handling, regular expressions. |
| *The Linux Command Line* | William Shotts (2012, 2019) | Practical introduction to shell scripting, pipelines, and text processing. |
| *Classic Shell Scripting* | Robbins & Beebe (2005) | Portable shell techniques, text processing with `sed`/`awk`, best practices. |
| *bash Cookbook* | Albing, Vossen & Newham (2007, 2017) | Recipe-driven solutions for common scripting tasks. |
| *Learning the bash Shell* | Newham & Rosenblatt (2005) | Thorough coverage of Bash syntax, job control, and customisation. |
| *Clean Code: A Handbook of Agile Software Craftsmanship* | Robert C. Martin (2008) | Small functions, meaningful names, SRP — applies to scripts too. |
