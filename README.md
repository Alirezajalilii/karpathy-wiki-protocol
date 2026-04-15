# Karpathy Wiki Protocol — Agent-Optimized Rules for LLM-Maintained Knowledge Bases

> **The only Karpathy wiki implementation born from a production codebase (139 pages, 594 wikilinks, 425 audited gaps), not a weekend experiment.**

Most Karpathy wiki implementations are demos. They ingest 5 articles, generate pretty pages, and stop.

This repo is different. It's the **operational ruleset** — extracted from a real project where AI agents maintain a 139-page wiki across 22 domains, with a verification gate that catches every missed update. Every rule exists because an agent violated it in production and we had to write the rule to stop it from happening again.

---

## What This Is

A complete, copy-paste-ready protocol for making LLM agents (Claude Code, Codex, Cursor, any agent with filesystem access) build and maintain a Karpathy-style wiki. It's not a tool. It's not a CLI. It's a **set of rules** you drop into your `CLAUDE.md` or `AGENTS.md` and your agent immediately knows how to maintain your knowledge base.

## What This Is NOT

- Not a RAG pipeline
- Not an Obsidian plugin  
- Not a CLI tool
- Not a SaaS product

It's a prompt. A very battle-tested one.

---

## The Problem It Solves

Every new AI agent session on your codebase starts from zero. It re-reads your docs, re-derives relationships, re-discovers gaps. Thousands of tokens burned on initialization, every single time. Knowledge doesn't accumulate.

The Karpathy wiki pattern fixes this: a **compiled knowledge layer** (markdown files in `wiki/`) that agents read FIRST instead of your raw docs. But the pattern only works if the wiki stays maintained. That's where every implementation fails — the wiki rots because nobody defined the maintenance rules.

This repo defines those rules.

---

## Quick Start (5 minutes)

### Option A: Drop the Full Protocol Into Your Project

```bash
# Copy the protocol file to your project root
curl -O https://raw.githubusercontent.com/{YOUR_USERNAME}/karpathy-wiki-protocol/main/WIKI_PROTOCOL.md

# Or clone and copy
git clone https://github.com/{YOUR_USERNAME}/karpathy-wiki-protocol.git
cp karpathy-wiki-protocol/WIKI_PROTOCOL.md your-project/docs/
```

Then reference it from your `CLAUDE.md`:
```markdown
## Wiki Rules
See docs/WIKI_PROTOCOL.md for complete wiki maintenance protocol.
Compressed rules are in §5 below.
```

### Option B: Embed the Compressed Version (~250 tokens)

Copy the compressed version from [Section 12 of WIKI_PROTOCOL.md](./WIKI_PROTOCOL.md#12-compressed-version-for-claudemd-embedding-250-tokens) directly into your `CLAUDE.md`. This gives agents the 7 mandatory rules without loading the full protocol every session.

### Option C: Use Task-Specific Prompts

Copy individual sections as needed:
- **Building a wiki from scratch?** → Use [Section 10: Multi-Session Compilation](./WIKI_PROTOCOL.md#10-multi-session-compilation-initial-wiki-build)
- **Setting up verification gates?** → Use [Section 8: Post-Task Verification Gate](./WIKI_PROTOCOL.md#8-post-task-verification-gate)
- **Running a lint pass?** → Use [Section 3.3: Lint](./WIKI_PROTOCOL.md#33-lint-health-check)

---

## Setup Guide for AI Agents

### Step 1: Create the Wiki Directory Structure

Tell your agent:
```
Create the following directory structure:
wiki/
├── domains/        ← One page per feature domain
├── entities/       ← One page per database entity
├── endpoints/      ← Grouped by domain
├── screens/        ← Grouped by domain (if applicable)
├── decisions/      ← Architecture Decision Records
├── gaps/           ← Gap tracker + per-domain gap pages
├── cross-cuts/     ← Cross-cutting concerns (auth, i18n, errors)
├── index.md        ← Master catalog of all pages
├── log.md          ← Append-only operation history
└── schema.md       ← Page templates and conventions
```

### Step 2: Create the Schema File

Your `wiki/schema.md` defines page templates. See [examples/schema.md](./examples/schema.md) for a starter template.

### Step 3: Bootstrap Index and Log

```markdown
<!-- wiki/index.md -->
---
type: meta
updated: {TODAY}
---
# Wiki Index
Pages: 0 | Last compilation: {TODAY}

## Domains
(none yet)

## Entities
(none yet)
```

```markdown
<!-- wiki/log.md -->
---
type: meta
updated: {TODAY}
---
# Wiki Log

## [{TODAY}] init | Wiki scaffolded
Pages created: index.md, log.md, schema.md
```

### Step 4: Add Protocol to CLAUDE.md

Paste the [compressed version](./WIKI_PROTOCOL.md#12-compressed-version-for-claudemd-embedding-250-tokens) into your agent init file.

### Step 5: Run First Compilation Phase

Give your agent:
```
Read WIKI_PROTOCOL.md Section 10 (Multi-Session Compilation).
Execute Phase 1: Scaffold.
Read my source documents and create the wiki directory structure with
proper templates. Do NOT attempt to compile all content in one session.
Log progress to wiki/log.md. STOP when Phase 1 is complete.
```

Then start new sessions for each subsequent phase.

---

## What's in the Box

| File | Purpose | When to Use |
|------|---------|-------------|
| `WIKI_PROTOCOL.md` | Complete 12-section agent protocol | Reference doc. Load sections as needed per task. |
| `WIKI_PROTOCOL_V1.md` | Human-readable version with explanations | Reading/understanding the pattern. Not for agents. |
| `examples/schema.md` | Starter wiki schema template | Bootstrapping a new wiki |
| `examples/domain-template.md` | Domain page template with frontmatter | Creating new domain pages |
| `examples/entity-template.md` | Entity page template | Creating new entity pages |
| `examples/claude-md-snippet.md` | Drop-in CLAUDE.md section (~250 tokens) | Adding wiki rules to your agent init file |
| `docs/GROWTH.md` | How this repo got traction (for the curious) | Community building |

---

## Architecture at a Glance

```
┌─────────────────────────────────────┐
│  Raw Sources (IMMUTABLE)            │
│  docs/, PDFs, specs, blueprints     │
│  Agent: READ ONLY                   │
└──────────────┬──────────────────────┘
               │ compile
               ▼
┌─────────────────────────────────────┐
│  Wiki (AGENT-MAINTAINED)            │
│  wiki/domains, entities, endpoints  │
│  wiki/gaps, cross-cuts, decisions   │
│  wiki/index.md + wiki/log.md        │
│  Agent: READ + WRITE                │
└──────────────┬──────────────────────┘
               │ governs
               ▼
┌─────────────────────────────────────┐
│  Schema (CO-EVOLVED)                │
│  CLAUDE.md + wiki/schema.md         │
│  Human + Agent: defines structure   │
└─────────────────────────────────────┘
```

---

## Why This One Is Different

| Feature | Most Implementations | This Protocol |
|---------|---------------------|---------------|
| **Origin** | Weekend demo with 5 articles | Production codebase: 22 domains, 139 pages, 594 wikilinks |
| **Maintenance rules** | "Keep the wiki updated" | 7 binary rules with explicit PASS/FAIL conditions |
| **Verification** | None — trust the agent | Post-task gate: 8-point checklist runs after every task |
| **Multi-session** | "Do it all at once" | 8-phase compilation with hard STOP boundaries |
| **Ambiguity** | "Use your judgment" | Decision tree: infer/ask/escalate with explicit triggers |
| **Contradiction handling** | Silently pick one | Document both positions, flag PENDING, escalate to human |
| **Anti-corruption** | None | 9 NEVER rules + 8 ALWAYS rules, binary enforcement |
| **Format** | Prose paragraphs | Code blocks + tables. Zero prose in operational sections. |

---

## Battle-Tested Lessons

These rules exist because agents violated them:

1. **"Small changes don't need wiki updates"** — They do. Small changes compound into stale wikis. Rule 1 exists because of this.
2. **"Read all docs at session start just in case"** — Context suicide. Agents hallucinate the later docs. The wiki exists to prevent this.
3. **"Placeholder pages are fine, fill them later"** — "Later" never comes. No placeholders, ever.
4. **"The agent will remember to update the wiki"** — It won't. The verification gate (Section 8) forces it.
5. **"One session can compile the whole wiki"** — It can't. Agents silently truncate or hallucinate coverage. 8 phases, hard stops.
6. **"Contradictions should be resolved immediately"** — By the agent? No. Flag both positions, escalate to human.

---

## Contributing

Found a rule that's too loose? An edge case the protocol misses? A better way to structure the verification gate?

Open an issue or PR. The protocol is version-controlled — it evolves the same way the wiki it describes evolves.

---

## Credits

- **Pattern**: [Andrej Karpathy's LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) (April 2026)
- **Production implementation**: Built on PlanChin — a Persian-first mobile platform (22 domains, 139 wiki pages, 425 audited gaps)
- **Protocol design**: Iteratively refined across 8+ design sessions, compression passes, and real agent failures

---

## License

MIT — use it, fork it, adapt it to your project. The protocol is just markdown files.
