# Project: appointment

> Referenced from root `CLAUDE.md`. Product definition and domain context.

## What it is

An **AI-first, voice-first appointment and operations platform for salons.**

Two interfaces over one system:

1. **Voice assistant (primary).** Users state intent in natural language; the
   system interprets and executes it. Designed to be language-agnostic and
   usable across age groups.
2. **Classic scheduling UI (secondary).** Conventional booking screens for
   everything the voice path does.

The product's wedge is removing the **phone call** from salon operations — not
by answering calls, but by removing the reasons to place them.

## Who it serves

| Role | Core need |
| --- | --- |
| Customer | Book, see queue position and live ETA, know when to arrive |
| Hair specialist | See queue, prep for who's coming, stop taking calls with wet hands |
| Receptionist | Supported where one exists; the product's pitch is making one unnecessary |
| Owner | Staffing, utilisation, revenue, CRM, broadcast |
| Admin | Configuration |

Whether owner and admin are one role is **undecided** — see
`docs/feature/raw-features/SCOPE-AND-CONFLICTS.md`.

## Iteration 1 scope

**In:**
- Web application, **mobile-first**
- Voice-first interaction
- Booking, live queue/ETA, shop config, specialist workspace

**Out (stated explicitly):**
- **Managing calls via agents.** The product removes reasons to call; it does
  not answer phones. A consequence is that the missed-call problem is
  **accepted as unsolved** in iteration 1.
- Native mobile apps

## Feature map

Ten features derived from 9 raw problem statements. Index and per-feature
detail: **`docs/feature/raw-features/`**.

F01 Booking core · F02 Live queue & ETA · F03 Voice assistant ·
F04 Shop configuration · F05 Specialist workspace · F06 Customer CRM ·
F07 Business analytics · F08 Marketing & broadcast · F09 Billing & payments ·
F10 Identity & roles

Source problems: `docs/Initial raw problem List.md`.

## Decisions still open — read before planning anything

These are upstream of architecture, not details. Do not let work proceed as
though they are settled.

1. **Salon SaaS or marketplace?** Raw problem 8 (customers finding a new shop)
   implies cross-salon discovery — a different product. Changes F04 tenancy and
   F10 identity, which are upstream of everything.
2. **How generic is per-salon config?** Fixed schema with per-tenant values, or
   a rules engine where salons define their own modes and pricing. Largest
   single driver of build size.
3. **VIP vs. ETA integrity.** Queue-jumping invalidates the live estimates the
   product promises. Both are committed to; neither is reconciled.
4. **Payment timing.** Cancellation charges are unenforceable unless payment
   details are captured at booking. Decides whether iteration 1 takes money.

## Stack

**Not yet chosen.** When it is, record language, framework, package manager,
test runner, entry points, and the commands to build/test/run here.
