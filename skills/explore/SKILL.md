---
name: lynko-explore
description: >
  Navigate and read content through Lynko collections — git repos, Google Drive,
  documents, PDFs. Use when browsing codebases, reading documentation, searching
  across files, or understanding project structure.
---

# Explore Content with Lynko

## Getting Oriented

Start every session by checking what's available:

```
artifacts()                                  # All collections and their operations
my-project.ls()                              # Top-level files
my-project.ls("src/")                        # Subdirectory listing
my-project.tree()                            # Full directory structure
```

## Reading Documents

Navigate documents top-down — don't read the whole file when you need one section:

```
my-project[docs/guide.md].toc()           # Table of contents with line numbers
my-project[docs/guide.md].section("Setup") # Read just that section
my-project[docs/guide.md].lines("10-30")  # Read specific lines
my-project[docs/guide.md].read()          # Full file (last resort for small files)
```

When sections share common titles, use ` > ` to disambiguate:

```
my-project[paper.md].section("Methods > Data Collection")  # Specific subsection
```

Intermediate levels can be skipped — partial paths resolve if unique.

## Reading Code

Use structure-aware navigation instead of reading entire source files:

```
my-project[src/server.go].outline()        # Types, functions, signatures
my-project[src/server.go].expand("Main")   # One function's full body
my-project[src/server.go].lines("40-60")   # Specific line range
```

`outline()` shows the structure. `expand()` shows the details. This keeps context minimal.

## Searching

Search across files with scope control:

```
my-project.grep("pattern")                   # All files
my-project.grep("pattern", context_lines=3)  # With surrounding context
my-project[src/].grep("pattern")             # Under a directory
my-project[**/*.go].grep("error")            # Only .go files at any depth
my-project[*.md].grep("TODO")               # Only .md files at root level
my-project.grep(/error|warn/)                # Regex pattern
```

## Finding Symbols

Jump to definitions and trace references across a codebase:

```
my-project.find_definition("UserService")    # Where is it defined?
my-project.find_references("UserService")    # Where is it used?
my-project[src/].find_definition("Handler")  # Scoped to a directory
```

## Finding Files

Locate files by name pattern:

```
my-project.find(*.go)                        # All .go files
my-project.find(*.test.ts)                   # All test files
my-project.find(*config*, type=file)         # Files matching "config"
```

## Navigation Hierarchy

Follow this pattern for efficient exploration:

1. **Structure** → `ls()`, `tree()`, `toc()`, `outline()` — "What's here?"
2. **Targeted** → `section()`, `expand()`, `lines()`, `grep()` — "Show me this"
3. **Full** → `read()` — "Everything" (only when needed)

## Multi-Command Syntax

Combine multiple commands in one call for efficiency:

```
my-project[src/server.go].outline()
my-project[src/handler.go].outline()
my-project[src/].grep("TODO")
```

This executes all three in one round trip.

## Tips

- `grep()` results include line numbers and section names — use them to target follow-up reads
- `outline()` shows line numbers — use `lines("N-M")` to read specific ranges
- Use directory scope (`my-project[src/]`) to keep searches fast and relevant
- `toc()` on markdown files shows headings as a navigable map — use `section()` to drill in