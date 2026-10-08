---
name: assistant-knowledge-sync
description: >
  Update what a site's AI chat assistant knows after the site changes: find
  the gap between the pages and the assistant, move page copy into one data
  module both read, add the new pages to the assistant's knowledge pack and
  searchable corpus, derive its site map from the sitemap source, write the
  routing and "never quote" guardrails, and prove it with unit tests and live
  probes. Use when: (1) a page, section, program, catalog or route was added or
  rewritten and the assistant should know about it, (2) the assistant answers
  with stale facts, wrong links, invented prices or sends people to the wrong
  form, (3) the user mentions: update Tim's knowledge, update the assistant's
  knowledge, teach the chatbot about, the bot doesn't know about the new page,
  sync the AI assistant with the site, knowledge pack, system prompt facts,
  site map for the assistant, the bot invents prices, probe the assistant.
  Carries the seven surfaces where assistant knowledge lives, the single-source
  refactor that keeps page and assistant from drifting, and the failure modes
  found doing this on a production marketing site. For an assistant whose
  prompt is assembled in code from repository content (AI SDK or similar, with
  search/read tools); the host's prompt, knowledge, tools and probe modules are
  the seam. Not for building a chatbot, not vector/RAG infrastructure, not
  fine-tuning, and not for writing the page copy itself.
---

# Assistant knowledge sync

A site's assistant goes stale the day a page ships without it. The fix is not a longer prompt: **the
assistant must read what the page renders, never a retyped copy of it.** Every fact gets one source of
truth in the repository, the page and the assistant both import it, and a probe proves the assistant
uses it.

## When to use

- A page, section, partner program, product, catalog entry or route was added or rewritten.
- The sitemap changed and the assistant's site map did not.
- The assistant gives stale versions, wrong links, invented numbers, or routes people to the wrong form.
- After a content sync (a catalog pulled from another repository) the assistant still lists the old one.

## When NOT to use

- **Building the assistant, its widget or its transport**: `ai-sdk` (chat on the site) or `slack-ai-bot`.
- **Embeddings, a vector store or hosted RAG**: this skill keeps an in-repo pack and keyword corpus.
- **Writing or editing the page copy**: the host's content workflow; this skill moves copy, never rewrites it.
- **Changing the model, provider or failover**: the host's AI config.
- **A one-word fact fix in the hand-written prompt**: just edit it; the full procedure is for new surfaces.

## Architecture

```
 repo content ──────────────┐            page.tsx ◄── data module (plain text)
 (markdown, i18n, data/*.ts) │                               │
                             ▼                               ▼
                 ┌───────── knowledge module ─────────────────────┐
                 │ corpus: docs {kind,title,url,summary,content}   │──► search / read tools
                 │ pack:   sections + site map + indexes           │──► system prompt
                 │ live:   external index ⨝ local facts           │──► system prompt / list tool
                 └─────────────────────────────────────────────────┘
 prompt module: hand-written facts + routing/link/"never quote" rules, per channel
 probe script:  question → expect[] / reject[] regexes against the running assistant
```

## Critical facts

1. **Tool descriptions gate retrieval.** If the search tool's description lists "help, blog, case
   studies", the model will not search for a skill page you just added to the corpus. Update the text.
2. **A hand-written site map drifts.** Build it from the same arrays and routes the sitemap uses.
3. **Copy inline in a TSX file is invisible to the assistant.** It has to move to a data module first.
4. **An external index can lag the synced content.** When both exist, show the version the page shows.
5. **The model fills silence with numbers.** Anything not yet published (payouts, prices, dates) needs
   an explicit "not published; never quote" line, or it will be invented.
6. **Routing is knowledge too.** "Who goes where" (join vs. buy vs. support) belongs next to the facts.

## Hard rules

> **Never retype page copy into the prompt.** Import it. A retyped copy is correct for one deploy.

> **Never rewrite the copy while moving it.** A single-source refactor is byte-for-byte; the page must
> render identically. Wording changes are a separate commit and a separate decision.

> **Never add a document kind without updating the tool descriptions and link rules.**

> **Never ship without a probe per new fact and one per guardrail.** Unit tests prove the pack
> contains the line; only a live probe proves the model uses it.

> **Never let the pack grow unbounded.** Indexes and summaries go in the pack; full bodies stay behind
> the read tool. Measure the pack's size before and after.

## Quick start

1. **Probe the host**: find the seven surfaces and fill the seam table —
   [surfaces.md](references/surfaces.md).
2. **Audit the gap**: what changed, which facts are new, what the assistant says today —
   [gap-audit.md](references/gap-audit.md).
3. **Single-source the copy** that lives only in components —
   [single-source.md](references/single-source.md).
4. **Extend corpus, pack, site map and live index** —
   [pack-and-corpus.md](references/pack-and-corpus.md).
5. **Write the guardrails**: routing, links, unpublished facts, kinds, tool descriptions, every
   channel — [guardrails.md](references/guardrails.md).
6. **Verify**: typecheck, lint, unit tests, page render, live probes —
   [verification.md](references/verification.md).
7. **Update the project docs line** that says what the assistant knows, then commit the refactor and
   the knowledge change as two commits.

## Reference directory

| Scenario | Trigger keywords | Reference |
|---|---|---|
| Where the assistant's knowledge lives in this host | system prompt, knowledge pack, corpus, tools, channel, Discord, MCP, seam | [surfaces.md](references/surfaces.md) |
| What the assistant is missing | stale, doesn't know, new page, git log, sitemap diff, baseline | [gap-audit.md](references/gap-audit.md) |
| Page copy only exists inside a component | inline constants, FAQ array, roles, cards, data module, drift | [single-source.md](references/single-source.md) |
| Making a page searchable and listed | KnowledgeDocument, kind, searchKnowledge, readKnowledgeDocument, site map, live index, version | [pack-and-corpus.md](references/pack-and-corpus.md) |
| The assistant says the wrong thing | invents prices, wrong form, wrong link, never quote, tool vs module, allowed topics | [guardrails.md](references/guardrails.md) |
| Proving it | bun test, probe, expect, reject, regex, dev server, render check | [verification.md](references/verification.md) |
| Where this came from | provenance, fixed, kept deliberately | [provenance.md](references/provenance.md) |
