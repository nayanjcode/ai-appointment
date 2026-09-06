# F03 — Voice-first AI assistant

**From:** problem 1 (managing phone calls)

Stated as the product's *primary* interface; the classic UI is secondary. Its
job is to absorb the work the phone currently does — but **not** by answering
phones. Call handling via agents is explicitly out of scope.

## The problem it replaces

Source names five conditions where a specialist cannot take a call:
wet hands (shampoo, facial, dye, beard, waxing), peak hours, off days,
night-time advance bookings, back-to-back queued calls.

Note these are all reasons the phone *fails*, which is why the fix is removing
the phone from the loop — not automating it.

## Capabilities

- Voice intent → system action (book, cancel, reschedule, query availability,
  query price, check my turn)
- Language-agnostic; usable across age groups (stated goal: "feels like talking
  to a real human")
- Heavy confirmation before any state change — source is emphatic: "many many
  constraints... so that false information is not passed"
- Never infers the specialist or the customer; always confirms explicitly
- Refuses to drift off-task ("does not deviate from the main work")
- Fallback to the classic UI

## Dependencies

Every transactional feature. F03 is an interface over F01/F02/F04/F09, not a
system of its own.

## Open questions

- **Cross-talk is unsolved.** Source raises it and does not resolve it:
  multiple specialists in one room means one person's voice command is picked
  up by another's session. Confirmation-before-commit reduces blast radius but
  does not solve capture. This is the single largest technical risk in the
  product.
- **Booking on behalf of another specialist** — source asks whether this is
  acceptable and never answers.
- **Who is the voice user?** Customer, specialist, or both? The narrative
  implies both, but a customer-facing voice UI and a staff-facing one have
  different vocabularies, auth, and failure costs.
- Ambient noise in a salon (dryers, clippers, music) — no mitigation stated.
- What happens on repeated misrecognition? No escape hatch defined.
