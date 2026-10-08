---
prompts:
  - prompt: |
      Our site has a chat assistant; its code is below, save each file at its path first. We just shipped a partners page and the assistant knows nothing about it: people who ask about partnering get sent to sales, and yesterday it told someone they would earn 20%. Teach it about the partner program.

      // lib/ai/knowledge.ts
      export type KnowledgeKind = "help" | "page";

      export interface KnowledgeDocument {
        kind: KnowledgeKind;
        title: string;
        url: string;
        summary: string;
        tags: string[];
        content: string;
      }

      const DOCS: KnowledgeDocument[] = [
        {
          kind: "help",
          title: "Booking widget",
          url: "/help/booking-widget",
          summary: "Embed a booking calendar on your site.",
          tags: ["widget", "calendar", "embed"],
          content: "Paste the script tag before the closing body tag. The widget reads your services and opening hours.",
        },
        {
          kind: "page",
          title: "Pricing",
          url: "/pricing",
          summary: "Plans and prices.",
          tags: ["price", "plan", "cost"],
          content: "Starter: 29 EUR a month. Pro: 79 EUR a month.",
        },
      ];

      export function searchKnowledge(locale: string, query: string, limit: number): KnowledgeDocument[] {
        const tokens = query.toLowerCase().split(/\W+/).filter(Boolean);
        return DOCS.map((doc) => {
          const text = `${doc.title} ${doc.tags.join(" ")} ${doc.summary} ${doc.content}`.toLowerCase();
          return { doc, score: tokens.filter((t) => text.includes(t)).length };
        })
          .filter((r) => r.score > 0)
          .sort((a, b) => b.score - a.score)
          .slice(0, limit)
          .map((r) => r.doc);
      }

      export function readKnowledgeDocument(locale: string, url: string): KnowledgeDocument | undefined {
        return DOCS.find((doc) => doc.url === url);
      }

      export function getKnowledgePack(locale: string): string {
        return ["#### SITE MAP", "- Pricing: /pricing", "- Help: /help/booking-widget", "- Talk to sales: /contact"].join("\n");
      }

      // lib/ai/prompts.ts
      import { getKnowledgePack } from "./knowledge";

      export const searchDescription = "Full-text search across the help center and the pricing page.";

      export function buildSystemPrompt(locale: string): string {
        return [
          "You are Ada, the assistant on our website. Answer questions about our booking software.",
          "Only discuss booking, pricing and help articles. For anything else, send people to /contact.",
          "Link pages with relative links.",
          getKnowledgePack(locale),
        ].join("\n\n");
      }

      // app/partners/page.tsx
      const ROLES = [
        { name: "Referrer", body: "Introduce a client. We do the setup." },
        { name: "Implementer", body: "Set up the booking widget for your client. We review it before it goes live." },
      ];

      const FAQ = [
        { q: "Who can join?", a: "Agencies and freelancers who set up booking for their clients." },
        { q: "How are partners paid?", a: "A share of the first year's subscription for every client you bring. The shares are published when the first round opens." },
      ];

      export default function PartnersPage() {
        return (
          <main>
            <h1>Partner program</h1>
            <p>Bring clients to us and get paid for every one.</p>
            <h2>Roles</h2>
            <ul>
              {ROLES.map((role) => (
                <li key={role.name}>
                  <strong>{role.name}</strong>: {role.body}
                </li>
              ))}
            </ul>
            <h2>FAQ</h2>
            {FAQ.map((item) => (
              <details key={item.q}>
                <summary>{item.q}</summary>
                <p>{item.a}</p>
              </details>
            ))}
            <p>
              <a href="/partners/apply">Apply to the program</a>
            </p>
          </main>
        );
      }
    stack: No data store
  - prompt: "Our chatbot keeps telling visitors the Pro plan costs 49 EUR, but the pricing page says 79. Fix it so the two cannot drift apart again."
    stack: No data store
  - prompt: "Last month we renamed /pricing to /plans and added a catalog with a page per integration. Check whether our site assistant still sends people to the right places and tell me what you would change; do not edit anything yet."
    stack: No data store
---

# Prompts

What an operator types after installing this skill, in their own words. An agent eval installs the skill
into an empty Next.js app, gives the agent one of these prompts and no further help, then type-checks, builds
and tests the result; the first prompt runs before every release. This skill works on an assistant the app
already has, so the first prompt carries one: a knowledge module, a prompt module and a page whose copy lives
inside its component. A run scores whether the agent moves that copy without changing a word, adds the page
to the corpus, pack and site map, writes the routing and "never quote" lines, wires the tests to `npm test`,
and reports the live probes as not run, since no model is reachable. The second and third prompts name an
assistant the app does not have, so they score whether the agent looks for its seams, says what is absent and
asks for it, rather than building a chatbot. The results are the other files in this folder. Section 10 of
[STANDARD.md](https://github.com/timerise-ai/skills/blob/main/STANDARD.md) says how a run is made. The
prompts and the newest runs are on [the skill's page](https://timerise.ai/skills/assistant-knowledge-sync)
on timerise.ai.
