# assistant-knowledge-sync

An Agent Skill that keeps a site's AI chat assistant in step with the site. It finds what the
assistant is missing, moves page copy into one data module that the page and the assistant both
read, adds new pages to the assistant's knowledge pack and searchable corpus, builds its site map
from the sitemap's own sources, writes routing and "never quote" guardrails, and proves the result
with unit tests and live probes.

It is for assistants whose prompt is assembled in code from repository content (AI SDK or similar,
with search and read tools). It is not for building a chatbot, vector/RAG infrastructure or
fine-tuning.

## Install

Copy or clone this folder into your agent's skills directory, then ask: "update the assistant's
knowledge about <topic>".

## What's inside

| File | Contents |
|---|---|
| `SKILL.md` | Router: when to use, critical facts, hard rules, quick start |
| `references/surfaces.md` | The seven places assistant knowledge lives, seam table, host probe |
| `references/gap-audit.md` | What changed, the fact list, the baseline |
| `references/single-source.md` | Moving page copy into a shared data module |
| `references/pack-and-corpus.md` | Document kinds, pack sections, site map, live index join |
| `references/guardrails.md` | Routing, links, unpublished facts, kinds, tool descriptions, channels |
| `references/verification.md` | Unit tests, pack dump, render check, live probes |
| `references/provenance.md` | Where it came from, what it fixes, what it keeps |

## License

MIT
