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

- **Cross-talk: partially mitigated, not solved (D12).** The accepted controls
  — confirmation, availability gating, per-artist auth — are *authorization*,
  and the collision case is two authenticated on-duty artists in one room, where
  every check passes. See [VOICE-CROSSTALK.md](VOICE-CROSSTALK.md) for capture-
  level options and the v1 scoping recommendation.
- **Decided (D15): both customers and staff**, phone audio, no external
  hardware. Wake word plus clarify-and-reject. Customers cannot start or stop a
  service, which bounds cross-talk to staff writes.
- Residual: wake-word accuracy under salon noise with the phone at counter
  distance. See [VOICE-CROSSTALK.md](VOICE-CROSSTALK.md).
- **Booking on behalf of another specialist** — source asks whether this is
  acceptable and never answers.
- **Mode vocabulary is now finite (D9).** Developers author mode types, so the
  assistant never meets an unknown mode — this removes a risk flagged earlier.
- Ambient noise in a salon (dryers, clippers, music) — no mitigation stated.
- What happens on repeated misrecognition? No escape hatch defined.
