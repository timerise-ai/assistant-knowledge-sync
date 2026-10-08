# assistant-knowledge-sync

[![Agent Skills](https://img.shields.io/badge/Agent_Skills-open_format-059669)](https://agentskills.io)
[![skills.sh](https://img.shields.io/badge/skills.sh-npx_skills_add-059669)](https://www.skills.sh)
[![Claude Code](https://img.shields.io/badge/Claude_Code-compatible-059669)](https://docs.claude.com/en/docs/claude-code/skills)
[![Codex CLI](https://img.shields.io/badge/Codex_CLI-compatible-059669)](https://developers.openai.com/codex/skills)
[![Gemini CLI](https://img.shields.io/badge/Gemini_CLI-compatible-059669)](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/skills.md)

An [Agent Skill](https://agentskills.io) that teaches an agent to keep a **Next.js App Router** site's AI chat
assistant in step with the site. It finds what the assistant is missing, moves page copy into one data module
that the page and the assistant both read, adds new pages to the assistant's knowledge pack and searchable
corpus, builds its site map from the sitemap's own sources, writes routing and "never quote" guardrails, and
proves the result with unit tests and live probes. It is for assistants whose prompt is assembled in code from
repository content (AI SDK or similar, with search and read tools).

**The assistant must read what the page renders, never a retyped copy of it.** A longer prompt is correct for
one deploy. A fact with one source of truth in the repository, imported by the page and by the knowledge
module alike, stays correct for as long as the page does, and a probe asked in a visitor's words shows the
model actually uses it.

This skill was written by the engineer who has shipped this work. The earlier implementation it was audited
against was the assistant of a marketing site, answering on more than one channel from one knowledge module.
The procedure holds the properties an assistant update has to hold: page copy moved byte-for-byte so the page
renders identically, a site map that gains a route whenever the sitemap does, tool descriptions that name
every document kind in the corpus, the page's version shown where an external index lags it, and an explicit
"never quote" line for every number the site has not published. Unit tests, a render check and regex probes
verify each one; [`references/provenance.md`](references/provenance.md) has the record.

## Install

One command, via the [skills.sh](https://www.skills.sh) CLI, which installs the skill into every
skills-compatible agent it detects, including Claude Code, Codex CLI and Gemini CLI:

```bash
npx skills add timerise-ai/assistant-knowledge-sync
```

Name the agents instead with `-a`, for example `npx skills add timerise-ai/assistant-knowledge-sync -a
claude-code -a codex`.

### Manual install

Nothing here is Claude-specific: the skill is a plain [Agent Skills](https://agentskills.io) folder,
`SKILL.md` plus markdown references with no file that calls a model, so cloning it into an agent's skills
directory is all an install is. For Claude Code:

```bash
git clone https://github.com/timerise-ai/assistant-knowledge-sync.git ~/.claude/skills/assistant-knowledge-sync
```

To scope it to a single project instead, clone it into that project's `.claude/skills/` directory. For another
agent, clone into that agent's skills directory, or symlink the Claude Code copy so one `git pull` updates
every agent:

```bash
mkdir -p ~/.agents/skills
ln -s ~/.claude/skills/assistant-knowledge-sync ~/.agents/skills/assistant-knowledge-sync
```

Update the skill with `git pull` in its directory. The current release is **0.1.1**. See
[`CHANGELOG.md`](CHANGELOG.md). The [skills index](https://github.com/timerise-ai/skills) lists the other
Timerise Skills and how to install them all at once.

## Activation

The skill activates automatically when a task matches its description: teaching the site's assistant about a
new page, program or catalog, fixing an assistant that gives stale facts, wrong links or invented prices, or
bringing its site map back in line with the sitemap. Phrases such as "update the assistant's knowledge", "the
bot doesn't know about the new page" or "the chatbot invents prices" match it. Invoke it explicitly with
`/assistant-knowledge-sync` in Claude Code, `$assistant-knowledge-sync` in Codex CLI, or from `/skills` in
Gemini CLI.

Each host matches a task against the description its own way, so invoke the skill explicitly on a first run
rather than assuming it fired. Only `SKILL.md` is read up front; the `references/` files load on demand, so
the skill stays cheap in context until a topic is actually needed.

## What's inside

| File | Contents |
|---|---|
| `SKILL.md` | Entry point: architecture diagram, critical facts, hard rules, unattended runs, quick start, and the reference directory |
| `references/adaptation.md` | The seam contract: what the host must have, the seam table, the knowledge interface, the rename table, integration points, order of work, the non-negotiables |
| `references/surfaces.md` | The seven places assistant knowledge lives and the read-only host probe |
| `references/gap-audit.md` | What changed, the sitemap comparison, the fact list, the baseline probes |
| `references/single-source.md` | Moving page copy into a shared data module without changing a word |
| `references/pack-and-corpus.md` | Document kinds, catalog and page documents, pack sections, the site map, the live index join, size budgets |
| `references/guardrails.md` | Routing, links, unpublished facts, kind distinctions, tool descriptions, every channel |
| `references/verification.md` | Unit tests, the pack dump, the render check, live probes, a run with no model reachable |
| `references/provenance.md` | The engineering ledger: what the audit of the earlier implementation changed and how it is verified, what was kept on purpose, what is new in the skill |
| `evals/` | The prompts an operator types after installing (`prompts.md`) and one file per agent eval: the skill installed into an empty Next.js app, one prompt, no help, then type-checked, built and tested |
| `.github/workflows/agent-eval.yml` | The caller of the index's reusable eval workflow, run on every published release |

The seam is the table in `references/adaptation.md`. It names the host's prompt, knowledge, tools, live index,
sitemap, data and probe modules, and the three knowledge functions the templates call. The skill extends
those modules and never replaces them: the corpus registry, the search scorer, the model, the provider, the
widget and every channel's transport stay the host's.

## The five non-negotiables

These travel with every update and are never optional. Each is stated as a hard rule in `SKILL.md`:

1. **Never retype page copy into the prompt.** Import it from the module the page renders. A retyped copy is
   correct for one deploy; a unit test on the pack proves the imported line is there.
2. **Never rewrite the copy while moving it.** A single-source refactor is byte-for-byte and the page renders
   identically, which the render check verifies. Wording changes are a separate commit and a separate
   decision.
3. **Never add a document kind without updating the tool descriptions and link rules.** The model decides to
   search from the tool's description, not from what the corpus holds; a unit test finds the new kind and a
   probe expects its site path.
4. **Never ship without a probe per new fact and one per guardrail.** Unit tests prove the pack contains the
   line; only a live probe proves the model uses it. With no model reachable, probes are reported as not
   run, never as passing.
5. **Never let the pack grow unbounded.** Indexes and summaries go in the pack; full bodies stay behind the
   read tool. The pack's size is measured before and after.

Everything else is the host's: file layout, function names, routes, the search scorer, the model and the
channels.

## Not this

| Not this | Use instead |
|---|---|
| Building the assistant, its chat widget or its streaming transport | [ai-sdk](https://ai-sdk.dev) for chat on the site, or [slack-ai-bot](https://github.com/timerise-ai/slack-ai-bot) for a Slack bot |
| Embeddings, a vector store or hosted RAG | A RAG stack; this skill keeps an in-repo pack and a keyword corpus |
| Writing or editing the page copy | The host's content workflow; this skill moves copy and never rewrites it |
| Changing the model, provider or failover | The host's AI configuration |
| A help center or blog the assistant should read | [help-center-markdown](https://github.com/timerise-ai/help-center-markdown) or [blog-markdown](https://github.com/timerise-ai/blog-markdown) to build it, then this skill to teach the assistant |

## Contributing

Issues and pull requests are welcome here. Pure markdown, with no build step, but the code blocks are
checked: every code block names its destination on the first line or continues one that did, and the
TypeScript blocks are written to compile under `strict` and `noUncheckedIndexedAccess` against the knowledge
interface in `references/adaptation.md`; `CLAUDE.md` says how to run that check. Claims in this skill are
meant to be verifiable: if you change a factual claim, say how you verified it, whether against the AI SDK
documentation, the Next.js documentation, the TypeScript compiler, or a reproduction against a running
assistant.

Adding, removing or renaming a file in `references/` means updating the quick start and the reference
directory table in `SKILL.md`, the file table above, and any relative cross-links. Every odd-looking part of
the procedure is there for a reason, and `references/provenance.md` is the ledger that must stay truthful:
read it before simplifying anything, and add an entry for anything you change. Commits follow Conventional
Commits and releases follow [STANDARD.md](https://github.com/timerise-ai/skills/blob/main/STANDARD.md) in the
index; `CLAUDE.md` carries the full editing conventions.

## Part of the Timerise Skills

This is one of the [Timerise Skills](https://github.com/timerise-ai/skills): modules for **Next.js App
Router** apps written by our own senior engineers from the modules they have shipped, not synthetic, each
published as its own repository and indexed there. They share one layout, so an agent that has read one knows
how to read the next: a `SKILL.md` entry point, `references/` loaded on demand, and a seam contract carrying
the module's non-negotiables.

## Author

Built and maintained by [Timerise](https://timerise.ai).

## License

MIT. See [LICENSE](LICENSE).
