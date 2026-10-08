# Changelog

All notable changes to this skill are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and versions follow
[Semantic Versioning](https://semver.org/).

## [0.1.2] - 2026-10-08

Fix release, from scoring the second round of prompt-1 agent eval runs against 0.1.1.

### Changed

- The project docs line is never an absent seam: a host without one gets it as a line in `README.md`
  (`SKILL.md` *Working unattended* and quick-start step 7, the seam table and order of work in
  `adaptation.md`).

## [0.1.1] - 2026-10-08

Fix release, from scoring the prompt-1 agent eval runs against 0.1.0.

### Changed

- An unattended run commits nothing and creates no branch; the handover lists the files of the refactor
  commit and the knowledge commit, in that order (`SKILL.md`, `adaptation.md`, `verification.md`).
- A seam the host lacks stays absent: with no sitemap source the route is added to the pack's existing site
  map, and with no transport `scripts/ask.mjs` is a stub that throws until wired; no routes module, endpoint
  or environment variable is invented (`SKILL.md`, `pack-and-corpus.md`, `verification.md`).
- The shipped tests a host copies are named: those whose seams it has, assertions unchanged, 2 of the 4
  without a catalog; extra tests go beside them (`SKILL.md`, `verification.md`).
- The host's search scorer stays as it is, and a canonical path is a role, not a file to create
  (`pack-and-corpus.md`, `adaptation.md`).
- The handover names the probes not run, the absent seams, the pack size before and after, the tests that
  ran and the two commits' files (`SKILL.md`).

## [0.1.0] - 2026-10-08

First release: a procedure for teaching a Next.js site's AI chat assistant a new page, program or catalog
from the repository's own content, verified by unit tests, a render check and regex probes.

### Added

- Seven knowledge surfaces and a read-only host probe.
- The seam contract: seam table, knowledge interface, rename table, integration points, order of work.
- Gap audit: changes since the last knowledge commit, sitemap comparison, fact list, baseline probes.
- Single-source refactor for page copy held in components.
- Corpus document kinds, pack sections, a site map built from the sitemap's sources, live index join.
- Guardrails: routing, link preference, unpublished facts, kind distinctions, tool descriptions, channels.
- Verification: unit tests with a vitest config for hosts without a runner, pack dump, render check, a
  probe runner, and the rule for a run with no reachable model.
- Agent evals: three prompts in `evals/prompts.md` and the eval workflow caller.
