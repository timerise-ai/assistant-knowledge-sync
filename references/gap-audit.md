# Gap audit

Find out exactly what the assistant is missing before writing anything. Three steps: what changed,
what facts it adds, what the assistant says today.

## 1. What changed since the knowledge was last touched

```bash
# The last commit that touched the assistant's knowledge (paths from the seam table)
LAST=$(git log -1 --format=%h -- lib/ai/knowledge.ts lib/ai/prompts.ts lib/ai/tools.ts)

# Routes added or removed since then
git diff --stat "$LAST"..HEAD -- 'app/**/page.tsx' app/sitemap.ts

# Data and content that pages render from
git diff --stat "$LAST"..HEAD -- data content

# The commit subjects, for the human view
git log --oneline "$LAST"..HEAD | grep -Ei "feat|page|program|catalog|sitemap|seo|nav"
```

A repository with no history for the knowledge module (a fresh clone, a single commit) has no `LAST`: read
the routes under `app/` and the sitemap source directly instead.

Then compare the sitemap with the assistant's site map line by line:

| In the sitemap source | In the assistant's site map | Action |
|---|---|---|
| route present | present | none |
| route present | missing | add, from the same array (see [pack-and-corpus.md](pack-and-corpus.md)) |
| route removed | still present | remove; check redirects so old links still answer |
| intentionally excluded (noindex, form-only) | present | keep if visitors ask for it, and say what it is |

Pages left out of the sitemap on purpose (a noindex help center, an apply form) still belong in the
assistant's site map when visitors ask for them. The sitemap answers "what should rank"; the assistant
answers "where do I go".

## 2. The fact list

One row per new fact. The source column is the decision that matters.

| Fact | Visitor question it answers | Source of truth | Surface | Notes |
|---|---|---|---|---|
| The program has several roles | "Can I only refer clients?" | data module (after the refactor) | pack and corpus | |
| Payout shares not published | "How much do I earn?" | page FAQ | guardrail | never quote |
| Latest version of a catalog entry | "What's the latest version?" | synced changelog | live index joined with local facts | the index can lag the page |
| `/partners/apply` is the form | "How do I join?" | route | site map and routing rule | not the sales brief |

A fact whose source is "inline in a component" triggers the single-source step
([single-source.md](single-source.md)) before anything else.

A fact with no source in the repository (a decision someone told you) goes to the hand-written prompt,
and only if it is stance or identity. Otherwise ask for it to be put on a page first.

## 3. The baseline

Ask the running assistant the visitor's questions before editing. Write each as a probe now (question,
expect, reject) and run it: it should fail. A probe that already passes means the fact was not missing;
drop it from the plan. When no model is reachable, write the probes anyway and record the baseline as not
run.

Good baseline questions are the ones a visitor types, not the ones that name the internal label:

- "I run a small agency. Can I refer clients to you and get paid? How much?"
- "What do I need before I can use the booking widget, and what does it cost?"
- "Can you build the starter template for me, and what would it cost?" (a catalog entry that is not for sale)

## Checklist

- [ ] Last knowledge commit found; diff of routes, data and content read
- [ ] Sitemap and the assistant's site map compared row by row
- [ ] Fact list written with a source and a surface per fact
- [ ] Inline-only copy flagged for the single-source step
- [ ] Baseline probes written and failing, or recorded as not run
