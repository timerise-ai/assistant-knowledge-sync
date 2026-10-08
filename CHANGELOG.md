# Changelog

All notable changes to this skill are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and versions follow
[Semantic Versioning](https://semver.org/).

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
