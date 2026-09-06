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

## ~~Problem 8 has no feature~~ — RESOLVED (D1)

**This section's original reading was wrong.** It read problem 8 as customer-side
cross-salon discovery, implying a marketplace. The intended meaning was the
salon's view of the same event: detecting and capturing lost demand. See
[DECISIONS.md](DECISIONS.md) D1.

Scope is unchanged: single-salon B2B. No marketplace decision is needed.

## Contradictions in the source

1. **Receptionist: role or target?** Listed as a supported role layer, while the
   stated value is eliminating the position. Both can be true; confirm which.
2. **Owner vs. admin** used interchangeably. Four roles or five?
3. ~~**VIP vs. ETA integrity.**~~ **Resolved (D5).** ETAs are explicitly mutable,
   not promises. VIP inserts are one of several reshuffle triggers. Residual
   question is only whether VIP can preempt an *in-progress* service.
4. **Cancellation charges vs. payment timing.** F01 mandates charges on two
   modes; F09 never establishes that payment details are captured at booking.
   Unenforceable as written.

## Largest unresolved risks

| # | Risk | Feature |
| --- | --- | --- |
| 1 | Voice cross-talk between specialists in one room — raised in source, never solved | F03 |
| 2 | Artist-confirmation timeout undefined; strands the customer mid-booking (D3/O2) | F01/F05 |
| 3 | Rules-engine config confirmed (D2) — but who authors a new mode is open (O1) | F04 |
| 4 | Payment timing undecided; cancellation charges unenforceable without it | F09 |
| 5 | Salon acoustics vs. voice-first as the *primary* interface | F03 |
| 6 | Reliability rests on artists pressing three separate controls (D3/D4) | F05 |

~~Service-duration estimates~~ resolved by D6.

## Not in the source, but load-bearing

Raised here because their absence will surface as rework, not because the
source discussed them:

- No-show and late-arrival policy (F01/F02)
- ~~Notification channel~~ — resolved (D8: SMS/WhatsApp). Deliverability and
  per-message cost are now open instead
- Consent, data retention, and messaging opt-out (F06/F08)
- Cold-start behaviour: a salon with no history has no durations, no trends, no
  analytics
