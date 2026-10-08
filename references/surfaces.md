# The seven surfaces

An assistant's knowledge is spread over seven places. Each fact belongs in exactly one of them. Most
staleness comes from a fact that lives in the wrong one, usually hand-written into the prompt when it
should have been generated from content.

| # | Surface | Holds | Changes when | Who writes it |
|---|---|---|---|---|
| 1 | **Hand-written prompt facts** | Identity, positioning, stance, what the company sells, things no page states | The business changes | A person, rarely |
| 2 | **Generated pack** | Sections built from site copy: FAQ, pricing, process, program summaries, indexes, site map | Any deploy | Code, from content |
| 3 | **Searchable corpus** | One document per page with a body: help, blog, case studies, catalog entries, program pages | Any deploy | Code, from content |
| 4 | **Live external index** | A list maintained in another repository (a skills or plugins index) | Without a deploy | Fetched, cached, with a static fallback |
| 5 | **Tool descriptions** | What the search/read/list tools cover, in words the model reads | A document kind is added | A person, with every new kind |
| 6 | **Routing and link rules** | Who goes where, link style, allowed topics, "never quote" lines | A new form, program or audience | A person, with every new destination |
| 7 | **Channel variants** | Link format and memory rules per channel: web (relative links), chat apps (absolute links, length cap), agent protocol (MCP; plain facts, tools instead of forms), interview/brief prompt | A channel is added | A person |

**Rule of thumb:** if a visitor could read the fact on a page, it belongs in 2 or 3, generated. If it
says how to behave, it belongs in 6. Only identity and stance belong in 1.

## The seam table

Fill the right-hand column before editing anything. Every later step refers to these names.

| Seam | The skill calls it | The host supplies |
|---|---|---|
| Assistant name | `<assistant>` | e.g. Tim |
| Prompt module | `prompts` | the file that exports the system prompt builders |
| Knowledge module | `knowledge` | the file that builds corpus, pack and context |
| Tools module | `tools` | the file that declares search / read / list tools |
| Live index module | `liveIndex` | the fetcher with its static fallback, if any |
| Sitemap source | `sitemap` | the file that generates `sitemap.xml` |
| Data modules | `data/*` | plain-text arrays pages import |
| Probe script | `probe` | the script that asks the running assistant questions |
| Unit tests | `knowledge.test` | tests next to the knowledge module |
| Project docs line | `docs` | the line in CLAUDE.md / README that lists what the assistant knows |

If the host has no probe script, create one (see [verification.md](verification.md)). If it has no
corpus and tools, this skill still applies to surfaces 1, 2 and 6; skip the corpus steps.

## Host probe

Read-only. Run from the repository root and note each hit in the seam table.

```bash
# Prompt module: where the system prompt text is assembled
grep -rln "system prompt\|buildSystemPrompt\|SYSTEM_PROMPT\|systemPrompt" src | head

# Knowledge module: pack, corpus, search
grep -rln "knowledge\|corpus\|searchKnowledge\|readKnowledgeDocument" src/lib src/config | head

# Tools: AI SDK tool() declarations and their descriptions
grep -rn "tool({" src | head
grep -rn "description:" $(grep -rln "tool({" src) | head -20

# Live index: remote fetch of a README or JSON list
grep -rn "raw.githubusercontent\|revalidate:" src/lib | head

# Sitemap source and the arrays it maps over
ls src/app/sitemap.ts 2>/dev/null; grep -n "\.\.\.\|map(" src/app/sitemap.ts | head -30

# Channels: every caller of the knowledge context
grep -rn "buildKnowledgeContext\|getKnowledgePack" src | grep -v test

# Probe script and its probes
ls scripts | grep -i probe; grep -n "name:" scripts/*probe* | head -30
```

Record the pack's current size as the baseline: print it with a one-off script that imports the
knowledge module (see [verification.md](verification.md)) and note the character count.

## Checklist

- [ ] All ten seams named, or marked absent
- [ ] Every channel that receives the knowledge context listed
- [ ] Pack size baseline recorded
- [ ] Existing probes listed, so new ones follow their shape
