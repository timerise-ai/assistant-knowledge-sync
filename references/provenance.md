# Provenance

Extracted from a production marketing site whose assistant answers on three channels (a website
widget, a Discord bot and an MCP server for other AI agents) and in a sales-interview prompt, all fed by
one knowledge module. The update this skill generalizes taught the assistant a new catalog of 15+
entries with per-entry pages and agent test results, a partner program page, and a reworked sitemap.
Before the update, the assistant knew nothing about the program and its site map was missing the
catalog and program routes. After it, 13 of 13 live probes passed, three of them new.

## Fixed

### 1. Page copy unreachable by the assistant
The program page kept its promises, roles, steps, badges, permissions and FAQ as constants inside page
and component files. The assistant had no way to read them, so it had nothing to say about the
program. **Shipped:** the single-source refactor, [single-source.md](single-source.md).

### 2. Hand-written site map drifted from the sitemap
The assistant's site map was a hand-kept list. The sitemap gained the catalog, the program, the
articles and two legal pages; the assistant's list gained none of them. **Shipped:** a site map built
from the sitemap's arrays, [pack-and-corpus.md](pack-and-corpus.md).

### 3. Live index older than the synced content
The fetched index README said v0.1.5 for an entry whose synced changelog, and page, said v0.1.8.
**Shipped:** the join that prefers the page's version, [pack-and-corpus.md](pack-and-corpus.md).

### 4. Catalog pages not in the corpus
Entries were listed in the pack by name only; their requirements, rules and test results were on the
page but unreachable. **Shipped:** the `catalog` document kind.

### 5. Tool descriptions listed the old kinds
The search tool said "help center, blog, manifesto and case studies", so the model had no reason to
search for catalog or program pages. **Shipped:** [guardrails.md](guardrails.md), tool descriptions.

### 6. Links pointed off-site
The web prompt linked catalog entries to their GitHub repositories although each had a page on the
site. **Shipped:** the link-preference guardrail.

### 7. Unpublished numbers
Payout shares are not published. Without a line saying so, the model is free to estimate one.
**Shipped:** the "never quote" guardrail, with a probe that rejects any percentage.

### 8. Wrong kind, wrong offer
The prompt said every catalog entry builds a module for one framework. One entry is a tool for
maintainers with nothing to sell, and another targets a different platform. **Shipped:** the
kind-distinction guardrail and `catalogOffer()`.

### 9. Proper nouns lowercased
A pack line ran `.toLowerCase()` over whole sentences to fit them mid-line, which turned the company
name into lowercase in the prompt. Found while writing this skill. **Shipped:** the "print the pack and
read it" step in [verification.md](verification.md).

## Kept deliberately

- **Hand-written company facts stay in the prompt.** Identity, stance and what the company sells are
  not on any one page; generating them would mean inventing a source.
- **Full help-article bodies stay out of the pack.** The pack holds an index; bodies come through the
  read tool. The pack was ~49,000 characters after the update; inlining bodies would multiply it.
- **Keyword search, not embeddings.** A few hundred documents with good tags rank well enough, and
  there is no index to rebuild on deploy.
- **Regex probes, not an LLM judge.** Routes and numbers are what go wrong, and a regex catches them
  without a second model's opinion.

## Added (designed while extracting, not yet run on a second host)

- The host probe in [surfaces.md](surfaces.md) and the seam table.
- The baseline step in [gap-audit.md](gap-audit.md): in the source update, probes were written after
  the change; asking first is the better order and is recommended here.
- The gap-audit fact table.

## If you are applying this to an existing assistant

Fix order, most damaging first: unpublished numbers (7), routing (wrong form), tool descriptions (5),
then site map (2) and corpus (4), then versions (3) and links (6).
