# Provenance

The engineering ledger for the person editing this skill. It separates what the audit of the earlier
implementation changed and how the procedure now verifies it, what was kept deliberately and why it is safe,
and what was designed in the skill and has never run. Read it before simplifying anything; add an entry for
anything you change.

The earlier implementation was the assistant of a marketing site, answering on more than one channel from
one knowledge module. The update this skill is written from taught it a new catalog with a page per entry, a
partner program page and a reworked sitemap. Nine entries below were fixed, four kept, and seven added.

## Fixed

### 1. Page copy unreachable by the assistant
The program page kept its promises, roles, steps, badges, permissions and FAQ as constants inside page
and component files. The assistant had no way to read them, so it had nothing to say about the
program. **Now:** the single-source refactor in [single-source.md](single-source.md); the render check in
[verification.md](verification.md) proves the page unchanged, and a unit test proves the pack holds the
moved copy.

### 2. Hand-written site map drifted from the sitemap
The assistant's site map was a hand-kept list. The sitemap gained the catalog, the program, the
articles and the legal pages; the assistant's list gained none of them. **Now:** `siteMapLines` built
from the sitemap's arrays in [pack-and-corpus.md](pack-and-corpus.md), and the sitemap comparison table in
[gap-audit.md](gap-audit.md).

### 3. Live index older than the synced content
The fetched index README listed an older version of an entry than its synced changelog and its page.
**Now:** `joinedRows`, which prefers the page's version, in [pack-and-corpus.md](pack-and-corpus.md).

### 4. Catalog pages not in the corpus
Entries were listed in the pack by name only; their requirements, rules and details were on the
page but unreachable. **Now:** the `catalog` document kind and `catalogDocument`, with a unit test that
finds an entry by what it does.

### 5. Tool descriptions listed the old kinds
The search tool named only the help center, the blog and case studies, so the model had no reason to
search for catalog or program pages. **Now:** the tool description step in [guardrails.md](guardrails.md),
and hard rule 3.

### 6. Links pointed off-site
The web prompt linked catalog entries to their source repositories although each had a page on the
site. **Now:** the link-preference guardrail, with a probe that expects the site path.

### 7. Unpublished numbers
Payout shares are not published. Without a line saying so, the model is free to estimate one.
**Now:** the "never quote" guardrail, with a probe that rejects any percentage.

### 8. Wrong kind, wrong offer
The prompt said every catalog entry was something to buy and build. Some entries had nothing to sell, and
one targeted a different platform. **Now:** the kind-distinction guardrail and `catalogOffer()`, which
states the offer exactly as the page does, with a unit test on an entry that is not for sale.

### 9. Proper nouns lowercased
A pack line ran `.toLowerCase()` over whole sentences to fit them mid-line, which lowercased the company
name in the prompt. Found while writing this skill. **Now:** the "print the pack and read it" step in
[verification.md](verification.md).

## Kept deliberately

- **Hand-written company facts stay in the prompt.** Identity, stance and what the company sells are
  not on any one page; generating them would mean inventing a source.
- **Full help-article bodies stay out of the pack.** The pack holds an index; bodies come through the
  read tool. Inlining bodies would multiply the pack on every turn, which is hard rule 5.
- **Keyword search, not embeddings.** A few hundred documents with good tags rank well enough, and
  there is no index to rebuild on deploy.
- **Regex probes, not an LLM judge.** Routes and numbers are what go wrong, and a regex catches them
  without a second model's opinion.

## Added (designed in the skill, not yet run on a second host)

- The host probe in [surfaces.md](surfaces.md) and the seam table and rename table in
  [adaptation.md](adaptation.md).
- The baseline step in [gap-audit.md](gap-audit.md): in the earlier implementation, probes were written
  after the change; asking first is the better order and is recommended here.
- The gap-audit fact table.
- The probe runner `scripts/probe.mjs` with a host-supplied `ask`. The earlier implementation had its own
  probe script; this one is a minimal shape for a host without one.
- The vitest config for a host with no test runner; the earlier implementation ran its tests under bun.
- The rule for a run with no reachable model: probes written, reported as not run, never as passing.
- The build-output variant of the render check for a host where no dev server can be started.

## If you are applying this to an existing assistant

Fix order, most damaging first: unpublished numbers (7), routing (wrong form), tool descriptions (5),
then site map (2) and corpus (4), then versions (3) and links (6).
