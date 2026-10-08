# Adaptation

The seam contract with the host app: what the host must already have, the names this skill uses and how they
map to the host's, where the host's storage, i18n and styling meet the skill, the order of work, and the
non-negotiables. Fill the seam table before editing anything; every other reference uses its left-hand names.

## What the host must already have

- **An assistant whose system prompt is assembled in code** from strings and functions in the repository.
  An assistant configured only in a vendor dashboard has no seam to work on.
- **A Next.js App Router site** whose pages render from components, markdown or data files in the same
  repository, and an `app/sitemap.ts` (or equivalent) that maps over arrays of routes.
- **Optionally, retrieval tools**: a search tool and a read tool the model calls, backed by an in-repo corpus.
  Without them the skill still applies to surfaces 1, 2 and 6 of [surfaces.md](surfaces.md); skip the corpus
  steps and say so in the handover.

## The seam table

| Seam | The skill calls it | Canonical path | The host supplies |
|---|---|---|---|
| Assistant name | `<assistant>` | | the name visitors see in the widget |
| Prompt module | `prompts` | `lib/ai/prompts.ts` | the file that exports the system prompt builders |
| Knowledge module | `knowledge` | `lib/ai/knowledge.ts` | the file that builds corpus, pack and context |
| Tools module | `tools` | `lib/ai/tools.ts` | the file that declares the search, read and list tools |
| Live index module | `liveIndex` | `lib/ai/live-index.ts` | the fetcher with its static fallback, if any |
| Sitemap source | `sitemap` | `app/sitemap.ts` | the file that generates `sitemap.xml` |
| Data modules | `data/*` | `data/program.ts` | plain-text arrays that pages import |
| Probe script | `probe` | `scripts/probe.mjs` | the script that asks the running assistant questions |
| Unit tests | `knowledge.test` | `lib/ai/knowledge.test.ts` | tests next to the knowledge module |
| Project docs line | `docs` | | the line in `CLAUDE.md` or `README.md` that lists what the assistant knows; never absent, added to `README.md` when missing |

Mark a seam the host does not have as absent; never invent a file to fill the column. A host with no probe
script gets one, and a host with no transport to the assistant gets the `ask` stub, both from
[verification.md](verification.md). The canonical path names a role, not a file to create: a host whose data
module is `data/partners.ts` uses that path, and never adds `data/program.ts` beside it.

## The knowledge interface

The templates call three functions the host's knowledge module already exposes, under these names or the
host's own:

| Function | Signature | Returns |
|---|---|---|
| `searchKnowledge` | `(locale: string, query: string, limit: number) => KnowledgeDocument[]` | ranked documents |
| `readKnowledgeDocument` | `(locale: string, url: string) => KnowledgeDocument \| undefined` | one document by site path |
| `getKnowledgePack` | `(locale: string) => string` | the text appended to the system prompt |

A host whose functions take other arguments keeps them; the tests in [verification.md](verification.md) are
adjusted to the host's signature, never the host to the tests.

## Rename table

| Canonical | Meaning | Host renames to |
|---|---|---|
| `KnowledgeKind`, `KnowledgeDocument` | the corpus document type and its kinds | the host's existing type, extended |
| `catalog` kind, `CatalogEntry`, `catalogOffer`, `catalogDocument` | per-entry pages of a listed set (products, plans, templates, integrations) | the host's catalog vocabulary |
| `offer` and `free` entry kinds | an entry that is sold, and one that is not | the host's distinction, if it has one |
| `page` kind, `programDocument`, `programLines` | a standalone page with substance | one builder per page |
| `PROGRAM_NAME`, `PROGRAM_PROMISES`, `PROGRAM_ROLES`, `PROGRAM_FAQ` | the moved copy of one page | `<PAGE>_*` per page |
| `/partners`, `/partners/apply` | the example page and its form | the host's routes |
| `siteMapLines`, `joinedRows` | the site map and the live index join | the host's pack builders |

The authoring contract is not renamed: the five hard rules, the probe shape `{ name, question, expect,
reject }`, and the order refactor commit first, knowledge commit second. On an unattended run the operator
makes the two commits from the handover's file lists; the agent commits nothing and creates no branch.

## Integration points

| Concern | Where it meets the skill |
|---|---|
| Storage | None. Content ships in the repository with each deploy; the only fetch is the live index, cached with a static fallback |
| i18n | Every knowledge function takes a locale. A data module with translated copy exports one object per locale, or holds message keys the page and the knowledge module resolve through the same i18n helper |
| Styling | Stays in the page. Data modules hold words only, so icons, colours and art attach in the component |
| Auth and tenancy | Out of scope. The assistant answers from public site content; an assistant that reads per-user data is not this skill |
| Channels | Every caller of the knowledge context: web widget, chat app bot, MCP server, interview prompt. Each gets the guardrails of [guardrails.md](guardrails.md) |

## Order of work

1. Probe the host and fill the seam table ([surfaces.md](surfaces.md)).
2. Audit the gap and run the baseline probes ([gap-audit.md](gap-audit.md)).
3. Single-source the copy, as its own commit ([single-source.md](single-source.md)).
4. Extend corpus, pack, site map and live index ([pack-and-corpus.md](pack-and-corpus.md)).
5. Write the guardrails on every channel ([guardrails.md](guardrails.md)).
6. Verify, then commit the knowledge change ([verification.md](verification.md)).
7. Update the `docs` line, or add it to `README.md` when the host has none.

## The non-negotiables

1. **Never retype page copy into the prompt.** Import it from the module the page renders.
2. **Never rewrite the copy while moving it.** The refactor is byte-for-byte and the page renders identically.
3. **Never add a document kind without updating the tool descriptions and link rules.**
4. **Never ship without a probe per new fact and one per guardrail.**
5. **Never let the pack grow unbounded.** Indexes and summaries in the pack, full bodies behind the read tool.
