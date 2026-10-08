# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

An [Agent Skill](https://agentskills.io) package, markdown only. There is no build, no lint, no test runner
and no `package.json` here; nothing in this repository executes. It teaches an agent to keep a Next.js App
Router site's AI chat assistant in step with the site: audit the gap, move page copy into one data module
that the page and the assistant both import, extend the knowledge pack, corpus and site map, write routing
and "never quote" guardrails, and prove the result with unit tests and live probes.

Keep the two straight: the commands and code in `references/` describe the **host** app the skill is applied
to, not this repository. The `git` and `grep` probes in `surfaces.md` and `gap-audit.md`, the TypeScript in
`single-source.md`, `pack-and-corpus.md` and `guardrails.md`, and the vitest suite, render check and probe
script in `verification.md` all run in the host.

The skill was written by the engineer who has shipped this work; the earlier implementation it was audited
against was the assistant of a marketing site answering on more than one channel from one knowledge module.
`references/provenance.md` is the rationale layer: nine entries fixed, four kept deliberately with the reason
each is safe, seven added in the skill and never run on a second host. Read it before "simplifying"
anything.

## Structure

- `SKILL.md`: entry point, loaded whole on every activation, so it stays between 130 and 160 lines, the
  closing index line aside. The frontmatter is `name` and `description`, and the description is the trigger
  surface. The body carries the framing paragraph, the architecture diagram, six **critical facts**, five
  **hard rules**, *Working unattended*, the seven-step quick start, the **reference directory table** and the
  closing line linking the skills index.
- `README.md`: the human-facing front door, in the section order of the skill standard: badges, intro,
  install, activation, the file table, the five non-negotiables, *Not this*, contributing, footer.
- `CHANGELOG.md`: Keep a Changelog, one section per release, newest first; the version there, the README's
  current-release line and the git tag agree.
- `references/*.md`: one topic per file, loaded on demand. `adaptation.md` (the seam table, the knowledge
  interface, the rename table) is the design entry point; `surfaces.md`, `gap-audit.md`,
  `single-source.md`, `pack-and-corpus.md`, `guardrails.md` and `verification.md` follow the quick-start
  order; `provenance.md` is the audit ledger.
- `evals/`: `prompts.md` holds what an operator types after installing, in their words; the first prompt
  carries a small assistant (knowledge module, prompt module, a page with inline copy) because the eval
  fixture has none, and is the agent eval run before every release. Every other file there is one eval run:
  measured frontmatter that is never edited, then the notes of the person who ran it, scored against the hard
  rules. Add a prompt rather than rewording one that has results. The procedure is section 10 of the index's
  STANDARD.md.
- `.github/workflows/agent-eval.yml`: the caller of the index's reusable eval workflow, run on every
  published release and on a maintainer's dispatch. It is copied verbatim from section 10 of the index's
  STANDARD.md and is the same in every skill; do not edit it, and never add a trigger on `push` or
  `pull_request`.

## Editing conventions

- **Code blocks name their destination on the first line** as a comment, using the canonical paths of the
  seam table: `// lib/ai/knowledge.ts`, `// data/program.ts`, `// app/partners/page.tsx`. A continuation
  block that extends a file already introduced omits it. Shell blocks are commands run in the host.
- **The TypeScript blocks compile and the example tests pass.** Extract every `ts`, `tsx` and `js` block to
  its named path in a scratch directory (blocks without a comment append to the file named last), add a
  stub `components/icons.tsx` exporting the three icons, and append to `lib/ai/knowledge.ts` a stub host
  implementing `searchKnowledge`, `readKnowledgeDocument` and `getKnowledgePack` over two catalog entries
  (`booking-widget`, an offer; `starter-template`, free) and `programDocument()`, using the scorer weights in
  `pack-and-corpus.md`. Then:

  ```bash
  npm i -D typescript @types/react @types/node vitest
  npx tsc --noEmit     # strict, noUncheckedIndexedAccess, skipLibCheck, jsx react-jsx, paths {"@/*": ["./*"]}
  npx vitest run       # the 4 tests in verification.md
  node --check scripts/probe.mjs
  ```

  The three files carried in `evals/prompts.md` prompt 1 must also compile under the fixture's `strict`.
  Re-run after editing any block.
- **Identifiers are shared across files.** `KnowledgeKind`, `KnowledgeDocument`, `CatalogEntry`,
  `catalogOffer`, `catalogDocument`, `programDocument`, `programLines`, `siteMapLines`, `joinedRows`,
  `IndexRow`, `searchKnowledge`, `readKnowledgeDocument`, `getKnowledgePack`, the `PROGRAM_*` constants, the
  entry kinds `offer` and `free`, and the routes `/partners`, `/partners/apply`, `/catalog/<slug>` appear in
  several references. Rename in all of them or none.
- **Keep the three tables in sync** with `references/`: the quick start and the reference directory in
  `SKILL.md`, and the file table in `README.md`. Links are relative: `[x.md](references/x.md)` from
  `SKILL.md`, `[x.md](x.md)` between references.
- **Numbered lists are cited by number.** The seven surfaces ("2 or 3", "surfaces 1, 2 and 6"), the hard
  rules ("hard rule 3") and the provenance entries (the fix order) are referenced elsewhere; renumbering
  means updating every reference.
- **The odd-looking parts stay.** The helper named `block` rather than `section`, `catalogOffer` returning a
  sentence for free and unpriced entries, the page's version winning in `joinedRows`, index rows kept
  without a page link, the regex `reject` on any percentage: each is a ledger entry. Check `provenance.md`
  before touching one.
- **The numbers that remain are load-bearing.** The scorer weights (title 8, tags 5, summary 3, body capped
  at 10, multiplied by 1.5 on a full match), the 14,000-character read cap, the 20% pack growth threshold, the
  one-hour live index cache, the 4 example tests, and the ledger's counts. They are design parameters or
  facts about this repository. Figures describing the earlier implementation's deployment do not appear
  anywhere, and examples stay generic ("the program", "the catalog", "Example Partners").
- **Mark additions as additions.** Anything designed in the skill and never run in the earlier
  implementation goes under "Added" in `provenance.md`, stated as such.
- **Never present the non-negotiables as optional.** The five hard rules in `SKILL.md`, the non-negotiables
  in `README.md` and the list at the end of `adaptation.md` are the same five, in the same order.
- **Evals are not skill content.** A new prompt or an eval result is committed as `chore(evals): ...`,
  never causes a version bump and never rides in a release commit. A failing run stays committed; the fix is
  the next release.
- **What the host renames and what it does not.** File paths, function names, routes, the program name and
  the catalog vocabulary are renamed by the host through the table in `adaptation.md`. The hard rules, the
  probe shape `{ name, question, expect, reject }` and the two-commit order (refactor, then knowledge) are
  the authoring contract and are not renamed.
- **Writing style** follows section 7 of the index's STANDARD.md: no em-dashes, en-dashes, arrows, middle
  dots or smart quotes (diagrams use ASCII `-->` and `+--`), prose wrapped at 110 columns, and no tool or
  model named as author in any file or commit.
