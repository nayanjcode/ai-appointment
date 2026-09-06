# F02 — Live queue & ETA

**From:** problem 2 (long waiting in shop)

The source's sharpest problem: customers get irritated not by waiting but by
*opaque* waiting. Two named causes — no visibility of who is ahead, and the
false signal of "a specialist is free so my turn came" when that specialist
cannot perform the requested service.

## Capabilities

- Queue position: how many are ahead of me
- Estimated start time for my turn
- Live re-estimation as reality moves: service start, stop, pause, overrun,
  specialist break, walk-in
- Live "who is being served now" view
- Proactive notification X minutes/hours before turn
- Arrival rule: customer must arrive 10 min before estimated turn (stated)
- Transparency of delay — owner-side and customer-side see the same truth
- Specialist-aware queueing: only count specialists who can perform *my* service

## Dependencies

F01 (bookings feed the queue), F05 (specialist start/stop/pause is the input
signal), F04 (per-service duration estimates).

## Decided (see DECISIONS.md)

- **Durations (D6).** Seeded per specialist, then adjusted from that
  specialist's own history. Cold start is manual, not modelled.
- **Live estimation (D5).** ETAs are explicitly mutable. Reshuffle triggers:
  service start, service end, break start/return, VIP insert.
- **Channel (D8).** SMS or WhatsApp — not app push.
- **Shop open state (D4).** Artist start/stop control gates availability.
- **Slot protection (D13).** Slot bookings are fixed points in the schedule.
  Next-available work fills the gaps around them and must not displace them.

## Open questions

- **What is the notification lead time?** "X minutes/hours" is a variable with
  no value. Per-customer, per-salon, or derived from travel time?
- **What happens when a customer is late?** The 10-min-early rule is stated;
  the penalty is not. This directly determines whether ETAs hold.
- **Per-message cost is now real.** D8 puts every notification on SMS/WhatsApp,
  which is metered. Reshuffles (D5) fire often; naive "notify on every change"
  is both expensive and the irritation it was meant to prevent.
- How often may an ETA change before the update itself becomes the irritation?
- **Admission rule decided (D16):** next-available shifts later, the slot holds,
  and the affected customer is warned on their schedule card as the slot nears.
  Open detail: how far ahead the warning fires, and whether a customer shifted
  repeatedly gets any priority compensation.
