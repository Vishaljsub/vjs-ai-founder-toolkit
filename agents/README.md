# Virtual Leadership Team — template

A virtual leadership team of advisor agents for a solo/small founder, in one place. A **Chief of Staff** manages the team; a **Chief Growth Officer** owns growth and directs a set of growth specialists; five functional executives cover finance, marketing, technology, sales, and people. They **think, advise, and draft** — they never touch the product's code, deploys, or build tooling. The founder decides and builds.

This is a **template** — every agent file has `{{PRODUCT_NAME}}` / `{{FOUNDER_NAME}}` / etc. placeholders. Fill them in for your product (or point the agent at a `PRODUCT.md`/`FEATURES.md` file with the real details) before use.

## Who's here (`.` / `specialists/`)
| Agent | Owns | Talk to it when… |
|---|---|---|
| **Chief of Staff** | Runs the whole team; routes questions; reconciles cross-functional decisions | "What needs my attention this week?" · anything cross-functional or "who owns this?" |
| **Chief Growth Officer (CGO)** | The overall growth plan + 90-day motion | "Where should I start?" · "What's the highest-leverage move?" |
| **CFO** | Money — runway, cost-to-serve, pricing, fundraising | "How long is our runway?" · "What does one customer cost us?" |
| **CMO** | Story & demand — brand, message, content, awareness | "What's our one-line story?" · "Where do we tell it?" |
| **CTO** | Technology — reliability, security/compliance, what to build next, tech hiring | "What should I build next?" · "What's most likely to break?" |
| **Head of Sales** | Pipeline to cash — deals, demos, pilots, closing | "How do I run this pilot conversation?" |
| **Head of People** | Team — who to hire, when, on what terms; culture | "Who should I hire first — and can I afford it?" |

**Growth specialists** (`specialists/`, directed by the CGO/CMO/Head of Sales, not called directly): `positioning-strategist`, `competitive-intel-analyst`, `content-engine`, `demand-gen-strategist`, `sales-enablement-lead`, `launch-pr-lead`.

**Repeatable playbooks** live in `skills/`: `positioning-canvas`, `icp-personas`, `messaging-house`, `launch-plan`, `content-calendar`, `outbound-sequence`.

## How it all fits
```
                         Founder (CEO — you)
                                │
                        Chief of Staff  ← single point of contact, routes & reconciles
                                │
     ┌─────────┬───────────────┬───────────────┬─────────┬──────────┐
    CFO       CGO             CTO             HR       (advisors report up)
              │  (owns the growth plan)
      ┌───────┴────────┐
     CMO           Head of Sales
 (create demand)  (turn it into cash)
      │                 │
      └──── growth specialists (positioning, content, demand-gen,
            competitive, sales-enablement, launch) ────┘
```

## How to use it
1. **Install as Claude Code agents.** Copy the files in this folder (and `specialists/`) into your project's `.claude/agents/` so each becomes an invocable agent by filename (e.g. `cfo.md` → `use the cfo agent: ...`).
2. **Fill in the placeholders.** Every file has `{{PRODUCT_NAME}}`, `{{FOUNDER_NAME}}`, and a few product-specific blanks — fill them in once, consistently, across all files (find/replace works fine).
3. **Ask the Chief of Staff** for anything cross-functional or unsure: `use the chief-of-staff agent: what needs my attention this week?` — it routes to the right expert(s) and returns one reconciled answer with a decision and next action.
4. **Or go direct** to an executive: `use the cfo agent: what's our runway?`, `use the head-of-sales agent: …`, etc.
5. They read their own mandate and write outputs into a `workspace/<their area>/` folder in your project (create it, or point them elsewhere).

## Shared rules (baked into every agent)
- **Advisory only.** No touching code, deploys, migrations, or build tooling. This folder is strategy, not the product.
- **No invented numbers, customers, or results.** Unproven items are marked `[needs proof]` or `[need founder input]`.
- **Plain language.** No unexplained jargon.
- **Everything ties to the business** — runway, customers, revenue, reliability, or team health — or it gets cut.
- **One story across the team.** Pricing, message, numbers, and roadmap must reconcile; the Chief of Staff enforces this.
