# Drop-in CLAUDE.md Section (~250 tokens)

Copy everything below the line into your CLAUDE.md or AGENTS.md file.

---

## Wiki (Karpathy Pattern)

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
