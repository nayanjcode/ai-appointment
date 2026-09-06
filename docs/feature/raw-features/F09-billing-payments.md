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

## Open questions

- **Prepay or pay-after?** This changes F01 entirely. Cancellation *charges* on
  VIP and specific-slot bookings only work if payment details are captured at
  booking time — otherwise the charge is unenforceable and the policy is
  decorative.
- Payment provider, settlement, and refund flow: unspecified.
- Partial service, walkout, or dispute: unspecified.
- Cash remains common in this segment. Is cash recorded in-app so F07 sees
  total revenue, or is the system blind to it?
- **Handling payments raises compliance obligations** (PCI-DSS scope, refunds,
  tax invoicing) that no other feature here does. Worth an explicit decision on
  whether iteration 1 takes money at all, or defers to cash + record-keeping.
