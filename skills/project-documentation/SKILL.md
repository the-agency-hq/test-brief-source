---
name: project-documentation
description: Use when writing, naming, or looking for a design document, spec, or implementation plan — and before creating any new file under `docs/`. Covers which directory a document belongs in and how its filename is formed.
---

# Project documentation

Design documents and implementation plans live in fixed directories under `docs/`, with a date-prefixed filename.
Check here before creating a document; putting one in the wrong place is much cheaper to avoid than to correct
once it is linked from elsewhere.

| Directory              | Holds                                                          |
|------------------------|----------------------------------------------------------------|
| `docs/design/`         | Design documents and specs — what is being built and why       |
| `docs/implementation/` | Implementation plans — the ordered task breakdown that builds it |

Other files may sit at the `docs/` root (a project's `idea.md`, for example). Only design documents and
implementation plans are placed by this convention.

## Naming

Both directories use the same filename form:

```
YYYY-MM-DD-<slug>-design.md
YYYY-MM-DD-<slug>-implementation.md
```

- **The date prefix is the document's creation date**, and it never changes. It is not the date of the last edit
  and not the date the work shipped. Revising a document in place keeps its original prefix.
- **The slug is kebab-case** and names the work, not the document type.
- **A design document and its implementation plan share a slug**, which is what pairs them:

  ```
  docs/design/2026-07-30-brief-pipeline-design.md
  docs/implementation/2026-07-30-brief-pipeline-implementation.md
  ```

  Their date prefixes are independent — the plan is usually written days after the design it implements — so pair
  documents by slug, never by date.

## Finding an existing document

The date prefix means a plain directory listing sorts chronologically, so `ls docs/design/` is a project history in
order. To find the document for a piece of work, match on the slug rather than guessing a date:

```bash
ls docs/design/ docs/implementation/ | grep <slug>
```

## Writing one

Write the document straight into its file under the correct directory. Settle open questions and trade-offs in
conversation first, then write — the document is the deliverable, so do not render its full content in chat and
then write the same thing to disk.
