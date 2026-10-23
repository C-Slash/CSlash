# C/ Language Specification (draft)

## 1. Lexical rules

- System-level directives begin with `#`.
- Statements end with a semicolon (`;`).
- Single (`'`) and double (`"`) quotes are interchangeable, so nested quotes need no escape characters.
- Comments start with `* //` (for example `* // This is a native comment line!`).
- File extension: `.cslash`.

## 2. Parameter chaining

Arguments and properties (such as `from:` and `to:`) can be chained across lines with a comma layout:

```text
#create: "byte_target.txt", to: dir/default;
from: "pr.cache";
```

## 3. Directives

| Directive | Purpose |
|-----------|---------|
| `#include: "file";` | Raw, blind text paste of the file. |
| `#import: "file";` | Loads the file exactly once into the engine scope (idempotent, duplicate-protected). |
| `#create: "name", to: path;` | Creates a resource at the target directory. |
| `#convert: path, to: cslash;` | Ingests data or foreign code, translates it, and restructures it into native C/. |
| `#if: "file" = dir, from: "cache";` | Checks whether the file exists in the given directory (resolved through the `from:` cache). Evaluates to true or false. |
| `then: action` | Runs the action only if the preceding `#if` was true; skipped if false. |
| `#run: "file", then: ...` | Runs a validated `.cslash` file, then continues with the next directive. |
| `#byte_generate: "file";` | Generates a binary output file. |

### 3.1 Conditionals: `#if` / `then:`

`#if` evaluates to true or false. It checks whether a file is present in a directory, and the `from:` parameter tells it which cache to resolve that directory from. `then:` runs only when the check is true.

**Directory resolution:**

- On the Private Runtime (PR), a `pr.cache` is required. It defines the directories, and the set directory is resolved from it.
- On the normal Runtime, no `pr.cache` is needed: it is already configured with `dir/default` set to the Downloads folder.
- Off the PR, the file being checked cannot be locally hosted yet, so it must be a link to a file in a GitHub repository. Local file checks are planned for the next version.

Example (on a PR):

```text
#import: "pr.txt";
#if: "2.2081.lol" = dir/default,
from: "pr.cache";
    then: create ".gitattributes";
```

On a PR, this checks whether `2.2081.lol` exists in the local directory found in `pr.cache`. If it does, `.gitattributes` is created. If not, nothing happens.

### 3.2 `pr.cache`

`pr.cache` maps C/ directory names to real paths on the host. Each entry is `name = path`, and the whole file is wrapped in braces. Example for Linux:

```text
{
dir/default = /home/C-Slash/Downloads
}
```

With this cache, `dir/default` in `#if` and `#create:` resolves to `/home/C-Slash/Downloads`. The paths are platform-specific, so each OS has its own cache values.

Example for Android:

```text
{
dir/default = storage/emulated/0/Download
}
```

## 4. Pluggable grammar

The native core is intentionally minimal. A spec file loaded at the top of a source file (for example `#include: "pr.txt";`) may modify the lexer's grammar rules mid-stream, such as enabling single-quote strings.

## 5. Staged compilation

1. Cache a resource.
2. Convert it to native C/ (`#convert:`).
3. Validate it (`#if ... = valid`).
4. Run it (`#run:`) and emit outputs.

Intermediate files go to a default directory (for example `dir/default`).

## 6. Private Runtime (PR)

The Private Runtime (PR) is a self-hosted version of the C/ Runtime. The full Runtime itself is not public. A PR is a sandboxed execution layer built in C++ that:

- manages a unified memory pool,
- translates data types on the fly,
- blocks unauthorized file system and network access.

## 7. Importable languages

JavaScript, TypeScript (types stripped first), Java (`.java`, `.class`, `.jar`), C, C++, and C# (`.cs`, `.csx`, `.csproj`, `.sln`, `.slnx`).

## Open questions

- Whether `#if` supports an else branch or other operators beyond `=`.
- Grammar for chaining multiple actions after `then:`.
- Whether `#import:` and `then:` lines require a trailing `;` (the section 1 rule says yes; informal examples omit it).
- How the `pr.txt` spec format declares lexer changes.
