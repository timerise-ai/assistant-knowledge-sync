# Single-source the copy

When a page's facts live as constants inside its component (an FAQ array, role cards, a permissions
table), the assistant cannot read them, and any copy you paste into the prompt drifts from the page on
the next edit. Move the **words** into a plain data module. The page keeps the **presentation** (icons,
art, colours) and attaches it by index or by name.

```
before:  page.tsx { FAQ, ROLES(with icons), STEPS }        prompt: retyped summary (drifts)
after:   data/program.ts { FAQ, ROLES, STEPS }  ◄── page.tsx (adds icons)
                                                ◄── knowledge (builds pack + corpus doc)
```

## The data module

Plain TypeScript, no React, no icon imports, no `"use client"`. It must be importable from a server-only
knowledge module and from a client component alike.

```ts
// src/data/program.ts
export const PROGRAM_NAME = "Example Builders";

/** The page's headline promises. Order matters: the page attaches icons by index. */
export const PROGRAM_PROMISES = [
  { title: "Paid per project", body: "A fixed share of the project value for each role." },
  { title: "One contract", body: "The client signs with us. Your name is on the offer." },
  { title: "Every line reviewed", body: "CI, an agent and a senior on every pull request." },
] as const;

export interface ProgramStage {
  name: string;
  who: string;
  can: string;
  entry: string;
}

/** Roles keyed by `kind`, so the component can look up its colours by name, not position. */
export const PROGRAM_ROLES: ReadonlyArray<{ kind: string; stages: ProgramStage[] }> = [
  {
    kind: "Sales",
    stages: [
      {
        name: "Referrer",
        who: "Sells, does not build",
        can: "Bring clients",
        entry: "A first offer approved by us",
      },
    ],
  },
];

export type ProgramFaqIcon = "code" | "banknotes";

/** The icon is a string key, so the FAQ list component maps it and this file stays plain. */
export const PROGRAM_FAQ: ReadonlyArray<{ icon: ProgramFaqIcon; q: string; a: string }> = [
  {
    icon: "banknotes",
    q: "How are partners paid?",
    a: "Per project, as a fixed share for each role. The shares are published when the first round opens.",
  },
];
```

## The page side

Attach presentation without changing a word.

```tsx
import { BanknotesIcon, DocumentTextIcon, ShieldCheckIcon } from "@heroicons/react/24/outline";
import { PROGRAM_PROMISES } from "@/data/program";

/** Icons in the order the data module lists the promises. */
const PROMISES = PROGRAM_PROMISES.map((item, i) => ({
  ...item,
  icon: [BanknotesIcon, DocumentTextIcon, ShieldCheckIcon][i],
}));
```

For keyed data, map by name so a reorder in the data module cannot swap colours:

```tsx
const ROLE_STYLE: Record<string, { tint: string; ink: string }> = {
  Sales: { tint: "bg-amber-100", ink: "text-amber-600" },
};
const ROLES = PROGRAM_ROLES.map((role) => ({ ...role, ...ROLE_STYLE[role.kind] }));
```

| Data shape | Attach presentation by | Why |
|---|---|---|
| Short fixed list (3–4 promises) | index | Stable, and an index array is the least code |
| Named entities (roles, stages, badges) | name / key | A reorder or insertion must not shift icons |
| Elements with JSX art | index, with `key` on each element | Art is not data; keep it in the component |
| Values interpolated from config (program name, badge) | template in the data module | So the assistant sees the same interpolated text |

Keep page metadata (title, description) in the data module too. The assistant's corpus document
summary should be the page's meta description, word for word.

## Failure modes

| Symptom | Cause | Fix |
|---|---|---|
| Page renders a missing icon | Data array grew, icon array did not | Map by name, or add the icon in the same commit |
| Client bundle pulls server code | Data module imported a server-only helper | Data modules import only constants and types |
| Copy changed during the move | "While I'm here" edits | Revert; wording changes are their own commit |
| Two arrays with the same name | Component kept its local constant | Delete the local one; grep the component for the old name |
| Type error on `FaqListItem[]` | Readonly data into a mutable prop type | `[...PROGRAM_FAQ]` at the call site |

## Checklist

- [ ] Every visitor-facing string from the component now lives in the data module
- [ ] Component diff contains only imports and presentation mapping
- [ ] Typecheck passes; the page renders every section (see [verification.md](verification.md))
- [ ] Committed on its own, as a refactor, before the knowledge change
