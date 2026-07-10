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

Snapshots: navigation only.

## Content navigation

| Content | Structural operations | Conditions |
|---|---|---|
| Code | `outline()` · `expand()` · `find_definition()` · `find_references()` · `lines()` | language supported by the code index |
| Markdown / structured text | `toc()` · `section()` · `lines()` | — |
| PDF | `pages()` · `toc()` / `section()` · `grep()` | `toc()`/`section()` need native bookmarks; `grep()` needs extractable text |
| Google Docs | `toc()` · `section()` · `paragraphs()` · `tab()` | `tab()` for multi-tab documents |
| Google Sheets | `sheets()` · `rows()` · `cells()` · `grep()` | — |

Collections expose `ls()`, `find()`, and — when hierarchical — `tree()`. Files expose `read()` plus their type-specific operations. `grep()` works where text is extractable or materialized. PDF is navigate-only today, regardless of the containing collection.

## Optional nodes

| Node | Verb | Prerequisites |
|---|---|---|
| Embedding | `search()` | node attached; qualifying content; index builds on first use |
| CI | `test()` | pushable remote git collection · node attached · targets from CI config |
| Runner | `run()` | remote git collection · node attached · machine registered¹ |
| Runner | `invoke()` | remote git collection · node attached · agent installed · a delegable role (`author`/`full` excluded; sub-agent scope ⊆ yours)¹ |
| Skills | `skills.search()` · read · pin · curate | node attached; qualified committed `SKILL.md` files |

¹ Read-only remotes: no-draft fast path; drafts without push access are rejected at dispatch.

Collections and content are the nouns; nodes add the verbs — and every verb acts at the scope the handle names.