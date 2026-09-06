# F01 — Booking & scheduling core

**From:** problems 1 (advance bookings, off days), 3 (availability), 9 (VIP booking)

The transactional heart of the product. Everything else decorates it.

## Capabilities

**Booking modes** — the set is per-salon configurable, not fixed:
- Next available — join the live queue, no fixed time
- Specific slot — a named time
- VIP / priority — jumps queue; source doc frames this as "for relatives / close ones"
- Prebook — advance booking for a future date

**Specialist selection modes:**
- Fastest turn (any available specialist)
- Best artist
- Named specialist

**Cancellation** — policy varies *by booking mode*:
- Next available: free cancellation up to 30 min before (stated)
- Specific slot: charge always applies (stated)
- VIP slot: charge always applies (stated)

**Other:**
- View my appointments (current, upcoming, history)
- Availability lookup: open/closed, holidays, per-specialist availability
- Reschedule

## Dependencies

F04 (mode/pricing config), F10 (who may book for whom), F02 (queue insert), F09 (cancellation charges).

## Open questions

- **VIP semantics.** Does VIP preempt an in-progress service, insert at queue
  head, or reserve held capacity? Preemption breaks F02's ETA guarantees for
  everyone behind it. Unspecified in source.
- **VIP eligibility.** "For relatives / close ones" is a social rule, not a
  system rule. Who grants VIP — owner allowlist, paid tier, both?
- **"Best artist" is undefined.** By rating, seniority, owner ranking, or
  service-specific skill? No rating system exists in the source.
- **Overbooking / no-show policy.** Not mentioned anywhere. A queue system
  without a no-show rule degrades immediately in practice.
- Can a customer hold more than one active booking?
