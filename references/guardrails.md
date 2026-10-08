# Guardrails

Facts make the assistant informed; guardrails make it say the right thing. Every new page needs at
least a routing line and a link line, and anything not yet public needs a "never quote" line. These
live in the prompt module (surface 6) or in the pack section next to the facts they guard.

## The five kinds

| Kind | Prevents | Template |
|---|---|---|
| **Unpublished** | Invented numbers, dates, names | "Not published yet: `<what>` (published `<when>`). Never quote a percentage or an amount." |
| **Routing** | The right answer with the wrong next step | "Someone who wants to `<intent>` goes to `<route>`, not to `<other route>`. Someone who wants `<other intent>` goes to `<other route>`." |
| **Kind distinction** | Selling what is not for sale | "Entries marked `<marker>` are `<what they are>`; there is nothing to build or quote, so never send them to `<sales route>`." |
| **Link preference** | Sending visitors off-site when a page exists | "Link `<thing>` to its page, for example `[Name](/catalog/slug)`; link the code repository only as where the code lives." |
| **Scope** | "I can't help with that" for a page the site has | Add the new topic and its route to the allowed-topics list |

Write guardrails as instructions with the exact route, not as descriptions. "Partners apply at
/partners/apply" is a fact; "Someone who wants to join goes to /partners/apply, not to the brief" is a
guardrail the model follows.

## Tool descriptions

The model decides whether to search from the tool's description, not from what the corpus contains.
Whenever a document kind is added, update both:

```ts
// lib/ai/tools.ts
export const searchDescription =
  "Full-text search across the help center, blog, case studies, catalog entry pages and the partner program page. Use it before answering any how-to, feature, catalog or program question.";

export const readDescription =
  "Reads the full text of one document by its site path (for example /help/<category>/<slug>, /blog/<slug>, /catalog/<slug> or /partners). Use after search when you need exact steps or details.";
```

Also add a grounding line naming when to read the new kind directly: "For what a catalog entry
requires, includes or costs, read its page (/catalog/<slug>) and answer from it."

## Every channel

A guardrail added to the web prompt only is a guardrail missing on the others. Walk the channel list
from [surfaces.md](surfaces.md):

| Channel | What changes for a new destination |
|---|---|
| Web widget | Relative links: `[Partners](/partners)` |
| Chat app (Discord, Slack) | Absolute links on the site URL; length cap; never ask for contact details in a shared channel. Name the new audience if the community is part of it ("candidates for the partner program") |
| Agent protocol (MCP) | Absolute links; where the web says "go to the form", say which tool does it, if one exists |
| Interview or brief prompt | Usually inherits the pack; check it does not route program candidates into the sales interview |

## Wording rules for guardrails

- Name the route literally. The model copies it.
- Say "never" once, for the thing that must not happen, and give the alternative in the same line.
- Keep stance-level rules (tone, what the company is) in the hand-written facts, not repeated per page.
- If a guardrail contradicts a hand-written fact, fix the fact; two conflicting lines make the model pick.

## Checklist

- [ ] One routing line per new destination
- [ ] One "never quote" line per unpublished number
- [ ] Kind distinctions written for entries that are not for sale
- [ ] Link rules prefer the site's own page
- [ ] Allowed topics include the new topic
- [ ] Search and read tool descriptions name the new kind and an example path
- [ ] Every channel checked
