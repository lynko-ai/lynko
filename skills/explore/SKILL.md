---
name: lynko-explore
description: >
  Navigate and read content through Lynko collections — git repos, Google Drive,
  documents, PDFs. Use when browsing codebases, reading documentation, searching
  across files, discovering skills in the pod, or understanding project structure.
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

## Reading PDFs

PDFs use pages as their natural unit. `pages()` defaults to extracted text; image-only pages return rendered images automatically.

```
my-project[paper.pdf].toc()                   # Bookmarks as table of contents
my-project[paper.pdf].pages("1-3")            # Text per page (image fallback for scanned)
my-project[paper.pdf].pages("5", as="image")  # Force rendered PNG (precision read)
my-project[paper.pdf].grep("methodology")     # Search extracted text
```

**Default to text. Switch to image mode when:**

- The answer lives in a multi-column table — financial statements, segment tables, multi-year comparisons. Text extraction can lose column-to-header alignment; vision reads the rendered page reliably.
- Text shows weird symbols, dropped ligatures (`scal` for `fiscal`), or other glyph corruption.
- The question is inherently visual — charts, figures, layout-dependent diagrams.

Typical workflow: `toc()` or `grep()` to locate the right page → `pages(N)` for prose → `pages(N, as="image")` if the page is tabular or the text looked off. Image mode runs ~2–4× the input-token cost of text at 150 DPI, so use it as the precision step, not the default.

## Reading Google Docs

Google Docs in Drive collections use paragraphs as their natural unit, with headings forming a hierarchical table of contents. Multi-tab documents expose each tab as a navigable unit.

```
my-drive[Roadmap].toc()                      # Heading hierarchy (+ tab list for multi-tab)
my-drive[Roadmap].section("Phase 1")         # One section by heading
my-drive[Roadmap].paragraphs("1-5")          # Paragraph range (single-tab)
my-drive[Roadmap].paragraphs("Tab 1:1-5")    # Tab-qualified (multi-tab)
my-drive[Roadmap].tab("Q4 Plan")             # Full content of one tab
my-drive[Roadmap].grep("milestone")          # Search with section + tab context
```

`tab()` is advertised only on multi-tab documents. On single-tab Google Docs, `read()` returns the full document. Run `my-drive[Roadmap].ls()` to see which operations are available for a specific doc.

## Reading Spreadsheets

Google Sheets in Drive collections use rows as their natural unit. Start with `sheets()` for the schema, then drill into data:

```
my-drive[Budget].sheets()                              # Sheet names, headers, dimensions
my-drive[Budget].rows("1-10")                          # First 10 rows (single-sheet)
my-drive[Budget].rows("Sales:1-10")                    # Sheet-qualified rows (multi-sheet)
my-drive[Budget].rows("Sales:1-10", columns="Revenue,Cost")  # Column projection by header name
my-drive[Budget].cells("Sales!B5:D15")                 # A1-notation access
my-drive[Budget].grep("Widget")                        # Search with sheet+row context
```

`grep()` on spreadsheets returns sheet-aware results like `Sales Row 3:` — use the same addressing with `rows()` to follow up: `rows("Sales:3")`.

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

**Semantic vs exact.** `grep()` matches exact text or regex; `search()` ranks results by *meaning* via embeddings — reach for it when you don't know the exact words:

```
my-project.search("how auth tokens are refreshed")   # Semantic; ranked by meaning
my-project.search("retry backoff", max_results=10)    # Widen the result set
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

## Finding Skills

The skills node surfaces qualified `SKILL.md` files across every collection in the pod as one searchable set:

```
skills.ls()                                  # All skills, grouped by collection
skills["lynko-prompts/"].ls()                # Skills from one collection
skills.search("write an engineering plan")   # Semantic search across all skills
skills["lynko-prompts/"].search("eval doc")  # Search within one collection
skills["lynko-prompts/tasks/adr-draft/SKILL.md"].info()  # One skill's details
```

In a crowded pod — hundreds of skills across many collections — a bare generic query gets out-ranked by unrelated skills. Scope to the collection, or name it in the query, to surface the one you want: `skills["lynko-prompts/"].search("code review")` or `skills.search("lynko code review")`.

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