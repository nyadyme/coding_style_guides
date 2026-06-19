# Bash — AI Coding Rules

Apply these rules when generating or reviewing Bash scripts.

## When to Use Bash

- Glue code, automation, thin wrappers. Max ~200 lines.
- Beyond that, rewrite in Python or Go.

## Boilerplate (every script)

- Shebang: `#!/usr/bin/env bash` (never `#!/bin/bash`).
- Strict mode: `set -euo pipefail` immediately after shebang.
- Wrap all logic in `main() { ... }` and call `main "$@"` at end of file.
- Header comment: description and usage.

## Formatting

- 2 spaces indentation. Never tabs.
- 80 characters hard line limit. Break with `\` (backslash-newline).
- Indent continuations by 4 spaces.
- Break after: pipes (`|`), logical operators (`&&`, `||`), flags, arguments, redirections.
- Two blank lines between top-level function definitions.
- No blank line at start/end of function body.
- Opening brace on same line: `func() {`. `then`/`do` on same line as `if`/`for`/`while`.
- Configure `shfmt -i 2 -ci`.

## Naming

- `snake_case`: functions, local variables.
- `UPPER_SNAKE_CASE`: globals, constants (with `readonly`).
- Prefix booleans: `is_verbose`, `has_error`.
- Script names: `kebab-case.sh` or `snake_case.sh`, describing the action.

## Variables

- Always `local` inside functions.
- `readonly` for constants.
- Separate `local` declaration from command substitution assignment.
- `declare -a` for arrays, `declare -A` for associative arrays (Bash 4+).
- Minimise globals. Pass data through arguments and stdout.

## Quoting

- **Always double-quote** variable expansions and command substitutions: `"${var}"`, `"$(cmd)"`.
- Exceptions: inside `(( ))`, on left side of `=~` in `[[ ]]`.
- Always brace variables: `${var}`, not `$var`.
- Use arrays instead of space-delimited strings.

## Control Flow

- Use `[[ ]]`, not `[ ]` or `test`.
- `case` instead of long `if`/`elif` chains.
- `(( ))` for arithmetic, not `expr` or `let`.
- `while IFS= read -r line` for line-by-line processing. Never `for line in $(cat file)`.

## Functions

- Small, under 20 lines, one thing.
- `name() { }` form (not `function name { }`).
- Validate args: `${1:?Error: arg required}`, `${2:-default}`.
- Return status via `return 0`/`return 1`. Return data via stdout and command substitution.

## Error Handling

- `set -euo pipefail` is the foundation.
- `trap cleanup EXIT` for guaranteed cleanup.
- Errors to stderr: `echo "ERROR: ..." >&2`.
- `die()` function: `echo "ERROR: $*" >&2; exit 1`.
- `command -v tool >/dev/null 2>&1 || die "Required: tool"` for dependency checks.
- Handle expected failures explicitly: `cmd || true`, `if ! cmd; then ...`.
- Inside trap/cleanup functions, use `if`-blocks, not `[[ ... ]] && cmd` chains — the short-circuit form returns the test's exit status and can corrupt the script's final exit code under `set -e`.
- Traps are NOT inherited by subshells (`( ... )`, command substitutions, pipeline stages) unless you use `set -E` (ERR trap) or re-install them; remember this when wrapping logic in `$(...)` or `( )`.

## I/O & Redirection

- Here-documents for multi-line output.
- Break long pipelines across lines (after `|`).
- Process substitution `< <(cmd)` to avoid subshell variable loss.
- Always `mktemp` for temp files + `trap 'rm -f "${tmp}"' EXIT`.
- Bash passes **text** through pipes, not typed objects — the opposite of PowerShell.
- For machine consumption, use structured output formats (JSON via `jq`, TSV, CSV).
- Prefer `jq` for JSON processing — never `grep`/`sed`/`awk` on JSON.
- Parse external command output carefully — do not assume format stability across versions or platforms.
- When passing data between functions, use clear variable contracts and document expected formats.

## Performance

- Prefer built-ins: parameter expansion over `basename`/`dirname`/`tr`.
- `${var##*/}` (basename), `${var%/*}` (dirname), `${var,,}` (lowercase).
- No useless `cat`: `grep pattern < file`, not `cat file | grep pattern`.
- No `echo "$var" | grep`: use `[[ "$var" == *pattern* ]]`.
- Brace expansion: `mkdir -p project/{src,tests,docs}`.

## Security

- Validate all external input (regex, `case` for allowed values).
- Never `eval`. Use arrays for dynamic commands: `cmd=(curl -sS "$url"); "${cmd[@]}"`.
- Never parse `ls`. Use globs or `find`.
- Never hardcode secrets. Use env vars or restricted files.
- `mktemp` only (never predictable `/tmp/` paths).
- **Command injection:** always quote variables in commands — unquoted variables undergo word splitting and globbing.
- Never use `eval` or backticks with user input.
- Use `--` to separate options from arguments when passing user input to commands.

## Testing

- ShellCheck: non-negotiable, every script must pass.
- Bats for unit/integration tests.
- Test public interface: args, stdout, stderr, exit code.
- `--dry-run` flag for destructive scripts.

## Defensive Programming

- Always quote variables: `"${var}"` not `$var`.
- Validate all script arguments and external input at the top of `main()`.
- Check argument count: `if [[ $# -lt 2 ]]; then ...`.
- Validate types with regex: `[[ "${port}" =~ ^[0-9]+$ ]]`.
- Validate string lengths: `[[ ${#input} -le 255 ]]`.
- Validate ranges: `[[ "${port}" -ge 1 && "${port}" -le 65535 ]]`.
- Validate file existence and permissions before operations: `-f`, `-r`, `-d`, `-w`.
- `set -euo pipefail` for fail-fast behaviour.
- Never use `eval` with user input.
- Never pass unvalidated input to `rm`, `mv`, or other destructive commands.
- Sanitise input used in file paths or command construction.
- Use `--` to separate options from arguments when passing user input to commands.

## Portability

- Document minimum Bash version. Check at runtime if needed.
- GNU vs BSD: `sed -i.bak`, portable `date` forms.
- Prefer Bash-specific scripts with documented requirement over POSIX sh.

## Tooling

- ShellCheck (lint), shfmt (format), Bats (test).
- `set -x` with `PS4='+(${BASH_SOURCE}:${LINENO}): '` for debugging.

## SBOM Creation & Dependency Management

- Document external dependencies in comments at script top.
- Validate required commands exist: `command -v curl &>/dev/null || exit 1`.
- For containers: use Dockerfile to declare and pin dependency versions.
- Scan container images with Trivy for vulnerabilities.
- Include LICENSE headers in scripts.
- Pin versions in requirements/dependencies files, not "latest".
