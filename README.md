# Coding Style Guidelines & AI Rules

Comprehensive, language-specific **coding style guides** and **AI-optimized rule files** grounded in official language references, community conventions, and established software engineering literature (GoF *Design Patterns*, *Clean Code*, *The Pragmatic Programmer*).

## Quick Index

- **[Using These Guides](#using-these-guides)** — Best practices by use case
- **[What Each Guide Covers](#what-each-guide-covers)** — Overview of guide structure and topics
- **[Guides](#guides)** — Full comprehensive guides for all 16 languages
- **[AI-Assisted Development](#ai-assisted-development)** — AI rule files and how to use them
- **[Project Organization](#project-organization)** — Folder structure and file layout

---

## Using These Guides

### For Code Review & Team Standards
- Use the full **style guide** as the authoritative reference for your language
- Link to it in your project's CONTRIBUTING.md or README
- Use sections on Testing, Database ACID, Error Handling, and Defensive Programming as code review checklists

### For AI-Assisted Development
- Copy the **`ai_rules_*` file** into your AI tool's context (Claude Code, Cursor, VS Code extensions)
- Or reference it in project instructions (`.cursorrules`, `CLAUDE.md`, `.github/copilot-instructions.md`)
- The concise format (~200 lines) fits efficiently in AI context windows

### For Security & Compliance
- Reference the **Defensive Programming** and **Database / ACID** sections for input validation and SQL injection prevention
- Use **SBOM Creation** sections for dependency documentation and vulnerability scanning workflows
- Consult **Security** sections (HTML/CSS) for XSS prevention and safe API usage

### For Onboarding
- New team members can start with the **Philosophy** section to understand language values
- Reference **Naming** and **Formatting** for immediate style consistency
- Use **Testing** and **Error Handling** sections to learn the team's patterns

---

## What Each Guide Covers

Every guide follows a consistent structure adapted to the language:

### Core Concepts
- **Philosophy** — language-specific principles and values
- **Formatting** — indentation, line length, blank lines (per official style)
- **Naming** — conventions for all language entities
- **Functions / Methods** — size, parameters, return values
- **Types & composition** — structs/classes/traits/interfaces, SOLID principles
- **Documentation** — doc-comment format (rST, GoDoc, rustdoc, YARD, TSDoc)
- **Error handling** — idiomatic error patterns per language
- **Imports / dependencies** — ordering, no redundant modules for the same task
- **Design patterns** — GoF patterns adapted to each language's paradigm
- **Testing** — naming, structure, mocking guidelines

### Production & Security
- **Database / ACID** — transaction discipline, isolation levels, parameterised queries, **SQL injection protection** (all except Bash/HTML)
- **Defensive programming & input validation** — boundary validation, type/length/range checks, dangerous operations, sanitisation for all external input
- **Security** — SQL injection prevention, XSS prevention (HTML/CSS), command injection avoidance, safe API usage

### Performance & Deployment
- **Concurrency** — language-appropriate concurrency models (goroutines, async/await, actors, threads, etc.)
- **Performance & idioms** — language-specific best practices, copy semantics, idiomatic constructs
- **Build tools** — language-specific build systems (Maven/Gradle, Cargo, npm, Swift PM, Poetry, etc.) — *all except Bash/PowerShell*
- **Project structure** — recommended layouts, dependency management, package organization
- **SBOM creation** — documenting dependencies, vulnerability scanning, license compliance
- **Tooling** — formatter, linter, test runner, dependency checker, CI checks

### Data Flow & Consistency
- **Data passing** — pass typed/structured objects (not raw dicts/strings), parse external data at boundaries, maintain consistency across modules

### Reference
- **References** — official language docs, RFCs/PEPs, seminal books

---

## Guides

| File | Language | Key References |
|---|---|---|
| [python_style_guide.md](python/python_style_guide.md) | Python | PEP 8, PEP 20 (Zen of Python), PEP 257, PEP 484 |
| [golang_style_guide.md](golang/golang_style_guide.md) | Go | Effective Go, Go Proverbs, Go Code Review Comments |
| [rust_style_guide.md](rust/rust_style_guide.md) | Rust | The Rust Book, The Rustonomicon, Rust API Guidelines |
| [rust/cargo_toml_reference.md](rust/cargo_toml_reference.md) | Rust (Cargo.toml) | Official Cargo Book, Cargo.toml manifest reference |
| [ruby_style_guide.md](ruby/ruby_style_guide.md) | Ruby | Ruby Style Guide, POODR (Sandi Metz), The Well-Grounded Rubyist |
| [typescript_style_guide.md](typescript/typescript_style_guide.md) | TypeScript | TypeScript Handbook, Google TS Style Guide, Effective TypeScript |
| [bash_style_guide.md](bash/bash_style_guide.md) | Bash | Advanced Bash-Scripting Guide, Google Shell Style Guide, ShellCheck |
| [zsh_style_guide.md](zsh/zsh_style_guide.md) | Zsh | Zsh Manual, Google Shell Style Guide, ShellCheck |
| [posix_style_guide.md](posix/posix_style_guide.md) | POSIX Shell | IEEE Std 1003.1, Shell & Utilities, Google Shell Style Guide |
| [batch_style_guide.md](batch/batch_style_guide.md) | Windows Batch | Microsoft Batch Docs, SS64 Batch Reference, RobvanderWoude Batch Guide |
| [powershell_style_guide.md](powershell/powershell_style_guide.md) | PowerShell | PowerShell Practice and Style Guide, about_Functions_Advanced, Pester |
| [java_style_guide.md](java/java_style_guide.md) | Java | Google Java Style Guide, Effective Java, JLS |
| [swift_style_guide.md](swift/swift_style_guide.md) | Swift | Swift API Design Guidelines, The Swift Programming Language |
| [perl5_style_guide.md](perl5/perl5_style_guide.md) | Perl 5 | perlstyle, Perl Best Practices (Conway), Modern Perl |
| [perl6_style_guide.md](perl6/perl6_style_guide.md) | Raku (Perl 6) | Raku Documentation, Think Raku, Perl 6 Deep Dive |
| [csharp_style_guide.md](csharp/csharp_style_guide.md) | C# | Microsoft C# Coding Conventions, .NET Framework Design Guidelines, C# in Depth |
| [html_css_style_guide.md](html_css/html_css_style_guide.md) | HTML5 / CSS | WHATWG HTML Living Standard, W3C CSS, Google HTML/CSS Style Guide, WCAG 2.1 |

## AI-Assisted Development

When using AI tools (code assistants, chat-based code generation, AI-powered IDEs),
feed the corresponding style guide for the language you are working in as context.
This ensures generated code follows the same conventions, patterns, and best
practices defined in these guides. If the AI supports project-level instructions
(e.g. `CLAUDE.md`, `.cursorrules`, `.github/copilot-instructions.md`), reference
or include the relevant style guide there.

### AI Rule Files

Each guide has a condensed, AI-optimised rule file — concise directives without
prose or examples, designed for direct consumption by AI code assistants:

| AI Rule File | Language |
|---|---|
| [ai_rules_python.md](python/ai_rules_python.md) | Python |
| [ai_rules_golang.md](golang/ai_rules_golang.md) | Go |
| [ai_rules_rust.md](rust/ai_rules_rust.md) | Rust |
| [ai_cargo_toml_instructions.md](rust/ai_cargo_toml_instructions.md) | Rust (Cargo.toml) |
| [ai_rules_ruby.md](ruby/ai_rules_ruby.md) | Ruby |
| [ai_rules_typescript.md](typescript/ai_rules_typescript.md) | TypeScript |
| [ai_rules_bash.md](bash/ai_rules_bash.md) | Bash |
| [ai_rules_zsh.md](zsh/ai_rules_zsh.md) | Zsh |
| [ai_rules_posix.md](posix/ai_rules_posix.md) | POSIX Shell |
| [ai_rules_batch.md](batch/ai_rules_batch.md) | Windows Batch |
| [ai_rules_powershell.md](powershell/ai_rules_powershell.md) | PowerShell |
| [ai_rules_java.md](java/ai_rules_java.md) | Java |
| [ai_rules_swift.md](swift/ai_rules_swift.md) | Swift |
| [ai_rules_perl5.md](perl5/ai_rules_perl5.md) | Perl 5 |
| [ai_rules_perl6.md](perl6/ai_rules_perl6.md) | Raku (Perl 6) |
| [ai_rules_csharp.md](csharp/ai_rules_csharp.md) | C# |
| [ai_rules_html_css.md](html_css/ai_rules_html_css.md) | HTML5 / CSS |

Use the full style guide as a reference for humans; use the `ai_rules_*` file
as context input for your AI tool of choice.

---

## Project Organization

All 16 language guides are organized in dedicated subfolders for easy navigation:

```
coding_style_rules/
├── README.md
├── python/
│   ├── python_style_guide.md
│   └── ai_rules_python.md
├── golang/
├── rust/
├── ruby/
├── typescript/
├── bash/
├── zsh/
├── posix/
├── batch/
├── powershell/
├── java/
├── swift/
├── perl5/
├── perl6/
├── csharp/
└── html_css/
```

Each language folder contains:
- **`<lang>_style_guide.md`** — comprehensive guide for humans (15–20 sections, ~1500–2500 lines)
- **`ai_rules_<lang>.md`** — condensed AI-optimised rules (~200–300 lines)
