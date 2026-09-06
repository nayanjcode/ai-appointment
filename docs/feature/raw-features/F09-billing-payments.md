# F09 — Billing & payments

**From:** problem 1 (receptionist duties include "manage payment")

Appears twice in the owner narrative — customer-side "can pay from here itself"
and owner-side "billing and all can be done here itself" — but has no dedicated
problem section. Under-specified relative to its risk.

## Capabilities

- In-app payment for a service
- Billing / invoice generation
- Cancellation charges, per booking mode (F01)
- Coupon redemption (F08)
- Revenue feed into F07

## Dependencies

F01 (what is owed), F04 (price), F08 (discount), F07 (reporting).

## Decided (see DECISIONS.md)

- **D11.** Slot booking blocks 50% at booking time and carries a cancellation
  charge. Next-available blocks nothing and charges nothing. This makes the slot
  charge enforceable — the gap flagged earlier.

## Open questions

- **C1 supersedes most of this feature for v1.** D18 defers payments to a later
  version, which removes the 50% block (D11) and any collectible cancellation
  charge including VIP (D17). Decide C1 before treating this feature as v1 scope
  at all.
- If C1 lands on option 2 (record as owed, settle in cash), this feature reduces
  to a small ledger. If option 3 (no-show tracking), it leaves F09 entirely and
  becomes an F06/F07 concern.
- Payment provider, settlement, and refund flow: unspecified.
- Partial service, walkout, or dispute: unspecified.
- Cash remains common in this segment. Is cash recorded in-app so F07 sees
  total revenue, or is the system blind to it?
- **Handling payments raises compliance obligations** (PCI-DSS scope, refunds,
  tax invoicing) that no other feature here does. Worth an explicit decision on
  whether iteration 1 takes money at all, or defers to cash + record-keeping.
