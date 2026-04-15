---
type: meta
updated: 2026-04-16
---
# Wiki Schema & Conventions

## Purpose

This wiki is a compiled knowledge layer. AI agents read wiki pages FIRST
before drilling into raw source documents.

## Page Types

| Type | Location | Frontmatter `type` | Purpose |
|------|----------|-------------------|---------|
| Domain | wiki/domains/ | domain | Full picture per feature domain |
| Entity | wiki/entities/ | entity | One page per DB entity |
| Endpoint | wiki/endpoints/ | endpoint-group | Grouped by domain |
| Screen | wiki/screens/ | screen-group | Grouped by domain |
| Decision | wiki/decisions/ | decision | ADRs: why X over Y |
| Gap | wiki/gaps/ | gap-tracker | Open deficiencies by severity |
| Cross-Cut | wiki/cross-cuts/ | cross-cut | Concerns spanning domains |

## Frontmatter Contract (required on every page)

```yaml
---
type: {page-type}
domain: {domain-name}
status: draft | compiled | stale | superseded
sources:
  - {raw-source-path}
updated: YYYY-MM-DD
tags: [keyword1, keyword2]
---
```

## Wikilink Format

```
[[domain:{name}]]     → wiki/domains/{name}.md
[[entity:{Name}]]     → wiki/entities/{name}.md
[[endpoints:{name}]]  → wiki/endpoints/{name}.md
[[screens:{name}]]    → wiki/screens/{name}.md
[[decision:{slug}]]   → wiki/decisions/{slug}.md
[[gaps:{name}]]       → wiki/gaps/{name}.md
[[cross-cuts:{name}]] → wiki/cross-cuts/{name}.md
```

## Maintenance Protocol

1. Update relevant wiki page(s) when you discover new info or complete a gap
2. Append to wiki/log.md with what changed and why
3. Set `status: stale` if a page's source data has changed
4. Never delete pages — mark `status: superseded` with pointer to replacement
