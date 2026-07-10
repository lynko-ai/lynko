# Capabilities

What each collection type, content type, and node supports today.

> Capabilities shown are the collection maximum — a connection role may expose a narrower subset. Roles are additive, allow-only scopes; `whoami()` shows what your connection can see.

## Collections & write-back

| Collection | Refresh | Draft state | Publish / write-back |
|---|---|---|---|
| Git — writable remote | `pull()` | ✓ scoped drafts · `diff()` / `status()` / `restore()` / `merge()` | ✓ `commit()` |
| Git — public / read-only remote | `pull()` · `compare()` · `log()` | — | — |
| Git — snapshot | — | — | — |
| Google Drive | `pull()` | Coming soon | Coming soon |

Git snapshots are navigable but expose no `pull()`, drafts, or commit. Drive draft/write-back is in active implementation and lands soon; until then Drive collections are read + navigate.

## Content navigation

| Content | Structural operations | Conditions |
|---|---|---|
| Code | `outline()` · `expand()` · `find_definition()` · `find_references()` · `lines()` | language supported by the code index |
| Markdown / structured text | `toc()` · `section()` · `lines()` | — |
| PDF | `pages()` · `toc()` / `section()` · `grep()` | `toc()`/`section()` need native bookmarks; `grep()` needs extractable text |
| Google Docs | `toc()` · `section()` · `paragraphs()` · `tab()` | `tab()` for multi-tab documents |
| Google Sheets | `sheets()` · `rows()` · `cells()` · `grep()` | — |

`read()`, `grep()`, `find()`, `ls()`, and `tree()` are available across collections and content, subject to the same conditions. Drive `grep()` searches materialized, navigable content and skips catalog-only entries. Structural PDF editing is not part of the current text DSL — PDF is navigate-only today regardless of the containing collection.

## Optional nodes

| Node | Verb | Prerequisites |
|---|---|---|
| Embedding | `search()` | node attached; qualifying content; index builds on first use |
| CI | `test()` | node attached; pushable remote Git collection; targets come from your CI configuration |
| Runner | `run()` | node attached; machine registered |
| Runner | `invoke()` | node attached; requested agent installed; a concrete delegable role — `author`/`full` cannot be a delegation target, and the sub-agent's scope is always a subset of your connection's |
| Skills | `skills.search()` · read · pin · curate | node attached; qualified committed `SKILL.md` files |

Collections and content are the nouns; nodes add the verbs — and every verb acts at the scope the handle names.