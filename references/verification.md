# Verification

Three layers, cheapest first. Unit tests prove the knowledge contains the fact. A render check proves
the refactor changed nothing visible. A live probe proves the model uses the fact.

| Layer | Proves | Runs |
|---|---|---|
| Typecheck + lint | The refactor and builders compile | Every change |
| Unit tests | Search ranks the new page; read returns the key line; pack holds the route and guardrail | Every change, CI |
| Render check | The page shows every section after single-sourcing | After a refactor |
| Live probe | The model answers from the fact and obeys the guardrail | After the change, against a dev server |

## Unit tests

One test per new kind of retrieval and one per guardrail line. Query with words a visitor types.

```ts
import { describe, expect, test } from "bun:test";
import { getKnowledgePack, readKnowledgeDocument, searchKnowledge } from "../knowledge";

describe("catalog and program", () => {
  test("finds a catalog page by what it builds", () => {
    const hits = searchKnowledge("en", "touchscreen kiosk on-screen keyboard", 3);
    expect(hits[0]?.url).toBe("/catalog/booking-kiosk");
    expect(hits[0]?.kind).toBe("catalog");
  });

  test("an entry that is not for sale says so", () => {
    expect(readKnowledgeDocument("en", "/catalog/eval-loop")?.content).toContain(
      "nothing to build or quote",
    );
  });

  test("finds the program page for partner questions", () => {
    expect(searchKnowledge("en", "partner program referrer ambassador", 3)[0]?.url).toBe("/program");
  });

  test("the pack lists the new routes and the guardrail", () => {
    const pack = getKnowledgePack("en");
    expect(pack).toContain("/program/apply");
    expect(pack).toContain("Never quote a percentage");
  });
});
```

## Printing the pack

Read what the model reads. A one-off script in the repository root (so path aliases resolve), deleted
afterwards:

```ts
// .dump.ts — run with `bun ./.dump.ts`, then delete it
import { getKnowledgePack, readKnowledgeDocument } from "@/lib/ai/knowledge"; // your knowledge module

const pack = getKnowledgePack("en");
console.log(pack.slice(pack.indexOf("#### PARTNER PROGRAM"), pack.indexOf("#### NEXT SECTION")));
console.log("PACK CHARS", pack.length);
console.log(readKnowledgeDocument("en", "/program")?.content.slice(0, 1500));
```

Look for: versions that disagree with the page, lowercase-mangled proper nouns from `.toLowerCase()`
on sentences, duplicated rows, and anything the page does not say.

## Render check

After single-sourcing, start the dev server and grep the HTML for one string from every section that
moved:

```bash
bun dev > /tmp/dev.log 2>&1 &
for i in $(seq 1 40); do curl -s -o /dev/null -w "%{http_code}" localhost:3000/program | grep -q 200 && break; sleep 2; done
curl -s localhost:3000/program | grep -o "Certify on one skill\|How are partners paid\|Merge to main" | sort | uniq -c
```

Every string must appear. A missing one is an array that did not get mapped.

## Live probes

A probe is a visitor question, regexes the answer must match, and regexes it must not. One probe per
new fact, one per guardrail. Keep them in the host's probe script next to the earlier ones; they are
regression tests for the next model or prompt change.

```js
const probes = [
  {
    name: "program",
    question:
      "I'm a freelance developer. Can I build modules for my own clients and get paid? How much?",
    expect: [/program/i, /\/program\/apply/i, /per project/i],
    reject: [/\d+\s?%/], // shares are unpublished: any percentage is an invention
  },
  {
    name: "not for sale",
    question: "Can you build the eval loop for me, and what would it cost?",
    expect: [/\/catalog\/eval-loop/i, /tool|maintainer/i],
    reject: [/\$\s?\d|usd\s?\d|\d+\s?usd/i],
  },
];
```

| Write | Avoid |
|---|---|
| Questions in the visitor's words | Questions that name the internal label ("tell me about the Builders section") |
| `expect` on routes and key terms | `expect` on whole sentences the model will paraphrase |
| `reject` on the invention you fear (numbers, wrong route, "we don't publish") | `reject` on words that appear in correct answers |

Run the full probe suite, not just the new probes: a new pack section can crowd out an old answer.

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
- [ ] Every probe passes, old and new
