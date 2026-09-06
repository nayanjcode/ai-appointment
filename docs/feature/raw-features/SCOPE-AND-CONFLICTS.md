# Scope, conflicts and unresolved items

Everything here is a decision the source did not make. These are stage-1
(`/grill-me`) inputs — none should reach a PRD unresolved.

## Explicitly out of scope (stated)

- **Managing calls via agents.** Stated directly by the owner.
- Native mobile apps — iteration 1 is **web, mobile-first**.

## Consequences of ruling out call agents

The source document reached for call agents twice. Both are now dead, and the
underlying problems return unsolved:

| Source item | Status |
| --- | --- |
| §1 "Sol1: Call agent as a solution" | Dead |
| §1 "**TO INVESTIGATE**: website to have call logs permission" | Moot — nothing consumes call logs |
| §1 "Missed calls: No solution" | **Still no solution** |

This is the honest position: **the missed-call problem is accepted as unsolved
in iteration 1.** The product removes reasons to call rather than handling calls
that still happen. That is defensible — but it should be an explicit decision,
because §1 argues missed calls are a real lost-customer channel, especially for
the out-of-area regular who only visits occasionally.

It also removes F07's ability to measure the largest category of lost deal.

## Problem 8 has no feature — and implies a different product

> "Customers pain in finding a new shop in case of unavailability and emergency"

Every other problem is **one salon's** operational pain. This one is a
**customer's cross-salon discovery** need. Serving it means a marketplace with
multiple salons, public search, and cross-salon availability — a different
business model, go-to-market, and data model from B2B salon software.

**Decision needed before any roadmap:** is this a salon SaaS or a marketplace?
Deferring it is fine. Leaving it ambiguous is not — it changes F04's tenancy
model and F10's identity model, both of which are upstream of everything.

## Contradictions in the source

1. **Receptionist: role or target?** Listed as a supported role layer, while the
   stated value is eliminating the position. Both can be true; confirm which.
2. **Owner vs. admin** used interchangeably. Four roles or five?
3. **VIP vs. ETA integrity.** F01's VIP mode and F02's live-ETA promise are in
   direct tension — every queue jump invalidates the estimates F02 just sent to
   everyone behind it. The source promises both without reconciling them.
4. **Cancellation charges vs. payment timing.** F01 mandates charges on two
   modes; F09 never establishes that payment details are captured at booking.
   Unenforceable as written.

## Largest unresolved risks

| # | Risk | Feature |
| --- | --- | --- |
| 1 | Voice cross-talk between specialists in one room — raised in source, never solved | F03 |
| 2 | "Very very generic" config is unbounded — fixed schema vs. rules engine | F04 |
| 3 | Service-duration estimates have no stated source; all ETAs depend on them | F02 |
| 4 | Salon acoustics vs. voice-first as the *primary* interface | F03 |
| 5 | Payment/compliance scope not decided | F09 |

## Not in the source, but load-bearing

Raised here because their absence will surface as rework, not because the
source discussed them:

- No-show and late-arrival policy (F01/F02)
- Notification channel and deliverability (F02/F08)
- Consent, data retention, and messaging opt-out (F06/F08)
- Cold-start behaviour: a salon with no history has no durations, no trends, no
  analytics
