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
- Next available: no charge, no block (D11)
- Specific slot: 50% blocked, charge applies (D11) — **but see C1**
- VIP: per-salon configurable, no charge or an extra charge (D17) — **see C1**

**C1: D18 defers all payments to v2, which removes the ability to block or
charge anything in v1.** Cancellation policy is unresolved until C1 is decided.

**Other:**
- View my appointments (current, upcoming, history)
- Availability lookup: open/closed, holidays, per-specialist availability
- Reschedule

## Dependencies

F04 (mode/pricing config), F10 (who may book for whom), F02 (queue insert), F09 (cancellation charges).

## Decided (see DECISIONS.md)

- **Artist confirmation (D3).** A booking is not accepted until the artist
  confirms, within a fixed salon-configured window. Adds a `pending` state.
- **No off-hours booking (D7).** Owner can change shop timings at will.
- **"Wanted to book" capture (D1).** Conditional button when demand cannot be
  served — e.g. queue full, limited hours left.
- **Modes are per-salon configurable (D2)**, including which exist and their
  prices. **Developers author new mode types (D9)** — salons configure, they do
  not build.
- **State machine (D10).** Pending → Booked/Cancelled → In Progress → Completed.
  Pending bookings unconfirmed by end of office hours are rejected.
- **Payment (D11).** Slot: 50% blocked, cancellation charge applies.
  Next-available: no block, no charge. VIP undecided.
- **Slots are protected capacity (D13).** Next-available delays must not push a
  slot booking.

## Open questions

- **VIP semantics.** Partly answered: D5 confirms VIP reshuffles the schedule
  and ETAs update. Still unspecified whether it preempts an in-progress service
  or only inserts ahead in the queue.
- **VIP eligibility.** "For relatives / close ones" is a social rule, not a
  system rule. Who grants VIP — owner allowlist, paid tier, both?
- **"Best artist" is undefined.** By rating, seniority, owner ranking, or
  service-specific skill? No rating system exists in the source.
- **Overbooking / no-show policy.** Not mentioned anywhere. A queue system
  without a no-show rule degrades immediately in practice.
- Can a customer hold more than one active booking?
