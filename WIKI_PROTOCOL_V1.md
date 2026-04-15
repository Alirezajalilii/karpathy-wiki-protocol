# Karpathy Wiki Pattern — Comprehensive Rules & Prompt Reference

> Compiled from Karpathy's original gist (2026-04-04), PlanChin's production implementation (139 pages, 594 wikilinks), and 8 iterative design sessions.
> This document is a **reusable prompt template**. Adapt the project-specific parts (page types, domains, wikilinks) to your own codebase.

---

## 1. The Core Idea (Why This Exists)

Most LLM-document workflows are RAG: upload files, retrieve chunks at query time, generate an answer. The LLM re-derives knowledge from scratch on every question. Nothing accumulates.

The Karpathy wiki is different. Instead of retrieving from raw documents at query time, the LLM **incrementally builds and maintains a persistent wiki** — a structured, interlinked collection of markdown files that sits between agents and raw sources. When new information arrives, the LLM reads it, extracts key information, and **integrates it into the existing wiki** — updating entity pages, revising summaries, noting contradictions, strengthening or challenging the evolving synthesis.

**The wiki is a persistent, compounding artifact.** Cross-references are already there. Contradictions are already flagged. Synthesis already reflects everything processed. The wiki gets richer with every source added and every question asked.

The human's job: curate sources, direct analysis, ask good questions, think about meaning.
The LLM's job: summarizing, cross-referencing, filing, bookkeeping — everything that makes a knowledge base useful over time but that humans abandon because the maintenance burden grows faster than the value.

**Compiler analogy:** `raw/` is source code, the LLM is the compiler, `wiki/` is the executable output, lint is tests, queries are runtime.

---

## 2. Three-Layer Architecture

| Layer | Location | Owner | Mutability |
|-------|----------|-------|------------|
| **Raw Sources** | `raw/` or `docs/` | Human | **Immutable** — LLM reads but NEVER modifies |
| **Wiki** | `wiki/` | LLM | LLM-written, LLM-maintained. Human reads. |
| **Schema** | `CLAUDE.md` / `AGENTS.md` + `wiki/schema.md` | Human + LLM | Co-evolved — defines structure, conventions, workflows |

### The Schema Is the Most Important File

The schema document is what turns a generic LLM into a disciplined wiki maintainer. It encodes:
- What types of entities and relationships exist in your domain
- When to create a new page vs. update an existing one
- What page templates look like (frontmatter, required sections)
- What workflows to follow for ingest, query, and lint
- What's private vs. shared

You and the LLM co-evolve this over time. The first version will be rough. After a few dozen sources and a few lint passes, you'll have a schema that reflects how your domain actually works.

---

## 3. Three Core Operations

### 3.1 Ingest

Drop a new source into `raw/` and tell the LLM to process it.

**Flow:**
1. LLM reads the source
2. Discusses key takeaways with you (optional — you can batch-ingest with less supervision)
3. Writes a summary page in wiki
4. Updates `wiki/index.md`
5. Updates relevant entity and concept pages across the wiki
6. Appends entry to `wiki/log.md`

A single source might touch 10–15 wiki pages. Ingest one at a time if you want to stay involved and guide emphasis. Batch if you trust the schema.

**Prompt pattern:**
```
I added a new source to raw/. Please ingest it:
- Read the source
- Create/update wiki pages for key entities, concepts, and connections
- Update index.md
- Log the operation to log.md
- Flag any contradictions with existing wiki content
```

### 3.2 Query

Ask questions against the wiki. The LLM searches for relevant pages, reads them, synthesizes an answer with citations.

**Critical insight: good answers should be filed back into the wiki as new pages.** A comparison you asked for, an analysis, a connection you discovered — these are valuable and shouldn't disappear into chat history. Your explorations compound just like ingested sources.

**Prompt pattern:**
```
Search the wiki for information about [topic].
Synthesize an answer with citations to specific wiki pages.
If this answer contains novel analysis worth preserving, offer to save it as a new wiki page.
```

### 3.3 Lint

Periodic health-check of the wiki.

**What to look for:**
- Contradictions between pages
- Stale claims that newer sources have superseded
- Orphan pages with no inbound links
- Important concepts mentioned but lacking their own page
- Missing cross-references
- Data gaps that could be filled with a web search
- Broken `[[wikilinks]]`
- Template placeholders that were never filled
- Pages missing required frontmatter fields

**Prompt pattern:**
```
Run a lint pass on the wiki:
1. Check all [[wikilinks]] resolve to existing pages
2. Find orphan pages (in index but no inbound links)
3. Find contradictions between pages
4. Find stale content (pages not updated in >30 days whose sources have new data)
5. Write severity-tiered report: 🔴 errors / 🟡 warnings / 🔵 info
6. Log to wiki/log.md
```

---

## 4. Index and Log — Two Special Files

### `wiki/index.md` — Content-Oriented

A catalog of everything in the wiki. Each page listed with a link, one-line summary, and metadata. Organized by category. Updated on every ingest.

**How the LLM uses it:** Reads index first to find relevant pages for a query, then drills into them. Works well at moderate scale (~100 sources, hundreds of pages) without embedding-based RAG infrastructure.

### `wiki/log.md` — Chronological

Append-only record of what happened and when — ingests, queries, lint passes, updates.

**Format tip:** Start each entry with a consistent prefix so it's parseable with unix tools:
```markdown
## [2026-04-16] ingest | Article Title
## [2026-04-16] query | What is X?
## [2026-04-16] lint | Weekly health check
## [2026-04-16] update | Fixed contradictions in entity pages
```

`grep "^## \[" wiki/log.md | tail -5` gives you the last 5 operations.

---

## 5. Page Structure & Frontmatter

Every wiki page MUST have YAML frontmatter for machine-parseable metadata:

```yaml
---
type: domain | entity | endpoint-group | screen-group | decision | gap-tracker | cross-cut
domain: auth  # which domain(s) this relates to
status: draft | compiled | stale | superseded
sources:
  - docs/API_INTERFACE_CONTRACT.md
  - docs/ServerImplementationBlueprint.md
updated: 2026-04-16
tags: [authentication, OTP, sessions]
---
```

### Page Types (adapt to your project)

| Type | Purpose | Template |
|------|---------|----------|
| **Domain** | Full picture per feature domain: requirements, endpoints, entities, screens, status, gaps | `wiki/domains/` |
| **Entity** | One page per DB entity: schema, relations, business rules, consumers | `wiki/entities/` |
| **Endpoint Group** | Grouped by domain: request/response, auth, errors, impl status | `wiki/endpoints/` |
| **Screen Group** | Grouped by domain: components, store bindings, nav path | `wiki/screens/` |
| **Decision** | ADRs: why X over Y, rationale, consequences | `wiki/decisions/` |
| **Gap Tracker** | Open deficiencies from audit, by severity and domain | `wiki/gaps/` |
| **Cross-Cut** | Concerns spanning domains: auth flow, i18n, offline, errors | `wiki/cross-cuts/` |
| **Source** | Summary of a raw source document | `wiki/sources/` |
| **Concept** | Abstract concept that spans multiple entities/domains | `wiki/concepts/` |
| **Synthesis** | Cross-source analysis, filed from a query | `wiki/syntheses/` |

---

## 6. Wikilinks — The Connective Tissue

Use `[[wikilinks]]` between pages. These are the primary navigation mechanism for both humans and LLMs.

```
[[domain:auth]]              → wiki/domains/auth.md
[[entity:User]]              → wiki/entities/user.md
[[endpoints:tours]]          → wiki/endpoints/tours.md
[[screens:events]]           → wiki/screens/events.md
[[decision:why-nestjs]]      → wiki/decisions/why-nestjs.md
[[gaps:cross-cutting]]       → wiki/gaps/gap-tracker.md
[[cross-cuts:rtl-layout]]    → wiki/cross-cuts/rtl-layout.md
```

Every new page MUST be wikilinked from its parent domain page and from `index.md`.

---

## 7. Hard Rules for Wiki Maintenance

These are non-negotiable. Every agent, every session, every task.

### Rule 1: Update on Discovery

If you find or fix a gap, change implementation status, find a contradiction, add a module/entity/endpoint, or discover undocumented behavior — **update the relevant wiki page BEFORE ending your session.**

This is the rule that makes the wiki compound instead of rot.

### Rule 2: Log Every Edit

Append to `wiki/log.md`: date, pages changed, reason, source reference.

### Rule 3: Frontmatter Stays Current

Set `updated:` on every page you touch. Mark stale pages `status: stale`. Never delete pages — use `status: superseded` with a pointer to the replacement.

### Rule 4: Link New Pages

New entity/endpoint/screen/domain → create wiki page per `wiki/schema.md` template → add to `wiki/index.md` → wikilink from parent domain page.

### Rule 5: Gaps Don't Disappear

Mark resolved gaps `✅ Resolved` with date + evidence. Don't delete rows. The history of what was broken and how it was fixed is itself valuable knowledge.

### Rule 6: Trace Everything

Every wiki claim → raw source file, code file, or audit artifact. Can't trace it? Write `NOT VERIFIED`. Untraceable claims are worse than missing claims.

### Rule 7: Flag Contradictions, Don't Guess

When raw sources contradict each other, document BOTH positions with file references. Mark `PENDING — escalate to human`. Never silently pick one.

---

## 8. Hard Don'ts

- **Don't read all raw sources at session start.** The wiki exists to prevent that. Read wiki first, drill into raw sources only when needed.
- **Don't leave template placeholders** (`{N}`, `{text}`, `TODO`, `TBD`). Every field has real data or `NOT FOUND IN SOURCES`.
- **Don't modify raw source documents to match wiki.** Wiki follows sources, not the reverse. Raw sources are immutable.
- **Don't create wiki pages with placeholder content.** If you don't have the data, don't create the page yet.
- **Don't duplicate wiki content in code comments.** Link to the wiki page instead.
- **Don't treat wiki as append-only.** Update existing pages when their data changes. Don't create new pages for corrections.
- **Don't skip wiki updates because "it's a small change."** Small changes compound. That's the entire point.

---

## 9. Agent Session Startup Protocol

Every new agent session follows this read order:

```
1. Read CLAUDE.md (or your schema file)         — operating rules
2. Read wiki/index.md                            — find relevant pages
3. Read wiki/domains/{relevant-domain}.md        — full domain picture
4. Follow [[wikilinks]] to entity/endpoint/screen/cross-cut pages as needed
5. ONLY read raw source docs when:
   - Wiki says NOT FOUND IN SOURCES or status: stale
   - You need exact DTO shapes or line-level code reference
   - You're resolving a contradiction the wiki flagged
```

**The wiki exists so agents don't have to re-read thousands of lines of raw docs every session.** If your agents are reading raw sources first, the wiki isn't doing its job.

---

## 10. Task-Specific Workflows

### 10.1 Writing a New Feature

```
1. Read wiki/domains/{domain}.md — understand the domain
2. Read wiki/endpoints/{domain}.md — existing endpoints
3. Read wiki/entities/{relevant}.md — schema and relations
4. Read wiki/gaps/gap-tracker.md — what's missing
5. Implement the feature
6. Update wiki:
   - wiki/endpoints/{domain}.md — add new endpoint
   - wiki/entities/{entity}.md — if schema changed
   - wiki/domains/{domain}.md — update implementation status
   - wiki/gaps/gap-tracker.md — mark resolved gaps
   - wiki/log.md — log what you did
```

### 10.2 Fixing a Bug

```
1. Read wiki/domains/{domain}.md — understand the context
2. Find and fix the bug in code
3. SCAN GLOBALLY for the same anti-pattern elsewhere
4. Update wiki:
   - If the bug revealed a gap → add to gap-tracker
   - If the bug changed behavior → update domain/endpoint/entity page
   - If the bug contradicted a wiki claim → fix the wiki claim
   - wiki/log.md — log the fix, root cause, and affected files
```

### 10.3 Updating the Wiki (Recompilation)

A full recompilation of a wiki page is needed when:
- The underlying raw source document was significantly rewritten
- More than 5 fields on a page are stale
- A domain's architecture changed (new module split, merged modules)
- Human explicitly requests: "recompile wiki for {domain}"

```
1. Re-read the raw sources for that page
2. Cross-reference with current code state
3. Rewrite the page completely
4. Update wiki/log.md with recompile operation type
5. Check all outbound [[wikilinks]] still resolve
6. Check all inbound links from other pages are still accurate
```

### 10.4 Adding a New Domain

```
1. Create wiki/domains/{domain}.md from schema.md template
2. Create wiki/endpoints/{domain}.md
3. Create wiki/screens/{domain}.md (if has UI)
4. Create wiki/entities/{entity}.md for each entity
5. Add all pages to wiki/index.md
6. Wikilink from related domain pages
7. Add gap entries to wiki/gaps/gap-tracker.md
8. Log to wiki/log.md
```

---

## 11. PostToolUse: Wiki Verification Gate

**Runs after every completed task — before reporting "done."**

### Gate 1: Was a Wiki Update Required?

| # | Question | If YES → Wiki Action |
|---|----------|----------------------|
| 1 | Created, renamed, or deleted an entity/table/model? | Update schema + domain page |
| 2 | Added, modified, or removed an API endpoint? | Update endpoints page |
| 3 | Added, modified, or removed a screen/component? | Update screens page |
| 4 | Changed a business rule, validation, or state machine? | Update domain page |
| 5 | Resolved a gap from gap-tracker? | Mark ✅ Resolved with date + evidence |
| 6 | Discovered a new gap, contradiction, or undocumented behavior? | Add to gap-tracker with severity |
| 7 | Changed error codes or error handling? | Update error-handling cross-cut |
| 8 | Changed localization, layout, or data format conventions? | Update relevant cross-cut page |

All 8 NO → no wiki update needed. Any YES → proceed to Gate 2.

### Gate 2: Is the Update Complete?

For each required update:
- Page exists (create from template if not)
- Content matches actual implementation (code is truth)
- All fields/params/states documented
- Every fact has a source path
- `updated:` set to today
- Page in `wiki/index.md` and wikilinked from parent
- Gap tracker synced
- `wiki/log.md` entry added

### Gate 3: Final Self-Check

1. No placeholder text left in any wiki page touched
2. Raw source documents NOT modified
3. Did not skip because "it's a small change"
4. If unsure whether update was needed → updated

---

## 12. Multi-Phase Wiki Compilation (Initial Build)

Building a wiki from scratch on a large codebase CANNOT be done in one session. Context windows fill up. Agents hallucinate coverage they didn't do.

### Phase Strategy

| Phase | Input Sources | Output |
|-------|-------------|--------|
| **1. Scaffold** | schema.md only | Directory structure, empty templates, index.md, log.md |
| **2. Domain Batch A** | PRD + API contract + first N domains from audit | 6–8 domain pages + their entity/endpoint pages |
| **3. Domain Batch B** | PRD + blueprints + remaining domains | Remaining domain pages + their sub-pages |
| **4. Entity Pass** | Server blueprint + code | All entity pages with schema, relations, rules |
| **5. Endpoint Pass** | API contract + code | All endpoint group pages |
| **6. Screen Pass** | App blueprint + code | All screen group pages |
| **7. Cross-Cuts & Decisions** | All sources | Cross-cutting pages + decision records |
| **8. Validation & Index** | wiki/ directory | Broken link fix, orphan check, index finalization |

### Hard Rules for Phased Compilation

- **Hard STOP between phases.** Start a new session. Don't let the agent continue.
- **Each phase reads ONLY the sources it needs.** Don't load all 7 source docs.
- **Each phase self-logs.** If a session dies mid-phase, `wiki/log.md` shows where to resume.
- **No placeholder pages.** If a phase doesn't have data for a page, skip it — next phase will create it.
- **Checkpoint after every domain.** Write findings before moving on. If session dies, work survives.
- **If the agent starts producing skeleton pages with placeholder text, kill the session.** Restart from where log.md left off.

---

## 13. When to Use Wiki vs. Raw Sources

| Use wiki/ for | Use raw sources for |
|--------------|-------------------|
| Understanding a domain's full picture | Implementing a feature (exact task steps) |
| Checking implementation status | Exact DTO / request-response shapes for coding |
| Finding which entities a feature touches | Verifying a wiki claim you doubt |
| Reviewing open gaps and priorities | Resolving a contradiction the wiki flagged |
| Understanding an architecture decision | Reading code during debugging |
| Onboarding a new agent session | Line-by-line code reference |

---

## 14. Anti-Corruption Rules

The wiki is only valuable if it stays accurate. These rules prevent rot:

1. **Code is truth, wiki is documentation.** When wiki and code disagree, code wins. Update wiki.
2. **Stale is worse than missing.** A page that was accurate 3 months ago but isn't now is actively harmful. Mark it stale immediately.
3. **One source of truth per fact.** If the same information appears in 3 wiki pages, one is the canonical page and the others link to it. Don't duplicate.
4. **Every fact has provenance.** If you can't point to a raw source, code file, or audit artifact, the fact is `NOT VERIFIED`.
5. **Contradictions are features, not bugs.** Flagging a contradiction is more valuable than silently picking one interpretation.

---

## 15. CLAUDE.md Integration (Compressed Version for Agent Init)

This is the minimal wiki section for your agent init file. ~250 tokens.

```markdown
## N. Wiki (Karpathy Pattern)

`wiki/` = compiled knowledge layer. Read wiki, not raw sources.

Links: `[[domain:auth]]` `[[entity:User]]` `[[endpoints:tours]]`
`[[screens:events]]` `[[decision:why-X]]` `[[cross-cuts:topic]]`
`[[gaps:domain]]`

### Maintenance (mandatory)

1. **Update on discovery.** Fix a gap, change impl status, add a
   module/entity/endpoint, or find a contradiction → update the wiki
   page BEFORE ending your session.
2. **Log every edit.** Append to `wiki/log.md`: date, pages changed,
   reason, source.
3. **Frontmatter stays current.** Set `updated:` on every page you touch.
   Stale → `status: stale`. Never delete → `status: superseded`.
4. **Link new pages.** Create per `wiki/schema.md` template, add to
   `wiki/index.md`, wikilink from parent domain page.
5. **Gaps don't disappear.** Mark `✅ Resolved` with date + evidence.
   Don't delete rows.
6. **Trace everything.** Every claim → raw source, code file, or audit
   artifact. Untraceable → `NOT VERIFIED`.
7. **Flag contradictions, don't guess.** Document both positions with
   file refs, mark `PENDING — escalate to human`.

**Don't**: read all raw sources upfront; leave placeholders; modify
raw sources to match wiki.
```

---

## 16. Why This Works

The tedious part of maintaining a knowledge base is not the reading or the thinking — it's the bookkeeping. Updating cross-references, keeping summaries current, noting when new data contradicts old claims, maintaining consistency across dozens of pages.

Humans abandon knowledge bases because the maintenance burden grows faster than the value.

LLMs don't get bored, don't forget to update a cross-reference, and can touch 15 files in one pass. The wiki stays maintained because the cost of maintenance drops to near zero.

The schema is transferable. Share it with someone else working on a similar domain and they get a running start.

**Minimal viable wiki:** raw sources + wiki pages + index.md + a schema that describes ingest/query/lint workflows. Everything else is optional and modular. Pick what's useful, ignore what isn't.
