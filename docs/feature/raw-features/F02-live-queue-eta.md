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

## Open questions

- **Where do duration estimates come from?** Static per-service config,
  per-specialist historical average, or ML? Source implies F07 measures actual
  durations — but the cold-start estimate is unspecified.
- **What is the notification lead time?** "X minutes/hours" is a variable with
  no value. Per-customer, per-salon, or derived from travel time?
- **What happens when a customer is late?** The 10-min-early rule is stated;
  the penalty is not. This directly determines whether ETAs hold.
- **Notification channel.** App push requires the app open/installed. SMS and
  WhatsApp are not mentioned. Web-first + mobile-first (stated scope) makes
  push unreliable.
- How often may an ETA change before the update itself becomes the irritation?
