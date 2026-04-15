# Karpathy Wiki Pattern — Agent Operating Rules

```
Target:    LLM agents with filesystem access
Authority: CLAUDE.md > this document > agent defaults
Override:  Human instructions override this document
Format:    Flat rules. No prose. Every rule is binary PASS/FAIL.
```

---

## 1. ARCHITECTURE

```
Layer 1: raw/     → Immutable source documents. Agent: READ ONLY. Never modify.
Layer 2: wiki/    → Compiled knowledge. Agent: READ + WRITE. Maintain actively.
Layer 3: schema   → CLAUDE.md + wiki/schema.md. Defines structure and conventions.
```

**Decision tree — which layer to read:**

```
START → Is the answer in wiki/?
  YES → Use wiki. Done.
  NO  → Is wiki page status: stale?
    YES → Read raw source, update wiki page, then answer.
    NO  → Read raw source, answer, create/update wiki page.
```

---

## 2. SESSION STARTUP (execute in order, no skipping)

```
STEP 1: Read CLAUDE.md
STEP 2: Read wiki/index.md
STEP 3: Read wiki/domains/{task-relevant-domain}.md
STEP 4: Follow [[wikilinks]] to entity/endpoint/screen pages as needed
STEP 5: Read raw sources ONLY IF:
        - wiki page says NOT FOUND IN SOURCES
        - wiki page status: stale
        - task requires exact DTO shapes or line-level code
        - resolving a wiki-flagged contradiction
```

**FAIL condition:** Reading all raw sources at session start.
**FAIL condition:** Skipping wiki and going straight to raw sources.

---

## 3. OPERATIONS

### 3.1 INGEST (new source → wiki)

```
INPUT:  New file in raw/
OUTPUT: Updated wiki pages + index + log entry

STEPS:
1. Read the source file completely.
2. For each entity, concept, or fact extracted:
   a. Wiki page exists → UPDATE existing page.
   b. Wiki page missing → CREATE from wiki/schema.md template.
3. Update wiki/index.md — add new pages.
4. Check for contradictions with existing wiki content.
   - Contradiction found → document BOTH positions, mark PENDING.
   - No contradiction → proceed.
5. Append to wiki/log.md: date, pages changed, source file, operation type.
```

**SUCCESS:** Every fact from source is in wiki. Index updated. Log appended. Zero placeholders.
**FAIL:** Any `TODO`, `TBD`, `{N}`, or `{text}` in output pages.

### 3.2 QUERY (question → answer from wiki)

```
INPUT:  User question
OUTPUT: Answer with wiki page citations

STEPS:
1. Read wiki/index.md.
2. Identify relevant pages by title and tags.
3. Read identified pages.
4. Synthesize answer. Cite specific wiki pages.
5. If answer contains novel analysis:
   → Offer to save as new wiki page.
   → If saved: update index.md, log.md.
```

**SUCCESS:** Answer grounded in wiki content with page citations.
**FAIL:** Answer derived from raw sources when wiki had the information.
**FAIL:** Answer with no citations.

### 3.3 LINT (health check)

```
INPUT:  wiki/ directory
OUTPUT: Severity-tiered report + fixes applied

CHECKS (in order):
1. Broken [[wikilinks]] — link target does not exist        → 🔴 ERROR
2. Missing frontmatter fields (type, domain, status, updated) → 🔴 ERROR
3. Placeholder text (TODO, TBD, {N}, {text})                 → 🔴 ERROR
4. Orphan pages — in index.md but zero inbound wikilinks     → 🟡 WARNING
5. Stale pages — updated: date >30 days old                  → 🟡 WARNING
6. Contradictions between pages                               → 🟡 WARNING
7. Concepts mentioned but lacking own page                    → 🔵 INFO
8. Missing cross-references between related pages             → 🔵 INFO

AFTER: Append report summary to wiki/log.md.
```

**SUCCESS:** All 🔴 resolved. Report logged.
**FAIL:** Any 🔴 left unresolved without documented blocker reason.

---

## 4. SEVEN MAINTENANCE RULES

All rules mandatory. All sessions. All models. No exceptions.

```
RULE 1: UPDATE ON DISCOVERY
  Trigger: gap fixed, impl status changed, entity/endpoint/module added,
           contradiction found, undocumented behavior discovered.
  Action:  Update relevant wiki page(s).
  Timing:  BEFORE ending session. BEFORE reporting "done."
  FAIL:    Session ends with stale wiki.

RULE 2: LOG EVERY EDIT
  Action:  Append to wiki/log.md.
  Format:  ## [YYYY-MM-DD] {operation} | {summary}
           Pages: {list}
           Source: {raw file or code path}
  FAIL:    Wiki page modified without log entry.

RULE 3: FRONTMATTER STAYS CURRENT
  Action:  Set updated: to today on every page touched.
  Stale:   If source data changed → set status: stale.
  Delete:  NEVER delete pages. Set status: superseded + link to replacement.
  FAIL:    Page touched without updated: change.

RULE 4: LINK NEW PAGES
  Trigger: New entity, endpoint, screen, domain, concept, or decision page.
  Action:  Create from wiki/schema.md template.
           Add to wiki/index.md.
           Add [[wikilink]] from parent domain page.
  FAIL:    Page exists but missing from index.md or unlinked from parent.

RULE 5: GAPS DON'T DISAPPEAR
  Resolve: Mark ✅ Resolved + date + evidence (code path or commit).
  NEVER:   Delete gap rows.
  FAIL:    Gap row deleted or resolved without evidence.

RULE 6: TRACE EVERYTHING
  Every wiki claim → raw source path, code file path, or audit artifact.
  Cannot trace → write NOT VERIFIED.
  NEVER:   Leave a claim without provenance.
  FAIL:    Untraceable claim without NOT VERIFIED label.

RULE 7: FLAG CONTRADICTIONS
  Trigger: Source A says X, source B says Y.
  Action:  Document BOTH positions with file references.
           Mark: PENDING — escalate to human.
  NEVER:   Silently pick one interpretation.
  FAIL:    Contradiction resolved without human input.
```

---

## 5. BANNED ACTIONS

```
NEVER: Read all raw sources at session start.
NEVER: Leave placeholder text in wiki pages.
NEVER: Modify raw source documents.
NEVER: Create wiki pages without frontmatter.
NEVER: Duplicate content across wiki pages (link instead).
NEVER: Skip wiki update because "it's a small change."
NEVER: Invent information not in raw sources.
NEVER: Delete wiki pages (use status: superseded).
NEVER: Resolve contradictions without flagging both positions.
```

---

## 6. REQUIRED ACTIONS

```
ALWAYS: Read wiki before raw sources.
ALWAYS: Update wiki when code changes affect documented behavior.
ALWAYS: Log every wiki edit to wiki/log.md.
ALWAYS: Set updated: date on every page touched.
ALWAYS: Add new pages to wiki/index.md.
ALWAYS: Wikilink new pages from parent domain page.
ALWAYS: Scan globally after bug fixes for same anti-pattern.
ALWAYS: Provide source path for every wiki claim.
```

---

## 7. PAGE STRUCTURE

### 7.1 Frontmatter (required on every page)

```yaml
---
type: domain | entity | endpoint-group | screen-group | decision | gap-tracker | cross-cut | source | concept | synthesis
domain: {domain-name}          # string or list
status: draft | compiled | stale | superseded
sources:                        # list of raw source file paths
  - docs/API_INTERFACE_CONTRACT.md
updated: YYYY-MM-DD
tags: [keyword1, keyword2]
---
```

**FAIL condition:** Any field missing. Any field with placeholder value.

### 7.2 Page Types

```
wiki/domains/{domain}.md        — Full picture per feature domain
wiki/entities/{entity}.md       — One page per DB entity
wiki/endpoints/{domain}.md      — Grouped by domain
wiki/screens/{domain}.md        — Grouped by domain
wiki/decisions/{slug}.md        — ADRs: why X over Y
wiki/gaps/gap-tracker.md        — Open deficiencies by severity
wiki/gaps/{domain}.md           — Per-domain gap details
wiki/cross-cuts/{topic}.md      — Concerns spanning domains
wiki/sources/{slug}.md          — Summary of a raw source
wiki/index.md                   — Master catalog (ALWAYS current)
wiki/log.md                     — Append-only operation history
wiki/schema.md                  — Page templates and conventions
```

### 7.3 Wikilink Format

```
[[domain:auth]]              → wiki/domains/auth.md
[[entity:User]]              → wiki/entities/user.md
[[endpoints:tours]]          → wiki/endpoints/tours.md
[[screens:events]]           → wiki/screens/events.md
[[decision:why-nestjs]]      → wiki/decisions/why-nestjs.md
[[gaps:payments]]            → wiki/gaps/payments.md
[[cross-cuts:rtl-layout]]    → wiki/cross-cuts/rtl-layout.md
```

---

## 8. POST-TASK VERIFICATION GATE

**Runs after every completed task. Before reporting done. Skipping = FAIL.**

### Gate 1: Was Wiki Update Required?

```
CHECK 1: Created/renamed/deleted entity or table?       → Update schema + domain page
CHECK 2: Added/modified/removed API endpoint?            → Update endpoints page
CHECK 3: Added/modified/removed screen or component?     → Update screens page
CHECK 4: Changed business rule, validation, state machine? → Update domain page
CHECK 5: Resolved a gap from gap-tracker?                → Mark ✅ Resolved + evidence
CHECK 6: Discovered new gap or contradiction?            → Add to gap-tracker
CHECK 7: Changed error codes or error handling?          → Update error-handling cross-cut
CHECK 8: Changed localization, layout, data formats?     → Update relevant cross-cut

ALL NO → No update needed. Report: Wiki: no update (task type: {type})
ANY YES → Proceed to Gate 2.
```

### Gate 2: Is Update Complete?

```
For EACH required update:
  □ Page exists (create from template if not)
  □ Content matches actual implementation (code is truth)
  □ All fields, params, states documented
  □ Every fact has source path
  □ updated: set to today
  □ Page in wiki/index.md
  □ Wikilinked from parent domain page
  □ Gap tracker synced
  □ wiki/log.md entry added
```

### Gate 3: Final Self-Check

```
□ Zero placeholder text in any touched page
□ Raw source documents NOT modified
□ Did not skip because "small change"
□ If unsure whether update needed → updated
```

### Report Format

```
Wiki: no update required (task type: refactoring|test-only|styling)
Wiki: updated {N} pages — {page1}, {page2}. Log entry added.
Wiki: BLOCKED — {reason}. Escalate to human.
```

---

## 9. TASK WORKFLOWS

### 9.1 Write New Feature

```
READ:    wiki/domains/{domain}.md → wiki/endpoints/{domain}.md →
         wiki/entities/{relevant}.md → wiki/gaps/gap-tracker.md
EXECUTE: Implement feature.
UPDATE:  endpoints page + entity page (if schema changed) +
         domain page (impl status) + gap-tracker (mark resolved) + log.md
```

### 9.2 Fix Bug

```
READ:    wiki/domains/{domain}.md
EXECUTE: Find and fix bug.
SCAN:    Global search for same anti-pattern in other files.
UPDATE:  If bug revealed gap → add to gap-tracker.
         If bug changed behavior → update domain/endpoint/entity page.
         If bug contradicted wiki claim → fix wiki claim.
         log.md → root cause, affected files, fix applied.
```

### 9.3 Recompile Wiki Page

```
TRIGGER: Raw source rewritten, >5 stale fields, architecture changed,
         human requests recompile.
EXECUTE: Re-read raw sources for that page.
         Cross-reference with current code state.
         Rewrite page completely.
UPDATE:  log.md (operation: recompile).
         Verify all outbound [[wikilinks]] resolve.
         Verify all inbound links from other pages still accurate.
```

### 9.4 Add New Domain

```
CREATE:  wiki/domains/{domain}.md (from schema template)
         wiki/endpoints/{domain}.md
         wiki/screens/{domain}.md (if has UI)
         wiki/entities/{entity}.md (for each entity)
ADD:     All pages to wiki/index.md
LINK:    Wikilinks from related domain pages
GAPS:    Add entries to wiki/gaps/gap-tracker.md
LOG:     Append to wiki/log.md
```

---

## 10. MULTI-SESSION COMPILATION (initial wiki build)

**Single-session full-codebase coverage is not feasible.** Agents hallucinate coverage when context fills up.

```
PHASE 1: Scaffold     → Directory structure, templates, index.md, log.md
          INPUT: schema.md only
          STOP. New session.

PHASE 2: Domains A    → First batch of domain pages + sub-pages
          INPUT: PRD + API contract + audit (batch 1 domains only)
          STOP. New session.

PHASE 3: Domains B    → Remaining domain pages + sub-pages
          INPUT: PRD + blueprints + audit (remaining domains)
          STOP. New session.

PHASE 4: Entities     → All entity pages
          INPUT: Server blueprint + code
          STOP. New session.

PHASE 5: Endpoints    → All endpoint group pages
          INPUT: API contract + code
          STOP. New session.

PHASE 6: Screens      → All screen group pages
          INPUT: App blueprint + code
          STOP. New session.

PHASE 7: Cross-cuts   → Cross-cutting pages + decision records
          INPUT: All sources (targeted reads, not full load)
          STOP. New session.

PHASE 8: Validation   → Broken link fix, orphan check, index finalization
          INPUT: wiki/ directory only
          DONE.
```

**FAIL conditions:**
- Agent continues past STOP boundary
- Agent reads sources not listed for current phase
- Agent produces placeholder pages
- Phase ends without log.md entry showing progress

---

## 11. AMBIGUITY RESOLUTION

```
IF missing parameter AND can infer from code/wiki:
  → Infer. Document assumption. Proceed.

IF missing parameter AND cannot infer:
  → Stop. Ask human. Do not guess.

IF wiki says X AND code says Y:
  → Code wins. Update wiki. Log discrepancy.

IF source A says X AND source B says Y:
  → Document both. Mark PENDING. Escalate to human.

IF wiki page is stale:
  → Re-read raw source. Update page. Set status: compiled. Log.

IF unsure whether wiki update is needed:
  → Update. False positive (unnecessary update) < false negative (stale wiki).
```

---

## 12. COMPRESSED VERSION (for CLAUDE.md embedding, ~250 tokens)

```markdown
## N. Wiki (Karpathy Pattern)

`wiki/` = compiled knowledge layer. Read wiki, not raw sources.

Links: `[[domain:X]]` `[[entity:X]]` `[[endpoints:X]]` `[[screens:X]]`
`[[decision:X]]` `[[cross-cuts:X]]` `[[gaps:X]]`

### Maintenance (mandatory)

1. **Update on discovery.** Gap fixed, impl status changed, entity/endpoint
   added, contradiction found → update wiki page BEFORE session end.
2. **Log every edit.** Append to `wiki/log.md`: date, pages, reason, source.
3. **Frontmatter stays current.** `updated:` on every touch. Stale →
   `status: stale`. Never delete → `status: superseded`.
4. **Link new pages.** Create per `wiki/schema.md` template, add to
   `wiki/index.md`, wikilink from parent.
5. **Gaps don't disappear.** Mark `✅ Resolved` + date + evidence. Keep row.
6. **Trace everything.** Claim → source path. Else `NOT VERIFIED`.
7. **Flag contradictions.** Both positions + file refs. `PENDING — escalate`.

**Don't**: read all raw sources upfront; leave placeholders; modify raw sources.
```
