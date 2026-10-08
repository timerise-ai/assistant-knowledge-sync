# Verification

Three layers, cheapest first. Unit tests prove the knowledge contains the fact. A render check proves
the refactor changed nothing visible. A live probe proves the model uses the fact.

| Layer | Proves | Runs |
|---|---|---|
| Typecheck and lint | The refactor and builders compile | Every change |
| Unit tests | Search ranks the new page; read returns the key line; pack holds the route and guardrail | Every change, CI |
| Render check | The page shows every section after single-sourcing | After a refactor |
| Live probe | The model answers from the fact and obeys the guardrail | After the change, against a running assistant |

## Unit tests

One test per new kind of retrieval and one per guardrail line. Query with words a visitor types. The
calls follow the knowledge interface of [adaptation.md](adaptation.md); adjust them to the host's
signatures, never the other way round.

```ts
// lib/ai/knowledge.test.ts
import { describe, expect, test } from "vitest";
import { getKnowledgePack, readKnowledgeDocument, searchKnowledge } from "./knowledge";

describe("catalog and program", () => {
  test("finds a catalog page by what it does", () => {
    const hits = searchKnowledge("en", "embed a booking calendar on my site", 3);
    expect(hits[0]?.url).toBe("/catalog/booking-widget");
    expect(hits[0]?.kind).toBe("catalog");
  });

  test("an entry that is not for sale says so", () => {
    expect(readKnowledgeDocument("en", "/catalog/starter-template")?.content).toContain(
      "nothing to build or quote",
    );
  });

  test("finds the program page for partner questions", () => {
    expect(searchKnowledge("en", "partner program referrer", 3)[0]?.url).toBe("/partners");
  });

  test("the pack lists the new routes and the guardrail", () => {
    const pack = getKnowledgePack("en");
    expect(pack).toContain("/partners/apply");
    expect(pack).toContain("Never quote a percentage");
  });
});
```

The host's runner runs it: `bun test` takes the `vitest` import as its own, and a vitest host runs it as it
is. A host with no runner gets vitest (`npm i -D vitest`, the package registry is not an external service),
a `"test": "vitest run"` script, and this config so the `@/` alias resolves:

```ts
// vitest.config.ts
import { fileURLToPath } from "node:url";
import { defineConfig } from "vitest/config";

export default defineConfig({
  resolve: { alias: { "@": fileURLToPath(new URL("./", import.meta.url)) } },
});
```

## Printing the pack

Read what the model reads. A one-off script in the repository root (so path aliases resolve), deleted
afterwards. The two markers are the headings of the new pack section and the one after it.

```ts
// .dump.ts, run with `bun ./.dump.ts` or `npx tsx ./.dump.ts`, then delete it
import { getKnowledgePack, readKnowledgeDocument } from "@/lib/ai/knowledge";

const pack = getKnowledgePack("en");
console.log(pack.slice(pack.indexOf("#### PARTNER PROGRAM"), pack.indexOf("#### NEXT SECTION")));
console.log("PACK CHARS", pack.length);
console.log(readKnowledgeDocument("en", "/partners")?.content.slice(0, 1500));
```

Look for: versions that disagree with the page, proper nouns lowercased by `.toLowerCase()` run over
whole sentences, duplicated rows, and anything the page does not say.

## Render check

After single-sourcing, start the dev server and grep the HTML for one string from every section that
moved:

```bash
npm run dev > .dev.log 2>&1 &
for i in $(seq 1 40); do curl -s -o /dev/null -w "%{http_code}" localhost:3000/partners | grep -q 200 && break; sleep 2; done
curl -s localhost:3000/partners | grep -o "Paid per project\|How are partners paid\|Referrer" | sort | uniq -c
kill %1; rm .dev.log
```

Every string must appear. A missing one is an array that did not get mapped. Where no server can be
started, `npm run build` and a grep of the prerendered HTML under `.next/server/app/` is the same check for a
static page.

## Live probes

A probe is a visitor question, regexes the answer must match, and regexes it must not. One probe per
new fact, one per guardrail. Keep them in the host's probe script next to the earlier ones; they are
regression tests for the next model or prompt change. A host without one gets this script; `ask` is the
host's, sending one question to the running assistant and resolving to the answer text.

```js
// scripts/probe.mjs
import { ask } from "./ask.mjs";

const probes = [
  {
    name: "program",
    question: "I run a small agency. Can I refer clients to you and get paid? How much?",
    expect: [/partner/i, /\/partners\/apply/i, /per project/i],
    reject: [/\d+\s?%/], // shares are unpublished: any percentage is an invention
  },
  {
    name: "not for sale",
    question: "Can you build the starter template for me, and what would it cost?",
    expect: [/\/catalog\/starter-template/i, /free/i],
    reject: [/\$\s?\d|usd\s?\d|\d+\s?usd|eur\s?\d|\d+\s?eur/i],
  },
];

let failed = 0;
for (const probe of probes) {
  const answer = await ask(probe.question);
  const missing = probe.expect.filter((re) => !re.test(answer));
  const found = probe.reject.filter((re) => re.test(answer));
  const ok = missing.length === 0 && found.length === 0;
  if (!ok) failed += 1;
  console.log(`${ok ? "PASS" : "FAIL"} ${probe.name}`, ok ? "" : { missing, found, answer });
}
process.exit(failed === 0 ? 0 : 1);
```

| Write | Avoid |
|---|---|
| Questions in the visitor's words | Questions that name the internal label ("tell me about the Partners section") |
| `expect` on routes and key terms | `expect` on whole sentences the model will paraphrase |
| `reject` on the invention you fear (numbers, wrong route, "we don't publish") | `reject` on words that appear in correct answers |

Run the full probe suite, not just the new probes: a new pack section can crowd out an old answer.

**When no model is reachable** (no key in the environment, an unattended run), write the probes and run
every other layer. The handover lists each probe as written and not run, with the command that runs them.
A probe is never reported as passing on a run that did not happen.

## Run order

1. Typecheck, lint, format.
2. Unit tests.
3. Print the pack; read the new sections.
4. Dev server: render check (if refactored), then the full probe suite.
5. Stop the dev server.
6. Two commits: the refactor, then the knowledge change with tests and probes.

## Checklist

- [ ] Typecheck and lint clean on touched files
- [ ] New unit tests pass, old ones still pass
- [ ] Pack printed and read; size compared with the baseline
- [ ] Render check passed for every moved section
- [ ] Every probe passes, old and new, or is listed as not run with the reason
