# Corpus, pack, site map and live index

The knowledge module has two outputs. The **corpus** is a list of documents the search and read tools
reach. The **pack** is a compact text appended to the system prompt on every turn. A new page usually
needs both: a corpus document for detail, and a short pack section plus a site map line so the
assistant knows the page exists without searching.

## The document shape

```ts
export type KnowledgeKind = "help" | "blog" | "case-study" | "catalog" | "page";

export interface KnowledgeDocument {
  kind: KnowledgeKind;
  title: string;
  /** Site path, exactly as the router serves it: the read tool resolves by this. */
  url: string;
  /** The page's meta description or tagline, verbatim. */
  summary: string;
  /** Words a visitor types that the title may not contain: synonyms, role names, the kind. */
  tags: string[];
  /** Plain text the read tool returns; sections separated by blank lines. */
  content: string;
}
```

| Kind | One per | Content built from |
|---|---|---|
| `catalog` | catalog entry page (`/catalog/<slug>`) | the same parsed object the page renders: problem, when to use, rules, requirements, install line, offer, test results, releases |
| `page` | standalone page with real substance (a program, a pricing page) | the page's data module, section by section in reading order |

Tags are the cheapest ranking lever. A program page whose title is its brand name ("Example Builders")
will lose to a blog post for the query "partner program referrer" unless those words are tags.

## Building catalog and page documents

```ts
interface CatalogEntry {
  slug: string;
  name: string;
  repoUrl: string;
  category: string;
  /** `tool` entries are used by an agent on something else; there is nothing to sell. */
  kind: "module" | "tool";
  tagline: string;
  problem: string;
  whenToUse: string[];
  requirements: string[];
  version: string | null;
  priceFrom: number | null;
  currency: string;
  days: number | null;
  results: Array<{ agent: string; result: "pass" | "partial" | "fail" }>;
}

/** The offer exactly as the page states it; never a number the page does not show. */
export function catalogOffer(entry: CatalogEntry): string {
  if (entry.kind === "tool") return "A tool, not a module: nothing to build or quote.";
  if (entry.priceFrom === null) return "Built on a brief; the price comes from the brief.";
  const days = entry.days ? `, about ${entry.days} working days` : "";
  return `From ${entry.priceFrom.toLocaleString("en-US")} ${entry.currency}${days}.`;
}

export function catalogDocument(entry: CatalogEntry): KnowledgeDocument {
  const block = (title: string, lines: string[]) =>
    lines.length > 0 ? `${title}\n${lines.map((l) => `- ${l}`).join("\n")}` : "";
  return {
    kind: "catalog",
    title: `${entry.name} (catalog)`,
    url: `/catalog/${entry.slug}`,
    summary: entry.tagline,
    tags: ["catalog", entry.slug, entry.category, entry.kind],
    content: [
      `Source: ${entry.repoUrl}. Category: ${entry.category}.`,
      entry.version ? `Latest release: v${entry.version}.` : "",
      catalogOffer(entry),
      entry.problem,
      block("When to use:", entry.whenToUse),
      block("Requirements:", entry.requirements),
      block(
        "Test results:",
        entry.results.map((r) => `${r.agent}: ${r.result}`),
      ),
    ]
      .filter(Boolean)
      .join("\n\n"),
  };
}
```

Name the local helper `block`, not `section`: knowledge modules usually already have a `section()`
that reads i18n copy, and shadowing it compiles but confuses the next reader.

The program page document reads the data module from [single-source.md](single-source.md):

```ts
import { PROGRAM_FAQ, PROGRAM_NAME, PROGRAM_PROMISES, PROGRAM_ROLES } from "@/data/program";

export function programDocument(): KnowledgeDocument {
  const stages = PROGRAM_ROLES.flatMap((role) =>
    role.stages.map((s) => `- ${s.name} (${role.kind}): ${s.who}. Can: ${s.can}. Entry: ${s.entry}.`),
  );
  return {
    kind: "page",
    title: PROGRAM_NAME,
    url: "/program",
    summary: `${PROGRAM_NAME} is our partner program.`,
    tags: ["partner program", "partners", ...PROGRAM_ROLES.flatMap((r) => r.stages.map((s) => s.name))],
    content: [
      PROGRAM_PROMISES.map((p) => `- ${p.title}: ${p.body}`).join("\n"),
      `Roles:\n${stages.join("\n")}`,
      `FAQ:\n${PROGRAM_FAQ.map((f) => `- Q: ${f.q}\n  A: ${f.a}`).join("\n")}`,
    ].join("\n\n"),
  };
}
```

Register both in the corpus builder next to the existing kinds, then check the search scorer weights
title, tags and summary above body hits. A typical scorer: title 8, tags 5, summary 3, body occurrences
capped at 10, ×1.5 when every query token matched.

## The pack section

Short: what it is, the destinations, the guardrails. Detail stays in the corpus document; point to it.

```ts
export function programLines(): string {
  return [
    `- ${PROGRAM_NAME}: our partner program. Page: /program; form: /program/apply.`,
    `- ${PROGRAM_PROMISES.map((p) => `${p.title}: ${p.body}`).join(" ")}`,
    `- Not published yet: the payout shares. Never quote a percentage or an amount.`,
    `- Someone who wants to join goes to /program/apply, not to the sales brief.`,
    `- Full page text: read "/program".`,
  ].join("\n");
}
```

## The site map, from the sitemap's own sources

Never maintain a second list of routes. Import the arrays the sitemap maps over (articles, products,
catalog categories) and write one line per destination, with what a visitor finds there.

```ts
interface Article { slug: string; href: string }

export function siteMapLines(articles: readonly Article[], categories: readonly string[]): string {
  return [
    `- Catalog (search and a category filter: ${categories.join(", ")}): /catalog; one page per entry at /catalog/<slug>`,
    `- Partner program: /program, apply at /program/apply`,
    `- Articles: ${articles.map((a) => a.href).join(", ")}`,
    `- Legal: ${["/privacy", "/terms"].join(", ")}`,
  ].join("\n");
}
```

When the router maps content slugs to different routes (`ai-delivery` → `/ai-systems`), derive that
map from the same data array too; a hand-kept `Record<slug, route>` is the next drift.

## Joining a live index with local facts

When a list is fetched from another repository and also synced into this one, the fetched list knows
what exists, and the local files know what the page shows. Join them and let the page win on facts it
renders.

```ts
interface IndexRow { name: string; url: string; version?: string; builds: string }

export function joinedRows(index: IndexRow[], local: Map<string, CatalogEntry>): string {
  return index
    .map((row) => {
      const page = local.get(row.name);
      // The synced changelog can be ahead of the index README; show what the page shows.
      const version = page?.version ?? row.version;
      const facts = page ? ` [${page.category}; page: /catalog/${page.slug}]` : "";
      const offer = page ? ` ${catalogOffer(page)}` : "";
      return `- **${row.name}**${version ? ` v${version}` : ""} (${row.url})${facts}: ${row.builds}.${offer}`;
    })
    .join("\n");
}
```

Rows the index lists but the site has no page for stay in (they exist), without a page link.

## Size and caching

| Budget | Guidance |
|---|---|
| Pack | Measure before and after. Indexes are one line per item; growth beyond ~20% for one feature means detail belongs in the corpus |
| Corpus document | Capped by the read tool (e.g. 14,000 characters, with a truncation marker) |
| Memoization | Corpus and pack are cached per locale for the life of the server instance; content ships with deploys, so no invalidation is needed |
| Live index | Cached in memory for about an hour, with a fetch timeout of a few seconds and a static fallback |

## Checklist

- [ ] New kinds added to `KnowledgeKind` and registered in the corpus builder
- [ ] Tags include the words visitors use, not only the brand name
- [ ] Pack section is short and points to the corpus document
- [ ] Site map built from the sitemap's arrays; every new route present
- [ ] Live index joined with local facts; page version wins
- [ ] Pack size measured; growth justified
