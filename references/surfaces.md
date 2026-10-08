# The seven surfaces

An assistant's knowledge is spread over seven places. Each fact belongs in exactly one of them. Most
staleness comes from a fact that lives in the wrong one, usually hand-written into the prompt when it
should have been generated from content.

| # | Surface | Holds | Changes when | Who writes it |
|---|---|---|---|---|
| 1 | **Hand-written prompt facts** | Identity, positioning, stance, what the company sells, things no page states | The business changes | A person, rarely |
| 2 | **Generated pack** | Sections built from site copy: FAQ, pricing, process, program summaries, indexes, site map | Any deploy | Code, from content |
| 3 | **Searchable corpus** | One document per page with a body: help, blog, case studies, catalog entries, program pages | Any deploy | Code, from content |
| 4 | **Live external index** | A list maintained in another repository (a plugin or integration index) | Without a deploy | Fetched, cached, with a static fallback |
| 5 | **Tool descriptions** | What the search, read and list tools cover, in words the model reads | A document kind is added | A person, with every new kind |
| 6 | **Routing and link rules** | Who goes where, link style, allowed topics, "never quote" lines | A new form, program or audience | A person, with every new destination |
| 7 | **Channel variants** | Link format and memory rules per channel: web (relative links), chat apps (absolute links, length cap), agent protocol such as MCP (plain facts, tools instead of forms), an interview or brief prompt | A channel is added | A person |

**Rule of thumb:** if a visitor could read the fact on a page, it belongs in 2 or 3, generated. If it
says how to behave, it belongs in 6. Only identity and stance belong in 1.

The names used for each surface's file are the seam table in [adaptation.md](adaptation.md). Fill it from
the host probe below before editing anything.

## Host probe

Read-only. Run from the repository root and note each hit in the seam table. The directories cover both
layouts, with and without `src/`.

```bash
# Prompt module: where the system prompt text is assembled
grep -rln "system prompt\|buildSystemPrompt\|SYSTEM_PROMPT\|systemPrompt" app lib src 2>/dev/null | head

# Knowledge module: pack, corpus, search
grep -rln "knowledge\|corpus\|searchKnowledge\|readKnowledgeDocument" lib src 2>/dev/null | head

# Tools: AI SDK tool() declarations and their descriptions
grep -rn "tool({" app lib src 2>/dev/null | head
grep -rn "description:" $(grep -rln "tool({" app lib src 2>/dev/null) | head -20

# Live index: remote fetch of a README or JSON list
grep -rn "raw.githubusercontent\|revalidate:" lib src 2>/dev/null | head

# Sitemap source and the arrays it maps over
ls app/sitemap.ts src/app/sitemap.ts 2>/dev/null
grep -n "\.\.\.\|map(" app/sitemap.ts src/app/sitemap.ts 2>/dev/null | head -30

# Channels: every caller of the knowledge context
grep -rn "buildKnowledgeContext\|getKnowledgePack" app lib src 2>/dev/null | grep -v test

# Probe script and its probes
ls scripts 2>/dev/null | grep -i probe; grep -n "name:" scripts/*probe* 2>/dev/null | head -30
```

Record the pack's current size as the baseline: print it with the one-off script in
[verification.md](verification.md) and note the character count.

## Checklist

- [ ] All ten seams named, or marked absent
- [ ] Every channel that receives the knowledge context listed
- [ ] Pack size baseline recorded
- [ ] Existing probes listed, so new ones follow their shape
