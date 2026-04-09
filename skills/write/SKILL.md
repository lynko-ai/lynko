---
name: lynko-write
description: >
  Draft and edit documents, notes, and written content through Lynko. Use when
  writing new documents, updating existing content, reorganizing sections, or
  collaborating on text-based artifacts.
---

# Write with Lynko

## Creating New Files

Create a new file in a collection:

```
my-project[docs/new-guide.md].draft("# Getting Started\n\nIntroduction goes here.")
```

The file exists as a draft until you commit.

## Editing Existing Content

### Replace specific text

```
my-project[docs/guide.md].draft.edit("old paragraph text", "new paragraph text")
```

### Scoped editing

Target a specific section to avoid accidental matches:

```
my-project[docs/guide.md].section("Installation").draft.edit("npm install", "npm install my-package")
```

### Multi-line content with raw strings

Use `@@@@@` delimiters for blocks of content:

```
my-project[docs/guide.md].draft.edit(
@@@@@
## Old Section

Old content here.
@@@@@
,
@@@@@
## Updated Section

New content with more detail.
@@@@@
)
```

Opening and closing delimiters must be on their own lines. If your content contains `@@@@@`, use `@@@`, `"""`, or `'''` instead.

### Append to a document

```
my-project[docs/guide.md].draft.append("\n\n## New Section\n\nContent here.")
```

Note: `draft.append()` does raw concatenation — always include `\n` for newlines.

## Reading Before Writing

Always check existing content before editing:

```
my-project[docs/guide.md].toc()           # See the structure
my-project[docs/guide.md].section("Setup") # Read the section you'll edit
```

This ensures your edits match the actual current text.

## Reviewing and Committing

```
my-project.status()                          # See which files have drafts
my-project[docs/guide.md].diff()          # Review changes in detail
my-project.commit("docs: update installation guide")
```

## Discarding Changes

```
my-project[docs/guide.md].draft.discard()  # Discard one file
my-project.restore()                          # Discard all drafts
```

## Tips

- Read the section before editing — `draft.edit()` requires exact text matching
- Use `toc()` to understand document structure before making changes
- Prefer `section().draft.edit()` over file-level `draft.edit()` for large documents
- Include surrounding context in your old text to ensure unique matching