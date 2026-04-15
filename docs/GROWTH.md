# Growth Strategy — How to Get Stars on This Repo

This isn't a "go viral" playbook. It's an honest assessment of what works
for developer tool repos in 2026, applied to this specific project.

---

## Your Competitive Position

The Karpathy wiki gist has 5,000+ stars and 4,000+ forks. Within one week,
20+ implementations appeared. Most are:

- CLI tools that wrap ingest/query/lint
- Obsidian plugins
- Full SaaS products
- Agent skill packages

**None of them are the rules.** They're all tools. Your repo is the only
one that provides the operational protocol — the actual agent instructions
that make the wiki maintenance work. That's your differentiation.

### Why This Matters

The #1 failure mode of Karpathy wikis is rot. The wiki stops being
maintained because nobody defined what "maintained" means. Your repo
defines it with binary PASS/FAIL rules, verification gates, and
explicit workflows. That's the thing people actually need.

---

## Traction Channels (ranked by effort/impact)

### 1. Karpathy Gist Comments (Effort: Low, Impact: High)

Post a comment on the original gist. Be useful, not promotional.

```
After implementing this pattern on a production codebase (139 pages,
22 domains), the biggest lesson was: the wiki rots without explicit
maintenance rules. We extracted the 7 rules that prevent this into
a standalone protocol: {link}

It's just markdown — drop it into CLAUDE.md and your agent knows
how to maintain the wiki. No tools, no CLI, no dependencies.
```

**Why this works:** 485+ comments, high engagement, people actively
looking for implementations. A non-promotional, experience-based
comment with a useful link gets attention.

### 2. Reddit r/ClaudeAI + r/LocalLLaMA (Effort: Low, Impact: Medium)

Post titled: "After 139 wiki pages, here are the 7 rules that prevent
your Karpathy wiki from rotting"

Lead with the lessons learned, not the repo. List the 7 rules. Link
the repo at the end as "full protocol with verification gates and
templates."

**Why this works:** Experience posts outperform "I built X" posts.
The number (139 pages) provides credibility. The rules provide
immediate value even without clicking the link.

### 3. X/Twitter Thread (Effort: Medium, Impact: Medium-High)

Structure:
```
Tweet 1: "We built a 139-page Karpathy wiki for a production app.
It rotted within 2 weeks. Here's what we learned: 🧵"

Tweet 2-4: The 3 worst failure modes (reading all docs at startup,
skipping small updates, one-session compilation)

Tweet 5-6: The fix — binary rules with PASS/FAIL, verification
gate after every task

Tweet 7: "Full protocol (just markdown, no tools): {link}"
```

**Why this works:** Failure stories get more engagement than success
stories. "I built X and it broke" is more relatable than "I built X
and it's great."

### 4. Hacker News (Effort: Low, Impact: Unpredictable)

Title: "Show HN: The 7 rules that prevent your LLM wiki from rotting"

Link directly to the repo. HN loves opinionated, minimal tools.
The fact that it's "just a prompt" (no CLI, no dependencies) is a
feature on HN, not a weakness.

### 5. Dev.to / Hashnode Blog Post (Effort: High, Impact: Medium)

Full write-up: "Building a 139-Page LLM Wiki: What Broke and How We
Fixed It." Include concrete examples of each failure mode and the
rule that prevents it. Embed code blocks from the protocol.

---

## Star Growth Patterns for This Type of Repo

**Week 1 (launch):** 20-50 stars from direct posting (gist comment,
Reddit, Twitter). This is your "is there demand?" signal.

**Month 1:** 100-300 stars if the content is genuinely useful and
gets picked up by a couple of newsletters or influential accounts.

**Month 2-3:** Plateau or compound. If people actually use the
protocol and reference it in their own repos/posts, it compounds.
If not, it plateaus.

**Honest expectation:** A well-positioned prompt/protocol repo in
this space should reach 200-500 stars in 3 months if the content
is good. 1,000+ requires either a celebrity signal boost or the
repo becoming a canonical reference that other implementations
link to.

---

## What Gets Stars vs. What Doesn't

| Gets Stars | Doesn't Get Stars |
|------------|-------------------|
| Opinionated rules with clear reasoning | Generic "best practices" lists |
| Concrete numbers (139 pages, 594 wikilinks) | Vague claims ("battle-tested") |
| Failure stories + fixes | "Everything works great" |
| Copy-paste ready files | "Adapt to your needs" without templates |
| Zero dependencies | Complex setup requirements |
| Active maintenance + issues engagement | Repo goes silent after launch |

---

## Post-Launch Maintenance

- Respond to every issue within 24 hours
- Accept PRs that add rules for edge cases you haven't covered
- Post updates when you refine rules based on real usage
- Add a "Testimonials" or "Used By" section when people reference it
- Cross-link from your PlanChin repo

---

## What NOT to Do

- Don't spam the Karpathy gist comments with self-promotion
- Don't buy fake stars (GitHub detects and removes them)
- Don't claim this is "the official" Karpathy implementation
- Don't oversell — "protocol" not "framework" or "platform"
- Don't add features that aren't rules (no CLI, no UI, no SaaS)
  The simplicity IS the product
